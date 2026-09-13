# Voice Log — CURRENT

Updated: 2026-09-13

## Current phase
Requirements resolved / architecture drafting / pre-implementation.

## Current working branch
`docs/initial-requirements`

## Canonical requirements
`docs/requirements.md` — DRAFT v0.2

## Current architecture
`docs/architecture.md` — DRAFT v0.1

## Implemented
None. No production app implementation has started.

## Documentation status
- START-HERE / authority hierarchy: created
- Canonical requirements: initial product decisions resolved
- Current architecture: created
- ADRs: ADR-001 and ADR-002 accepted
- Tests / CI: not yet created

## Resolved product decisions
- V1 target: iOS 26+
- V1 cloud: iCloud Drive with configurable destination folder
- Transcription timing: post-recording by default
- Local audio retention: cloud-verified audio retained at least 7 days; eligible cleanup may remove local copy on/after day 7
- AI primary: Apple on-device Foundation Models when runtime-supported
- AI fallback: Google Gemini API with explicit cloud opt-in; prefer a current suitable free-tier model
- Transcription primary: Apple SpeechTranscriber / SpeechAnalyzer path

## Important product invariants
- Recording reliability has priority over transcription and AI.
- Original audio survives independently of transcript/summary.
- Local DB is not the source of truth for audio durability.
- Recoverable filesystem manifests permit DB/index reconstruction.
- Cloud upload state must not be reported as successful before provider confirmation.
- Local audio must not be evicted before cloud verification and 7-day retention eligibility.
- Minimal UI must not bypass or hide iOS privacy indicators.
- Cloud AI must not receive transcript/audio silently.

## Accepted ADRs
- `docs/adr/ADR-001-durable-recording-recovery.md`
- `docs/adr/ADR-002-on-device-ai-fallback.md`

## Next canonical work
1. Define implementation work packages and acceptance-test matrix.
2. Freeze the first implementation slice.
3. Start production implementation only against the frozen requirements/architecture/ADRs.
