# TRU Core 0.05 — PoW Security Audit Patch Work

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

HARDENING — PATCHED
────────────────────────────────────────────────────────────
07  P2P incomplete-frame timeout  PATCHED
09  VERSION handshake deadline    PATCHED / INSTALLED VERIFIED
13  RPC auth input bound          PATCHED
15  Log injection                 PATCHED

HARDENING — REMAINING
────────────────────────────────────────────────────────────
04  Dust policy                   NEXT
11  Evolution queue fairness      PENDING
12  HTLC safety margin            PENDING
14  Wallet secret memory          PENDING

DEFERRED CONSENSUS
────────────────────────────────────────────────────────────
03  Difficulty retarget           DEFERRED
05  CHECKSEQUENCEVERIFY / CSV     DEFERRED
