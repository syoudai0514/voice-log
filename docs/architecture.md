# Voice Log — Current Architecture

Status: DRAFT v0.1  
Target: iPhone / iOS 26+  
Canonical requirements: `docs/requirements.md`

## 1. Architectural principles

1. Recording reliability outranks all secondary features.
2. Audio durability must not depend on the local database.
3. Cloud synchronization is resumable and idempotent.
4. Transcription and AI processing happen after recording by default.
5. The original M4A audio and raw transcript are immutable source artifacts.
6. UI presentation mode must never change recording durability semantics.
7. Apple on-device capabilities are preferred; external cloud AI is opt-in fallback.

## 2. High-level component model

```text
Action Button / App Shortcut
            |
            v
     RecordingCoordinator
            |
      +-----+----------------------+
      |                            |
      v                            v
AudioSessionManager         RecordingStateStore
      |                            |
      v                            v
ChunkRecorder  -------> Recovery Manifest
      |                            |
      +------------+---------------+
                   v
             SyncCoordinator
                   |
                   v
          StorageProvider protocol
                   |
                   +--> iCloudDriveProvider (V1)
                   +--> future providers

After recording:
                   |
                   v
        TranscriptionCoordinator
                   |
                   +--> Apple SpeechTranscriber (V1 primary)
                   +--> future providers
                   |
                   v
             Raw Transcript
                   |
                   v
             AIProcessor
                   |
                   +--> Apple Foundation Models (primary)
                   +--> Gemini provider (explicit opt-in fallback)
                   |
                   +--> Clean Transcript
                   +--> Summary
                   +--> Suggested Title
```

## 3. Recording subsystem

### 3.1 RecordingCoordinator

Owns the explicit session state machine:

- idle
- starting
- recording
- finalizing
- pendingUpload
- syncing
- synced
- interrupted
- recoverable
- failed

Only this coordinator may transition the primary recording state.

### 3.2 AudioSessionManager

Responsibilities:

- microphone permission,
- AVAudioSession configuration,
- route-change handling,
- interruption handling,
- background audio recording configuration,
- deterministic error reporting.

Recording must continue after display lock using Apple-supported background audio mechanisms.

### 3.3 ChunkRecorder

V1 shall record AAC/M4A in recoverable chunks.

Initial design default:
- target chunk duration: 60 seconds,
- finalize each chunk atomically,
- write manifest update after each finalized chunk,
- never require the full session to close successfully before earlier chunks are usable.

Exact duration may be adjusted after device testing without changing the product requirement.

### 3.4 Emergency storage reserve

Voice Log shall maintain a small reserved local file that may be released if storage becomes critically low, giving the app room to finalize the active chunk and persist recovery metadata.

Initial target: 20 MB reserve.

This reserve is a safety mechanism, not normal recording capacity.

## 4. Persistence model

### 4.1 Audio files are source-of-truth artifacts

Audio lives as files/chunks.

The local metadata database is an index/cache only.

### 4.2 Session manifest

Each session directory contains a durable manifest, for example:

```text
session/
  manifest.json
  audio/
    000001.m4a
    000002.m4a
    000003.m4a
  transcript/
    raw.json
    raw.txt
    clean.md
    summary.md
```

The manifest includes at least:

- schemaVersion
- sessionID
- startedAt
- endedAt
- chunk sequence
- chunk checksums / sizes
- local state
- cloud destination identifier
- per-chunk upload state
- cloud verification state
- transcription state
- AI artifact state
- recovery notes

Manifest writes use atomic replace semantics.

### 4.3 Local database

Preferred V1 technology: SwiftData unless implementation evidence shows a reliability limitation that materially affects recovery.

The DB indexes:
- sessions,
- titles,
- duration,
- presentation metadata,
- sync state,
- transcript/summary state.

The DB can be rebuilt from manifests and cloud/local artifacts.

## 5. Storage architecture

### 5.1 StorageProvider protocol

Provider interface conceptually supports:

- chooseDestination()
- validateDestination()
- uploadArtifact()
- verifyArtifact()
- listRecoverableArtifacts()
- fetchArtifact()
- evictLocalCopy()
- deleteCloudArtifact() — explicit user action only

### 5.2 iCloud Drive V1

V1 provides iCloud Drive as the first-class destination.

The user selects/configures the destination folder through supported iOS file/folder-selection APIs.

The app stores a durable destination reference appropriate to iOS sandbox/security-scoped access rules.

### 5.3 Upload policy

Upload runs independently of the DB.

For every finalized chunk:

1. persist chunk locally,
2. persist manifest state,
3. enqueue sync,
4. upload when possible,
5. verify provider-preserved state,
6. persist verified state,
7. only then make the item eligible for local eviction.

A network failure never ends an otherwise healthy recording.

### 5.4 DB-loss recovery

At launch and after storage/database inconsistency:

1. scan local session roots,
2. read manifests,
3. discover finalized chunks,
4. reconstruct DB/index rows,
5. identify pending uploads,
6. resume synchronization idempotently.

Cloud-side manifests/artifacts may also be used to rebuild sessions if the provider exposes them locally through iCloud Drive.

### 5.5 Seven-day retention

After cloud preservation is verified:

- keep local audio for at least 7 days from recording completion,
- on/after day 7, an eligible cleanup execution may evict/remove local audio,
- never remove local audio whose cloud verification is missing/failed,
- cloud copies remain until the user explicitly deletes them.

Cleanup is opportunistic; exact execution at the seven-day boundary is not guaranteed.

## 6. Transcription architecture

### 6.1 Primary provider

Use Apple's Speech framework `SpeechTranscriber` / `SpeechAnalyzer` path for iOS 26+ normal conversation / long-form transcription when the locale/device reports support.

Runtime capability must be checked rather than assumed.

### 6.2 Processing timing

Default flow:

```text
recording completed
      |
      v
audio stable + manifest finalized
      |
      v
transcription requested
      |
      v
raw transcript persisted
```

Transcription does not run concurrently with active recording by default.

### 6.3 Transcript artifact model

Raw transcript is retained independently.

When timing information is available, store segments structurally:

```json
{
  "segments": [
    {
      "startMs": 0,
      "endMs": 2480,
      "text": "..."
    }
  ]
}
```

Readable `.txt` is generated alongside structured JSON.

## 7. AI cleanup / summary / title

### 7.1 Primary provider: Apple Foundation Models

Use Foundation Models on-device when:

- the framework is available,
- Apple Intelligence is enabled/available,
- Japanese locale is supported,
- runtime availability check passes.

Primary tasks:
- clean transcript,
- summarize transcript,
- suggest title.

These operations do not alter raw transcript or original audio.

### 7.2 Fallback: Google Gemini API

Gemini is optional fallback only.

Requirements:
- explicit user opt-in before first external transmission,
- clear indication that content leaves the device,
- no automatic fallback that silently uploads transcript/audio,
- exact Gemini model/free-tier limits rechecked at implementation time.

Prefer sending transcript text rather than raw audio when transcription already succeeded locally.

### 7.3 Provider abstraction

`TextIntelligenceProvider` conceptually supports:

- cleanTranscript(raw)
- summarize(rawOrClean)
- suggestTitle(rawOrClean)

This keeps Apple/Gemini provider choice out of the UI/domain logic.

## 8. User experience

### 8.1 Entry

Primary entry:
- iPhone Action button assigned to Voice Log App Shortcut.

Target behavior:
- one user action reaches recording as quickly as platform privacy rules permit,
- haptic confirms start,
- subsequent display lock does not end recording.

Exact AppIntent/foreground handoff is selected using current iOS behavior during implementation.

### 8.2 Standard mode

Shows clearly:

- recording status,
- elapsed time,
- stop control,
- audio level/waveform if useful,
- local-save state,
- cloud-sync state,
- errors/interruption warnings.

### 8.3 Minimal mode

Low-distraction app UI:

- no large red recording surface,
- no waveform by default,
- no unnecessary animation,
- no start/stop sound by default,
- small elapsed/status indicator,
- deterministic haptic start/stop acknowledgement.

Minimal mode does not suppress, spoof, or bypass iOS privacy indicators.

### 8.4 Recording detail

Suggested information architecture:

```text
Recording Detail
  Audio
  Raw
  Clean
  Summary
```

Actions:
- Play
- Copy
- Share
- Export
- Regenerate clean text
- Regenerate summary
- Suggest title
- Edit title

## 9. Export model

Audio:
- M4A native export
- WAV optional conversion when required

Text:
- TXT
- Markdown
- JSON structured export

Each of Raw / Clean / Summary has:
- one-tap Copy,
- Share Sheet,
- export action.

## 10. Background work

Recording uses the audio background mode.

Post-recording processing uses appropriate current iOS mechanisms:
- foreground work when app is active,
- background URLSession where appropriate for network transfer,
- BackgroundTasks / continued processing only where platform rules fit.

No design assumes arbitrary unlimited background execution.

## 11. Verification strategy

Real-device acceptance testing is mandatory for:

- Action button/App Shortcut entry,
- screen-lock recording,
- Low Power Mode recording,
- phone/audio interruption,
- Bluetooth route change,
- network disconnect/reconnect,
- iCloud unavailable/recovery,
- critically low local storage,
- DB deletion/corruption and rebuild,
- 7-day cleanup eligibility,
- on-device Japanese transcription,
- on-device Japanese cleanup/summary/title,
- Gemini opt-in fallback.

## 12. Non-goals for V1

- live transcription during active recording,
- automatic speaker diarization unless Apple APIs provide sufficiently reliable support without affecting recording,
- multi-cloud implementation beyond iCloud Drive,
- automatic cloud AI transmission,
- exact-at-midnight seven-day cleanup scheduling.
