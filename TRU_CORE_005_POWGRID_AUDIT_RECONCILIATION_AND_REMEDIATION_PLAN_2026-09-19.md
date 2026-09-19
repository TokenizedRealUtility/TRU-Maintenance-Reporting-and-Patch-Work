# TRU Core 0.05 — PoWGrid Round 1 Audit Reconciliation & Remediation Plan

**Source-grounded disposition of all 15 audit tracks**  
**Date:** September 19, 2026

> Baseline: supplied post-Core-0.05 / post-LOGGING-01C source archive, TRU security roadmap, maintenance record, and PoWGrid Round 1 audit.

## Executive conclusion

This document reconciles the PoWGrid “TRU Core v0.05 Security Audit — Round 1” against the supplied post-Core-0.05 / post-LOGGING-01C TRU source archive, the TRU Security Hardening / Tokens / UI Roadmap, and the Public Engineering Progress, Maintenance & Verification Record.

The external audit is useful, but its headline “15 verified security findings” must not be treated as “15 currently exploitable, unpatched vulnerabilities.” Several tracks identify genuine residual defects; several identify defense-in-depth work; several overlap controls that TRU has already implemented; and two proposed changes affect consensus and therefore must not be slipped into an ordinary maintenance release.

The governing rule for remediation is: prove the finding against the current source, preserve already-closed security architecture, add deterministic regression tests before changing behavior, and isolate consensus changes behind an explicit network-upgrade plan.

## Disposition vocabulary

ALREADY CLOSED — The current TRU architecture already contains the substantive defense the audit says is missing. No replacement patch should be applied; only regression coverage or documentation may be added.

REAL / PATCH — A concrete defect remains in the supplied source and can be corrected without intentionally changing the network’s consensus rules. These items belong in the first pinned remediation package.

REAL / DEFER CONSENSUS — The issue is technically real or the current behavior is incomplete, but changing it changes consensus interpretation. It must be implemented as a separately versioned, activation-controlled network upgrade rather than a maintenance hotfix.

HARDENING — The current system already has meaningful defenses or the finding requires additional assumptions, but an additional bounded defense is worthwhile. These changes should be added without overstating the current risk.

FALSE / OVERSTATED FINDING — The audit’s specific claim does not accurately describe the current source. The underlying area may still deserve regression tests, but the claimed missing defense is already present or the stated impact is materially broader than the code supports.

## Master disposition

| Track | Topic | Disposition |
|---|---|---|
| 01 | Binary serialization / VarInt cursor | **REAL / PATCH** |
| 02 | Token self-transfer accounting | **REAL / PATCH** |
| 03 | PoW retarget interval arithmetic | **REAL / DEFER CONSENSUS** |
| 04 | Dust / tiny-output state growth | **HARDENING** |
| 05 | OP_CHECKSEQUENCEVERIFY | **REAL / DEFER CONSENSUS** |
| 06 | RPC storage pointer safety | **REAL / PATCH** |
| 07 | P2P incomplete-frame slow drip | **HARDENING** |
| 08 | Strict DER / Low-S transaction signatures | **FALSE / OVERSTATED FINDING** |
| 09 | Outbound peer handshake completion deadline | **HARDENING** |
| 10 | Crash-safe multi-block reorganization | **ALREADY CLOSED** |
| 11 | Living Token anchor queue fairness | **HARDENING** |
| 12 | HTLC claim/refund safety margin | **HARDENING** |
| 13 | RPC authentication token length bounding | **HARDENING** |
| 14 | Wallet secret memory zeroing / page locking | **HARDENING** |
| 15 | Log injection / control characters | **HARDENING** |

## Track 01 — Binary serialization / VarInt cursor

**Disposition: REAL / PATCH**

### Source-grounded finding

Confirmed in current source. `readVarInt()` uses `read32LE(raw, pos)` / `read64LE(raw, pos)`, whose cursor parameter is already advanced by the helper, and then advances `pos` by another 4 or 8 bytes.

### Why this disposition is correct

This is a parser correctness bug, not a theoretical scanner-only result. It can desynchronize parsing for 0xFE/0xFF CompactSize forms.

### Required action

Patch `tx.cpp` so each extended form advances the cursor exactly once. Add deterministic boundary vectors for FC/FD/FE/FF, truncation, maximum supported values, transaction round-trip serialization, and malformed-input rejection. Apply to both NEW_TRU and TRU-Core with exact pre/post hashes. Build production and test targets, then run parser/regression tests before runtime deployment.

## Track 02 — Token self-transfer accounting

**Disposition: REAL / PATCH**

### Source-grounded finding

Confirmed in current `updateTokenOwnership()`. Sender and receiver keys are identical when `fromAddress == toAddress`; the later receiver `Put` can overwrite the debit computed earlier in the same LevelDB batch.

### Why this disposition is correct

This violates the token balance invariant and can create an inflation path if the vulnerable function is reached with a self-transfer.

### Required action

Add an explicit self-transfer invariant before constructing the batch. A valid self-transfer must never increase supply or balance. Prefer a deterministic no-op/validated success only after proving ownership and amount rules, or reject it consistently. Add regression tests for full-balance, partial-balance, zero/invalid quantity, repeated self-transfer, restart persistence, mempool-to-confirmed behavior, and token U4/reorg round trips.

## Track 03 — PoW retarget interval arithmetic

**Disposition: REAL / DEFER CONSENSUS**

### Source-grounded finding

Current code walks `DIFFICULTY_RETARGET_INTERVAL - 1` ancestor edges while the desired timespan is `DIFFICULTY_RETARGET_INTERVAL * target spacing`. The arithmetic question identified by the audit is therefore real.

### Why this disposition is correct

Difficulty is consensus-critical. Even a mathematically cleaner formula is unsafe as an uncoordinated hotfix because old and new nodes could calculate different required bits at a retarget boundary.

### Required action

One dedicated consensus patch: first freeze test vectors reproducing the current rule at historical and synthetic retarget boundaries; then implement the corrected rule behind an explicit activation height/version; require identical activation configuration across production nodes/miners; test pre-activation compatibility, activation block, next retarget, side-chain candidate ancestry, reorg behavior and restart; publish the activation rule; only then roll out network-wide.

## Track 04 — Dust / tiny-output state growth

**Disposition: HARDENING**

### Source-grounded finding

The audit is directionally valid that fee policy alone does not impose a per-output economic floor. However, TRU already has minimum relay fee, fee-rate admission, bounded mempool size and fee-aware eviction from Patch 12B, so the claim of an entirely unbounded mempool dust surface is overstated.

### Why this disposition is correct

A fee-paying transaction can still create economically tiny spendable outputs that become durable UTXO state after confirmation. That is a state-growth policy issue, not evidence that the existing mempool is unbounded.

### Required action

Define TRU-native dust policy from serialized spend cost and relay economics rather than copying Bitcoin’s 546-unit constant blindly. Add a mempool/relay standardness rule first, with exemptions for canonical protocol outputs that legitimately carry small values. Test token, burn, contract, HTLC, TRUscription and ordinary P2PKH outputs. Do not make it block consensus unless a later explicit consensus decision requires it.

## Track 05 — OP_CHECKSEQUENCEVERIFY

**Disposition: REAL / DEFER CONSENSUS**

### Source-grounded finding

The opcode is named/mapped, but current authoritative opcode support/execution does not implement CSV semantics. This is a real missing feature.

### Why this disposition is correct

The audit overstates the immediate exploitability: the current behavior is fail-closed rather than silently accepting a relative timelock. Implementing CSV changes which scripts are valid/spendable and is therefore consensus behavior.

### Required action

One dedicated consensus patch: define TRU CSV semantics and sequence encoding; create negative/positive vectors; implement interpreter behavior; gate activation by explicit height/version; verify mempool and block-validation parity; test reorg/restart around activation; then roll all validating nodes/miners to the same activation schedule. Do not combine CSV activation with unrelated maintenance fixes.

## Track 06 — RPC storage pointer safety

**Disposition: REAL / PATCH**

### Source-grounded finding

Current RPC code still contains direct `chain.getStorage()->...` dereferences. Even though normal production construction should provide storage, externally reachable handlers should not rely on an unchecked pointer invariant.

### Why this disposition is correct

A defensive null check converts a potential process crash into a controlled RPC error. The audit’s assertion that arbitrary malformed JSON alone necessarily makes storage null is not proven, but the dereference is still worth fixing.

### Required action

Capture storage once per handler (`auto* storage = chain.getStorage()`), fail with a deterministic internal-service error when null, and use the checked pointer thereafter. Audit all RPC storage dereferences, not only the two line numbers cited externally. Add tests with unavailable storage/test doubles and confirm no SIGSEGV, no partial writes and stable HTTP/JSON error semantics.

## Track 07 — P2P incomplete-frame slow drip

**Disposition: HARDENING**

### Source-grounded finding

TRU already bounds the receive buffer, throttles inbound bytes/messages, uses SO_RCVTIMEO/SO_SNDTIMEO, a 15-second select loop, malformed-frame scoring, and disconnects on receive-buffer overflow.

### Why this disposition is correct

The residual issue is narrower: an incomplete but size-valid frame can remain buffered while a peer periodically supplies small amounts of data, continually making progress without completing the frame. Therefore “no transport controls” is false, but an elapsed incomplete-frame deadline is a legitimate defense.

### Required action

Track the monotonic time when the current incomplete frame begins and optionally last meaningful frame progress. Disconnect and score a peer when a frame remains incomplete beyond a bounded deadline, regardless of trickle traffic. Reset the deadline only when a complete frame is consumed or a genuinely new frame begins. Add slow-drip, fragmented-valid-frame, shutdown and reconnect tests.

## Track 08 — Strict DER / Low-S transaction signatures

**Disposition: FALSE / OVERSTATED FINDING**

### Source-grounded finding

The current source already has `ECDSAKey::isStrictDERLowS`, `verifyCanonicalTransactionSignature`, local Low-S normalization, and the script interpreter uses the canonical transaction verifier.

### Why this disposition is correct

The roadmap records Patch 11 as DONE with strict DER, Low-S transaction signatures, local signer normalization, SIGHASH_ALL enforcement and an authoritative sighash path. The audit appears to infer absence because it searches for a particular library normalization symbol or expects Low-S enforcement in generic `ECDSAKey::verify()`.

### Required action

Do not replace the existing Patch-11 transaction-signature path. Instead inventory every remaining caller of generic `ECDSAKey::verify()` and classify whether it signs consensus transactions, VAH records, telemetry or other domain-specific messages. Add regression vectors proving high-S transaction rejection and low-S acceptance. Only tighten a non-transaction signature domain if its protocol explicitly requires canonical Low-S.

## Track 09 — Outbound peer handshake completion deadline

**Disposition: HARDENING**

### Source-grounded finding

Core 0.05 already closed the larger peer-recovery problem through PEER-REDIAL-01B: only VERSION-verified endpoints are promoted for recovery, duplicate reconnects are avoided, backoff/jitter is bounded and shutdown is clean.

### Why this disposition is correct

The audit identifies a narrower state-machine gap: a connected outbound socket can exist before VERSION verification without an explicit application-layer maximum handshake lifetime. Socket timeouts and read-loop activity do not fully substitute for a handshake-state deadline.

### Required action

Add a monotonic VERSION-handshake deadline for unverified outbound sessions. If valid TRU VERSION is not completed within the configured window, disconnect without promoting the endpoint and allow the existing Core-0.05 redial/backoff machinery to handle retry. Test silent peer, garbage peer, slow-but-valid peer, wrong-network VERSION and successful redial.

## Track 10 — Crash-safe multi-block reorganization

**Disposition: ALREADY CLOSED**

### Source-grounded finding

The audit’s architecture description is stale relative to the supplied TRU roadmap. TRU implemented durable U4 undo journals, exact PRE/POST classification, isolated sandbox validation, a durable reorg state machine, controlled multi-block execution, authenticated recovery and end-to-end interrupted-state recovery.

### Why this disposition is correct

The roadmap records 08B.4C as CLOSED / FULL END-TO-END PASS and shows recovery from PREPARED, CONNECT_PUBLISHED and DISCONNECT_COMMITTED states converging to the same logical LevelDB state. Later 08B.4D work adds bounded side-chain lifecycle/DoS controls.

### Required action

Do not replace this with the audit’s generic “single atomic batch” recommendation. Preserve the existing state-machine/journal architecture and add the audit scenario to Patch-17 crash/recovery regression coverage.

## Track 11 — Living Token anchor queue fairness

**Disposition: HARDENING**

### Source-grounded finding

Current source has a global 256-item / 64 KiB anchor queue bound. The bound itself prevents unlimited memory growth, but a shared global queue can create cross-token fairness pressure if one authorized activity stream occupies the available slots.

### Why this disposition is correct

The audit’s statement that any single token spammer can necessarily freeze all evolution is broader than what has been proven because authorization, canonical lineage and trigger admission still apply. The global bottleneck is nevertheless real.

### Required action

Replace pure global first-come capacity with bounded fairness: per-token pending caps plus a global cap, deterministic compaction/deduplication, reserved progress for distinct token IDs, and no bypass of VAH authorization/canonical election rules. Add saturation tests showing one token cannot starve unrelated authorized tokens and restart serialization remains deterministic.

## Track 12 — HTLC claim/refund safety margin

**Disposition: HARDENING**

### Source-grounded finding

The audit identifies a valid cross-chain operational risk class: claim and refund windows should leave enough separation for confirmation and reorg uncertainty.

### Why this disposition is correct

A universal consensus constant is not automatically correct because the safe margin depends on both chains, block intervals, confirmation policy and coordinator behavior. The finding is therefore a swap-policy hardening requirement, not proof that TRU block consensus is broken.

### Required action

Enforce minimum safety deltas in the swap coordinator/agent before constructing or accepting a swap. Express policy in confirmations/time for each chain, require refund ordering, reject unsafe proposals, persist the selected safety parameters in the durable swap record, and test shallow reorg/restart/late-claim scenarios. Keep this outside base TRU consensus unless the HTLC script itself requires a future protocol change.

## Track 13 — RPC authentication token length bounding

**Disposition: HARDENING**

### Source-grounded finding

TRU already uses a constant-time comparison helper and Patch 05 provides authenticated privileged RPC, bounded request bodies, worker/queue limits, timeouts and authentication throttling.

### Why this disposition is correct

The helper loops over `max(a.size(), b.size())`; an oversized supplied Authorization value can therefore add avoidable CPU work. This is resource hardening, not evidence that token equality is non-constant with respect to equal-length secrets.

### Required action

Before constant-time comparison, reject Authorization/Bearer credentials outside a small explicit maximum and require the expected Bearer framing. Keep the constant-time comparison for permitted lengths. Add oversized-header, missing-prefix, wrong-length, wrong-token and correct-token tests; verify 401/appropriate transport behavior and no credential logging.

## Track 14 — Wallet secret memory zeroing / page locking

**Disposition: HARDENING**

### Source-grounded finding

The roadmap itself does not claim the broader wallet-encryption closeout complete. Existing encrypted wallet/session work reduces persistent-secret exposure, but ordinary C++ strings/buffers can leave plaintext key material in process memory and may be swapped.

### Why this disposition is correct

This is primarily a local-host/process-memory defense. It requires a compromised host, debugger/core dump, swap inspection or similar access; it is not a remote key-extraction proof.

### Required action

Treat as a dedicated wallet-security patch: introduce a secure secret container with deterministic zeroization, minimize copies, prevent secrets in logs/exceptions, use page locking where supported with a safe fallback, wipe on lock/logout/destruction, disable/secure core dumps where appropriate, and test unlock/relock, wrong password, restart, backup/recovery and memory-lifetime behavior. Do not rush this into the parser hotfix.

## Track 15 — Log injection / control characters

**Disposition: HARDENING**

### Source-grounded finding

LOGGING-01C already addresses bounded log size, rotation/retention, levels, configured path and flush behavior. The remaining audit point is different: `formatLineLocked` appends caller-provided message text without normalizing CR/LF/control characters.

### Why this disposition is correct

Therefore the audit is wrong if read as “logging is unbounded/unhardened,” but correct that attacker-influenced strings such as peer user-agent text could make multi-line records that visually resemble separate log entries.

### Required action

Sanitize untrusted/control characters at the logger boundary: encode or replace CR, LF, NUL and non-printing controls while preserving useful tabs only if intentionally supported. Keep one physical record per logger call. Add malicious user-agent, embedded newline, ANSI/control-sequence and ordinary Unicode/ASCII tests; verify rotation and severity behavior remain unchanged.

## Remediation release sequence

### Stage 1 — Non-consensus remediation package
Build one pinned, reversible package for Tracks 01, 02 and 06 plus the low-risk hardening portions selected from 07, 09, 13 and 15. The installer must recognize exact NEW_TRU and TRU-Core source hashes, back up every changed file, support `--check` and `--revert`, refuse unknown lineage, build both trees, and run deterministic regression tests before any runtime restart.

### Stage 2 — Architectural hardening
Complete Tracks 04, 07, 09, 11, 12, 13, 14 and 15 as bounded defense-in-depth work. Keep each change within its proper layer: mempool policy, P2P state machine, VAH/evolution queue, swap coordinator, RPC boundary, wallet secret handling and logging. Do not turn policy hardening into block consensus accidentally.

### Stage 3 — One consensus-upgrade patch
Tracks 03 and 05 belong in one explicitly versioned consensus-upgrade release, but they must remain separately testable inside that release. Freeze the old behavior in regression vectors first; implement corrected retarget and CSV semantics; activate at a declared future height/version; deploy compatible binaries before activation; verify activation and subsequent retarget/reorg behavior; retain rollback/recovery procedures. Track 08 is not included because current transaction Low-S enforcement is already present.

### Stage 4 — Patch 17 regression closure
Convert every PoWGrid track into a deterministic test, including findings classified ALREADY CLOSED or FALSE/OVERSTATED. A closed finding stays closed only if the regression suite continuously proves the defense. This turns the external audit from a one-time report into permanent security coverage.

## Release gates

No package is considered closed from source editing alone. Required gates are: exact lineage recognized; static/source tests pass; full native builds pass; unit/regression vectors pass; production test hooks remain absent; no chain/database mutation during install; runtime smoke test passes; clean shutdown/restart passes; and the maintenance/roadmap documents are updated with exact post-build hashes and observed runtime evidence.

## Final disposition summary

The current evidence supports three immediate code defects (Tracks 01, 02 and 06), two consensus changes that must be deferred to an activation-controlled release (Tracks 03 and 05), one already-closed major architecture claim (Track 10), one materially false/overstated transaction-signature claim (Track 08), and eight defense-in-depth hardening tracks (04, 07, 09, 11, 12, 13, 14 and 15). This classification intentionally avoids counting already-deployed controls as missing vulnerabilities while still retaining useful improvements identified by the external review.