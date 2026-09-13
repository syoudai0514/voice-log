# ADR-001 — Durable Recording and Recovery

Status: Accepted  
Date: 2026-09-13

## Context

Voice Log's highest priority is to avoid losing recordings. A local database can be deleted, corrupted, migrated incorrectly, or become unavailable. A single large audio file finalized only at stop time also creates excessive loss risk.

## Decision

1. Record into finalized AAC/M4A chunks rather than relying on one final monolithic file.
2. Persist a filesystem-level session manifest independently of the local metadata DB.
3. Treat the local DB as rebuildable index/cache only.
4. Sync finalized chunks independently and idempotently.
5. Never evict a local chunk until cloud preservation has been verified.
6. Retain verified local audio for at least 7 days; on/after day 7, eligible cleanup may evict it.
7. Maintain a small emergency local reserve file that can be released to finalize the active chunk and recovery metadata when storage becomes critically low.
8. Network/cloud failure must not terminate a healthy recording.
9. On launch, scan for recoverable sessions and rebuild/resume state where required.
10. The Action Button entry uses toggle semantics: stopped -> start, recording -> stop, subject to current iOS privacy/foreground requirements.

## Consequences

- More files and manifest bookkeeping than a single-file recorder.
- Much smaller blast radius from crash/storage/database failure.
- Upload can progress before a long recording is finished.
- DB technology can be changed later without redefining audio durability.
- Exact chunk duration is an implementation parameter and may change after device testing.

## Rejected alternatives

### Local DB as sole source of truth
Rejected because DB loss would make otherwise valid audio unreachable or unsynchronized.

### Single final audio file only
Rejected because a crash near the end can threaten the entire session and prevents incremental cloud preservation.

### Delete immediately after upload
Rejected because the user requires at least seven days of local retention after verified cloud preservation.
