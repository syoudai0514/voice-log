# Voice Log — CURRENT

Updated: 2026-09-13

## Current phase
Requirements definition / pre-implementation.

## Current working branch
`docs/initial-requirements`

## Canonical requirements
`docs/requirements.md` — DRAFT v0.1

## Implemented
None. No production app implementation has started.

## Documentation status
- START-HERE / authority hierarchy: created
- Canonical requirements draft: created
- Architecture: pending requirement decisions
- ADRs: pending
- Tests / CI: not yet created

## Open decisions blocking architecture freeze
- OD-001 Minimum iOS version
- OD-002 V1 cloud targets
- OD-003 Post-recording vs live transcription
- OD-004 Local retention policy after verified cloud preservation
- OD-005 Summarization provider

## Important product invariants already captured
- Recording reliability has priority over transcription.
- Original audio survives independently of transcript/summary.
- Local DB is not the source of truth for audio durability.
- Recoverable filesystem metadata permits DB/index reconstruction.
- Cloud upload state must not be reported as successful before provider confirmation.
- Minimal UI must not bypass or hide iOS privacy indicators.
