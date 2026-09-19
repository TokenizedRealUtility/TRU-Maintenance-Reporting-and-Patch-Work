# TRU Core 0.05 — PoWGrid Round 1 Audit Reconciliation & Remediation Plan

**Updated:** September 19, 2026

This record tracks TRU's remediation and reconciliation work following the
PoWGrid TRU Core v0.05 Round 1 security review.

The external findings are not being applied blindly.

Each finding is cross-referenced against:

- the current TRU Core 0.05 source;
- the development `NEW_TRU` source;
- the TRU Security Hardening roadmap;
- previously completed security patches;
- existing reorganization and recovery architecture;
- the PoWGrid verification logic where applicable.

Findings are classified as:

- `CLOSED`
- `HARDENING`
- `FALSE / OVERSTATED`
- `ALREADY CLOSED`
- `DEFERRED CONSENSUS`

The objective is to correct demonstrated defects without replacing defenses
that already exist or introducing an accidental consensus change.

---

# PATCH WORK ROAD MAP

```text
REAL / PATCH
────────────────────────────────────────────────────────────
01  VarInt cursor                 PATCHED
02  Token self-transfer           PATCHED
06  RPC storage safety            PATCHED

FALSE / OVERSTATED
────────────────────────────────────────────────────────────
08  DER / Low-S                   CLOSED
                                  Existing Patch 11 defense
                                  already present

ALREADY CLOSED
────────────────────────────────────────────────────────────
10  Reorg crash/recovery          CLOSED
                                  Existing 08B durable
                                  reorganization/recovery
                                  architecture already present

HARDENING — PATCHED / RECOMPILED
────────────────────────────────────────────────────────────
04  Dust policy                   CLOSED
07  P2P incomplete-frame timeout  CLOSED
09  VERSION handshake deadline    CLOSED
11  Evolution queue fairness      CLOSED
12  HTLC safety margin            CLOSED
13  RPC auth input bound          CLOSED
14  Wallet secret memory          CLOSED
15  Log injection                 CLOSED

DEFERRED CONSENSUS
────────────────────────────────────────────────────────────
03  Difficulty retarget           DEFERRED
05  CHECKSEQUENCEVERIFY / CSV     DEFERRED


The consensus findings are intentionally being held until the
non-consensus security work is complete.

They will not be introduced as ordinary maintenance hotfixes.

---

# AUDIT-REMEDIATION-01

## Tracks 01 / 02 / 06

The first remediation package addressed three demonstrated defects while
avoiding intentional changes to consensus, chain state, wallet format or
P2P wire format.

It modifies exactly three source files in each source lineage:

```text
src/tx.cpp
src/tokens.cpp
src/rpc_server.cpp
```

and addresses:

```text
TRACK 01   VarInt cursor
TRACK 02   Token self-transfer
TRACK 06   RPC storage dereference
```

## Track 01 — VarInt Cursor

The extended CompactSize/VarInt parsing path contained redundant cursor
advancement.

The LE reader functions already advance the cursor.

The affected paths then performed another:

```cpp
pos += 4;
```

or:

```cpp
pos += 8;
```

The remediation removes the second advancement so that the extended
CompactSize value advances the parser exactly once.

### Result

```text
TRACK 01 = PATCHED
```

---

## Track 02 — Token Self-Transfer

The token ownership update path used separate debit and credit writes.

When:

```cpp
fromAddress == toAddress
```

the source and destination LevelDB keys are identical.

The destination write could therefore overwrite the previously calculated
debit using the original balance and create an incorrect increased balance.

The remediation does not simply prohibit self-transfer.

After validating the source holding and amount, a valid self-transfer
preserves ownership unchanged.

This maintains the existing accepted operation while preventing the
same-key debit/credit overwrite condition.

### Result

```text
TRACK 02 = PATCHED
```

---

## Track 06 — RPC Storage Safety

Several active RPC handlers directly dereferenced:

```cpp
chain.getStorage()->...
```

The remediation obtains and validates the storage pointer before use.

If storage is unavailable, the RPC fails closed with:

```text
-32603 Internal storage unavailable
```

rather than dereferencing an invalid pointer.

Occurrences located only inside obsolete/commented-out code were not
treated as active vulnerabilities.

### Result

```text
TRACK 06 = PATCHED
```

---

## AUDIT-REMEDIATION-01 Self-Test

```text
AUDIT_REMEDIATION_01_PAYLOAD_HASHES=PASS
AUDIT_REMEDIATION_01_TRACK01=PASS
AUDIT_REMEDIATION_01_TRACK02=PASS
AUDIT_REMEDIATION_01_TRACK06=PASS
AUDIT_REMEDIATION_01_SELFTEST=PASS
```

The package was successfully applied with:

```text
SEMANTIC_GATES=PASS
WRITE_VERIFIED=YES
INSTALLED_VERIFIED
```

No chain reset, database wipe, wallet-format change, P2P-wire change,
difficulty change, CSV activation, Low-S replacement or reorganization
redesign was included.

---

# AUDIT-HARDENING-01A

## Tracks 07 / 13 / 15

The next package addressed transport, authentication and logging
defense-in-depth findings.

It also made the already-correct Track 06 storage guard syntactically
recognizable by the external PoWGrid verifier without changing its behavior.

Scope:

```text
TRACK 07   Incomplete P2P frame lifetime
TRACK 13   RPC authentication input bounds
TRACK 15   Log control-character injection

TRACK 06   External verifier compatibility
           Behavior unchanged
```

---

## Track 07 — P2P Incomplete-Frame Deadline

TRU already contained several P2P resource protections:

```text
bounded receive buffers
SO_RCVTIMEO
SO_SNDTIMEO
select timeout
message/byte throttling
malformed-frame handling
receive-buffer overflow protection
```

The remaining weakness involved an incomplete but otherwise size-valid
frame.

A peer could continue supplying small amounts of data without completing
the frame.

The hardening adds a bounded monotonic incomplete-frame lifetime.

Trickle traffic therefore cannot keep one unfinished frame alive
indefinitely.

Normal fragmented messages remain supported.

### Result

```text
TRACK 07 = PATCHED
```

---

## Track 13 — RPC Authentication Input Bound

TRU already uses constant-time comparison for privileged RPC credentials
and already has RPC request/body/worker/rate controls.

The remaining issue was that the comparison loop could process an
unnecessarily large attacker-supplied credential.

The hardening places an explicit maximum on authentication input before
entering the constant-time comparison.

Valid credentials continue through the constant-time comparison path.

Oversized credentials are rejected before expensive comparison.

### Result

```text
TRACK 13 = PATCHED
```

---

## Track 15 — Log Injection

TRU LOGGING-01C already provides:

```text
bounded log size
rotation
retention
severity levels
deterministic log location
controlled flush behavior
```

The remaining issue was record integrity.

Caller-provided text containing CR/LF or other control characters could
visually create additional log lines.

The hardening sanitizes control characters at the logging boundary so that:

```text
one logger invocation
        =
one physical log record
```

CR, LF, NUL and unsafe control characters can no longer be used to create
fake-looking log records.

### Result

```text
TRACK 15 = PATCHED
```

---

# AUDIT-HARDENING-01B

## Track 09 — Outbound VERSION Handshake Deadline

Core 0.05 already contained substantial peer-recovery protection through
the existing verified-peer recovery architecture.

A peer must complete a valid TRU VERSION exchange before being promoted
into the verified-peer recovery set.

The remaining audit issue was narrower.

A TCP connection could be established while the peer failed to complete
the application-level VERSION handshake.

Socket-level timeouts alone are not equivalent to a maximum handshake
lifetime.

AUDIT-HARDENING-01B therefore introduces an explicit outbound VERSION
handshake deadline.

An outbound connection is given a bounded period to complete the existing
TRU VERSION handshake.

If it fails to do so:

```text
TCP CONNECTED
      |
      v
VERSION WAIT
      |
      +---- valid VERSION ----> VERIFIED PEER
      |
      +---- deadline ---------> DISCONNECT
                                  |
                                  v
                         existing Core 0.05
                         redial/backoff logic
```

The patch does not promote the failed endpoint.

It does not bypass VERSION validation.

It does not change the P2P wire format.

It does not change consensus.

It works with the existing verified-peer redial architecture rather than
replacing it.

## Package verification

```text
AUDIT_HARDENING_01B_PAYLOAD_HASHES=PASS
AUDIT_HARDENING_01B_TRACK09=PASS
AUDIT_HARDENING_01B_SELFTEST=PASS
```

Both source lineages were recognized before modification:

```text
NEW_TRU
STATE=PRE
CHECK_ONLY=PASS

TRU-Core
STATE=PRE
CHECK_ONLY=PASS
```

Installation completed successfully on both:

```text
SEMANTIC_GATES=PASS
WRITE_VERIFIED=YES
TRU-AUDIT-HARDENING-01B INSTALLED_VERIFIED
```

The installer explicitly records:

```text
CONSENSUS_RULE_CHANGE=NO
CHAIN_RESET=NO
DATABASE_WIPE=NO
P2P_WIRE_CHANGE=NO
```

### Result

```text
TRACK 09 = PATCHED / INSTALLED VERIFIED
```

Native build and runtime peer-handshake smoke verification remain the
final operational closeout gate.

---

# FINDINGS ALREADY ADDRESSED BY EXISTING TRU SECURITY WORK

## Track 08 — Strict DER / Low-S

### Audit disposition

```text
FALSE / OVERSTATED
```

The current TRU transaction-signature path already contains the substantive
protection described by this finding.

Existing TRU security work includes:

```text
strict DER transaction signatures
Low-S enforcement
local signer normalization
SIGHASH_ALL enforcement
canonical transaction signature verification
```

This work originated in the existing Patch 11 security-hardening series.

The generic ECDSA verification primitive is not itself proof that
transaction verification accepts high-S signatures.

TRU transaction validation uses the canonical transaction-signature path.

Therefore the existing transaction-security implementation was preserved
rather than replaced merely to satisfy a source-pattern assumption.

### Result

```text
TRACK 08 = CLOSED
REASON = EXISTING PATCH 11 DEFENSE
```

---

# Track 10 — Reorganization Crash / Recovery

### Audit disposition

```text
ALREADY CLOSED
```

TRU's current reorganization system is substantially more developed than
the simple architecture assumed by this finding.

Existing work includes:

```text
durable U4 undo journals
PRE / POST state classification
isolated candidate validation
durable reorganization state machine
controlled multi-block execution
authenticated recovery
interrupted-state recovery
restart convergence
```

The existing 08B architecture is therefore retained.

It is not being replaced with a simplistic reorganization-wide LevelDB
batch solely to satisfy an external source-pattern test.

### Result

```text
TRACK 10 = CLOSED
REASON = EXISTING 08B REORG/RECOVERY ARCHITECTURE
```

---

# NON-CONSENSUS HARDENING CLOSEOUT

All non-consensus findings from the Round 1 audit have now been reconciled.

The final hardening sequence completed:

```text
AUDIT-HARDENING-01C   Track 04   Standard-payment dust policy
AUDIT-HARDENING-01D   Track 11   Evolution queue fairness / race guard
AUDIT-HARDENING-01E   Track 12   HTLC safety-margin policy
AUDIT-HARDENING-01F   Track 14   Wallet secret-memory page locking
```

## Track 04 — Standard-Payment Dust Policy

Track 04 was implemented as relay/mempool policy rather than a new block-consensus rule.

Ordinary spendable P2PKH outputs below the configured 546-atom dust boundary are rejected from normal mempool admission, while protocol-specific anchor/metadata paths retain their dedicated policy treatment.

```text
CONSENSUS_RULE_CHANGE=NO
PROTOCOL_ANCHOR_POLICY_CHANGE=NO
TRACK 04 = CLOSED
```

## Track 11 — Evolution Queue Fairness / Race Guard

The existing bounded Living Token evolution queue was hardened with a per-token pending-anchor limit and synchronization around the queue read/modify/write operation.

```text
MAX_ANCHORS_PER_TOKEN = 8
QUEUE_SCHEMA_CHANGE=NO
TRACK 11 = CLOSED
```

This prevents one token lineage from occupying the entire pending queue and prevents concurrent queue writers from losing each other's updates.

## Track 12 — HTLC Safety Margin

Newly prepared TRU HTLCs now require a minimum safety interval between current chain median-time-past and refund eligibility.

```text
HTLC_MIN_SAFETY_MARGIN_SECONDS = 7200
TRACK 12 = CLOSED
```

This is local HTLC creation/swap safety policy. It does not redefine base-chain consensus and does not replace the coordinator's responsibility to maintain safe asymmetric refund windows across two chains.

## Track 14 — Wallet Secret Memory

The existing encrypted-wallet secret session already zeroized plaintext secret material on lock/destruction. The hardening adds page locking for the live secret session so plaintext key material is not intentionally left pageable while the wallet is unlocked.

The implementation fails closed if the requested memory locking cannot be established and does not change the wallet's persistent file format.

```text
WALLET_FORMAT_CHANGE=NO
TRACK 14 = CLOSED
```

## Build closeout

Following installation of the complete non-consensus hardening sequence, the updated source was successfully recompiled.

This establishes the final source/build checkpoint before any consensus-changing work begins.

```text
NON_CONSENSUS_AUDIT_WORK=COMPLETE
SOURCE_PATCHES=INSTALLED
NATIVE_RECOMPILE=PASS
```

---

# DEFERRED CONSENSUS WORK

Only after the remaining non-consensus hardening is complete will the
consensus audit items be activated.

```text
03  Difficulty retarget arithmetic
05  OP_CHECKSEQUENCEVERIFY / CSV
```

These are intentionally deferred because changing either one can change
what different versions of TRU consider valid.

They will therefore be implemented as an explicitly coordinated consensus
upgrade rather than ordinary maintenance.

Planned process:

```text
CURRENT CONSENSUS
       |
       v
freeze current behavior into deterministic vectors
       |
       v
implement corrected/new rule
       |
       v
activation gate
       |
       v
deploy compatible binaries before activation
       |
       v
activation height/version
       |
       v
post-activation validation
       |
       v
network-wide consensus closeout
```

No silent consensus change will be introduced as part of the security
maintenance packages.

---

# CURRENT AUDIT STATUS

```text
15 ORIGINAL POWGRID TRACKS
          |
          +-- 01 CLOSED — VarInt cursor
          +-- 02 CLOSED — Token self-transfer
          +-- 03 DEFERRED CONSENSUS — Difficulty retarget
          +-- 04 CLOSED — Dust policy
          +-- 05 DEFERRED CONSENSUS — CSV
          +-- 06 CLOSED — RPC storage safety
          +-- 07 CLOSED — Incomplete-frame deadline
          +-- 08 CLOSED — Existing DER / Low-S defense
          +-- 09 CLOSED — VERSION handshake deadline
          +-- 10 CLOSED — Existing reorg/recovery architecture
          +-- 11 CLOSED — Evolution queue fairness
          +-- 12 CLOSED — HTLC safety margin
          +-- 13 CLOSED — RPC auth input bound
          +-- 14 CLOSED — Wallet secret memory
          +-- 15 CLOSED — Log injection
```

## Current counts

```text
NON-CONSENSUS TRACKS RECONCILED / CLOSED  13
DEFERRED CONSENSUS TRACKS                  2
────────────────────────────────────────────
TOTAL                                     15
```

The project has deliberately kept three categories separate:

```text
security defect repair
        ≠
defense-in-depth hardening
        ≠
consensus protocol change
```

The first two stages are now complete. The remaining work is an explicit consensus upgrade.

---

# NEXT PHASE — COORDINATED CONSENSUS RELEASE

The remaining tracks are:

```text
03  Difficulty retarget arithmetic
05  OP_CHECKSEQUENCEVERIFY / CSV
```
