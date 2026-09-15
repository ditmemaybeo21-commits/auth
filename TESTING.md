# Verification ledger

No release has passed the definition of done yet.

## Phase gates

1. Architecture and pre-code audit written. Java 25 and Gradle downloaded with SHA-256
   verification. Official Paper compilation and initial state invariant test: pending.
2. SecureInput keyboard: pending.
3. Registration/login state machine: pending.
4. Argon2id and persistence: pending.
5. Player lock system: pending.
6. Sessions and rate limits: pending.
7. Public API: pending.
8. Concurrency and crash hardening: pending.
9. Paper/Leaf 26.2 runtime: pending.
10. Stress, fuzz and security audit: pending.

## Required automated coverage

Hash/verify/wrong password, salt uniqueness and rehash; session expiry/revocation;
backoff/lockout; legal state transitions; buffer erasure; concurrent operations and duplicate
callbacks; schema migration/corruption; input deadlines and all cleanup paths.

Required stress runs: 100,000 state transitions, 50,000 concurrent session operations,
20,000 duplicate-submit races, 10,000 login/logout cycles, 5,000 lockouts and 100,000
created/closed input sessions with zero active references afterward. Record actual counts,
failures, duration and environment. A loop count alone is not a leak or race proof.

## Live acceptance procedure (Paper and Leaf independently)

Use disposable offline-mode test worlds bound to localhost and a vanilla 26.2 client.
Record the exact server build, Java version and plugin checksum.

- Join a new name: remain locked, create a password through keys, enter mismatched
  confirmation, then matching confirmation. Inspect chat/history and GUI lore for secrets.
- Reconnect: enter a wrong password then a correct password; verify all restrictions
  before success and normal gameplay afterward.
- Logout; verify immediate lock and session revocation. Change password with wrong/right
  current password. Confirm old password fails and all old sessions are invalid.
- Reach backoff and lockout; reconnect and restart during lockout; confirm it persists.
- Restart normally and hard-kill during registration, login and password change. Check
  database integrity and ensure no partially committed credential grants access.
- Spam key/confirm/backspace/clear/close. Test shift, double click, number keys, offhand,
  drag, collect-to-cursor and creative inventory packets. No GUI items may transfer.
- Test spectator, death, external teleports/GUI opens, portals, vehicles, projectiles,
  crafting/trading, item pickup/drop and command suggestions while locked.
- Disconnect during queued and running hash work. Reconnect the same name/UUID and try
  duplicate/case-variant names; stale completions must never unlock a replacement connection.
- Make SQLite unavailable and corrupt a disposable copy. Authentication must fail closed.
- Enable randomized layouts and no-sound mode; test all configured characters and PIN.
- Exercise third-party input completion/cancel/timeout/quit/disable and verify that auth
  credentials cannot be requested through API accessors.
- Monitor tick timing during saturated hash queues; verify bounded workers/queue, rejection
  and cleanup. Inspect logs/database/artifacts using a distinctive disposable test secret.

Never mark these checks passed without executing them. Client tests and runtime tests
cannot be inferred from compilation or pure Java unit tests.
