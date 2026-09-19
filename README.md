### PATCH WORK ROAD MAP 
```
──────────────────────────────────────────────
01  VarInt cursor                 CLOSED
02  Token self-transfer           CLOSED
06  RPC storage safety            CLOSED

FALSE / OVERSTATED
──────────────────────────────────────────────
08  DER / Low-S                   CLOSED — existing Patch 11
                                    defense already present

ALREADY CLOSED
──────────────────────────────────────────────
10  Reorg crash/recovery          CLOSED — existing 08B
                                    architecture already present

NEXT
──────────────────────────────────────────────
04  Dust policy                   HARDENING
07  P2P incomplete-frame timeout  HARDENING
09  VERSION handshake deadline    HARDENING
11  Evolution queue fairness      HARDENING
12  HTLC safety margin            HARDENING
13  RPC auth input bound          HARDENING
14  Wallet secret memory          HARDENING
15  Log injection                 HARDENING

DEFERRED CONSENSUS
──────────────────────────────────────────────
03  Difficulty retarget
05  CSV
```


### AUDIT-REMEDIATION-01

It touches exactly **three source files** in each lineage:

```text
src/tx.cpp
src/tokens.cpp
src/rpc_server.cpp
```

and closes only:

```text
TRACK 01   VarInt cursor          
TRACK 02   Token self-transfer       
TRACK 06   RPC storage dereference   
```

### What the fixes actually do

**01 — VarInt:** removes the erroneous second `pos += 4` / `pos += 8`. The LE readers already advance the cursor, so extended CompactSize values now advance exactly once.

**02 — Token self-transfer:** importantly, I did **not** simply reject self-transfers. After validating that the source holding exists and has sufficient quantity, a transfer where:

```cpp
fromAddress == toAddress
```

returns success with ownership **unchanged**. That preserves historical acceptance semantics while preventing the same LevelDB key from receiving debit and credit writes that could inflate its balance.

**06 — RPC storage:** four live RPC handlers that were directly doing:

```cpp
chain.getStorage()->...
```

now acquire the storage pointer and fail closed with:

```text
-32603 Internal storage unavailable
```

if it is absent. I deliberately ignored two additional textual occurrences because they're inside the old commented-out `handleIssueTokenSigned` implementation—not executable code.

### The package self-test reports:

```text
AUDIT_REMEDIATION_01_PAYLOAD_HASHES=PASS
AUDIT_REMEDIATION_01_TRACK01=PASS
AUDIT_REMEDIATION_01_TRACK02=PASS
AUDIT_REMEDIATION_01_TRACK06=PASS
AUDIT_REMEDIATION_01_SELFTEST=PASS
```

No chain reset, database wipe, wallet-format change, P2P-wire change, difficulty change, CSV activation, Low-S replacement, or reorg redesign is included.

## Results
```
TRACK 01  VarInt cursor bug             PATCHED
TRACK 02  Token self-transfer inflation PATCHED
TRACK 06  RPC storage pointer safety    PATCHED
```


