# Voice Log — Canonical Product Requirements

Status: DRAFT v0.1  
Product name: Voice Log  
Repository: `syoudai0514/voice-log`

## 1. Purpose

Voice Log shall provide an iPhone-first way to start long-form audio recording with minimal interaction, continue recording while the screen is locked, preserve recordings resiliently to cloud storage, and later produce usable transcripts and summaries.

Priority order:

1. Do not lose the recording.
2. Make start/stop operation fast and dependable.
3. Preserve the original audio in a transcription-friendly format.
4. Make cloud destination and folder configurable.
5. Make transcription, cleanup, summarization, copying, and export easy.
6. Keep the UI simple and low-friction.

## 2. Supported primary scenario

A user invokes Voice Log from the iPhone Action button / App Shortcut, starts recording quickly, locks the screen, records a conversation or meeting for a long period, stops the recording, and later opens Voice Log to play, transcribe, clean up, summarize, copy, or export the content.

## 3. Recording requirements

### REC-001 One-action entry
Voice Log shall expose an App Shortcut suitable for assignment to the iPhone Action button.

### REC-002 Background/lock-screen continuity
Once recording has started, screen lock or automatic display sleep must not stop recording.

### REC-003 Low Power Mode
Recording must be verified on the target iPhone with Low Power Mode enabled. Low Power Mode alone must not cause the recording to terminate.

### REC-004 Long-form recording
The design shall support long-form conversations and meetings rather than only short voice notes.

### REC-005 Durable chunking
A recording session shall be written as recoverable audio chunks rather than relying on one large file that is only finalized at the end. A crash or interruption must not invalidate already finalized chunks.

### REC-006 Interruption handling
The app shall explicitly handle and record interruption state for events such as phone/audio-session interruption, route change, microphone loss, and app termination where the platform permits recovery.

### REC-007 Audio format
The default master recording shall be a broadly compatible `.m4a` AAC audio file/chunk format suitable for playback on iOS and later speech-to-text processing. The original recording must be retained independently of transcripts.

### REC-008 Start/stop confidence
The user shall receive a deterministic start/stop acknowledgement. Haptic acknowledgement is preferred; audible acknowledgement must be optional.

## 4. Storage and cloud requirements

### STO-001 Configurable destination
The user shall be able to select where recordings are stored.

### STO-002 iCloud Drive
iCloud Drive shall be a first-class storage target.

### STO-003 Configurable folder
For supported storage targets, the user shall be able to select or configure the destination folder rather than being forced to use a fixed folder.

### STO-004 Storage abstraction
Recording code shall not depend directly on one cloud provider. Storage must be behind a provider abstraction so additional destinations can be added without rewriting recording logic.

### STO-005 Audio independent from local database
The local database is an index/cache, not the sole source of truth for recorded audio. Loss or corruption of the local database must not make existing audio undiscoverable or prevent pending audio from being uploaded.

### STO-006 Recoverable manifest
Each recording session shall persist enough filesystem-level metadata outside the local database to reconstruct:
- session identity,
- chunk ordering,
- timestamps,
- upload state,
- finalization state,
- target storage destination.

### STO-007 Rebuild after DB loss
If the local database is missing or corrupt, the app shall scan recoverable manifests/audio artifacts, rebuild its index, and resume pending cloud synchronization.

### STO-008 Upload-before-evict
A local audio artifact must not be deleted/evicted merely because an upload was requested. The app must verify the cloud-side preservation state to the extent supported by the provider before local eviction.

### STO-009 Offline behavior
Network or cloud unavailability must not stop an otherwise healthy recording. Unsynced chunks shall remain in a durable pending state and retry when cloud access returns.

### STO-010 Critical local-storage behavior
If local storage becomes critically low during recording, Voice Log shall:
1. finalize the current recoverable audio state,
2. immediately attempt upload/synchronization of finalized chunks,
3. preserve an explicit unsynced/recovery state if cloud upload cannot complete,
4. never silently discard the current session.

The app must not claim a cloud upload succeeded when the provider has not confirmed the preservation state.

### STO-011 Cloud metadata
Provider-independent metadata may be stored locally and/or in a cloud metadata store, but audio preservation must not depend on a single database row surviving.

## 5. Recording presentation modes

### UX-001 Standard mode
Voice Log shall provide a clearly visible recording mode with an obvious recording state, timer, stop control, sync state, and error/warning state.

### UX-002 Minimal mode
Voice Log shall provide a low-distraction recording mode with subdued application UI, minimal animation, optional haptics, and no unnecessary sounds.

Minimal mode must not attempt to hide, suppress, bypass, spoof, or interfere with iOS microphone/privacy indicators or permission behavior.

### UX-003 Mode selection
The user shall be able to choose the default presentation mode and switch modes without changing the underlying recording durability behavior.

### UX-004 State must remain inspectable
Even in Minimal mode, the user must be able to intentionally inspect whether recording is active and whether cloud synchronization is healthy.

## 6. Recording library

### LIB-001 Session list
The app shall display recorded sessions with:
- title,
- date/time,
- duration,
- audio availability,
- cloud sync state,
- transcription state,
- summary state.

### LIB-002 Playback
The original audio shall be playable in-app.

### LIB-003 Manual title
The user shall be able to edit the recording title manually.

### LIB-004 Generated title
Automatic title generation from transcript content shall be supported as an optional feature and must never overwrite a user-edited title without confirmation.

## 7. Transcription

### TRN-001 Original/raw transcript
Voice Log shall be able to produce and retain a raw transcript close to the speech recognizer output.

### TRN-002 Clean transcript
Voice Log shall be able to produce a cleaned/readable transcript derived from the raw transcript while keeping the raw transcript separately accessible.

### TRN-003 Summary
Voice Log shall be able to produce a summary derived from the transcript.

### TRN-004 Independent artifacts
Raw transcript, cleaned transcript, and summary are independent artifacts and shall be individually viewable, copyable, exportable, and regenerable.

### TRN-005 Long-form support
The transcription implementation shall be chosen for long-form conversation/meeting audio, not only short dictation.

### TRN-006 On-device first
On iOS 26+, Apple SpeechAnalyzer / SpeechTranscriber is the preferred first-party transcription path because it supports long-form conversational audio and on-device processing. A provider abstraction shall allow alternate transcription engines later.

### TRN-007 Timestamp-capable model
Where the transcription engine provides timing information, Voice Log shall preserve timing metadata so transcript text can later be synchronized with audio playback.

### TRN-008 Recording stability wins
Transcription must not be allowed to compromise active recording reliability. If resource contention occurs, recording takes priority and transcription may be deferred.

## 8. Text output and sharing

### OUT-001 Copy
Raw transcript, cleaned transcript, and summary shall each have an explicit one-tap copy action.

### OUT-002 Share sheet
Each text artifact and the audio shall be shareable through the standard iOS share sheet.

### OUT-003 Text formats
Text artifacts shall support plain text export. Clean transcript and summary shall additionally support Markdown export.

### OUT-004 Structured export
A structured export format (JSON) shall be available for automation and future integrations and may include session metadata, timestamps, title, transcript segments, and summary.

### OUT-005 Audio export
The master audio shall be exportable in its native `.m4a` form. Additional PCM/WAV export may be added when a downstream transcription workflow requires it.

## 9. Future summarization architecture

### AI-001 Provider abstraction
Title generation, transcript cleanup, and summarization shall be implemented behind a provider abstraction.

### AI-002 Preserve source
Generated/cleaned text must never destroy or replace the raw transcript.

### AI-003 Explicit data boundary
If a cloud AI provider is used, the app shall clearly distinguish on-device processing from data sent to an external service.

### AI-004 Retryability
Failed title/cleanup/summary generation must be retryable without affecting the original recording or raw transcript.

## 10. Reliability and observability

### REL-001 Explicit state machine
A session shall have explicit states for recording, finalizing, pending upload, syncing, synced, interrupted, recoverable, and failed.

### REL-002 No false green
The UI must distinguish:
- saved locally,
- upload requested,
- upload in progress,
- cloud preserved/verified,
- sync failed.

### REL-003 Recovery on launch
On launch, the app shall scan for incomplete/recoverable sessions and offer or automatically perform safe recovery.

### REL-004 Idempotent upload
Retrying an upload must not create uncontrolled duplicate sessions/chunks.

### REL-005 Audit log
The app shall keep a lightweight technical event log sufficient to explain why a recording stopped or why a sync failed, without storing unnecessary transcript/audio content in logs.

## 11. Platform/privacy constraints

### PLT-001 iOS privacy behavior
Voice Log shall use supported iOS microphone permissions and privacy indicators. The app shall not attempt to conceal or defeat system privacy disclosures.

### PLT-002 Background audio
The implementation shall use Apple-supported background audio/recording mechanisms and must be validated on real hardware.

### PLT-003 Platform limitations
Requirements must not claim guaranteed cloud preservation when the device has no usable network path or when iOS terminates execution before a provider can confirm upload. In those cases the requirement is durable recovery and later retry, not a false success.

## 12. Acceptance scenarios — initial set

- AC-001: Action button/App Shortcut reaches recording with minimal interaction.
- AC-002: 10+ minute recording survives screen lock.
- AC-003: 10+ minute recording survives Low Power Mode.
- AC-004: App interruption does not corrupt already finalized chunks.
- AC-005: Network loss during recording does not stop recording.
- AC-006: Pending chunks synchronize after network recovery.
- AC-007: Deleting/corrupting the local index DB still allows session discovery/rebuild from manifests and files.
- AC-008: Cloud-preserved audio is not lost when local cache is later evicted.
- AC-009: Standard and Minimal presentation modes use the same recording engine and durability semantics.
- AC-010: Raw, cleaned, and summary text are independently copyable/exportable.
- AC-011: Original M4A remains accessible after transcription and summarization.
- AC-012: A failed transcription/summary does not alter or delete the recording.

## 13. Open decisions

These decisions remain intentionally open and must not be silently guessed during implementation.

### OD-001 Minimum iOS version
Proposed: iOS 26+ only for V1, to use SpeechAnalyzer as the primary transcription API.

### OD-002 V1 cloud targets
Proposed:
- V1: iCloud Drive with configurable folder.
- Architecture from day one: pluggable cloud target.
- Later: Files-provider folders and/or direct object storage such as S3-compatible/Supabase Storage.

### OD-003 Transcription timing
Proposed: post-recording transcription by default. Live transcription can be added later so it cannot degrade recording reliability.

### OD-004 Local retention after verified cloud preservation
Need to decide default policy:
- keep all local audio,
- evict automatically after verified cloud preservation,
- or retain for N days / only evict when storage is low.

### OD-005 Summarization provider
Need to decide default:
- on-device Apple Intelligence/Foundation Models when available,
- external LLM,
- or provider-selectable with on-device preferred.
