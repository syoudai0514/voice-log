# Voice Log — Canonical Product Requirements

Status: DRAFT v0.2  
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

### STO-012 Local retention policy
After a recording has been confirmed as preserved in the configured cloud destination, its local audio copy shall be retained for at least 7 days after recording completion. On or after day 7, the next eligible cleanup run may remove/evict the local audio copy.

Cleanup must not remove a local audio artifact unless the corresponding cloud preservation state is verified. If cloud verification is unavailable or failed, the local copy must remain.

The retention policy applies to audio files/chunks; lightweight metadata, manifests, transcript artifacts, and recovery records may be retained longer as needed for indexing and recovery.

### STO-013 Cleanup timing
Voice Log does not need to wake at an exact 7-day boundary. Cleanup shall run opportunistically when the app or an allowed background maintenance task executes, and shall remove only items that satisfy STO-012 at that time.

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

### AI-005 On-device default
For title generation, transcript cleanup, and summarization, Voice Log shall prefer Apple's on-device Foundation Models framework when the device/OS/language configuration supports it.

The V1 target is iOS 26+ on Apple Intelligence-capable iPhones. Japanese must be supported by the selected on-device path.

### AI-006 Runtime capability check
The app shall not assume the on-device model is always available. At runtime it must detect model availability / supported locale and expose a clear unavailable state rather than failing silently.

### AI-007 Google fallback
When the on-device model is unavailable or unsuitable, Voice Log may offer Google Gemini API as an optional fallback provider.

The fallback must not automatically transmit transcript/audio to Google without an explicit user action/consent for that operation or a previously configured explicit opt-in.

### AI-008 Cloud AI privacy disclosure
Before first use of a cloud AI provider, Voice Log shall clearly state that transcript/content will leave the device and identify the configured provider. The app shall preserve the raw transcript locally regardless of provider success/failure.

### AI-009 Free-tier preference
For the Google fallback, implementation should prefer a Gemini API model with a current no-cost/free tier when one is available and suitable. Exact model names/limits must be re-verified against current Google documentation at implementation time rather than hard-coded into canonical requirements.

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

## 13. Resolved product decisions

### RD-001 Minimum iOS version
V1 shall target iOS 26+.

### RD-002 V1 cloud targets
V1 shall implement iCloud Drive with a configurable destination folder.

The storage architecture shall be pluggable from day one so additional Files providers / object-storage providers can be added later without rewriting the recording engine.

### RD-003 Transcription timing
V1 transcription shall run post-recording by default. Live transcription is deferred so active recording reliability remains the higher priority.

### RD-004 Local retention after verified cloud preservation
After verified cloud preservation, local audio shall be retained for at least 7 days. On/after day 7, the next eligible cleanup execution may remove/evict the local audio copy, subject to STO-012.

### RD-005 Summarization provider
Primary provider: Apple on-device Foundation Models where runtime capability and Japanese support are available.

Fallback provider: Google Gemini API, preferably using a current free-tier model, only through an explicit cloud-AI opt-in flow.

### RD-006 Transcription provider
Primary transcription remains Apple SpeechAnalyzer / SpeechTranscriber on iOS 26+. Alternate transcription providers may be added behind the provider abstraction later.

## 14. Remaining design decisions

These are implementation/design choices, not unresolved product requirements, and may be decided in architecture/ADR work provided they do not change the requirements above:

- exact chunk duration,
- exact local metadata database technology,
- exact manifest serialization format,
- exact iCloud directory naming convention,
- exact background-task scheduling strategy,
- exact Google Gemini model chosen at implementation time,
- exact UI visual design within Standard/Minimal mode constraints.
