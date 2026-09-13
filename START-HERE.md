# Voice Log — START HERE

## Product
Voice Log is an iPhone-first personal audio logging app focused on reliable long-form recording, resilient cloud preservation, and later transcription/summarization.

## Source of truth order
When documents conflict, use this authority order:

1. `docs/requirements.md` — canonical product requirements
2. Accepted ADRs under `docs/adr/`
3. `docs/architecture.md` — current implementation design
4. `CURRENT.md` — current implementation / verification status
5. Source code and tests
6. README / issue / PR / chat history

A lower-authority artifact must not silently override a higher-authority artifact.

## Change rule
Any product behavior change must update the canonical requirement in the same change set when applicable.

## Current phase
Requirements definition in progress. Implementation has not started.

## Safety / privacy boundary
Voice Log may provide a low-distraction recording UI, but must not attempt to hide or bypass iOS microphone/privacy indicators, permissions, or platform disclosure behavior.
