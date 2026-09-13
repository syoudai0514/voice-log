# ADR-002 — On-device AI Primary, Explicit Cloud Fallback

Status: Accepted  
Date: 2026-09-13

## Context

Voice Log will later generate cleaned transcripts, summaries, and suggested titles. Conversation data can be sensitive. The user prefers no-cost processing when practical.

## Decision

1. Target iOS 26+.
2. Use Apple SpeechTranscriber / SpeechAnalyzer as the primary transcription path when runtime support exists.
3. Perform transcription after recording by default so it cannot compete with the recording engine.
4. Use Apple Foundation Models as the primary provider for transcript cleanup, summarization, and suggested titles when the on-device model supports the current device/language and is available.
5. Preserve raw transcript and original audio regardless of AI provider outcome.
6. Check on-device model capability at runtime; do not assume Apple Intelligence/model availability.
7. When on-device generation is unavailable/unsuitable, offer Google Gemini API as an optional fallback.
8. Do not silently send conversation text/audio to Google. First use requires explicit opt-in and clear disclosure that content leaves the device.
9. Prefer a suitable current Gemini free-tier model, but re-verify model name, limits, pricing, and data-use terms at implementation time.
10. Prefer sending transcript text rather than raw audio to the cloud fallback when local transcription succeeded.

## Consequences

- Most title/cleanup/summary operations can stay private and cost-free on supported devices.
- Cloud fallback remains available without becoming an invisible privacy downgrade.
- AI provider behavior remains replaceable behind a common protocol.

## Rejected alternatives

### Always use a cloud LLM
Rejected because it unnecessarily sends private conversation data off-device and creates cost/network dependency.

### Automatically fall back to Google
Rejected because free-tier data handling may differ from paid processing and external transmission should be an explicit user choice.

### Run AI during active recording
Rejected for V1 because recording durability has higher priority than immediate transcription/summary latency.
