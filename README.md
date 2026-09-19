**Updated: August 27, 2026** 

## September 19, 2026   Core maintenance addendum

This addendum records the maintenance track completed after the earlier roadmap body. It does not replace the historical patch ordering below.

```text
NETWORK-VERSION-01A    CLOSED
LOGGING-01A            CLOSED / RUNTIME VERIFIED
LOGGING-01B            CLOSED / INCLUDED IN CORE 0.04
PEER-REDIAL-01B        CLOSED / RUNTIME PROVEN
TRU CORE               0.05
LOGGING-01C            PREPARED / STATIC VERIFIED   CURRENT
```

### Core 0.05   automatic verified-peer recovery

`PEER-REDIAL-01B` closes the operational gap where a verified peer socket could disappear and remain absent until manual menu option 32 was used. The reconnect worker only considers endpoints already promoted by a valid TRU `VERSION` handshake, retains ban/abuse authority, avoids duplicate live-IP reconnects, uses bounded backoff/jitter, and exits cleanly during shutdown.

Runtime proof on `gw878` included deliberately killing `137.184.68.43:21833` and observing the peer return in `getpeerinfo` without manual reconnect.

**Consensus change:** NO  
**Wire-format change:** NO  
**Chain reset:** NO  
**Older Core P2P compatibility:** PRESERVED

### LOGGING-01C   final planned logging architecture patch

Current status: **PREPARED / STATIC VERIFIED.**

Purpose:

- finish the application-log work begun by LOGGING-01A/01B;
- bound `Tru_debug.log`;
- add rotation and retention;
- add `ERROR / WARN / INFO / DEBUG / TRACE`;
- default production logging to `INFO`;
- derive the live log path from the configured data/log location instead of a bare process-relative filename;
- preserve urgent/shutdown diagnostics while reducing per-line flush pressure.

Default policy prepared in 01C:

```text
active log maximum   32 MiB
retained rotations   4
level                 INFO
normal flush window   ~1 second
WARN / ERROR flush    immediate
```

Exact modified source surface:

```text
src/logging.cpp
src/logging.h
src/main.cpp
```

Package/static gates already passed:

```text
NEW_TRU exact PRE                  PASS
TRU-Core exact PRE                 PASS
payload hashes                    PASS
apply / exact POST                PASS
revert / exact PRE restore        PASS
logging.cpp C++17 syntax          PASS
level filtering self-test         PASS
rotation / retention self-test    PASS
re-initialization self-test       PASS
```

Required live closeout:

```text
01C apply to NEW_TRU + TRU-Core
        ↓
full native tru_advanced builds
        ↓
restart NEW_TRU
        ↓
verify LOGGING-01C startup marker + resolved path
        ↓
verify RPC / P2P / explorer health
        ↓
verify clean shutdown marker
        ↓
prove configured rotation / retention bound
        ↓
LOGGING-01C CLOSED / RUNTIME PROVEN
```

If those gates pass, **there is no additional planned logging patch in this maintenance sequence.** Later compression or deeper call-site level classification may be treated as optional operational refinement rather than a blocker.

---

## 1. Patch 01   Docker / Private-Config Containment   DONE

- Secrets and private configuration excluded from Docker build context.
- Clean public-config fallback.
- Private key/config containment.
- Patch backups and sensitive runtime files excluded from images/repositories.

---

## 2. Patch 02   Monetary Overflow Hardening   DONE

- Coinbase output bounds.
- Checked coinbase-output summation.
- Checked UTXO/input sums.
- Checked transaction fee accumulation.
- Checked total block fee accumulation.
- Checked `subsidy + fees`.
- `MAX_MONEY` enforcement.

---

## 3. Patch 03   Transaction-ID Recomputation   DONE

- Every block transaction ID recomputed from actual transaction content.
- Claimed txid must match canonical txid.
- Canonical txids verified before Merkle-root validation.
- Coinbase transaction included in txid verification.

---

## 4. Patch 04   Fail-Closed Fork / Reorg Containment   DONE

- Direct-tip-only active-chain acceptance.
- Prevent P2P path from overwriting active chain.
- Tip hash/height consistency enforcement.
- Unsafe rollback/reorganization disabled.
- Persisted-tip inconsistencies fail closed.
- Competing blocks cannot silently replace active state.

---

## 5. Patch 05   RPC / Website Gateway   APPLIED / SOFTWARE VALIDATED 

**Implementation:** `TRU_RPC_WEB_05_PRIVILEGED_RPC_AUTH_PUBLIC_GATEWAY_SITE_COMPAT_REBASED_EXACT_PINNED.sh`

**Status:** SOFTWARE INSTALL PASS / NATIVE BUILD PASS / 

Patch 05 establishes the production security boundary between privileged TRU Core RPC and public browser/web services.

Completed software work:

- Privileged Core `/rpc` defaults to `127.0.0.1`.
- Non-loopback RPC binding requires explicit `rpcAllowRemote=1`.
- Every privileged RPC POST requires Bearer/cookie transport authentication.
- Private RPC credential is generated/stored outside browser JavaScript.
- Direct browser access to privileged Core RPC is forbidden.
- Wildcard Core RPC CORS removed.
- RPC worker pool and queue bounded.
- RPC request-body size bounded.
- RPC transport timeouts added.
- Per-source RPC rate limiting / authentication throttling added.
- Wallet/signing RPC methods remain behind the privileged RPC boundary.
- `tru-cli` automatically authenticates using the Patch-05 RPC cookie/token.
- Native CPU miner authentication installed.
- GPU miner authentication installed.
- Native/internal RPC callers updated for authenticated transport.
- Existing `TRU_SWAP_RPC_TOKEN` retained as a separate inner authorization layer for protected swap RPC methods.
- Existing swap constant-time token comparison converged on the hardened comparison helper.
- Public browser wallet no longer communicates directly with privileged Core `/rpc`.
- Server-side public wallet gateway installed at `/api/wallet/rpc`.
- Server-side browser-mining gateway installed at `/api/mining/rpc`.
- Gateway uses an explicit RPC allowlist rather than unrestricted RPC proxying.
- Core RPC credential is never returned to browser JavaScript.
- Browser self-custody / SEC-14W V3 key generation and signing preserved.
- `wallet-self-custody-v3.js` custody boundary preserved.
- `force_miner_wallet` updated for browser self-custody compatibility.
- Explorer public API remains separately unauthenticated.
- Current Market / Swap Connect Basket generation preserved.
- Market and Swap JS remained outside the Patch-05 consensus/security mutation surface.
- Service-worker epoch advanced to `v23-rpc05-connect-basket-01`.
- Docker host RPC publishing remains loopback-only.
- Raw Docker RPC-cookie path standardized at `/app/data/.rpc-cookie-21832`.
- Docker Compose shared-cookie path remains `/run/tru-rpc/token`.
- Portable source lineage repinned.
- Post-Group-05 active lineage repinned.
- Exact Core/Web prehash and posthash gates passed.
- Exact rollback protection installed.
- Native Core build passed.
- GPU miner build passed.
- Compiled RPC/gateway security-marker gate passed.
- Portable runtime lineage verification passed.

Canonical Patch-05 software-install runtime identity:

- `tru_advanced` SHA-256:
  `b75918cfb99b8d0c4c314836ff2446d1eb1d76afba8667cb40fc16888b6ba846`

- Portable release ID:
  `059a96cfe149195eda9e23d3a0b68190f1edb9d8cc7a5b70757d08457e631e43`

Install closeout:

- `TRU_RPC_WEB_05_REBASED_EXACT=APPLIED`
- `TRU_RPC_WEB_05_NATIVE_BUILD=PASS`
- `COMPILED_RPC_GATEWAY_MARKERS=PASS`
- `PATCH05_RUNTIME_LINEAGE=PASS`
- `SOFTWARE_INSTALL=PASS`
- `CONSENSUS_RULES_CHANGED=NO`
- `CHAIN_RESET_REQUIRED=NO`
- `CHAIN_DB_MUTATED_DURING_INSTALL=NO`
- `WALLET_KEYS_MUTATED_DURING_INSTALL=NO`

### Remaining Patch-05 Runtime Closeout

Before Patch 05 is marked CLOSED, prove on the real GW runtime:

- unauthenticated privileged Core RPC → `401`;
- incorrect RPC credential → `401`;
- browser-origin privileged Core RPC → `403`;
- authenticated `tru-cli` → PASS;
- wildcard Core CORS → ABSENT;
- native Core RPC listens only on intended loopback interface;
- P2P remains publicly reachable on `21833`;
- Explorer `:8001` remains usable without privileged RPC credentials;
- `/api/wallet/rpc` public gateway → PASS;
- `/api/mining/rpc` public gateway → PASS;
- privileged/disallowed wallet RPC through public gateway → REJECTED;
- RPC cookie file mode → `0600`;
- RPC cookie directory mode → `0700`;
- Swap Agent remains compatible with authenticated Core transport;
- RPC credential persists across restart, or clients correctly adopt the regenerated credential;
- website remains functional without any Core RPC credential in browser JavaScript.

---

## 6. Patch 06   Authoritative Block-Acceptance Path   DONE

- `submitBlock()` is the sole public block-acceptance entry point.
- `connectTipBlock()` made private.
- Confirmed-state mutators kept private.
- P2P, sync, mining, and RPC routed through authoritative acceptance.
- Block submission serialized.
- Duplicate confirmed metadata write removed.
- Recovery deadlock corrected.

---

## 7. Patch 07   Candidate-Chain Ancestry   DONE

- Difficulty calculation follows candidate ancestry.
- Median-Time-Past follows candidate ancestry.
- Atomic best-tip snapshots.
- Mining/template generation uses parent-aware difficulty.
- Candidate ancestry moved away from assumptions tied only to the active `chain[]`.
- Provides foundation for isolated fork validation.

---

# 8. Fork / Reorganization Work

## 8A. Patch 08A   Bounded Side-Chain Scaffolding   DONE

- Persistent alternate-branch index.
- Known-parent side blocks accepted into bounded side storage.
- 256-block indexed side-block cap.
- 256 MiB serialized side-pool cap.
- Fork-depth limits.
- Branch-length limits.
- Per-parent child limits.
- Atomic side-block persistence.
- Restart reconstruction.
- Per-source side-fork accounting added in later P2P hardening.
- Active-chain reorganization remains disabled.

---

## 8B. Patch 08B   Actual Safe Reorganization   DONE

### 08B.1   Durable U4 Undo Journals   DONE

- Durable per-active-block undo journals.
- Journal format `P08B1-U4`.
- Exact raw LevelDB PRE-state stored.
- Exact POST-state witnesses stored.
- Canonical state-affecting block fingerprint.
- Exact PRE/POST crash-state classification.
- Safe stale-journal reapply.
- Atomic confirmed-state + undo-journal commit.
- UTXO rollback data captured.
- Token indexes captured.
- Address indexes and transaction counts captured.
- Persistent contract-related LevelDB mutations captured.
- Pre-commit mempool mutation removed.
- Corrupt/mixed journal state fails closed.
- Reorganization remains disabled.

### 08B.2   Authenticated Journal Reader + Safe Single-Tip Disconnect   DONE

- Authenticated `P08B1-U4` journal reader.
- Exact POST-state required before disconnect.
- Exact raw PRE-images decoded.
- Only exact current active tip may be disconnected.
- Genesis disconnect forbidden.
- Standalone disconnect refuses unsafe side-child conditions.
- PRE-state restoration and parent-tip metadata committed atomically.
- `bestTipHash`, `bestTipHeight`, `difficulty`, and height mapping moved atomically.
- In-memory state updated only after durable commit.
- Address/UTXO caches invalidated after disconnect.
- Wallet UTXO view rebuilt afterward.
- Block record and undo journal retained for safe PRE-state reapply.
- Mempool resurrection remains deferred.
- Reorganization remains disabled.

**Runtime note:** the primitive compiles successfully but has not yet been exercised through real reorganization logic.

### 08B.3   Isolated Fork-Point Stateful Candidate Validation   DONE

Current version: **08B.3 v3**

- Creates isolated LevelDB validation sandbox.
- Live confirmed chain is never rolled backward during candidate validation.
- Copies a byte-exact LevelDB snapshot.
- Historical `block:` records omitted from sandbox copy.
- Only required active-suffix U4 journals copied.
- Authenticated U4 journals rewind sandbox state to the exact fork point.
- Candidate branch replay uses normal:
  - `validateBlock()`
  - `applyBlock()`
  - `connectTipBlock()`
- Validation-only blockchain mode introduced.
- Sandbox avoids maintaining a height-sized `chain[]`.
- Candidate difficulty uses `blockIndex` ancestry.
- MTP uses candidate ancestry.
- Difficulty retarget interval hoisted to shared file-scope constant.
- Sandbox ancestry window derived from retarget interval instead of hardcoded `64`.
- Static preflight prevents direct consensus `chain[]` dependencies from being reintroduced.
- Explicit standalone and submission-lock-held validation entry points.
- Explicit standalone and submission-lock-held rollback entry points.
- Recursive `miningSubmitMutex` deadlock eliminated.
- Actual branch promotion remains disabled.
- `handleChainReorganization()` remains fail-closed.

### 08B.3a   LevelDB Registry Lifetime / Lock Hardening   DONE

Completed:

- Fixed process-global LevelDB registry races exposed by runtime sandbox creation/destruction.
- Constructor registry mutation uses exclusive locking.
- Destructor registry mutation uses exclusive locking.
- Eliminated unsafe `s_dbMutexes[dbPath]` lifetime patterns.
- Prevented map rehash/erase from invalidating a mutex being held or waited upon.
- Moved per-path mutex ownership to lifetime-safe handles.
- Protected LevelDB instance/refcount registry consistently.
- Fixed the existing `getContract()` checksum-migration self-deadlock.
- Consensus/reorg behavior remained unchanged.
- Reorganization activation remained disabled.

### 08B.3b   Sandbox Hygiene / Performance Hardening   DONE

Completed:

- Converted process-global `tokenCreationMutex` into per-`Blockchain` instance locking so sandbox replay does not unnecessarily serialize live block application.
- Added non-durable `putBatch(..., sync=false)` support for disposable sandbox writes.
- Live confirmed-state writes remain `sync=true`.
- Non-sync writes are used for disposable sandbox-only state, including:
  - sandbox snapshot copy;
  - sandbox journal rewind;
  - other validation-only sandbox writes.
- Copied snapshot is explicitly checked against the initiating in-memory snapshot:
  - `bestTipHash`
  - `bestTipHeight`
- Sandbox path selection hardened instead of blindly relying on `/tmp`.
- Avoids accidentally placing multi-GB sandbox state on tmpfs.
- Orphan sandbox cleanup added.
- PID/unique process identity included in sandbox path.
- `rollbackBlock*()` documented as a **live-chain primitive only**.
- Sandbox rewind continues to use authenticated manual U4 journal restoration.
- Sandbox database remains a **validation oracle**, not a directory that can simply be swapped into production.

### 08B.3T   Controlled Runtime Test Hooks   DONE

Test-only/debug functionality remains disabled by default.

Completed runtime coverage includes:

1. Authenticated U4 journal reading.
2. Exact active-tip disconnect.
3. PRE-state restoration.
4. Exact-PRE block reapply.
5. Sandbox construction/destruction.
6. LevelDB sandbox copying.
7. Multi-block sandbox rewind.
8. Stateful candidate validation.
9. Real difficulty-retarget-boundary coverage at height 180.
10. Candidate ancestry / MTP behavior exercised through the runtime harness.
11. Live persistent state verified unchanged by sandbox validation.
12. Restart after disconnect/reapply.
13. Missing U4 journal rejection at the real retarget boundary.
14. Durable POST → disconnect → exact PRE → SAFE-REAPPLY → exact POST round trip.
15. Tip hash/height/difficulty/height-mapping/in-memory difficulty assertions.
16. Production-like candidate-validation path exercised.
17. Test-hook production containment verified:
    - production binary contains no 08B.4A debug symbols;
    - production binary contains no hidden `08b4a` CLI;
    - production binary contains no test-enable or mutation magic strings.

Deferred negative tests still recorded for later regression/crash-suite coverage:

- Corrupt height-180 U4 journal rejection.
- Mixed PRE/POST-state rejection.
- Repeated sandbox creation/destruction under meaningful miner/P2P/RPC activity.

These deferred negatives do **not** enable reorganization and do not block the next manually gated executor stage.


### 08B.4   Crash-Safe Multi-Block Reorganization / Promotion   Done

Reorganization remains intentionally **DISABLED**.

#### 08B.4A v2   Durable Reorg State Machine + Full Preflight   DONE

Current rules version: `P08B4A-R2`

Completed:

- Durable `reorg:pending` state-machine plumbing.
- `PREPARED` is the only enabled reorg phase.
- No executor and no automatic activation.
- Strict candidate-winner requirement using Patch-09 authoritative `ChainWork256`.
- Reorg plan records:
  - original active tip;
  - candidate tip;
  - fork point;
  - ordered disconnect suffix;
  - ordered connect suffix;
  - active/candidate cumulative work;
  - captured byte budgets;
  - reorg policies.
- Full preflight verifies:
  - complete active disconnect suffix;
  - required authenticated U4 journals;
  - complete side-branch block availability;
  - side-child safety policy;
  - disconnect/connect/total byte budgets.
- Candidate branch is revalidated through the 08B.3 authenticated sandbox path before any mutation.
- Active tip is rechecked after expensive validation.
- Valid `PREPARED` startup records are deliberately abandoned/cleared while no executor exists.
- Corrupt `reorg:pending` records fail closed before worker startup.
- Hidden development CLI only:
  - `08b4a status`
  - `08b4a preflight <tip>`
  - `08b4a prepare <tip>`
- Production containment verified: hidden reorg test/debug CLI is absent from production builds.

Validated policies:

- `REORG_OLD_BRANCH_POLICY = "REJECT_DISCONNECT_SIDE_CHILDREN"`
- `REORG_MEMPOOL_POLICY = "DEFER_RESURRECTION_UNTIL_REORG_COMMIT"`

Validated limits:

- Undo-journal retention: 200 blocks.
- Disconnect capture budget: 256 MiB.
- Connect capture budget: 256 MiB.
- Total reorg capture budget: 512 MiB.

Runtime proof:

- Winning side branch from fork height 235 preflighted successfully.
- Candidate side tip at height 238 had greater cumulative work than the then-active tip.
- Sandbox validation succeeded.
- `PREPARED` marker write succeeded without changing active state.
- Restart cleared a valid dormant `PREPARED` marker.
- Corrupt marker produced exact fail-closed startup rejection.
- Automatic reorganization remained disabled throughout.

#### 08B.4A.1   `saveChainState()` Durability / Lifetime Fix   DONE

Completed:

- Dirty state is never cleared before durable persistence succeeds.
- Detached commit thread and captured-stack-reference lifetime hazard removed.
- Durable write uses synchronous `dbStorage.putBatch(batch, true)`.
- Dirty flags clear only after exact durable success.
- Failed persistence leaves dirty state available for retry.
- No asynchronous/detached chain-state persistence remains in this path.

#### 08B.4A.1a v2   Deterministic Shutdown / Scaling   DONE

Completed:

- `Blockchain::stop()` made idempotent and deterministic.
- Actual block-queue and member condition variables are notified.
- Block processor and transaction processor are synchronously joined.
- No detach/async shutdown joins.
- Workers quiesce before final chain-state persistence.
- `saveAllBlocksSync()` persists:
  1. dirty active blocks;
  2. height mappings;
  3. tip metadata last.
- Dirty blocks clear only after exact synchronous success.
- Restart reload marks persisted blocks clean.
- Missing active `blockIndex` entries fail closed.
- Main CLI/GUI shutdown ordering made deterministic:
  - stop network/node services;
  - join external threads;
  - stop blockchain;
  - persist wallet/peers.
- Fresh production and development builds passed.
- Production 08B debug/test-hook containment passed.
- Runtime shutdown-after-load/mining passed.
- Final blockchain stop path completed with `durableSave=PASS`.

Known cleanup retained for later:

- `saveHeightMappings()` remains O(height) and should be optimized.
- Global `blockQueue` / `queueMutex` / `queueCond` should eventually become `Blockchain` members.

#### 08B.4A.1b v3   Signal-Safe CLI Wake + Unwind-Safe Shutdown   DONE

The original CLI shutdown bug was reproduced and closed.

V2 established the self-pipe / signal-safe CLI wake path but exposed an exception-unwind join hang inside `startCLI()`.

V3 completed the fix:

- POSIX handlers only:
  - update `sig_atomic_t` state;
  - write one byte to a non-blocking/CLOEXEC self-pipe.
- No logging, mutex, terminal resize, or C++ atomic stores occur inside the signal handler.
- `sigaction()` used for:
  - `SIGINT`
  - `SIGTERM`
  - `SIGWINCH`
- Shutdown and WINCH handlers use `SA_RESTART`.
- CLI input waits use a `poll()` gate on stdin + signal self-pipe.
- All direct interactive:
  - `getline`
  - extraction (`>>`)
  - line-ignore
  sites route through signal-aware wrappers.
- Sticky `g_running` check prevents nested `catch (...)` paths from swallowing shutdown and blocking again.
- `CliWorkerStopGuard` clears local CLI worker stop flags **before** `ThreadJoiner` destructors join them during exception unwind.
- Reorganization logic was untouched.

Fresh-build validation:

- Production hooks OFF.
- Development hooks ON.
- Production debug/test-hook containment: `0 / 0 / 0 / 0`.
- Signal-wake marker present in both binaries.
- Development-only 08B.4A symbols/CLI remain present only in the dev build.

Runtime validation:

- Restarted chain:
  - chain size: 265
  - best tip height: 264
  - best tip hash: `2a02b309d4fd97af5dbecbdf70a831681709029b1679e13bddd3f3fd03000000`
  - chain valid: yes
- Idle-menu `SIGTERM`:
  - process exit: **612 ms**
  - no keyboard input required
  - external threads: **3/3 joined**
  - blockchain worker joins: **0 ms / 0 ms**
  - `durableSave=PASS`
  - `Blockchain::stop()`: **30 ms**
- Nested-input `SIGTERM`:
  - process exit: **1330 ms**
  - no keyboard input required
  - sticky nested unwind passed
  - `durableSave=PASS`
  - `Blockchain::stop()`: **34 ms**
- `Ctrl-C` / `SIGINT`:
  - signal 2 dispatched correctly
  - no Enter required
  - `durableSave=PASS`
  - `Blockchain::stop()`: **26 ms**
- `SIGWINCH → SIGTERM`:
  - node remained alive after `SIGWINCH`
  - SIGTERM process exit: **1229 ms**
  - `durableSave=PASS`
  - `Blockchain::stop()`: **81 ms**

**08B.4A.1a / 08B.4A.1b deterministic shutdown and signal-safe CLI work are CLOSED.**

#### 08B.4B   Controlled Multi-Block Reorg Executor   DONE

**Implementation:** Patch 08B.4B.1 v2  
**Runtime fault-test support:** Patch 08B.4B.1T v1  
**Status:** CLOSED / BUILD VALIDATED / PRODUCTION CONTAINMENT VALIDATED / HAPPY-PATH RUNTIME VALIDATED / FAIL-STOP RUNTIME VALIDATED

08B.4B implements the first real multi-block reorganization executor for TRU.

This stage DOES NOT enable automatic network reorganization.

Automatic fork reconsideration and automatic reorganization remain explicitly:

**DISABLED**

The executor is restricted to a manually gated, authenticated reorganization plan that has already passed the full 08B.3 sandbox validator and 08B.4A durable preflight.

---

##### Core safety rules

The controlled executor may operate only when all required gates pass:

- A valid authenticated `PREPARED` 08B.4A plan exists.
- The requested candidate hash exactly matches the durable PREPARED plan.
- The candidate is still a strict authoritative `ChainWork256` winner.
- The complete plan is regenerated immediately before mutation.
- The regenerated plan must be byte/topology equivalent to the PREPARED intent.
- Full 08B.4A durability preflight must pass again.
- Full 08B.3 authenticated sandbox rewind/replay validation must already have passed.
- The explicit development mutation gate must be enabled.
- Connected peer count must be zero.
- Active recent miner count must be zero.
- `miningSubmitMutex` is held across controlled execution.
- All required blocks, U4 journals, side-block records, ancestry, sizes, accounting reservations, and capacity limits are validated before the first durable mutation.
- Any inconsistency fails closed.
- No RPC or normal P2P path invokes the executor.
- No automatic fork reconsideration invokes the executor.

The durable execution rules version is:

`P08B4B-E2`

Execution uses two authenticated post-PREPARED phases:

- `DISCONNECT_COMMITTED`
- `CONNECT_PUBLISHED`

These phases are deliberately distinct from the harmless 08B.4A `PREPARED` phase. Until 08B.4C recovery is implemented, encountering either execution phase on startup causes a production fail-closed startup refusal. :contentReference[oaicite:0]{index=0}

---

##### Pre-mutation reservation

Before touching active-chain state, 08B.4B captures and verifies the complete transition:

- ordered disconnect suffix;
- ordered connect suffix;
- authoritative `BlockIndexEntry` data;
- serialized block sizes;
- side-block sources;
- disconnect byte total;
- connect byte total;
- resulting side-pool count;
- resulting side-pool byte usage;
- per-parent child capacity;
- per-source block counts;
- per-source byte accounting;
- per-source/per-parent child accounting.

The executor refuses to begin if any reservation would:

- exceed `MAX_INDEXED_SIDE_BLOCKS`;
- exceed `MAX_SIDE_POOL_BYTES`;
- exceed `MAX_SIDE_CHILDREN_PER_PARENT`;
- exceed reorg disconnect/connect byte budgets;
- cause global accounting underflow;
- cause per-source accounting underflow;
- lose a required candidate or disconnect block;
- change the authenticated PREPARED plan.

---

##### Active-chain disconnect execution

The old active suffix is disconnected in exact tip-to-fork order using authenticated U4 undo journals.

For each disconnect:

1. The expected post-disconnect parent hash and height are calculated.
2. A `DISCONNECT_COMMITTED` E2 execution marker is built.
3. `rollbackBlockWithSubmissionLockHeld()` performs the authenticated rollback.
4. The disconnected former-active block receives a durable `sideblock:<hash>` ownership record.
5. Restored UTXO/state changes, active-tip metadata, difficulty/height state, side-block ownership, and the E2 progress marker are committed as one durable disconnect transition.
6. The new active tip is verified from LevelDB.
7. The exact durable execution-marker payload is read back and byte-verified.
8. Only then may execution continue to the next disconnect.

The marker semantics are explicitly recorded as:

`DISCONNECT_STATE_TIP_AND_MARKER_ATOMIC`

The executor therefore never advances its durable progress witness ahead of the disconnect state it describes. :contentReference[oaicite:1]{index=1}

---

##### Candidate-side promotion

Candidate blocks are promoted from the already validated/indexed side branch in fork-child-to-candidate-tip order.

Promotion correctly handles the previous `blockIndex.count(hash)` problem:

- the candidate block's existing side-chain index/accounting entry is verified;
- its side ownership is removed from in-memory bounded-side accounting before normal active connection;
- its temporary side `blockIndex` entry is removed so `connectTipBlock()` cannot incorrectly treat the promoted candidate as an already-processed duplicate;
- durable `sideblock:<hash>` deletion occurs through the normal state-application path;
- the candidate is then connected through the normal authoritative active-chain connector.

Side accounting is reduced in lockstep for:

- `sideBlockCount_`;
- `sidePoolBytes_`;
- `sideChildCount_`;
- `sideBlockCountBySource_`;
- `sidePoolBytesBySource_`;
- `sideChildCountBySource_`;
- `sideBlockSource_`.

Any missing entry, underflow, parent mismatch, source mismatch, or accounting mismatch fails closed. :contentReference[oaicite:2]{index=2}

---

##### CONNECT_PUBLISHED atomicity

Candidate connection intentionally uses a two-stage persistence model:

1. `applyBlock()` commits candidate state/U4 data and deletes the candidate's durable side marker.
2. Active-tip publication then synchronously commits:
   - `bestTipHash`;
   - `bestTipHeight`;
   - active difficulty;
   - active `height:<N>` mapping;
   - exact `CONNECT_PUBLISHED` E2 marker.

The second batch makes the active-tip metadata and the execution marker atomic with one another.

Its declared marker semantics are:

`TIP_METADATA_AND_MARKER_ATOMIC_AFTER_STATE_BATCH`

After every candidate connect, the executor independently reads back:

- durable best-tip hash;
- durable best-tip height;
- durable difficulty;
- durable active height mapping;
- exact E2 marker payload.

A mismatch at any point enters fail-stop instead of continuing. :contentReference[oaicite:3]{index=3}

---

##### Authoritative ChainWork preservation

Promotion preserves the authoritative `BlockIndexEntry.chainWork` accumulated during side-chain indexing.

The legacy `block.chainWork` field remains non-authoritative and is zeroed for side-chain storage, matching the existing 08B.3/08B.4A indexing model.

After promotion the executor verifies that:

- the candidate index entry still exists;
- its height is unchanged;
- its parent is unchanged;
- its authoritative `ChainWork256` is unchanged.

Any change fails closed.

---

##### Old active branch demotion

Once the candidate branch is completely active, the old active suffix is installed into bounded side-chain accounting in fork-child-to-old-tip order.

The executor requires the durable side marker written during each disconnect to exist and authenticate correctly before installing the old block in side accounting.

It then restores:

- `blockIndex`;
- `sideBlockCount_`;
- `sidePoolBytes_`;
- `sideChildCount_`;
- `sideBlockCountBySource_`;
- `sidePoolBytesBySource_`;
- `sideChildCountBySource_`;
- `sideBlockSource_`.

The final side count and byte usage must exactly match the reservation calculated before mutation.

The demoted old branch must also continue resolving to the original authenticated fork point. :contentReference[oaicite:4]{index=4}

---

##### Height mappings / active metadata

Active height mappings are changed as part of the rollback/connect publication primitives rather than by rewriting the entire active chain after the reorg.

The active-tip durable witness explicitly verifies:

- `bestTipHash`;
- `bestTipHeight`;
- `difficulty`;
- `height:<bestTipHeight>`.

Disconnected active heights are no longer permitted to remain authoritative merely because their block records continue to exist as side-chain blocks.

A disconnected block may remain durably stored, but it is owned as a side block rather than through the authoritative active-chain height mapping.

**Coverage note:** the primary runtime reorg used a 2-block disconnect followed by a 3-block connect, so the disconnected heights 236 and 237 were subsequently replaced by candidate heights 236 and 237 and candidate height 238 was added. A shorter-but-higher-work candidate case, where a top `height:N` mapping remains completely unreplaced, was not separately isolated as its own runtime fixture.

---

##### Final commit

The executor does not clear `reorg:pending` merely because the candidate blocks connected.

Before final commit it verifies:

- candidate hash is the exact in-memory active tip;
- candidate height is exact;
- candidate exists in the authoritative block index;
- durable best-tip metadata matches the candidate;
- final bounded side count matches the pre-mutation reservation;
- final bounded side bytes match the reservation;
- the old branch resolves to the original fork point.

Only after those checks does it synchronously delete `reorg:pending`.

The deletion is then read back and verified absent.

A crash before that final deletion therefore leaves an authenticated E2 execution witness for 08B.4C instead of leaving an unmarked partial reorganization. :contentReference[oaicite:5]{index=5}

---

##### Mempool policy

Mempool resurrection remains explicitly:

`DEFER_RESURRECTION_UNTIL_REORG_COMMIT`

08B.4B does not resurrect transactions from disconnected blocks during an incomplete transition.

Mempool behavior will be revisited only after complete reorganization commit/recovery semantics are proven.

---

##### Fail-stop protection after durable mutation

Once the first durable reorg mutation occurs, any subsequent failure sets:

`reorgExecutionHalted_ = true`

The E2 `reorg:pending` record is deliberately retained.

While halted, live mutation and shutdown-save paths are blocked from republishing potentially stale in-memory state over the durable partial transition.

The halt gates include the confirmed mutation/persistence paths introduced during 08B.4B hardening, including:

- normal block submission;
- live tip connection;
- rollback mutation;
- normal chain-state publication;
- final synchronous block/state save;
- shutdown metadata publication.

The controlled executor logs that 08B.4C recovery is required before restart rather than attempting an unsafe rollback or continuing from uncertain state. The source explicitly sets the halt after durable mutation and retains the pending marker for recovery. :contentReference[oaicite:6]{index=6}

---

##### 08B.4B.1T v1 deterministic failure instrumentation

A development-only test patch was added to create exact crash/recovery witnesses.

Supported deterministic fault points:

- `AFTER_DISCONNECT_1`
- `AFTER_CONNECT_1`

`AFTER_DISCONNECT_1` fires only after:

- the first rollback state is committed;
- the new durable active tip is committed;
- the old-active durable side marker is committed;
- the exact `DISCONNECT_COMMITTED` marker is committed;
- both active-tip metadata and marker payload have been read back successfully.

:contentReference[oaicite:7]{index=7}

`AFTER_CONNECT_1` fires only after:

- all required disconnects have completed;
- candidate block 1 state has committed;
- its candidate side marker has been deleted;
- candidate height-236 tip metadata has committed;
- the exact `CONNECT_PUBLISHED` marker has committed;
- both durable tip and marker have been read back exactly.

:contentReference[oaicite:8]{index=8}

The fault hooks are compiled only under `TRU_08B3T_TEST_HOOKS`.

Production containment was verified:

- `debug08B4A` symbols: `0`
- `debug08B4B` symbols: `0`
- hidden `08b4a` CLI strings: `0`
- `TRU_08B4B_FAILSTOP_POINT`: `0`
- `AFTER_DISCONNECT_1`: `0`
- `AFTER_CONNECT_1`: `0`
- 08B3T enable magic: `0`
- 08B3T mutation magic: `0`

The production executable remained byte-identical to the pre-test-hook production executable:

`1e0765cef206e2e5fe9357c08e0f4317a7e687f4bbb85b6eb361b316646524a2`

This confirms the deterministic fault-injection surface compiled completely out of PROD.

---

##### Runtime validation   successful multi-block reorganization

Validated against a preserved height-237 disposable fixture.

Original active tip:

`2e48a0054cd88e2519116a94e4e1274d5d8be664f7a17f9d4387a79444000000`

Original active height:

`237`

Fork:

`21f43d9edbb597d254bd8c3aaec1dbfac4a06a3705b1b9c54ea9270458000000`

Fork height:

`235`

Winning candidate tip:

`61c11eb8bf19c744521eb10ac844115459f137a0487bd8eb80e3f82d07000000`

Candidate height:

`238`

Validated transition:

- disconnect old height 237;
- disconnect old height 236;
- connect candidate height 236;
- connect candidate height 237;
- connect candidate height 238.

Runtime result:

`08B.4B controlled reorg COMMIT PASS`

Observed:

- disconnected: `2`
- connected: `3`
- final active height: `238`
- final active hash: `61c11eb8...000000`
- final side count: `25`
- final side bytes: `13847`
- `reorg:pending`: `ABSENT`
- mempool resurrection: `DEFERRED`
- automatic reorganization: `DISABLED`
- node remained alive after successful execution.

Clean shutdown subsequently completed with:

`durableSave=PASS`

Restart loaded the same candidate height-238 tip.

The old active tip remained a recognizable indexed side candidate and was rejected by preflight for the correct reason:

`candidate chainwork is not strictly greater than active chainwork`

This proved that the former active branch survived demotion/restart rather than disappearing from ancestry/index state.

---

##### Negative-gate runtime validation

The following were explicitly runtime tested:

1. **Mutation gate disabled**
   - executor refused invocation;
   - no mutation occurred.

2. **Mutation gate enabled but no PREPARED marker**
   - executor refused;
   - log confirmed:
     `authenticated PREPARED reorg marker is absent | no durable reorg mutation committed`

3. **Authenticated PREPARED marker exists but requested candidate differs**
   - executor refused:
     `requested candidate does not match PREPARED marker`
   - active tip remained unchanged;
   - no durable reorg mutation occurred.

4. **Valid matching PREPARED plan**
   - executor successfully completed the real 2-disconnect / 3-connect reorganization.

---

##### Runtime validation   DISCONNECT_COMMITTED failure state

Deterministic failure was injected immediately after the first authenticated disconnect.

The durable state was:

- phase: `DISCONNECT_COMMITTED`
- durable height: `236`
- durable active hash:
  `c7c10365ef0503fc709cf6cf67521318647670533df3d169bdd20bcf43000000`
- `reorg:pending`: retained.

The node entered fail-stop.

Shutdown behavior:

- final chain-state republish refused;
- shutdown metadata republish refused;
- normal `saveChainState` republish refused;
- `durableSave=FAIL`;
- durable partial-reorg evidence preserved.

A hook-free production binary was then started against the interrupted database.

Production loaded the actual durable height-236 state and then exited with code `1`:

`in-progress 08B.4B execution marker phase=DISCONNECT_COMMITTED requires 08B.4C recovery; refusing startup fail-closed`

No CLI/node operation was allowed.

---

##### Runtime validation   CONNECT_PUBLISHED failure state

A second deterministic failure was injected after candidate connect 1.

The durable state was:

- phase: `CONNECT_PUBLISHED`
- durable height: `236`
- durable candidate hash:
  `4202d4364711542ba89190fec8463df9305d38ae0e204f03892526485c000000`
- `reorg:pending`: retained.

The node again entered fail-stop and shutdown completed with:

`durableSave=FAIL`

A hook-free production binary then loaded the actual candidate height-236 durable tip and exited with code `1`:

`in-progress 08B.4B execution marker phase=CONNECT_PUBLISHED requires 08B.4C recovery; refusing startup fail-closed`

This proves the state machine distinguishes a committed disconnect-prefix state from a committed candidate-connect-prefix state instead of inferring recovery position from memory.

---

##### Preserved 08B.4C recovery fixtures

Two immutable working copies have now been preserved for 08B.4C development:

`08b4c-fixture-disconnect-committed`

- size approximately `13M`
- durable phase: `DISCONNECT_COMMITTED`
- durable active tip: old branch height 236

`08b4c-fixture-connect-published`

- size approximately `13M`
- durable phase: `CONNECT_PUBLISHED`
- durable active tip: candidate branch height 236

These fixtures must remain untouched.

08B.4C tests should always operate on disposable copies of these preserved databases.

---

##### Final 08B.4B disposition

Patch 08B.4B.1 v1:

**REJECTED / NEVER APPLIED**

Patch 08B.4B.1 v2:

**DONE / APPLIED / BUILD VALIDATED / PRODUCTION CONTAINMENT VALIDATED / FULL HAPPY-PATH RUNTIME VALIDATED / FAIL-STOP RUNTIME VALIDATED**

Patch 08B.4B.1T v1:

**DONE / DEVELOPMENT-ONLY TEST SUPPORT / PRODUCTION CONTAINMENT VALIDATED**

08B.4B has now proven:

- authenticated manual reorganization execution;
- real multi-block disconnect/connect;
- bounded candidate promotion;
- bounded old-branch demotion;
- authoritative ChainWork preservation;
- durable progress markers;
- success marker clearing;
- clean restart persistence;
- old-branch persistence;
- pre-mutation rejection gates;
- deterministic post-disconnect failure;
- deterministic post-connect failure;
- fail-stop shutdown;
- prevention of stale shutdown republishing;
- production restart refusal from both known partial execution phases.

**Automatic network reorganization remains DISABLED.**

**08B.4C crash recovery is now the next reorganization stage.**

#### 08B.4C   Crash Recovery for Multi-Block Reorganization   DONE

Implemented and crash-tested.

Completed recovery guarantees:

- Durable execution phases beyond `PREPARED`:
  - `DISCONNECT_COMMITTED`
  - `CONNECT_PUBLISHED`
  - authenticated `CONNECT_STATE_BEFORE_PUBLISH` recovery gap
  - independent `FINALIZE`
- Deterministic restart classification for every tested transition.
- Restart recovery re-reads and reclassifies the authenticated E2 marker before every mutation.
- Live active-tip metadata, block identity, ancestry, Merkle state, U4 journal state, and cumulative-work witnesses are authenticated before recovery continues.
- `CONNECT_STATE_BEFORE_PUBLISH` is distinguished from an unapplied candidate block using exact U4 POST-state plus durable side-marker absence.
- Publish-only recovery never re-runs `applyBlock()`.
- Recovery only mutates when the explicit runtime gate is supplied:
  - `TRU_ENABLE_08B4C2_RECOVERY=I_ACCEPT_AUTHENTICATED_FINISH_FORWARD`
- Default startup remains fail-closed / classification-only when the recovery gate is absent.
- Interrupted DISCONNECT recovery authenticates already-demoted old-branch side children before allowing the next active-tip rollback.
- Standalone rollback side-child rejection remains intact outside authenticated recovery.
- Recovery failures retain `reorg:pending` and halt further recovery mutation.
- E2 is cleared only after independent final-state authentication.
- Mempool resurrection remains deferred until committed success.
- Automatic/network reorganization remains disabled.

Crash fixtures completed successfully:

1. `CONNECT_STATE_BEFORE_PUBLISH`
   - recovered using publish-only metadata/E2 completion
   - `applyBlock()` was never re-run
   - PASS

2. `CONNECT_PUBLISHED`
   - resumed at the next candidate CONNECT
   - PASS

3. `DISCONNECT_COMMITTED`
   - resumed after one old-branch block had already been disconnected
   - authenticated recovery-only side-child bridge accepted exactly the previously disconnected child
   - PASS

All three recovery paths converged to the exact same logical LevelDB state:

- Final tip:
  `61c11eb8bf19c744521eb10ac844115459f137a0487bd8eb80e3f82d07000000`
- Final height:
  `238`
- Canonical logical LevelDB SHA-256:
  `b4e2d861f506777abd5b27dbc36d70cb80d19fe934824276075b70574d35a7d0`
- Key count:
  `1613`
- Logical dump bytes:
  `1437066`

Final clean gate-OFF restart also passed:

- exact final tip/height loaded
- no interrupted-recovery classification
- no fatal recovery-state condition
- PRE/POST logical LevelDB dumps byte-identical
- recovery gate remained OFF

Pinned 08B.4C.2 v2.2 source:

- `src/blockchain.h`
  `7f27b4dcec6ed505336dce8ce3e683a4c0b608a4977aa416eea9857748cac456`
- `src/blockchain.cpp`
  `0d66b21bcf72bbef03654a3424ddaf3cc382d8a84b79f4d63ca5d0650525f5f6`
- `src/main.cpp`
  `568d114699b8d9bd002cbe8482ba0374a7358b447d665783d9d1b9734586d61c`

08B.4C status:

**CLOSED / FULL END-TO-END PASS**

---

#### 08B.4D   Production Reorg Activation / DoS Controls - DONE

Automatic/network fork promotion remains **DISABLED**.

Production activation will not be considered until the remaining bounded-side-chain lifecycle and DoS controls are implemented and regression-tested.

Planned order:
##### 08B.4D.1   Runtime Stale-Side Pruning   DONE

- Identify side blocks that can no longer participate in a valid bounded reorganization.

- Prune stale side-block payloads and their authenticated `sideblock:` ownership markers.

- Keep all side-pool counters/indexes consistent.

- Never prune blocks or U4 journals still required by:
  - active recovery
  - `reorg:pending`
  - allowed fork depth
  - candidate validation

- Make pruning deterministic and restart-safe.

- Bound pruning work per invocation.

- Enforce the reviewed stale-fork boundary:
  - `forkDepth=100` → retain
  - `forkDepth=101` → prune

- Preserve U4 journals during 4D.1 pruning; U4 retention/pruning is owned by 08B.4D.3.

- Verified runtime pruning at height 300.

- Verified clean restart with:
  - `sideCount=24`
  - no additional prune commits
  - byte-identical logical LevelDB state

- Automatic/network reorganization remains DISABLED.

**Status: CLOSED.**


##### 08B.4D.2   Work-Aware Side-Pool Admission / Eviction   DONE

- Prefer viable higher-work competing branches under bounded side-pool pressure.

- Prevent cheap low-work branches from exhausting the bounded side pool.

- Preserve deterministic source/accounting behavior.

- Use authoritative cumulative `chainWork` for replacement decisions.

- Permit replacement only when one deterministic eligible victim resolves all active capacity limits.

- Victim must be:
  - SIDE-owned
  - non-active
  - leaf-only
  - not the incoming block's parent
  - strictly lower cumulative work than the incoming block

- Equal-work candidates must never evict an existing side block.

- For untrusted sources:
  - an untrusted candidate cannot evict a `local` side block
  - per-source and per-source-parent limits remain enforced

- Deterministic victim selection:
  1. lowest cumulative work
  2. lexicographically lowest block hash on equal work

- Fail closed if `reorg:pending` exists or cannot be read reliably.

- Stage incoming in-memory state before durable mutation.

- Perform replacement as one atomic durable operation:
  - delete `block:<victim>`
  - delete `sideblock:<victim>`
  - add `block:<incoming>`
  - add `sideblock:<incoming>`

- Do not prune or modify U4 journals during 4D.2.

- Verified production/DEV compile separation:
  - PROD: 4D.2T test fixture compiled OUT
  - DEV: 4D.2T test fixture compiled IN

- Verified gate-OFF protection:
  - hidden fixture command rejected before candidate construction
  - no PoW started
  - no replacement occurred
  - logical DB remained byte-identical

- Verified equal-work pressure behavior through the real pipeline:
  - real PoW
  - real `validateBlockForIndex`
  - real `submitBlockInternal`
  - equal-work candidate rejected
  - `sideCount=24`
  - `rpcCount=22`
  - candidate remained non-durable
  - no replacement commit
  - DB remained byte-identical

- Verified strict-higher-work replacement through the real pipeline:
  - real PoW
  - real `validateBlockForIndex`
  - real `submitBlockInternal`
  - higher-work candidate accepted
  - exactly one deterministic lower-work RPC leaf evicted
  - `sideCount=24`
  - `rpcCount=22`
  - candidate durable = PRESENT
  - victim durable = ABSENT
  - victim U4 raw state unchanged
  - U4 journal count remained `301`
  - exactly one replacement commit

- Verified exact durable replacement delta:
  - removed only `block:<victim>`
  - removed only `sideblock:<victim>`
  - added only `block:<incoming>`
  - added only `sideblock:<incoming>`
  - no other common durable key/value changed

- Verified gate-OFF restart of the mutated fixture:
  - active height remained `300`
  - `sideCount=24`
  - `sideBytes=13311`
  - `rpcCount=22`
  - `U4_COUNT=301`
  - incoming replacement survived restart
  - victim remained absent
  - zero additional replacement commits
  - zero 4D.2T fixture executions
  - logical LevelDB state remained byte-identical across restart

- Automatic/network reorganization remains DISABLED.


##### 08B.4D.3   Undo Journal Retention / Pruning   DONE

**Status: CLOSED / BUILD VALIDATED / RUNTIME BOUNDARY VALIDATED / `reorg:pending` SUPPRESSION VALIDATED**

- Final active-chain U4 retention depth:
  - `UNDO_JOURNAL_RETENTION_DEPTH = 200`
  - `MAX_SIDE_FORK_DEPTH = 100`
  - retention margin preserves the full bounded reorganization window.
- Pruning is limited to active-chain U4 journals that are provably outside every protected window.
- Pruning remains suppressed whenever `reorg:pending` exists or cannot be authenticated safely.
- Non-active/side-branch U4 journals are not deleted merely because active retention advances.
- Pruning work is bounded and performed under the existing submission-ordering safety contract.
- Automatic/network reorganization remains disabled.

Runtime boundary proof:

- Starting active height: `300`
- Advanced through height: `314`
- Accepted new blocks: `14`
- Old active U4 journals removed: `112`
- Boundary result:
  - height `113`, depth `201` → **ABSENT**
  - height `114`, depth `200` → **PRESENT**
- Active-chain U4 journals after test: `201`
- Total U4 journals after test: `203`
- Preserved non-active U4 journals at heights `236` and `237` remained byte-identical.
- Source and original fixture remained unchanged.
- Host runtime/recovery gates remained OFF.

`reorg:pending` suppression proof:

- A real authenticated `PREPARED` marker was created through the DEV-only 08B.4A path.
- Active tip then advanced by one direct-tip block.
- 4D.3 detected the pending marker and suppressed pruning.
- Prune commits: `0`
- Existing old U4 journals: unchanged.
- One new active U4 was added for the newly accepted block.
- No 08B.4A executor ran.
- Automatic/network reorganization remained disabled.

**08B.4D.3 is CLOSED.**

---

##### 08B.4D.4   Bounded Positive Candidate Validation Cache   DONE

**Implementation:** 08B.4D.4 v1b  
**Rules:** `P08B4D4-V1B`  
**Status: CLOSED / APPLIED / FRESH PROD+DEV BUILD PASSED / RUNTIME PROOFS PASSED**

Purpose:

- Avoid repeating the expensive authenticated 08B.3 sandbox when the exact candidate, reorg plan, durable state, and relevant LevelDB state are unchanged.
- Positive results only.
- Generic negative 08B.3/08B.4A results are never cached here.

Cache bounds:

- Memory only.
- Deterministic FIFO eviction.
- Maximum entries: `32`
- Maximum accounted bytes: `65536`
- No durable cache writes.
- No consensus-state writes.

The cache key/witness is not candidate-hash-only. It binds:

- candidate tip identity;
- reorg plan SHA-256;
- exact active tip / height / difficulty;
- LevelDB mutation generation;
- disconnect/connect block identities, heights, and payload hashes;
- authoritative side-block records;
- authenticated U4 journal records;
- durable tip/height/difficulty/height mapping;
- contract-state snapshot, including explicit empty-byte-vector handling;
- other state-affecting payload fingerprints required by the 08B.3 sandbox.

LevelDB mutation-generation protection:

- Process-local generation is tracked per exact database path.
- Normal LevelDB mutators bump generation after successful mutation.
- Sandbox databases use different paths, so sandbox writes do not invalidate the live database generation.
- Generation wrap/bookkeeping failure disables cache use fail-closed rather than pretending state is unchanged.
- Process restart clears both the memory cache and generation registry.

Mandatory checks that a cache HIT does **not** bypass:

- initial 08B.4A durability preflight;
- reorg-plan rebuild/equivalence;
- final durability recheck;
- final witness rebuild/equality;
- normal fail-closed reorganization guards.

Runtime proof   positive MISS → STORE → HIT:

- First preflight:
  - cache MISS: `1`
  - sandbox executed: `1`
  - positive STORE: `1`
- Second identical preflight:
  - cache HIT: `1`
  - sandbox re-execution: `0`
- Active height remained `237`.
- No PREPARED marker.
- No controlled reorg commit.
- No mining.
- No 4D.3 prune commit.
- Logical LevelDB state remained byte-identical.
- Original fixture and source remained unchanged.

Runtime proof   same-tip LevelDB generation invalidation:

- First preflight produced a positive cache STORE.
- A real authenticated `PREPARED` marker then created a same-tip LevelDB mutation without changing:
  - active tip;
  - active height;
  - reorg plan;
  - U4 payloads;
  - side-block payloads.
- The LevelDB mutation generation changed the cache witness.
- The next preflight became a MISS and re-executed the full sandbox.
- Exact durable delta was only the expected `reorg:pending` key.
- This closed the stale-positive same-tip mutation hole.

Runtime proof   missing required U4 fails before cache:

- One required active-tip U4 journal was deleted **offline from a disposable database only**.
- Node still loaded exact active height `237`.
- 08B.4A durability preflight rejected:
  - `undo journal absent inside declared retention horizon`
- Cache MISS count: `0`
- Cache HIT count: `0`
- Cache STORE count: `0`
- 08B.4A preflight PASS count: `0`
- Failed preflight made zero additional durable DB mutations.
- Original fixture remained untouched.

Deferred to 08B.4D.8 regression:

- 33-entry FIFO churn stress for the 32-entry positive cache.

**08B.4D.4 is CLOSED.**

---

##### 08B.4D.5   Immutable-Only Negative Candidate Cache   DONE

**Implementation:** 08B.4D.5 v1b  
**Rules:** `P08B4D5-N1B`  
**Status: CLOSED / APPLIED / FRESH PROD+DEV BUILD PASSED / PROD RUNTIME PROOFS PASSED**

Purpose:

- Avoid repeatedly performing the same intrinsic validation work for candidate payloads that are permanently invalid.
- Never convert state-dependent, time-dependent, ancestry-dependent, durability-dependent, ZK-dependent, UTXO-dependent, or sandbox-dependent failures into durable negative assumptions.

Cache scope:

- `validateBlockForIndex()` intrinsic-only rejection classes.
- Memory only.
- Deterministic FIFO.
- Maximum entries: `256`
- Maximum accounted bytes: `65536`
- No LevelDB generation dependency is required because only immutable payload defects are cached.
- No TTL is required for immutable-only entries.
- No consensus-state writes.
- No durable cache writes.

Immutable rejection classes cached include:

- invalid transaction-ID encoding;
- txid/content mismatch;
- duplicate txid;
- Merkle-root mismatch;
- malformed coinbase;
- coinbase-height commitment failure;
- multiple coinbase transactions;
- outputs exceeding `MAX_MONEY`.

Explicitly **not cached**:

- future timestamp;
- Median-Time-Past failure;
- parent/index state failure;
- ancestry/difficulty state failure;
- UTXO-state failure;
- coinbase-maturity failure;
- durability failure;
- snapshot failure;
- ZK failure;
- generic 08B.3 sandbox failure;
- generic 08B.4A negative result.

Runtime proof   immutable rejection STORE → HIT:

- PROD RPC path.
- Authentic candidate header/PoW preserved.
- Transaction bytes were modified to create a permanent `TXID_CONTENT_MISMATCH`.
- First submission:
  - real validator executed;
  - intrinsic rejection occurred;
  - negative cache STORE count became `1`.
- Second identical submission:
  - negative cache HIT count became `1`;
  - txid/content mismatch validator count remained `1`;
  - full intrinsic rejection logic did not rerun.
- Active height remained `237`.
- No side-block store.
- No PREPARED marker.
- No controlled reorg commit.
- No mining.
- Logical LevelDB state remained byte-identical.
- Original fixture/source remained unchanged.

Runtime proof   time-dependent rejection remains uncached:

- PROD RPC path.
- A candidate was rebuilt with:
  - valid recomputed block hash;
  - valid TRU PoW;
  - unchanged transactions;
  - unchanged Merkle root;
  - unchanged parent;
  - unchanged difficulty bits;
  - timestamp approximately `NOW + 86400`.
- First submission:
  - rejected by the real future-time rule;
  - negative STORE count remained `0`;
  - HIT count remained `0`.
- Second identical submission:
  - future-time validation ran again;
  - future-time reject count became `2`;
  - STORE remained `0`;
  - HIT remained `0`.
- This proves time-dependent failures are re-evaluated rather than cached.
- Logical LevelDB state remained byte-identical.
- Original fixture/source remained unchanged.
- Automatic/network reorganization remained disabled.

**08B.4D.5 is CLOSED.**

---

##### 08B.4D.6   Per-Source Candidate-Validation Budget   DONE

08B.4D.6 protects the expensive authenticated 08B.3 candidate sandbox from repeated work by a single untrusted source.

Completed design and implementation:

* Candidate source ownership is resolved from authoritative indexed side-chain state through `sideBlockSource_`.
* Existing source attribution includes:

  * `peer:<IP>`
  * `rpc`
  * `explorer`
  * `p2p-legacy`
  * trusted `local`
* `preflightReorgCandidateWithSubmissionLockHeld()` does **not** accept a caller-supplied `sourceKey`.
* Candidate source is resolved from authoritative indexed side ownership.
* Positive-cache HIT/MISS flow remains authoritative.
* The expensive 08B.3 sandbox executes only on a positive-cache MISS.
* Per-source budget admission occurs immediately before actual expensive sandbox execution.
* Positive-cache HIT consumes **no** sandbox budget.
* Durability-preflight rejection consumes **no** sandbox budget.
* Witness-build failure consumes **no** sandbox budget.
* Trusted `local` source bypasses remote-source rate limiting.
* Budget timing uses `std::chrono::steady_clock`.
* Budget state is process-local memory only.
* No LevelDB persistence or budget-related durable writes.
* No consensus-validity change.
* No caller-supplied source override.
* Automatic/network reorganization remains disabled.

Final production policy:

* Untrusted per-source burst capacity: `4`
* Refill: `1 token / 30 seconds`
* Steady rate: `2 expensive sandbox executions / minute / source`
* Positive-cache HIT charge: `NO`
* Durability-preflight rejection charge: `NO`
* Witness-build failure charge: `NO`
* Trusted `local` bypass: `YES`
* Idle bucket expiry: `900 seconds`
* Maximum tracked sources: `1024`
* New untrusted source when map is full: **fail closed**
* Persistence: **memory only**
* Process restart resets budget state.

DEV-only deterministic pressure fixture:

* Runtime gate:

  `TRU_ENABLE_08B4D6T_TEST_HOOKS=I_ACCEPT_LOCAL_4D6_VALIDATION_BUDGET_PRESSURE`

* DEV pressure capacity: `1`

* DEV pressure refill: `2 seconds`

* Compiled out of PROD.

Completed runtime proofs:

1. First untrusted cache MISS consumed a source token.
2. Repeated positive-cache HIT consumed no token.
3. Exhausted source budget rejected before expensive sandbox execution.
4. Token refill permitted a later sandbox execution.
5. Trusted `local` source bypass remained unaffected.
6. Budget accounting caused no durable DB mutation.
7. Production gate-OFF containment verified.
8. Source/build integrity verified.
9. Automatic/network reorganization remained disabled.

**08B.4D.6 STATUS: CLOSED**

---

##### 08B.4D.7   Global Expensive-Fork DoS Budget   DONE

08B.4D.7 adds a global aggregate limit over expensive authenticated candidate validation so an attacker cannot bypass the 4D.6 per-source budget simply by rotating source identities.

Completed design and implementation:

* Global candidate-validation budget applies across all untrusted sources.
* Per-source 4D.6 and global 4D.7 admission use the same synchronization domain.
* Admission is atomic:

  * both source and global budgets must admit;
  * no partial token charge occurs if either side rejects.
* Positive-cache HIT consumes neither per-source nor global sandbox budget.
* Durability-preflight rejection consumes no sandbox budget.
* Trusted `local` source bypasses remote-source candidate-validation budgets.
* Global timing uses process-local monotonic time.
* No LevelDB persistence.
* No consensus-validity change.
* Existing source attribution remains authoritative.
* Automatic/network reorganization remains disabled.

Final production global policy:

* Global burst capacity: `8`
* Global refill: `1 token / 15 seconds`
* Global steady rate: `4 expensive sandbox executions / minute`
* Scope: aggregate across untrusted candidate sources.
* Trusted `local` bypass: `YES`
* Positive-cache HIT charge: `NO`
* Durability-preflight rejection charge: `NO`
* Partial per-source/global charge on rejection: `NO`
* Persistence: **memory only**
* Process restart resets global budget state.

DEV-only global pressure fixture:

* Runtime gate:

  `TRU_ENABLE_08B4D7T_TEST_HOOKS=I_ACCEPT_LOCAL_4D7_GLOBAL_VALIDATION_PRESSURE`

* DEV global capacity: `1`

* DEV global refill: `2 seconds`

* Compiled out of PROD.

Completed runtime proofs:

1. First qualifying untrusted sandbox execution consumed source + global admission.
2. Global exhaustion rejected additional expensive validation.
3. Rejected dual admission caused **no partial source-budget charge**.
4. Refill restored admission.
5. Trusted `local` bypass remained unaffected.
6. PROD gate-OFF containment verified.
7. No durable DB mutation from budget accounting.
8. Source/build integrity verified.
9. Automatic/network reorganization remained disabled.

**08B.4D.7 STATUS: CLOSED**

---

##### 08B.4D.8   Final Regression / Crash / Adversarial Suite   DONE

Completed 4D.8 coverage:

###### 4A / 4B / 4C regression family

* 08B.4A planning/preflight replay passed.
* Authentic PREPARED-marker behavior passed.
* Controlled 4B happy-path reorganization tests passed.
* 4B fail-stop crash points passed.
* 4C recovery classifications and finish-forward recovery passed.
* CONNECT-state and DISCONNECT-state recovery fixtures passed.
* Gate-OFF recovery behavior passed.
* PREPARED-marker gate-OFF startup clearing passed.
* Exact durable mutation for PREPARED startup cleanup verified:

  * remove `reorg:pending` only;
  * active mappings unchanged;
  * U4 unchanged;
  * side markers unchanged.

###### 4D.1 stale-side pruning regression   DONE

* H299 → H300 stale-side boundary replay passed.
* `forkDepth=100` retained.
* `forkDepth=101` pruned.
* Exactly one eligible stale side leaf removed.
* Restart produced zero second prune.
* Restart DB remained deterministic.
* U4 journals preserved.
* Automatic/network reorganization remained disabled.

###### 4D.2 work-aware admission/eviction regression   DONE

* Equal-work candidate rejection replay passed.
* Strict-higher-work single-leaf replacement replay passed.
* Deterministic one-victim replacement confirmed.
* Restart persistence confirmed.
* Active-chain and U4 state remained protected.
* Automatic/network reorganization remained disabled.

###### 4D.3 active U4 retention regression   DONE

* Retention depth: `200`

* H300 → H314 runtime boundary test passed.

* Old active U4 journals older than retention boundary pruned as expected.

* Depth `201` became prune eligible.

* Depth `200` remained retained.

* Exactly `112` old active U4 journals removed during the H300→H314 run.

* Final active U4 count: `201`

* Nonactive U4 journals remained preserved.

* Clean restart produced zero unexpected mutation.

* Authenticated `reorg:pending` suppression prevented destructive pruning while real prune-eligible work existed.

* PROD startup with gates OFF safely abandoned stale PREPARED marker.

* Exact startup mutation was:

  `DELETE reorg:pending ONLY`

* Active mappings, all retained U4 journals, and all side markers remained byte-identical.

**4D.8 / 4D.3 replay set: DONE

###### 4D.4 positive candidate-validation cache regression   DONE

Completed:

* Cache rules: `P08B4D4-V1B`
* Maximum entries: `32`
* Maximum accounted bytes: `65536`
* Positive-only / process-local cache behavior reconfirmed.
* Deterministic FIFO implementation reconfirmed.
* DEV-only 4D4T FIFO fixture compiled into DEV and completely out of PROD.
* Fresh PROD binary remained byte-identical to the already-closed 4D.7 production binary.

33-entry FIFO adversarial runtime proof passed:

* Fill entries `1..32`.
* Cache size at capacity: `32`.
* Accounted bytes: `9472`.
* Insert entry `33`.
* Entry `1` evicted.
* Entries `2..33` retained.
* Front after insertion: `2`.
* Back after insertion: `33`.
* Cache remained exactly `32` entries.
* Accounted bytes remained exactly `9472`.
* Duplicate entry `33` caused no count/byte/order change.
* Fixture restored original process-local cache state.
* Two consecutive FIFO runs produced identical results.
* Durable DB mutation: `ZERO`.
* No block submission.
* No PoW.
* No reorg/recovery/pruning/mining.

Positive-cache MISS → STORE → HIT replay passed:

* First candidate preflight:

  * `MISS`
  * authenticated sandbox `EXECUTED`
  * positive entry `STORE`
* Second identical preflight:

  * `HIT`
  * sandbox result `CACHED_POSITIVE`
  * no second STORE
  * no second sandbox execution
* LevelDB remained byte-identical.
* Active height remained unchanged.

Same-tip mutation-generation invalidation replay passed:

* Initial candidate preflight: `MISS_STORE`

* PREPARE internal validation reused cached positive result.

* Authentic PREPARED marker written through `LevelDBStorage`.

* Active tip remained unchanged.

* LevelDB mutation generation changed.

* Identical candidate against identical active tip subsequently produced a new `MISS_STORE`.

* First and second cache witnesses differed.

* Exact durable DB delta:

  `ADD reorg:pending ONLY`

* Active height mappings byte-identical.

* U4 set and payloads byte-identical.

* Side markers byte-identical.

* No controlled reorganization.

* No mining.

* No 4D.3 pruning.

Remaining immediate 4D.4 regression:

* Required-U4 fail-closed replay:

  * remove one required active U4 journal from a stopped disposable fixture;
  * candidate durability preflight must reject;
  * positive cache must not be consulted;
  * no cache HIT/MISS/STORE;
  * no active-chain mutation;
  * no additional durable mutation from failed preflight.

After that passes:

**4D.8 / 4D.4 replay set: DONE

###### Remaining deferred 4D.8-only adversarial coverage

These are regression/adversarial tests only. They do not reopen the completed 4D.5–4D.7 patches.

* 4D.5 negative-cache `257`-entry FIFO boundary/churn proof.
* Repeated candidate sandbox construction/destruction stress.
* Multi-source interaction across the already-closed 4D.6 per-source and 4D.7 global budgets.
* Final production gate-OFF verification.
* Final deterministic side-accounting/restart verification as needed.
* Final active-chain / U4 / side-marker accounting audit.
* Final source/build hash lock.
* Confirm all DEV-only pressure/test surfaces remain absent from PROD.
* Confirm automatic/network reorganization remains disabled.

**08B.4D.8 STATUS: CLOSED / FINAL REGRESSION + GATE-OFF ACCOUNTING + HASH-LOCK PASS**

---

##### 08B.4D.9   Production Activation Review   **IN PROGRESS**

08B.4D.9 began only after the complete 4D.8 regression suite closed.

**Important:** entering 08B.4D.9 does **not** automatically enable network reorganization.

Current production automatic/network reorganization state:

**DISABLED**

### 08B.4D.9 Step 1   Activation-Surface Inventory   DONE

Read-only review completed:

- Current source hashes were pinned before review.
- `handleChainReorganization()` remains an intentional fail-closed legacy stub.
- The hardened reorganization machinery is separate from that legacy function.
- Existing hardened core identified:
  - `buildReorgPlanLocked()`
  - `verifyReorgDurabilityPreflight()`
  - `preflightReorgCandidateWithSubmissionLockHeld()`
  - `prepareReorgTransactionWithSubmissionLockHeld()`
  - `executePreparedReorgWithSubmissionLockHeld()`
  - 08B.4C authenticated finish-forward recovery.
- No production automatic caller of the hardened prepare/execute core existed.
- DEV-only manual `08b4a prepare/execute` remained compiled out of PROD.
- Host runtime/recovery gates were OFF.
- No source modification, build, runtime, or DB mutation occurred.

### 08B.4D.9 Step 2   Executor/API + Shutdown-Boundary Deep Dive   DONE

Confirmed:

- Production should **not** resurrect the old `handleChainReorganization()` path.
- Future production activation should call the already-proven 4A/4B core behind an explicit OFF-by-default production policy gate.
- Startup recovery/classification remains owned by 4C.
- The existing normal shutdown path can be woken safely from normal C++ execution context through the signal self-pipe architecture.
- A production activation bridge must distinguish:
  - harmless PREPARED/pre-mutation rejection;
  - partial durable mutation requiring fail-stop + 4C recovery.
- A winning branch already present at startup requires a separate deterministic startup-reconciliation stage; incoming-block activation alone is insufficient.

### 08B.4D.9A   Production Gate + Fail-Stop Shutdown Plumbing   DONE

**Status: APPLIED / FRESH DEV+PROD BUILD PASS / PROD CONTAINMENT PASS / AUTOMATIC ACTIVATION STILL NOT WIRED**

Implemented:

- Exact production opt-in environment gate:
  - `TRU_ENABLE_08B4D9_AUTOMATIC_REORG`
  - exact magic:
    `I_ACCEPT_PRODUCTION_CHAINWORK_REORGANIZATION`
- Gate is OFF by default.
- Wrong magic remains OFF.
- Exact magic may report `ARMED`, but 9A itself does not call PREPARE or EXECUTE.
- Validation sandboxes cannot arm production activation semantics.
- Added thread-safe reorg fail-stop shutdown callback plumbing.
- Main installs the shutdown bridge before P2P exposure.
- Existing normal shutdown atomics/self-pipe are reused.
- No new DEV test surface was added to PROD.
- Legacy `handleChainReorganization()` remains fail-closed.

Protected functions remained unchanged during 9A:

- `submitBlockInternal()`
- `handleChainReorganization()`
- `prepareReorgTransactionWithSubmissionLockHeld()`
- `executePreparedReorgWithSubmissionLockHeld()`

9A canonical source lineage:

- `src/blockchain.h`
  `febbca9b8b4f784099a00e72bac6ef5f9ef6e7de4bd495d7b6f7ee8f0d4dbcc8`
- `src/blockchain.cpp`
  `b0f7b7e9f1be93908fd1a70dc9d466dd56fed734330df9b8daaadd647874637d`
- `src/main.cpp`
  `494c1e48aad2e4961f790fce92593585535bc977cce3c4e9841799ffb1849aba`

9A fresh build lineage:

- DEV `tru_advanced`
  `c51612efc772e5c9c76d4485f0af328048316b7ec82f5b23f1289b0f53ab6a0f`
- PROD `tru_advanced`
  `a3540c8ef6d96f4ec5eb44f63dd5ec0ab882e4eefd26fabe17058be1b1c3d8fc`
- PROD/DEV CLI
  `bd4dd49ba9ddc81dc2284273d1c7a3b35e73e1d993b80160f8885062f0b26c1d`

9A runtime containment proof:

- Gate absent → `OFF`
- Wrong magic → `OFF`
- Exact magic → `ARMED`
- All three:
  - activation = `NOT_WIRED`
  - PREPARED writes = `0`
  - reorg commits = `0`
  - sandbox constructions = `0`
  - recovery commits = `0`
  - logical DB mutation = `ZERO`
- OFF / WRONG / ARMED logical LevelDB dumps were byte-identical.
- Original H300 fixture remained unchanged.
- Closed-4D.8 build artifacts remained unchanged.
- Host gates returned OFF after testing.

**08B.4D.9A is CLOSED.**

### 08B.4D.9B   Incoming Strict-Winning Side-Tip Activation   DRY RUN / PAUSED BEFORE LIVE TEST

Purpose:

- Only after a side block has been fully validated, indexed, and durably persisted, allow an explicitly ARMED production node to consider that freshly accepted side tip.
- Require strict authoritative `ChainWork256` superiority.
- Reuse the existing 4A PREPARE and 4B EXECUTE core.
- Keep startup reconciliation out of this stage; startup is owned by 9C.

9B v1 dry-run:

- Static scope proof passed.
- Existing 4A/4B/4C core functions remained byte-identical.
- Only incoming-side activation wiring was introduced.
- Startup reconciliation remained unwired.
- Legacy `handleChainReorganization()` remained fail-closed.
- One production-semantic issue was found during review:
  - after `indexSideChainBlock()` had already durably accepted the block, a later fatal activation failure could return `false` from `submitBlockInternal()`;
  - this would misreport an already accepted/persisted block as rejected.

9B v1 disposition:

**SUPERSEDED / DO NOT BUILD**

Corrected 9B v1a policy:

- Once `indexSideChainBlock()` succeeds, the block remains reported as accepted.
- A later activation fail-stop may:
  - halt further reorg execution;
  - request orderly shutdown;
  - preserve recovery evidence;
  - but may not rewrite the already committed block-acceptance result.
- Gate OFF / wrong magic:
  - side accept only;
  - no PREPARE/EXECUTE.
- Gate ARMED + non-winner:
  - side accept only.
- Gate ARMED + strict winner:
  - require fail-stop shutdown bridge;
  - PREPARE through existing 4A;
  - EXECUTE through existing 4B.
- Pre-mutation failure:
  - clear only an exact matching PREPARED marker;
  - verify marker absence;
  - preserve accepted side state.
- Partial durable mutation:
  - halt;
  - retain authenticated recovery evidence;
  - request orderly shutdown;
  - 4C remains recovery authority.

**Current 9B status:** corrected v1a dry-run is the next reviewed candidate, but live automatic activation testing is intentionally paused behind the token-U4 proof gate below.

### Token Confirmed-State Containment + U4 Proof Gate Before Live 9B   CURRENT BLOCKER

A deeper source trace performed while building the U4 runtime test found an active out-of-band RPC mutation path that must be closed first.

Current `issuetoken` RPC behavior:

- wallet creates/admit the token transaction;
- `Blockchain::findTransaction()` searches the active chain and then the mempool;
- therefore an unconfirmed token transaction can satisfy the RPC lookup;
- `handleIssueToken()` then directly writes `tokenMetadata:<txid>` through `LevelDBStorage::putWithDataChecksum()`;
- it then calls `ensureTokenMetadataIndexed()`;
- that helper directly writes:
  - `tokenMetadata:<txid>`;
  - `tokenUTXO:<txid>:<vout>`;
  - `tokenOwnerUTXO:<tokenID>:<owner>:<txid>:<vout>`;
  - `tokenIssuance:<tokenID>`;
- those direct RPC writes are outside the block `mainBatch` and therefore outside the U4 journal for the eventual confirming block.

This means the roadmap statement from Patch 10 that no unconfirmed token metadata is persisted has regressed in the current RPC token-issuance path and must be restored before automatic reorganization.

**Immediate rule:** do not attempt to interpret a token issuance U4 round trip until the out-of-band RPC writes are removed/disabled; otherwise the PRE-state is already contaminated by unconfirmed token index state.

Required sequence before any live 9B automatic reorganization test:

0. Close Patch 16A.0 confirmed-state containment:
   - token issuance RPC may create/admit transactions;
   - it may return transaction/metadata information to the caller;
   - it must not mutate confirmed token LevelDB indexes before block confirmation;
   - confirmed token indexing must be owned by authoritative block application / `mainBatch` / U4.
1. Prove an unconfirmed token issuance causes zero confirmed token-index LevelDB mutations.
2. Create/use a disposable fixture containing a real token issuance committed through normal block application.

3. Prove the relevant token state keys are included in the active tip's authenticated `P08B1-U4` journal.
4. Run the existing DEV-only authenticated tip disconnect/reapply round trip.
5. Prove exact token-state restoration:
   - POST before disconnect;
   - exact PRE after disconnect;
   - exact POST after SAFE-REAPPLY.
6. Cover authoritative token state including, as applicable:
   - `tokenOwnership:`
   - `tokenUTXO:`
   - `tokenOwnerUTXO:`
   - issuance/index metadata mutated by block application.
7. Require the entire logical LevelDB state to return byte-identically to the starting POST state.
8. Original fixtures must remain untouched.

Static source review already indicates the active block-application path merges token state into `mainBatch`, after which U4 captures the final state batch. This runtime proof is required to verify that property end-to-end before automatic reorg activation.

### 08B.4D.9B Shadow Mode   DONE

Before allowing a peer-triggered strict winner to persist PREPARED/execute automatically:

- Run automatic winner selection and full durability/preflight logic in shadow mode.
- Log:
  - candidate tip;
  - active tip;
  - active/candidate cumulative work;
  - fork point;
  - disconnect/connect counts;
  - plan SHA-256;
  - source/budget/cache decisions.
- Stop before `persistPreparedReorgMarker()`.
- No active-chain mutation.
- No `reorg:pending` write.
- Use shadow results to validate real network behavior before live production activation.

### 08B.4D.9C   Startup Reconciliation   DONE

* Production automatic reorg remains **OFF by default** and requires the explicit 9A gate to be ARMED with the exact production activation value.
* Startup reconciliation runs **after the fail-stop shutdown bridge is installed and before `node->setBlockchain()`, outbound peer connections, or the P2P listener**.
* Startup deterministically scans only the **bounded, authoritative persisted side-chain set**; no unbounded DB scan is used.
* Candidate selection uses authoritative cumulative **`ChainWork256`** and activates only a **strictly greater-work** side tip.
* Equal-work behavior remains **first-seen / no activation**. If multiple persisted side tips tie for greatest strict-winning work, startup does not invent a new tie-break rule and performs no activation.
* Startup reuses the already-hardened **9B → 4A PREPARE → 4B EXECUTE** path, with existing **4C crash-recovery/reconciliation** machinery retaining precedence for durable pending state.
* Persisted side markers and block records are authenticated before selection. Unknown, inconsistent, corrupt, oversized, or unresolved durable state **fails closed**.
* `reorg:pending` must be absent after recovery before any fresh startup activation is attempted and is verified absent after a successful commit.
* Gate-OFF startup behavior is a verified **no-op**: persisted higher-work side branches remain dormant and active-chain state is unchanged.
* Gate-ARMED startup behavior is verified: a persisted higher-work H239 side tip was selected and committed automatically with **no recovery required**.
* Restart after successful activation is **idempotent**: no second activation and no second reorg commit occur.
* DEV-only reorg/test hooks remain **absent from PROD**; DEV/PROD compile containment passed.
* Legacy `handleChainReorganization()` remains **fail-closed**.
* **08B.4D.9A, 9B, and 9C are CLOSED.**
* Final production binaries are built and authenticated; **automatic network reorg remains disabled by default pending the final production activation review.**


### 08B.4D.9 Production Activation Rule

Production automatic/network reorganization remains:

TRU's reorganization implementation is patched, hardened, and successfully runtime-tested. The reorg development gate is CLOSED. To launch production: 
TRU_ENABLE_08B4D9_AUTOMATIC_REORG=I_ACCEPT_PRODUCTION_CHAINWORK_REORGANIZATION ./tru_advanced


---

## 9. Patch 09   Integer Difficulty + Authoritative Chainwork   DONE

- Integer-only retarget.
- Strict canonical compact targets.
- Exact `floor(2^256 / (target + 1))` block work.
- 256-bit cumulative chainwork.
- Active work reconstructed on restart.
- Side branches receive authoritative cumulative work.
- Peer/disk legacy `Block::chainWork` ignored.
- Boost dependency explicitly declared and pinned.
- Candidate difficulty uses ancestry rather than active-tip assumptions.
- Reorganization remains disabled pending Patch 08B completion.

---

## 10. Patch 10   Mempool + Transaction-Ingress Hardening   DONE

- Canonical txid recomputation.
- Duplicate-input rejection.
- Checked monetary arithmetic.
- 4 MiB transaction limit.
- 64 MiB mempool.
- Bounded P2P validation queue.
- O(1) queued/in-flight duplicate suppression.
- First-seen conflict policy.
- Removed per-transaction `std::async`.
- No unconfirmed token metadata persisted to LevelDB.
- Centralized mempool accounting/removal.
- RPC/wallet bounded-queue rejection propagated correctly.
- Broadcast only after successful local admission.

---

## 11. Patch 11   Transaction Uniqueness + Signature Canonicalization   DONE

- Duplicate txid rejection within blocks.
- CVE-2012-2459-style Merkle duplication defense.
- BIP30-style live UTXO collision protection.
- Coinbase-height commitment.
- Strict DER transaction signatures.
- Low-S transaction signatures.
- Local signers normalize to low-S.
- `SIGHASH_ALL` enforcement.
- Removed alternate `signInputRpc()` ASCII sighash.
- `Transaction::getSigHash()` becomes authoritative transaction sighash implementation.

Addresses major portions of original audit findings H1, H2, and H9.

---

# 12. Patch 12   Script / VM / Relay / P2P Resource Hardening   DONE

## 12A   Consensus Execution Limits   DONE

- Per-block sigop limit.
- Per-transaction and per-block input/output limits.
- Bound BIP30 UTXO lookup workload.
- Script size limit.
- Opcode-count limits.
- Per-input VM/gas ceiling.
- Per-transaction VM/gas ceiling.
- Per-block VM/gas ceiling.
- Stack-item limits.
- Stack-byte limits.
- Shared consensus resource limits between mempool and block validation.

### Remaining cleanup note

- Dead alternate validators such as unused `Transaction::verifyInput()` / `isCoinbaseSpendable()` should eventually be removed or made authoritative-safe rather than left as misleading alternate verification paths.

---

## 12A.1   Interpreter Correctness   DONE

- `OP_IF` / `OP_NOTIF` do not consume stack data inside inactive branches.
- Nested dead-branch correctness.
- Correct `OP_ELSE` execution-state handling.
- Unterminated conditional detection.
- Stack helpers/invariants hardened.
- CHECKMULTISIG stack accounting tested/hardened.
- Explicit nested conditional regression vectors.

---

## 12B   Transaction Relay / Admission Policy   DONE

- Minimum relay fee.
- Fee-rate-aware admission.
- Fee-aware mempool eviction.
- 64 MiB mempool pinning mitigation.
- Externally reachable RPC transaction submission routed through bounded admission.
- Direct `mempool->addTransaction()` paths audited.
- Verbose mempool failures isolated per transaction.
- BUSY/backpressure semantics propagated.

---

## 12B.1   Wallet Compatibility / Size-Aware Fees   DONE

- Size-aware transaction fee calculation.
- Large token metadata fee test.
- MagicLock fee handling.
- Contract/TRUScript fee handling.
- No-fee-bearing-output fail-loud behavior.
- ScriptSig sizing corrected before final fee calculation.

---

## 12C   P2P DoS Defense   DONE

- Per-peer message/request rate budgets.
- Invalid-message/block scoring.
- Temporary bans and eviction.
- Orphan limits.
- In-flight block/transaction limits.
- Peer resource ownership.
- Side-fork source attribution.
- Per-source side-fork quotas.
- Bounded request ceilings.
- P2P lifecycle/shutdown hardening.
- Keep actual **network magic in Patch 15**, where network identity belongs.

---

## 13. Patch 13   Docker / Build Supply-Chain Hardening   DONE

- Remaining dependency pinning.
- Consolidate build dependencies.
- Pin base-image digests.
- Verify/build `libkeccak.a` from source.
- Run runtime containers as `USER tru`.
- Repair healthcheck.
- Fix miner/node image naming.
- Review cookie/network architecture.
- Boost 1.83 installation already handled by Patch 09; do not duplicate it.

---

## 14. Patch 14   Wallet Encryption   PENDING

Required work:

- Encrypt wallet seed/private keys at rest.
- Enforce restrictive `0600` wallet/key-file permissions.
- Introduce a strong password-based KDF.
- Use authenticated encryption.
- Implement wallet lock/unlock behavior and timeout.
- Remove plaintext private-key storage.
- Remove designs that store an encryption key beside the ciphertext it protects.
- Verify encryption/decryption symmetry and failure behavior.
- Add migration/recovery handling appropriate for existing wallet files.
- Confirm secrets never leak through logs, CLI output, Docker layers, backups, or temporary files.

### Closeout gate

Patch 14 should not be marked CLOSED until encrypted-wallet creation, restart, unlock, relock, wrong-password rejection, corruption rejection, file-permission enforcement, and recovery/backup behavior have been runtime tested.

---

## 15. Patch 15   TRU Network / Address Identity   IN PROGRESS

### Status Summary

* **15A   Address / Genesis / HD Identity: CLOSED**
* **15B Audit   P2P Network Identity Mapping: CLOSED**
* **15B.1   P2P Wire Magic + VERSION Network Binding: CURRENT**
* **15B.2   TRU Default RPC / P2P Port Identity: NEXT**
* **15B Runtime / Two-Node Network Separation Proof: PENDING**
* **Patch 15 Final Canonical Apply / Rebuild / Closeout: PENDING**

---

### 15A   TRU Address / Genesis Identity   DONE

Implemented and runtime validated:

* TRU-specific Base58Check P2PKH address namespace.
* Mainnet P2PKH version:

  * `0x41`
  * canonical wallet addresses begin with `T`.
* Testnet P2PKH namespace reserved:

  * `0x7F`
  * intended lowercase `t...` namespace.
* Bech32 HRPs reserved:

  * Mainnet: `tru`
  * Testnet: `ttru`
* Project-local BIP44 coin type:

  * `0x00545255`
  * decimal `5526101`
  * intentionally not claimed as registered SLIP-0044.
* BTC/TRU P2PKH address crossover prevented.
* Existing Bitcoin-style `1...`, `3...`, and `bc1...` wallet address behavior retired from the TRU mainnet wallet path.
* Separate NOVO legacy bridge namespace intentionally preserved and excluded from TRU address rewriting.

#### Fresh TRU Mainnet Genesis   DONE

Canonical genesis identity:

* Genesis address:

  * `TCtVWvC1JtsKXWoEDueN2aNEkpVbjAftQM`
* Genesis HASH160:

  * `2004155a5d7120f6d3c792a9c881544f86ae4a7c`
* Genesis coinbase TXID:

  * `daca4ceae9b5f0ee1bd30eaa93522666bcf37560b6a7310657eb1f7cac7ec139`
* Genesis Merkle root:

  * `daca4ceae9b5f0ee1bd30eaa93522666bcf37560b6a7310657eb1f7cac7ec139`
* Genesis nonce:

  * `46045855`
* Genesis block hash:

  * `b62fba2600030d97a06916b17694bec8d97ca14c1db682b2e57b424bd6000000`
* Timestamp:

  * `1745982427`
* Difficulty bits:

  * `0x1e00ffff`

Genesis durability defect discovered during testing was fixed in Patch 15A v1c.

Runtime proof confirmed:

* `block:<genesis-hash>` is durably stored in LevelDB.
* `height:1` maps to the new genesis.
* `bestTipHash` maps to the new genesis.
* `bestTipHeight=1`.
* Genesis is not recreated after restart.
* Restart produces an identical logical LevelDB state.
* Retired genesis block record is absent.
* Fresh wallet creation generates only TRU `0x41 / T...` addresses.

**PATCH15A_STATUS=CLOSED**

---

### 15B   TRU P2P / Wire Network Identity    DONE 

Patch 15B separates TRU at the actual peer-to-peer protocol layer so a TRU node cannot silently communicate with an old/pre-Patch-15B node or another network using the same message format.

#### 15B Audit   DONE

Audit confirmed current pre-15B behavior:

* **No P2P network magic exists.**

* Current frame is:

  `[type][length][SHA256 checksum][protobuf payload]`

* VERSION messages currently have no explicit TRU network ID.

* A peer is not currently required to prove `tru-mainnet` identity before chain traffic.

* Legacy `8332 / 8333` defaults remain throughout multiple node, CLI, miner, wallet, explorer, and config paths.

Locked Patch 15B mainnet identity:

* Mainnet P2P magic:

  * hex `ec7c8fb9`
  * wire bytes `EC 7C 8F B9`
* Mainnet network ID:

  * `tru-mainnet`
* Reserved testnet network ID:

  * `tru-testnet`
* Reserved testnet P2P magic:

  * `77f1676c`
* Reserved regtest P2P magic:

  * `a476302e`
* Planned mainnet RPC port:

  * `21832`
* Planned mainnet P2P port:

  * `21833`

---

### 15B.1   P2P Wire Magic + VERSION Network Binding   CURRENT

Current active gate.

Target frame:

`[4-byte TRU magic][u32 LE type][u32 LE length][32-byte SHA256][protobuf payload]`

Required behavior:

* Every serialized TRU P2P message begins with:

  * `EC 7C 8F B9`
* Integer frame fields use explicit little-endian encoding rather than host-native memory layout.
* Incoming frame magic is validated before:

  * message type parsing,
  * length interpretation,
  * checksum processing,
  * protobuf parsing.
* Wrong network magic fails closed immediately.
* Pre-Patch-15B peers cannot be interpreted as valid TRU peers.

VERSION handshake changes:

* Add append-only protobuf field:

  * `networkId = 11`
* Outgoing VERSION must send:

  * `tru-mainnet`
* Incoming VERSION must contain exactly:

  * `tru-mainnet`
* Missing network ID fails closed.
* Wrong network ID fails closed.
* VERSION validation must succeed before the connection is considered network-authenticated.

Additional fail-closed handshake rule:

* HEIGHT, BLOCK, TX, GET_BLOCK, GET_HEIGHT, INV, GETDATA and other chain/application traffic must not be accepted until that peer has completed a valid `tru-mainnet` VERSION handshake.
* Correct P2P magic alone is not sufficient to join the TRU network.

Current dry-run target scope:

* `src/tru_network_params.h`
* `src/message_handler.cpp`
* `src/message_handler.h`
* `src/peer_connection.cpp`
* `src/peer_connection.h`
* `src/message.proto`
* regenerated `src/message.pb.h`
* regenerated `src/message.pb.cc`

No canonical 15B.1 source mutation has occurred yet.

**CURRENT NEXT GATE:**
Patch 15B.1 disposable-source dry run → exact diff review → isolated DEV/PROD build → wire negative tests.

Required 15B.1 runtime/static tests:

* Correct magic + valid `tru-mainnet` VERSION → accepted.
* Wrong magic → rejected before protobuf parsing.
* Missing network ID → rejected.
* Wrong network ID → rejected.
* Correct magic but BLOCK/HEIGHT before VERSION → rejected.
* Valid VERSION followed by normal chain traffic → accepted.
* Old/pre-15B framed peer → cannot silently peer with a Patch 15B node.

---

### 15B.2   TRU Default Port Identity   NEXT AFTER 15B.1

Centralize and replace legacy Bitcoin-associated/default ports where they represent TRU node defaults.

Target TRU mainnet defaults:

* RPC:

  * `21832`
* P2P:

  * `21833`

Expected areas include:

* main node defaults.
* blockchain configuration defaults.
* P2P advertised VERSION addresses.
* `tru-cli`.
* CPU miner.
* GPU miner.
* wallet CLI.
* wallet GUI.
* wallet external-node broadcast paths.
* smart-contract broadcast paths.
* block explorer P2P startup.
* example/default `tru.conf` values.

Explicit user-configured ports must continue to override defaults.

Patch 15B.2 should centralize these values in TRU network parameters rather than introducing another set of scattered literals.

---


### 15B Final Runtime Validation   CLOSED

Patch 15B runtime validation is complete.

All required network-separation and valid-peer synchronization behaviors were proven using fresh disposable nodes, isolated local ports, canonical Patch 15B binaries, and automatic reorganization disabled.

#### Matching TRU Node Test   PASS

Two isolated Patch 15B nodes were started using:

* P2P magic `ec7c8fb9`.
* Network ID `tru-mainnet`.
* Separate disposable databases.
* Isolated local RPC/P2P ports.
* Canonical Patch 15B binaries.

A fresh disposable Node A3 initialized successfully at genesis height 1.

A real block was mined and accepted on A3, producing:

* A3 height: `2`
* A3 best hash:
  `0310c4ad5d447ca4dc1565120b25bfa3da9c3bf308287414b8e4283ec5000000`

After the miner exited:

* A3 RPC remained available.
* A3 remained alive.
* A3 retained the accepted block and stable best tip.

A fresh Node B then connected to A3 using the valid `tru-mainnet` Patch 15B protocol.

Runtime proof:

* VERSION handshake succeeded.
* Valid `tru-mainnet` network ID was processed.
* Peers remained connected.
* HEIGHT/block synchronization succeeded.
* Node B reached height `2`.
* Node B best hash exactly matched Node A3:

  `0310c4ad5d447ca4dc1565120b25bfa3da9c3bf308287414b8e4283ec5000000`

Final results:

`VALID_TRU_MAINNET_HANDSHAKE=PASS`

`REAL_BLOCK_SYNC=PASS`

`BEST_HASH_MATCH=PASS`

`A3_POST_MINER_SURVIVAL=PASS`

`DISPOSABLE_DATADIR_CONTAINMENT=PASS`

`DISPOSABLE_NODES_EXITED=YES`

#### Wrong-Magic Test   PASS

A hostile fixture connected using non-TRU framing magic.

Runtime proof confirmed:

* Wrong magic was rejected before protobuf parsing.
* The frame did not enter normal message processing.
* The peer was disconnected/fail-closed.
* No chain synchronization began.
* No canonical database was used or mutated by the disposable hostile-peer test.

Final result:

`WRONG_MAGIC_REJECTED_BEFORE_PROTOBUF=PASS`

#### Wrong-Network Test   PASS

A genuine canonical VERSION frame was captured, its network ID was changed to a foreign value while preserving valid TRU framing, and its frame checksum was recomputed using TRU's actual wire checksum algorithm:

`SHA256(payload)`

The forged VERSION therefore passed:

* TRU mainnet magic validation.
* Frame type validation.
* Payload-length validation.
* Checksum validation.
* Protobuf deserialization.

It then reached the Patch 15B VERSION network validator and was rejected because the network ID did not equal:

`tru-mainnet`

Final result:

`WRONG_NETWORK_ID_REJECTED=PASS`

This proves that valid TRU framing alone is insufficient to join TRU mainnet; the VERSION message must also bind explicitly to the correct TRU network identity.

#### Pre-Patch-15B Compatibility Rejection   PASS BY WIRE-MAGIC ENFORCEMENT

Pre-Patch-15B framing used:

`[type][length][checksum][payload]`

Patch 15B framing requires:

`[magic][type][length][checksum][payload]`

The Patch 15B deserializer verifies the first four bytes against:

`ec7c8fb9`

before type, length, checksum, or protobuf parsing.

Therefore an old pre-Patch-15B frame cannot silently participate in the new TRU mainnet protocol. Its first four bytes are interpreted as network magic and rejected before protobuf processing unless they coincidentally equal the required TRU magic.

The wrong-magic runtime proof directly exercised this fail-closed boundary.

#### Patch 15B Runtime Containment   PASS

Runtime validation also confirmed:

* Canonical Release binary provenance matched.
* Tests used disposable databases.
* No private key was used.
* Automatic production reorganization remained disabled.
* Test nodes shut down cleanly.
* Canonical chain/database state was not used for the destructive portions of testing.

Final Patch 15B state:

`PATCH15B1=CLOSED`

`PATCH15B2=CLOSED`

`PATCH15B=CLOSED`

---

### Patch 15 Final Closeout   READY FOR FINAL IDENTITY SWEEP

Patch 15A and Patch 15B are both functionally complete and runtime validated.

Completed:

1. Patch 15A address/genesis/HD identity applied and validated.
2. Patch 15B.1 P2P wire magic and VERSION network binding applied and validated.
3. Patch 15B.2 canonical mainnet RPC/P2P default ports applied and validated.
4. Canonical DEV/Release binaries rebuilt.
5. Fresh-chain genesis startup validated.
6. Fresh disposable block production validated.
7. Two-node TRU mainnet P2P synchronization validated.
8. Wrong-magic rejection validated before protobuf parsing.
9. Wrong-network-ID VERSION rejection validated.
10. Valid TRU mainnet VERSION handshake validated.
11. Exact synchronized height and best-tip hash convergence validated.
12. Automatic reorganization confirmed disabled unless separately enabled through its existing production gate.
13. Patch 15B testing confirmed no change to the already-closed Patch 15A genesis/address consensus.
14. Canonical Patch 15B binary provenance recorded.
15. Disposable runtime containment and clean shutdown validated.

#### Remaining Administrative Closeout Gate

Before marking the overall Patch 15 package formally CLOSED, perform one final read-only source identity sweep confirming:

* Mainnet address version remains `0x41`.
* Reserved testnet P2PKH version remains `0x7F`.
* Mainnet P2P magic remains `ec7c8fb9`.
* Mainnet network ID remains `tru-mainnet`.
* Mainnet default RPC port remains `21832`.
* Mainnet default P2P port remains `21833`.
* Fresh Patch 15A genesis constants remain unchanged.
* Project-local BIP44 coin type remains `0x545255` / `5526101`.
* Reserved Bech32 HRPs remain `tru` and `ttru`.
* No unintended stale BTC/pre-Patch-15 network identity constants remain active in canonical TRU mainnet paths.
* NOVO/bridge legacy namespace remains intentionally separate from native TRU address/network identity.
* Final canonical source hashes are recorded.


`PATCH15_STATUS=CLOSED`

At that point TRU independently defines:

* Address identity.
* Genesis identity.
* HD wallet identity.
* P2P wire identity.
* P2P handshake/network identity.
* Mainnet RPC/P2P default port identity.
* Explicit separation from BTC-style addresses.
* Explicit separation from pre-Patch-15 TRU peers.
* Reserved testnet/regtest identities for future activation.


 **15B is absolutely closed.** 

TRU mainnet P2PKH       = 0x41
TRU testnet P2PKH       = 0x7F (reserved)

TRU mainnet Base58      = T...
TRU mainnet Bech32 HRP  = tru  (reserved)
TRU testnet Bech32 HRP  = ttru (reserved)

TRU BIP44 coin type     = 0x00545255 / 5526101
BIP32 HMAC label        = "Bitcoin seed"

TRU mainnet P2P magic   = ec7c8fb9
TRU mainnet network ID  = tru-mainnet

TRU mainnet RPC         = 21832
TRU mainnet P2P         = 21833

Genesis address =
TCtVWvC1JtsKXWoEDueN2aNEkpVbjAftQM

Genesis block =
b62fba2600030d97a06916b17694bec8d97ca14c1db682b2e57b424bd6000000

NOVO legacy 0x00        = BRIDGE NAMESPACE ONLY

AUTOMATIC REORG         = OFF

---

## 16. Patch 16   Token Consensus / State Integrity Hardening   DONE

**Priority:** production-readiness hardening.  
**Immediate prerequisite borrowed from this patch:** Token-U4 disconnect/reapply proof before live 08B.4D.9B automatic reorganization.

### 16A.0   Confirmed-State-Only Token Indexing   CURRENT BLOCKER

- Remove direct confirmed-state LevelDB mutation from the active `issuetoken` RPC path.
- `Blockchain::findTransaction()` currently searches both active chain and mempool; a mempool-only token transaction must never authorize confirmed-state index writes.
- `handleIssueToken()` must not directly persist `tokenMetadata:<txid>` before confirmation.
- `ensureTokenMetadataIndexed()` must not persist confirmed token indexes from an unconfirmed RPC transaction.
- Remove/re-plumb direct RPC writes to:
  - `tokenMetadata:`
  - `tokenUTXO:`
  - `tokenOwnerUTXO:`
  - `tokenIssuance:`
- Confirmed token state must be produced only by authoritative block application and the same `mainBatch` captured by U4.
- RPC may return constructed/admitted transaction metadata without declaring it confirmed.
- Restore the Patch-10 invariant: **no unconfirmed token metadata/index state persisted to confirmed LevelDB.**
- Add runtime proof:
  - issue token into mempool on a disposable fixture;
  - active height remains unchanged;
  - confirmed token LevelDB keyspace remains byte-identical;
  - only after mining/confirmation may token confirmed-state keys appear.

**16A.0 must close before the token U4 disconnect/reapply proof and before live 08B.4D.9B.**

### 16A   Token U4 / Reorganization Integrity

- Prove all consensus-relevant token mutations produced by normal `applyBlock()` flow through the caller-owned `mainBatch`.
- Prove U4 PRE/POST witnesses include every token key actually mutated by confirmed token issuance/transfer.
- Explicitly test:
  - `tokenOwnership:`
  - `tokenUTXO:`
  - `tokenOwnerUTXO:`
  - token issuance/index keys;
  - token metadata/index keys that are confirmed-state mutations.
- Real token issuance → mine/confirm → authenticated disconnect → exact PRE → SAFE-REAPPLY → exact POST.
- Real token transfer → mine/confirm → authenticated disconnect → exact PRE → SAFE-REAPPLY → exact POST.
- Require byte-identical logical LevelDB state after reapply.
- Audit/remove any out-of-band RPC or helper path that mutates confirmed token indexes outside authoritative block application.
- Any required confirmed token write outside `mainBatch` must be re-plumbed or disabled before automatic reorganization is enabled.

### 16B   Metadata Authentication / Unsafe Mutation Paths

- Disable `updateOffChainMetadata()` until a real authenticated signature design exists.
- Remove unconditional-success `verifyMetadataSignature()` behavior.
- If mutable metadata is retained:
  - canonical metadata serialization;
  - domain-separated signature digest;
  - signer authorization tied to the controlling token owner/key;
  - anti-replay/version semantics;
  - exact verification before any mutation.
- No direct confirmed-state metadata rewrite outside authoritative batched/journaled state transitions.
- Remove or fail-close demonstration/stub authentication code in production paths.

### 16C   Token Identifier Strength / Versioned Format

- Current 4-byte / 32-bit token identifier commitment is not sufficient as a security identifier.
- Design a versioned stronger token identifier format.
- If the existing 80-byte token payload ceiling must remain:
  - evaluate a 128-bit token ID (`16` bytes), which keeps the current binary payload within the existing ceiling.
- If a larger/versioned payload is adopted:
  - permit the full 256-bit token ID where practical.
- Database/index keys must use the canonical versioned token identifier consistently.
- Prevent deliberate prefix-grinding/collision between distinct tokens.
- Define migration/compatibility rules for already-issued legacy 32-bit token IDs.

### 16D   Token Amount / Parse Safety

* Removed the active `issuetoken` RPC path's pre-confirmation confirmed-state indexing behavior.
* Restored the invariant that unconfirmed token transactions do not directly create confirmed:

  * `tokenMetadata:`
  * `tokenUTXO:`
  * `tokenOwnerUTXO:`
  * `tokenIssuance:`
    state.
* Confirmed-state token indexing remains owned by authoritative block application / `mainBatch` / U4.

Token issuance U4 runtime proof:

* Real token issuance was confirmed through normal block application.
* Confirmed issuance token indexes were present in the active-tip `P08B1-U4` journal.
* Authenticated single-tip disconnect restored exact PRE-state.
* SAFE-REAPPLY restored exact POST-state.
* Final logical LevelDB state returned byte-identically to its original confirmed POST state.

Patch16D transfer-accounting corrections:

* `sendtoken` RPC now passes the controlling P2PKH vout to the wallet transfer path rather than the token metadata vout.
* `Wallet::findOneCoinUtxo()` now accepts canonical P2PKH fee UTXOs rather than incorrectly filtering them through the smart-contract-script allowlist.
* Existing protections remain in place for:

  * exact address ownership;
  * coinbase maturity;
  * mempool-spent exclusion;
  * nonzero/valid amount;
  * token-controlling UTXO exclusion.
* Confirmed token ownership accounting now collects both:

  * output credits;
  * spent-token debits.
* `tokenOwnership` PRE-state is reconciled against live indexed `tokenOwnerUTXO` / `tokenUTXO` state before net arithmetic.
* Historical or malformed ownership-summary disagreement fails closed rather than being silently carried forward.
* Token balance arithmetic uses checked integer accumulation and checked net credit/debit handling.
* Ownership reconciliation occurs before U4 capture so the complete confirmed mutation participates in the authenticated block transition.

Real partial-transfer runtime proof:

* Starting token amount: `1000`.
* Transfer amount: `400`.
* Recipient confirmed ownership: `400`.
* Sender confirmed change ownership: `600`.
* Final indexed ownership total: `1000`.
* Ownership conservation: **PASS**.
* Old controlling token indexes were removed.
* New recipient and sender-change `tokenUTXO` entries were created.
* New `tokenOwnerUTXO` entries were created for both resulting controlling outputs.
* Confirmed transfer was removed from the mempool after confirmation.
* Exactly one H407 block was accepted for the transfer fixture.

Authenticated H407 transfer fixture:

* Height: `407`
* Active tip:
  `3cf394d7bd5d6ab50eec1f99b76b3d24e9bc06b4fe9b4e80ddcdd2665b000000`
* Transfer txid:
  `7e46d364dffae1d44ba9ca1604f63516e1bccdd8cda39c5d8bcaf2dc81df8968`
* Logical LevelDB keys: `2561`
* Logical LevelDB SHA-256:
  `37b7bae6172295b7822a4ecd63f6b74c4bfd50595f095aa8c3d82150e295c407`

H407 U4 forensic proof:

* Active-tip identity: **PASS**
* Old controlling token indexes removed: **PASS**
* Recipient `400` token UTXO: **PASS**
* Sender-change `600` token UTXO: **PASS**
* Sender `tokenOwnership`: `600`
* Recipient `tokenOwnership`: `400`
* Total indexed ownership: `1000`
* Ownership conservation: **PASS**
* Token issuance mapping unchanged: **PASS**
* H407 U4 entry count: `25`
* Transfer U4 token mutation coverage: **PASS**
* U4 PRE-image semantics: **PASS**
* U4 POST operations: **PASS**
* Transfer `tokenMetadata:<transfer-txid>` remains absent under the current confirmed-state model.
* Existing issuance metadata remains available through the issuance-metadata fallback.
* No nonexistent transfer metadata mutation is incorrectly journaled.

Token-U4 transfer disconnect/reapply proof:

* Fresh disposable exact copy of authenticated H407 used.
* DEV-only local roundtrip hooks enabled explicitly.
* P2P disabled.
* No miner active.
* Automatic/network reorganization disabled.
* Exact H407 POST-state authenticated before disconnect.
* Authenticated single-tip disconnect: **PASS**
* Exact U4 PRE-state internal probe: **PASS**
* SAFE-REAPPLY: **PASS**
* Exact U4 POST-state internal probe: **PASS**
* Patch16D transfer accounting executed again during reapply: **PASS**
* Confirmed transfer transaction was not resurrected into the mempool.
* PRE logical keys: `2561`
* POST logical keys: `2561`
* PRE logical SHA:
  `37b7bae6172295b7822a4ecd63f6b74c4bfd50595f095aa8c3d82150e295c407`
* POST logical SHA:
  `37b7bae6172295b7822a4ecd63f6b74c4bfd50595f095aa8c3d82150e295c407`
* Exact post-roundtrip logical DB restoration: **PASS**
* Preserved H407 fixture unchanged.
* Preserved H406 issuance fixture unchanged.

Canonical Patch16D source:

* `src/blockchain.h`
  `febbca9b8b4f784099a00e72bac6ef5f9ef6e7de4bd495d7b6f7ee8f0d4dbcc8`
* `src/blockchain.cpp`
  `46895cf543766318a233568d579265aba59d6af5e0517f064d6c7a1804d02ce2`
* `src/main.cpp`
  `494c1e48aad2e4961f790fce92593585535bc977cce3c4e9841799ffb1849aba`
* `src/rpc_server.cpp`
  `d2c0041bc9ccfd6cf3240e4f59bd65a1388d364632a5e0f1f64262f5cc7d3dd4`
* `src/utxo.cpp`
  `7f83b35c15062fb14cce4cec86776d73b1a9def3d98ba144da522e1b1581d509`
* `src/tokens.cpp`
  `176b997ddb0a2b5534059e5ecf106b00cfcfe087b7b8dfbc3ca8a639146ef886`
* `src/wallet.cpp`
  `cbdd3b053a565adb015ed76c0e026e461581feabe80f5d05e8afd33eab89f956`
* `src/utils.cpp`
  `7dd63fe54131b447a1fba598d623fe8f41c144599e7ab9213a1b92c77792ec14`

Canonical Patch16D builds:

DEV:

* `tru_advanced`
  `1ab52c88437cdde14c2a9304742b9cbef8f06f4c30caca6a3d10cac32d00718c`
* `tru-cli`
  `bd4dd49ba9ddc81dc2284273d1c7a3b35e73e1d993b80160f8885062f0b26c1d`
* `tru_miner_cpu`
  `6c797c936dd6b39b86ae3cedb9bd9a01cde41242a55c0980de1c255bb455fa4e`

PROD:

* `tru_advanced`
  `ef2085eb96bb85740d280f8c8215b72bd134a7cf2ec39106e3a8b13147c519aa`
* `tru-cli`
  `bd4dd49ba9ddc81dc2284273d1c7a3b35e73e1d993b80160f8885062f0b26c1d`
* `tru_miner_cpu`
  `6c797c936dd6b39b86ae3cedb9bd9a01cde41242a55c0980de1c255bb455fa4e`

Canonical semantic/build containment:

* Modified-object pair count: `36`.
* Modified-object executable code sections identical between reviewed isolated and canonical builds: **YES**.
* Modified-object semantic symbol sets identical: **YES**.
* Normalized disassembly and text relocations identical: **YES**.
* Path-dependent final ELF delta accepted only after object-level semantic reproduction.
* DEV-only runtime/test surfaces present in DEV and absent from PROD: **PASS**.
* Patch16D safety witnesses present in both DEV and PROD: **PASS**.
* Patch16A.0 rollback builds remain unchanged.
* Canonical source hashlock: **PASS**.

Production reorganization state after Patch16D closeout:

* Production reorganization gate still exists.
* Gate remains OFF by default.
* Automatic incoming-side PREPARE/EXECUTE remains **NOT WIRED**.
* Legacy `handleChainReorganization()` remains fail-closed.
* No node was started during canonical apply/build.
* No database was accessed.
* No mining was performed.
* No P2P activity occurred.
* Automatic/network reorganization remains **DISABLED**.

The historical-inflation negative regression is retained for **16F Token Regression Coverage** and is not a prerequisite for resuming 08B.4D.9B.

**Disposition:**

* Patch16A.0 confirmed-state-only token indexing prerequisite: **CLOSED**
* Token-U4 issuance prerequisite: **CLOSED**
* Token-U4 transfer prerequisite: **CLOSED**
* Patch16D transfer-accounting / RPC-transfer prerequisite: **CLOSED**
* Canonical Patch16D DEV/PROD build and semantic containment: **CLOSED**
* 08B.4D.9B token blocker: **REMOVED**
* Automatic/network reorganization: **STILL DISABLED**
* Next reorganization stage: **RESUME 08B.4D.9B**

The canonical run supports the source/build hashes and the final disposition above: `CANONICAL_PATCH16D_SOURCE_APPLY=PASS`, both builds passed, semantic object/disassembly reproduction passed, and `NEXT=RESUME_08B4D9B`. 

---

## 17. Patch 17   Consensus / Regtest / Fuzz Regression Suite   PENDING


This is the broad final regression/security test suite.

Required coverage from the roadmap:

- Monetary overflow vectors.
- Txid manipulation.
- Duplicate transactions.
- Merkle mutation.
- Strict-DER signatures.
- High-S signatures.
- Coinbase-height commitment.
- Difficulty-retarget boundaries.
- Candidate ancestry.
- Median-Time-Past.
- Side-chain persistence.
- Fork/reorganization behavior.
- Reorganization crash recovery.
- U4 PRE/POST journal recovery.
- Patch 16 token issuance/transfer/reorg vectors.
- Mempool/resource limits.
- Persistence/restart.
- Synthetic fork harness.
- Sandbox validation.
- Reorganization interruption/restart.
- Parser fuzzing.
- Script/VM fuzzing.
- Token-surface fuzzing.
- Network/P2P parsing fuzzing.

### Carry-forward regression items

The roadmap also records cleanup/adversarial items that fit naturally into Patch 17 rather than reopening already-closed patches:

- Historical token-inflation negative regression retained for Patch 16F/regression coverage.
- Dead alternate transaction validators should be removed or proven authoritative-safe.
- Continue regression coverage for cache/budget/reorg accounting and restart determinism where appropriate.

### Closeout gate

Patch 17 should produce a repeatable automated suite with deterministic PASS/FAIL behavior and production-safe test-hook containment.

---

## Deferred Patch 05 Deployment   PENDING

After core consensus/security work stabilizes:

This patch remains intentionally reserved and unfinished.

Required work:

- Bind privileged node RPC to loopback/internal interfaces where practical.
- Add RPC authentication.
- Remove wildcard CORS.
- Protect wallet/signing RPC methods.
- Add RPC rate limits and backpressure.
- Account for synchronous transaction-validation resource exposure.
- Place public website/API traffic behind a server-side gateway.
- Keep RPC credentials completely out of browser JavaScript.
- Confirm the public website cannot directly reach privileged node RPC.
- Review Cloudflare/gateway trust boundaries.

---

## 18. Patch 18   Hard-Fork Commitment Cleanup   PENDING

Consensus-affecting hard-fork work:


This is consensus-affecting work and should be treated as a deliberate hard-fork/pre-genesis commitment decision.

Required work:

- Cryptographically commit non-coinbase `scriptSig` / authorization data.
- Commit witness-equivalent authorization data if retained.
- Commit block-level `tokenMetadata`.
- Commit every field capable of affecting consensus or confirmed state.
- Define/commit coinbase token metadata behavior.
- Eliminate any field capable of changing validated state without being covered by the canonical cryptographic commitment.
- Keep U4 undo-journal block fingerprints consistent with the final canonical block serialization/commitment.
- Revisit PoW / SHA256d architecture only after canonical commitment rules are finalized.

### Closeout gate

Patch 18 requires explicit serialization/txid/Merkle/block-hash compatibility vectors and negative tests proving that changing any consensus-affecting committed field changes the appropriate cryptographic commitment.

Because the project intends to start a fresh chain, this is an especially important patch to settle before that final canonical chain is launched.


---

## 19. Patch 19   Header-Only Block Index   PENDING

This is primarily scalability/resource hardening.

Required work:

- Remove full `Block` residency from every `blockIndex` entry.
- Retain only required index/header metadata:
  - block hash;
  - parent hash;
  - height;
  - bits/target;
  - timestamp;
  - cumulative chainwork;
  - required status/source metadata.
- Fetch complete blocks from LevelDB on demand.
- Preserve all active-chain, side-chain, candidate-validation, reorg, and recovery semantics.
- Reduce RAM amplification from active and side branches.
- Reduce sandbox ancestry-copy cost.
- Reduce deep-fork memory amplification.
- Validate operation with large blocks and long chains.

### Closeout gate

Patch 19 closes after restart, mining, sync, side-chain indexing, candidate validation, reorganization, crash recovery, and explorer/RPC block retrieval all operate correctly without relying on full blocks being resident in `blockIndex`.
---
## Stateful K/V V1   COMPLETE / RUNTIME PROVEN

The Stateful foundation established the reference lifecycle used by later stateful contract families:

```text
contractlineage:<current-anchor> -> <stable-root>
contractlive:<stable-root>       -> <current-live-anchor>
contractowner:<stable-root>      -> <owner-hash160>
contractstate:<stable-root>:<key> -> <value>
```

Proven properties include:

- stable root identity;
- explicit owner binding;
- durable state;
- owner-authorized continuation;
- committed calldata;
- live-anchor rollover;
- restart persistence;
- authoritative block-batch persistence.

The deferred Stateful U4 witness remains suitable for permanent regression coverage in `INT-28` / `SEC-17`.

## SC-17   Voting   COMPLETE

Voting is no longer a future roadmap placeholder. The completed contract work supersedes the older pre-activation roadmap entry.

Status:

```text
SC-17  ████████████████████ COMPLETE
```

## SC-18   Oracle   COMPLETE

Oracle lock/finalization work is complete for the current contract scope and remains accepted after SC-22.

Status:

```text
SC-18  ████████████████████ COMPLETE
```

## SC-19   Hash Lock   COMPLETE

Hash Lock end-to-end contract behavior is complete for the current scope and remained mempool accepted after SC-22.

Status:

```text
SC-19  ████████████████████ COMPLETE
```

## SC-20   Contract Execution   COMPLETE

The contract execution layer is complete for the currently activated contract families.

Status:

```text
SC-20  ████████████████████ COMPLETE
```

## SC-21A   Explorer Classification   COMPLETE

- Contract explorer classification moved to structural/canonical interpretation.
- Contract families are displayed without relying on broad heuristic guessing.

Status:

```text
SC-21A ████████████████████ COMPLETE
```

## SC-21B   RPC / Base58 Parity   COMPLETE

- Contract identifiers are no longer treated as though every identifier must be a Base58 address.
- RPC / contract display parity was brought in line with structural contract identity.

Status:

```text
SC-21B ████████████████████ COMPLETE
```

## SC-22   Script / Relay Policy Cleanup   COMPLETE / RUNTIME PROVEN

SC-22 replaced broad legacy byte/regex policy with compiled structural recognition.

Completed behavior:

- relay policy is compiled + structural rather than dependent on CWD-loaded JSON regexes;
- `allowed_scripts.json` is now a human-readable policy manifest rather than the relay decision engine;
- exact canonical recognition retained for Time Lock, Hash Lock, Oracle Lock, `f751` state anchors and MagicLock;
- Custom Script remains available through deterministic structural policy;
- pushed bytes are skipped before opcode classification;
- `OP_EXTERNALDATA` is non-standard;
- noncanonical `OP_DATAFEED` is non-standard;
- `OP_DELEGATECHECK` remains non-standard until activation;
- arbitrary state-domain scripts are non-standard;
- legacy NOVO/BSTY bridge opcodes are relay-disabled pending bridge redesign;
- reserved upgrade NOPs remain consensus-valid but non-standard;
- broad legacy token opcode regexes removed;
- Token Issuer V1 uses the canonical `f751` family;
- MagicLock fast-path uses exact parsing rather than substring guesses;
- `shouldRelayTransaction()` no longer double-hex-encodes `TxOut::scriptPubKey`;
- blockchain Custom fallback uses the same mempool policy;
- legacy SmartContract Custom Script validates compiled bytecode rather than textual opcode names against hex regexes.

### SC-22 runtime proof   PASS

```text
Existing Contract Vault                     PASS
Normal Hash Lock                            MEMPOOL ACCEPTED
Normal Time Lock                            MEMPOOL ACCEPTED
Patch-18 Oracle Lock                        MEMPOOL ACCEPTED
Stateful / Voting / Token Issuer f751       ACCEPTED
Supported deterministic Custom Script       ACCEPTED
Legacy NOVO/BSTY bridge-opcode creation      RELAY BLOCKED AS INTENDED
Legacy [loadAllowedPatterns] startup loads   ABSENT
tru_advanced rebuild                         100% PASS
```

### SC-22 canonical post-patch hashes

```text
src/allowed_scripts.json  1193d2e8eef74ba8d00df477079377d9baa65b378f3a74b9a5ff71d0d9071983
src/mempool.cpp           2c07dec888ddb21a767913b2a16d7a7e5adef60c75c53438d492065434f9cd22
src/mempool.h             1c25d754532f62226b618dbadb54e463e40a1052ac20c8e644b38504bd011b37
src/blockchain.cpp        7701bcd1667c49911305c1f2e0ad9d22b974679c445513e492f4aedab24b76e2
src/contract_call_policy.h 0eec1f321e4b0b3e5a61f5b5eddc86b1abea9bfad8cf4217b89ae8592ac6aaca
src/smart_contract.cpp    900c65223da390249be6cef0db41ab2e261eeaf852fcc6a8228a764c8f8a2cde
src/smart_contract.h      f3f7c8c20709c6c3707c7425f9aff6069f1fce25f8145d0fd93559863ed78020
```

**SC-22 did not require a consensus change, VM change, transaction-format change, database change, or chain reset.**

---

# 6. UI / Contract Vault Status

## UI-30A   COMPLETE

Contract-vault/UI work completed.

## UI-30B   COMPLETE

Contract-vault/UI work completed.

## UI-30C   COMPLETE WITH TINY COSMETIC FOLLOW-UP

Core UI-30C behavior is complete.

Remaining cosmetic item:

> Suppress or redirect the `[SYNC] Waiting for peers...` heartbeat while the fullscreen Contract Vault/pager owns the screen so the sync lane cannot draw through contract cards.

This is **not a blocker** for MS-01 contract development.
# 8. MS-01   Multisig / Escrow V1   NEXT

```text
MS-01  ░░░░░░░░░░░░░░░░░░░░ NEXT
```

## Goal

Create the first canonical TRU multisignature escrow contract/application using the already-hardened signature and CHECKMULTISIG execution foundation.

## V1 scope decision

Start with a deliberately narrow **2-of-3 escrow**:

```text
Party A / Buyer
Party B / Seller
Party C / Arbiter

Any 2 of 3 authorized keys may release the escrowed TRU.
```

This provides a real escrow use case while keeping the first standard script family deterministic and easy to test.

General `M-of-N` support can be added after the canonical 2-of-3 form is runtime proven.

## Architectural target

Prefer **no new consensus opcode** and **no new transaction format** if the existing CHECKMULTISIG semantics are sufficient.

MS-01 should primarily add:

- canonical script construction;
- exact structural relay recognition;
- wallet/CLI creation flow;
- signer discovery;
- partial-signature handling if needed;
- final transaction assembly;
- spend path;
- Contract Vault classification/display;
- RPC parity;
- negative tests.

## Canonicality rule

MS-01 must follow the SC-22 policy model:

> Parse the actual script structure. Never identify multisig by substring, wildcard regex, or opcode bytes found inside pushed data.

## Proposed MS-01 patch sequence

### MS-01A   Read-Only Multisig Architecture Audit - DONE

Confirm before mutation:

- exact CHECKMULTISIG VM semantics;
- existing compiler opcode names and encodings;
- signature ordering requirements;
- current dummy-stack behavior if applicable;
- current wallet signing assumptions;
- scriptSig construction path;
- P2PKH-only assumptions that need a multisig-safe branch;
- mempool standardness behavior;
- explorer/RPC script decoding behavior;
- fee estimation with multiple signatures;
- UTXO ownership/accounting treatment for multisig outputs.

**Gate:** no code mutation until the current execution/spend path is mapped.

### MS-01B   Canonical 2-of-3 Script + Structural Policy- DONE

Implement one exact standard family for 2-of-3 escrow.

Requirements:

- exactly three canonical public keys;
- deterministic public-key encoding requirements;
- threshold exactly `2`;
- participant count exactly `3` for V1;
- exact CHECKMULTISIG termination;
- no trailing arbitrary opcodes;
- pushed data skipped correctly by classifier;
- malformed M/N relationships rejected;
- duplicate-key policy explicitly defined;
- oversized/noncanonical pubkeys rejected.

### MS-01C   Wallet / CLI Escrow Creation - DONE

Create a wallet flow that accepts three participant identities/keys and an escrow amount.

Expected result:

```text
MULTISIG / ESCROW V1 CREATED
Threshold: 2 of 3
Amount: <TRU>
Funding TXID: <txid>
Vout: <n>
Status: MEMPOOL ACCEPTED
```

The wallet must not falsely claim confirmation before mining.

### MS-01D   Signing / Redemption - DONE

Support the spend lifecycle.

At minimum prove these independent combinations on disposable fixtures:

```text
A + B  -> PASS
A + C  -> PASS
B + C  -> PASS
A only -> FAIL
B only -> FAIL
C only -> FAIL
wrong key + valid key -> FAIL
malformed signature set -> FAIL
```

### MS-01E   Escrow UI / Explorer / RPC Parity

Contract Vault / explorer should show:

```text
Family: Multisig / Escrow V1
Threshold: 2 of 3
Participants: <canonical identities/pubkeys>
Funding outpoint: <txid>:<vout>
Amount: <TRU>
State: UNSPENT / SPENT
```

Do not mislabel the contract outpoint as a Base58 address.

### MS-01F   Runtime Closeout

Required closeout proof:

base) gw878@gw878:~/NEW_TRU$ bash ./TRU_MS_01F_MULTISIG_ESCROW_RUNTIME_CLOSEOUT_READ_ONLY.sh
============================================================
TRU MS-01F   MULTISIG / ESCROW V1 RUNTIME CLOSEOUT
READ-ONLY EVIDENCE HARNESS   NO SOURCE OR CHAIN MUTATION
============================================================

This harness does NOT manufacture PASS results.
It validates whatever runtime evidence is actually present.

===== MS-01E SOURCE BASELINE =====
PASS  src/contract_call_policy.h  b669f124484a50137be3c8b716f44906146f0195b07c274d6a2ccee15505c1d3
PASS  src/blockchain.cpp  609635482e6c9f854321346f19a1ac96f5923d5e71f6567aecba58bf01517d4e
PASS  src/wallet.h  f799a32ddff76be3cf113692d7a972aab88cc4e6b3c70041acc388cd8bd5a09b
PASS  src/wallet.cpp  422a74a72f34b750a360092dab74ffbab82b1cf0380b1409b510182e8f4e5b5d
PASS  src/main.cpp  f06cf5245b6728bb1d734d131f39877c1d1d2dec16b8ce9dcc9a81bc7d81db83
PASS  src/mempool.cpp  f656a18ff81c0c6585bac564d2402430a878e9fce22b13dcca2d3f1de398ad28
PASS  src/rpc_server.cpp  58b51fe7444549390c92d2ca6090c06119a652354c90b7dca58d4fbfddb34585
PASS  src/script_interpreter.cpp  d4a7ce919423f400c6dd4fe08c24206abe30a289c0364dcf0639ad72d1c438fd
PASS  src/tx.cpp  e9a1a769be5b536e41bdfbb2705cb6accb438f0eefbdbc95c3ea62a66424dd75
PASS  src/crypto_ecdsa.cpp  d64d8d45d25475d524cbc67bd3f76b4cc0c98b8a0613d21b35c5c15b9ad65547

PASS  tru_advanced exists

PASS  runtime log: build-native/bin/Tru_debug.log

===== CREATE / FUNDING =====
PASS  creation mempool accepted
PASS  funding outpoint applied in block
PENDING  confirmed escrow amount line not isolated by simple scan

===== SIGNING =====
PASS  deterministic redemption sighash observed
PASS  locally produced participant signature verified

===== REDEMPTION / 2-OF-3 UNLOCK =====
PENDING  no final OP_0 <sig> <sig> redemption unlock observed
PENDING  redemption txid not discovered
PENDING  redemption mempool acceptance not proven
PENDING  redemption block confirmation not proven
PENDING  recipient payout not proven
PENDING  post-redemption spent-state not proven

===== NEGATIVE / FAIL-CLOSED EVIDENCE =====
PASS  bad redemption signature rejected
PASS  wallet refused signing for unowned participant key

===== MS-01E CLASSIFICATION SOURCE =====
PASS  Contract Vault + authoritative family classification installed
PASS  exact structural parser used by authoritative classifier

Evidence file: /tmp/TRU_MS01F_CLOSEOUT_EVIDENCE.txt

============================================================
MS01F_RUNTIME_CLOSEOUT=PENDING
failures=0 pending=7

Development may continue, but final runtime closeout is not yet proven.
============================================================
(base) gw878@gw878:~/NEW_TRU$ 

## MS-01 closeout definition

Only mark MS-01 COMPLETE when the actual funded and redeemed 2-of-3 lifecycle has been runtime proven.

---

# 9. HTLC-01   Atomic Swap

```text
HTLC-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Compose Hash Lock + Time Lock semantics into a canonical Hashed Timelock Contract suitable for atomic-swap workflows.

Required design:

- claim branch with correct secret/preimage;
- refund branch after timeout;
- exact hash primitive parity across wallet/VM/RPC;
- exact lock-time semantics;
- deterministic branch structure;
- structural relay recognition;
- pre-maturity refund rejection;
- correct-secret claim success;
- wrong-secret rejection;
- refund success after maturity;
- restart / explorer / RPC parity.

This application should build on SC-19 and the established Time Lock path rather than introducing a new arbitrary VM mechanism.

---

# 10. NFT-01   Royalty NFT

```text
NFT-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Create a deterministic ownership-transfer asset with explicit royalty rules.

Required architecture review before activation:

- canonical asset identifier;
- owner / controlling outpoint;
- transfer authorization;
- royalty recipient;
- royalty amount or rate representation;
- prevention of bypass transfer paths;
- metadata commitment rules;
- supply rule, preferably one-of-one for V1;
- replay / duplicate creation protection;
- wallet/explorer ownership display.

Any consensus-affecting ownership covenant must be explicit and narrowly versioned.

---

# 11. GOV-01   Treasury Governance

```text
GOV-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Combine Voting with controlled treasury release.

Possible V1 model:

- funded treasury UTXO/root;
- proposal identifier;
- deterministic voting window;
- eligible voter policy;
- threshold/quorum rule;
- approved recipient and amount committed in proposal;
- release only after valid proposal outcome;
- no arbitrary post-vote destination substitution.

This should reuse SC-17 Voting state rules instead of creating a parallel voting engine.

---

# 12. VEST-01   Vesting Plan

```text
VEST-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Create deterministic scheduled release of TRU using established Time Lock semantics.

Potential V1:

- one beneficiary;
- fixed schedule;
- fixed tranches;
- no administrator override;
- each tranche represented by canonical time-locked outputs.

Prefer composition of existing primitives over new consensus rules.

---

# 13. POE-01   Proof of Existence

```text
POE-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Commit a document/data digest to TRU without placing the full private document on chain.

V1 should include:

- explicit digest algorithm/version;
- content hash;
- optional media/type label kept non-authoritative unless committed canonically;
- timestamp inherited from confirmed block context;
- explorer/RPC verification path;
- duplicate proof behavior defined.

---

# 14. INH-01   Inheritance Vault

```text
INH-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Create a recoverable inheritance-style vault using a narrow combination of multisig and time-based conditions.

Potential V1 design should be selected only after MS-01 and HTLC-01 are proven.

Possible model:

- owner/control key;
- beneficiary key;
- recovery/arbiter key;
- timeout or inactivity condition;
- no ambiguous dual-spend authority;
- clearly defined emergency/recovery branch.

---

# 15. TOK-01   Batch Distribution

```text
TOK-01  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

## Goal

Provide safe deterministic batch distribution for TRU tokens using the already-hardened confirmed token state model.

Required:

- bounded recipient count;
- checked amount sums;
- exact conservation;
- deterministic change;
- fee handling;
- duplicate-recipient behavior defined;
- mempool admission parity;
- U4/restart regression coverage;
- no direct confirmed-state RPC writes.

---

# 16. BRIDGE GENERATION PHASE

Legacy NOVO/BSTY script relay is intentionally disabled by SC-22 until this redesign phase.

That is the desired state.

## BR-23   Bitcoin / Generic External-Chain Verification Framework

```text
BR-23  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

Goals:

- external chain identity/version;
- Bitcoin header/proof verification;
- deposit commitment;
- deterministic TRU representation;
- replay prevention;
- bridge state namespace;
- confirmation policy;
- regtest/testnet testing without needing real BTC;
- reusable verifier interface for later bridges.

The generic framework should be built first so later chains are adapters, not separate consensus inventions.

## BR-24   GlobalBoost-Y Bridge Migration

```text
BR-24  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

- migrate legacy BSTY behavior into BR-23 framework;
- eliminate broad legacy script recognition;
- deterministic proof handling;
- deposit/redeem lifecycle;
- replay prevention;
- canonical bridge state namespace.

## BR-25   Solana Adapter

```text
BR-25  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

- Solana-specific proof/finality adapter;
- generic BR-23 interface integration;
- deterministic external-event representation;
- replay protection;
- asset mapping;
- selected mint/burn or escrow semantics.

## BR-26   User-Defined / Custom Bridge Interface

```text
BR-26  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

Required configuration domain:

- chain identifier;
- verifier version;
- proof format;
- finality policy;
- asset definition;
- replay domain;
- deterministic state keying;
- governance/activation controls.

A custom bridge must not mean arbitrary executable consensus code supplied by users.

---

# 17. FINAL CONTRACT INTEGRATION

## INT-27   Wallet / Compiler / Forms Parity

```text
INT-27  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

Every exposed contract/application must compile and spend the same canonical structure accepted by:

- wallet forms;
- CLI;
- RPC;
- compiler;
- VM;
- mempool;
- consensus;
- explorer.

No hidden legacy construction path should remain.

## INT-28   Full Contract Regression / Consensus Suite

```text
INT-28  ░░░░░░░░░░░░░░░░░░░░ PENDING
```

Permanent regression coverage should include:

### Stateful K/V

- create;
- owner bind;
- valid call;
- unauthorized call;
- stale anchor;
- malformed/changed calldata;
- restart;
- U4 PRE/disconnect/reapply/POST.

### Voting

- initialization;
- valid vote;
- duplicate/unauthorized vote;
- rollover;
- restart;
- undo/reapply.

### Token Issuer / Tokens

- initialization;
- checked arithmetic;
- authorization;
- balances;
- transfers;
- restart;
- U4/reorg behavior.

### Oracle

- deterministic input;
- boundary comparisons;
- malformed feed rejection;
- VM/RPC path parity.

### Hash Lock / Time Lock / HTLC

- valid and invalid preimages;
- pre-maturity rejection;
- maturity boundary;
- post-maturity redemption;
- HTLC claim/refund exclusivity.

### Multisig

- every valid 2-of-3 pair;
- every 1-of-3 failure;
- wrong-key failure;
- malformed signature stack;
- classifier push-awareness;
- wallet/RPC/explorer parity.

### Bridges

- valid proof;
- invalid proof;
- insufficient confirmations/finality;
- replay;
- duplicate deposit;
- redemption;
- restart;
- undo/reapply.

### Cross-cutting

- compiler/VM parity;
- RPC/VM parity;
- mempool/consensus parity;
- standardness/consensus parity;
- fee boundaries;
- resource limits;
- malformed serialization;
- duplicate inputs;
- persistence/restart;
- database recovery;
- U4 journal integrity.

---
 Namespace | Meaning | Example |
|---|---|---|
| `SEC-xx` | Core security / consensus hardening | `SEC-14 Wallet Encryption` |
| `REORG-xx` | Fork / reorganization safety work | `REORG-08B` |
| `NET-xx` | Network / RPC / deployment perimeter | `NET-05 RPC / Website Gateway` |
| `IDX-xx` | Index / scalability architecture | `IDX-19 Header-Only Block Index` |
| `UI-xx` | Wallet / terminal / contract-vault UI work | `UI-30C` |
| `SC-xx` | Smart-contract activation/hardening | `SC-22 Script / Relay Policy Cleanup` |
| `MS-xx` | Multisig / escrow application | `MS-01` |
| `HTLC-xx` | Atomic-swap application | `HTLC-01` |
| `NFT-xx` | NFT / royalty application | `NFT-01` |
| `GOV-xx` | Treasury governance application | `GOV-01` |
| `VEST-xx` | Vesting application | `VEST-01` |
| `POE-xx` | Proof-of-existence application | `POE-01` |
| `INH-xx` | Inheritance-vault application | `INH-01` |
| `TOK-xx` | Token application tooling | `TOK-01` |
| `BR-xx` | External-chain bridge framework | `BR-23` through `BR-26` |
| `INT-xx` | Final wallet / contract integration | `INT-27`, `INT-28` |

# Current Immediate Sequence
# TRU Token Evolution / Verifiable Attribute History Patch Plan
**Updated:** 2026-09-01  
**Status:** TOKEN-AI-03A applying; subsequent work sequenced below.

---

## 1. Product Direction

The current V1 feature is **AI-assisted token evolution for SFT and NCFT assets** with append-only metadata epochs, parent-hash lineage, atomic durable persistence, deterministic provenance anchoring, crash-safe prepared/submitted transaction recovery, exact rebroadcast of signed anchor transactions, active-chain verification, and wallet-native preview/commit flow.

The broader product direction is **TRU Verifiable Attribute History (VAH)**. AI is one future writer class among AI, human-signed, sensor/device, and authenticated software/system writers. V1 will close with AI as the only active writer class. Human/sensor/device writers are deferred to V2 so V1 can close as a proven feature instead of expanding into another 02-series-scale subsystem.

Canonical public framing:

> A TRU SFT or NCFT can maintain an append-only AI-assisted metadata history whose lineage is cryptographically chained, whose individual evolution records are durably persisted, whose commitments are anchored into TRU transactions, and whose provenance can be independently checked against the active blockchain   without granting the AI authority over ownership, money, token supply, authorization, or consensus.

Canonical disclaimer:

> Evolved metadata is an off-chain record. Its provenance chain is anchored on-chain and verifiable against the anchor. The chain proves what was recorded and when; it does not establish that an AI-generated claim is true.

---

## 2. Completed Foundation

### TOKEN-AI-01A   Integer Token Supply Scaling
**Status: CLOSED**

- Removed floating-point issuance supply math.
- Exact integer scaling.
- Decimal bounds enforced.

### TOKEN-AI-01B / 01B2R   64-Bit Token IDs
**Status: CLOSED**

- Canonical 64-bit token IDs.
- Canonical 16-hex representation.
- Legacy compatibility retained.
- Native/RPC normalization aligned.

### TOKEN-AI-01C   Native/Web Metadata Hash Parity
**Status: CLOSED**

- Canonical metadata hashing rules defined.
- Native and web parity established.
- Structured-value ambiguity removed.
- This patch is the model for all future canonical request/record hashing rules.

### TOKEN-AI-02A   Atomic Evolution Persistence
**Status: CLOSED**

- Sequential epoch enforcement.
- Parent-hash lineage enforcement.
- Duplicate epoch prevention.
- Canonical metadata hash validation.
- Bounded anchor queue.
- Atomic epoch + latest + queue persistence.
- Synced batch write.
- Read-after-write verification.
- Whole operation fails closed on persistence failure.

### TOKEN-AI-02B   Crash-Safe Anchor Preparation
**Status: CLOSED / HARDENED BY 02B2 + 02B3**

- Deterministic prepare/sign before submit.
- Durable prepared state.
- Exact transaction bytes retained.
- Receipt support.
- Queue drain tied to successful anchor submission path.

### TOKEN-AI-02B2   Durable Submitted Anchor Watch
**Status: CLOSED**

- Submitted-but-unconfirmed anchors survive restart.
- Durable watch namespace.
- Pre-02B2 receipts can reconstruct watch state.
- Confirmed anchors retire from watch.
- Mempool-loss rebroadcast supported.

### TOKEN-AI-02B3   Prepared TXID Materialization
**Status: CLOSED**

- Persisted prepared transaction is deserialized.
- `computeTxId()` runs on the actual reconstructed transaction object.
- Materialized txid must equal durable expected txid.
- Same signed transaction is rebroadcast; no re-signing.

### TOKEN-AI-02C   Full Epoch Chain / Receipt Verifier
**Status: CLOSED**

- Root verification.
- Contiguous epoch verification.
- Parent-hash lineage verification.
- Receipt/prepared/queue state validation.
- Full history verification API/CLI.

### TOKEN-AI-02D / 02D1 / 02D2 / 02D3   Live Provenance Verification
**Status: CLOSED**

- Active-chain anchor transaction lookup.
- Exact payload parsing.
- Durable record ↔ anchor payload comparison.
- MEMPOOL / CONFIRMED / MISSING states.
- Strict `require_confirmed` runtime verification.
- Canonical root/epoch repair.
- Oracle signer wallet lifetime repair.
- Fresh runtime fixture proven through confirmed anchor verification.

### CR-01A   Core Direct-Tip Crash-Atomic Publication
**Status: CLOSED**

Discovered during Token Evolution runtime testing.

- Direct-connect pending publication evidence.
- Exact durable block bytes.
- Startup publish-only recovery path.
- Atomic publication metadata handling.
- Normal live direct-connect path proven after patch.

### CR-01B   Height/State Recovery Repair
**Status: CLOSED**

- Rolled back contaminated replacement blocks.
- Restored clean pre-images.
- Reapplied canonical blocks.
- Rebuilt fresh U4 state.
- Removed orphan contamination.
- Final chain/state parity and containment proven.

---

## 3. Current Patch

### TOKEN-AI-03A   Wallet AI Evolution Preview / Exact Commit
**Status: CLOSED**

Purpose:

- Expose AI Evolution in the native wallet.
- List wallet-owned confirmed/live SFT and NCFT tokens.
- Select provider.
- Enter trigger.
- Preview evolution without persistence.
- Show exact epoch transition, hashes, permitted field changes, and resulting metadata.
- Commit the exact in-memory preview.
- No second AI call on commit.
- Require explicit `COMMIT`.
- Preserve current backend invariants.

Important runtime rule:

**Do not perform a real persistent wallet COMMIT until 03A1 lands.** Preview testing is allowed.

---

# 4. V1 Remaining Patch Plan

## TOKEN-AI-03A1   Preview/Commit Boundary Hardening
**Status: CLOSED**

03A1 makes critical backend invariants explicit and testable at the user-action boundary.

### Commit ordering

After the user types `COMMIT`:

1. Freshly reload current `latest:` state.
2. Require current epoch == `preview.epoch_before`.
3. Require current canonical parent hash == `preview.previous_metadata_hash`.
4. If either fails: refuse with **"State moved. Re-run preview."**
5. Re-prove confirmed ownership.
6. If ownership fails: refuse with **"You no longer own this token."**
7. Verify previous epoch anchor is confirmed.
8. Refuse no-op evolution.
9. Call `persistPreview(exactPreview)`.
10. TOKEN-AI-02A independently re-checks all persistence invariants again.

### Structural gate requirement

The gate must assert **ordering**, not mere symbol presence. It must prove that the fresh `loadLatest`/equivalent current-state read occurs after exact COMMIT confirmation, before ownership re-proof, and before `persistPreview`.

### Exact confirmation rule

Only literal:

```text
COMMIT
```

is accepted. `commit`, ` Commit`, `COMMIT `, and ` COMMIT` must refuse. No trimming. No case folding.

### Regenerate Preview freshness

Before spending another AI/provider call:

1. Freshly read current latest state.
2. Compare epoch/hash against the preview parent.
3. If state moved, stop and require a fresh preview from current state.
4. Do not spend a provider call against a stale parent.

### No-op evolution rule

**REFUSE.**

No-op detection happens after provider output, allow-list filtering, merge into parent metadata, and canonical resulting metadata hashing.

If:

```text
new_metadata_hash == previous_metadata_hash
```

then:

> AI proposed no permitted metadata changes; nothing committed.

Consequences: no epoch consumed, no persistence, no anchor, no transaction fee, no provenance noise.

This catches both zero allowed keys and allowed keys whose values are unchanged.

### One-confirmed-anchor-behind rule

Epoch N+1 may be previewed while epoch N is pending, but it may not be committed until epoch N's provenance anchor is confirmed on-chain.

Message:

> Previous evolution epoch is not yet confirmed on-chain. Preview is allowed; commit is temporarily unavailable.

Invariant:

> Every newly committed epoch has a confirmed on-chain parent anchor.

### Ownership/history rule

Evolution history belongs to the **token**, not the wallet. Ownership controls who may commit the next epoch.

If Alice commits epoch 5 and transfers the token before epoch 5 confirms, epoch 5 remains valid history for the token; the new owner sees it and may commit epoch 6 once epoch 5 is confirmed.

---

## TOKEN-AI-03A2   Evolution Record V2 / Provenance Format
**Status: CLOSED**

This patch freezes a richer, future-compatible record format before 03B builds the permanent history UI.

V1 still permits only AI writers, but the record format reserves clean provenance semantics for future VAH writers.

### Core new record fields

At minimum:

```text
record_format_version
writer_type
provider
provider_version
model_id
request_hash
input_metadata_hash
```

For V1:

```text
writer_type = ai
```

Future V2 record formats may add signatures, writer registry references, device identity, etc.

### `record_format_version`

Required. Do not infer record semantics from which fields happen to exist. The verifier must know exactly which historical verification rules apply.

---

## 5. TRU_EVOLUTION_REQUEST_V1 Canonicalization Specification

This is the most important part of 03A2.

`request_hash` must never depend on C++ struct member order, JSON object insertion order, provider implementation quirks, delimiter parsing, omitted/null ambiguity, or web/native formatting differences.

### Field order must be explicit

The canonical spec must publish the ordered field list as a numbered format definition.

Initial proposed semantic order:

1. domain/version tag
2. token_id
3. token_type
4. epoch_before
5. provider
6. provider_version
7. model_id
8. trigger
9. input_metadata_hash

The exact final list must be frozen in 03A2 after inspecting the actual current provider prompt construction.

### Absent field encoding

Every field must have a defined representation. No ambiguity between absent, empty string, null, or omitted field.

Recommended V1 rule:

- fields are never omitted from canonical serialization,
- unavailable optional text fields encode as zero-length byte strings,
- integer fields use a specified fixed or canonical integer encoding.

### Length-prefix every variable field

Do not use delimiter-only concatenation. Every variable byte/string field must carry an explicit length.

Conceptual form:

```text
DOMAIN_LEN | DOMAIN_BYTES
TOKEN_ID_LEN | TOKEN_ID_BYTES
TOKEN_TYPE_LEN | TOKEN_TYPE_BYTES
...
TRIGGER_LEN | TRIGGER_BYTES
...
```

The actual widths and byte order must be specified and shared by native/web implementations.

### Request-hash semantic claim

03A2 must inspect the actual current prompt/request builder before freezing the format.

Two possible claims:

**Strong form:** `request_hash` commits to the exact request bytes sent to the provider.

Use this only if canonical request serialization fully determines the actual provider request, including system/template text, provider-specific formatting, model identifiers, metadata content/hash, trigger, and all request-affecting parameters.

**Structured provenance form:** `request_hash` commits to the canonical structured evolution request from which provider-specific request text is constructed.

Use this if provider-specific formatting or hidden/template content is not fully captured.

Do not claim exact-request provenance unless the bytes actually sent are deterministically committed.

### Input metadata commitment

`input_metadata_hash` commits to the canonical parent metadata used for the request.

### Private prompt handling

The prompt/request text does not need to be publicly stored. A canonical hash can prove later correspondence:

```text
SHA256(canonical_request_bytes)
```

---

## TOKEN-AI-03B   Token-Centric Evolution History UI
**Status: CLOSED**

Build once against the final 03A2 record format.

The UI should show, per token:

- current epoch,
- provider,
- provider version/model when recorded,
- writer type,
- trigger,
- previous hash,
- new hash,
- record hash,
- anchor txid,
- anchor status,
- confirmation status,
- provenance verification result.

History follows the token across ownership transfers.

---

## TOKEN-AI-03C   SECURITY: Untrusted AI Output / Stored-XSS Containment
***Status: CLOSED**

This is not cosmetic UI work. Provider/model output is untrusted input.

Required work:

- enumerate all relevant `innerHTML` and equivalent rendering sinks,
- distinguish constant-template HTML from data-bearing insertion,
- move untrusted data to `textContent`, safe DOM construction, or context-appropriate escaping,
- ensure structured metadata remains inert after serialization/storage/reparse,
- prove hostile provider output cannot become executable content.

Hostile test corpus must include event-handler attributes, attribute-context breakouts, `javascript:` URLs, SVG/event payloads, nested/encoded markup, JSON round-trip payloads, and quote/angle-bracket edge cases.

Success requirement:

> Hostile AI/provider metadata renders as inert text/data only.

---

## TOKEN-AI-03D   Anchor / Provenance Verification UI
**Status: CLOSED**

Expose the proven 02C/02D machinery cleanly in wallet/web UI.

Show issuance root status, verified epoch count, contiguous lineage status, anchor payload validity, txid match, MEMPOOL / CONFIRMED / MISSING, pending epochs, missing anchors, and strict confirmed provenance result.

User-facing verified state must distinguish provenance verification from factual truth.

---

# 6. AI Provenance V1 Final Closeout
**Status: CLOSED**

V1 closes on adversarial runtime proof, not a happy-path demo.

### Preview / user-action negatives

- cancel preview → zero persistence,
- anything except literal `COMMIT` → zero persistence,
- double commit of same preview → one success at most,
- two concurrent previews for same next epoch → one may win, the other must refuse,
- state moves under preview → refuse,
- regenerate after state move → detect before provider call,
- ownership disappears after preview → refuse,
- malformed/mutated preview hash → refuse,
- duplicate epoch → refuse,
- provider exception → zero persistence,
- valid JSON with no permitted fields → refuse,
- permitted fields with unchanged values → refuse by canonical hash equality.

### Confirmed-parent behavior

- prior anchor CONFIRMED → next commit permitted,
- prior anchor MEMPOOL/SUBMITTED → preview permitted,
- prior anchor MEMPOOL/SUBMITTED → commit refused,
- ownership transfer after commit/before anchor confirmation → history remains token-centric,
- new owner can continue only after previous anchor confirms.

### Storage atomicity regression

Inject storage failures and prove no partial epoch/latest/queue state.

Test failure before batch, during atomic batch, after batch before readback, and readback mismatch.

A DEV-only fault injection mechanism may be used, but it must be impossible to accidentally ship enabled in production.

### Crash/restart boundaries

Kill/restart at least after epoch persistence, after prepared anchor persistence, after receipt creation, after submission, while anchor is in mempool, and after block confirmation.

### 02B3 regression

Permanent regression proof:

1. deserialize exact durable prepared tx,
2. materialize txid on actual object,
3. require expected txid equality,
4. rebroadcast exact signed bytes,
5. no re-signing,
6. no blank txid path.

### Web security

Run hostile provider payload corpus through actual storage/render path and prove all payloads remain inert.

### Strict final proof

Final V1 fixture must produce issuance root verified, full epoch continuity verified, metadata lineage verified, all anchor payloads valid, all required anchor txids confirmed, no missing anchors, no pending epochs, and `runtime_ok = true` under strict confirmed mode.

Only then:

```text
AI PROVENANCE V1 = CLOSED
```

---

# 7. V2   TRU Verifiable Attribute History

V2 generalizes the V1 evolution engine to stronger writer classes.

Do not activate external writers until writer authorization and canonical multi-node reconciliation are solved.

## VAH-01   Canonical Writer Registry / Historical Authorization
**Status: CLOSED**
Purpose:

- define authorized human/device/sensor writers,
- bind writer identities to keys,
- support writer class,
- support key rotation,
- preserve historical verification.

A cryptographically valid signature alone must never justify `Signature: VERIFIED`.

A meaningful verified writer requires signature mathematically valid, key authorized for this token, key authorized at that historical epoch, writer class authorized, and writer class allowed to modify the claimed fields.

Preferred direction: writer-registry changes are themselves owner-authorized historical records, so the verifier can derive which key set was in force at any epoch and old valid signatures remain historically verifiable after rotation/revocation.

---

## VAH-02   Multi-Node Canonical Epoch Reconciliation
**Status: CLOSED**

Two nodes can independently create competing candidate epoch N records. The confirmed blockchain anchor determines the canonical winner.

Required behavior:

```text
local candidate
      ↓
compare against active-chain confirmed epoch/hash
      ↓
match → canonical
mismatch → local divergence
      ↓
quarantine losing local candidate
      ↓
reconstruct latest from confirmed canonical history
```

Do not silently delete losing records. Keep forensic/conflict state such as:

```text
evolution_conflict:<tokenID>:<epoch>:<recordHash>
```

with explicit `NON_CANONICAL` status.

---

## VAH-03   Human / Sensor / Device Writers

External writers activate only after VAH-01 and VAH-02.

The authorization model becomes:

```text
token type
+
writer class
+
field capability
```

Example:

```text
SFT + AI
    description_ai
    learning_mode
    growth_algorithm
    adaptation_rate
    ai_version

SFT + SENSOR
    operating_hours
    temperature
    pressure
    service_counter
    location_zone

SFT + HUMAN
    asset_label
    service_status
    inspection_note
    certification_reference
```

A sensor must not be able to alter AI description fields. An AI must not be able to write telemetry as though it were a device measurement.

Ownership/supply/consensus fields remain outside all evolution writer authority.

---

## VAH-04   Sensor Batching / Event Feed Architecture

Do not make one blockchain epoch per high-frequency sensor reading. The confirmed-parent rule remains intact.

### High-frequency readings

Accumulate readings off-chain in an authenticated local/device log. Periodically anchor a batch Merkle root, reading count, time range, summary attributes, and device identity/authorization reference.

### Important events

Threshold crossings and meaningful milestones may become discrete epochs: overheat event, service interval reached, fault code, inspection completed, or 2,500 operating-hour milestone.

This makes sensors an event/checkpoint writer rather than using the blockchain as a telemetry bus.

---

# 8. Token Identity Rules

For V1/V2, keep canonical issuance identity outside normal evolution authority:

```text
name
symbol
decimals
token_type
owner/supply/balance/authorization fields
```

Potentially evolvable presentation/asset fields:

```text
display_name
asset_label
```

A mutable display label is useful; a writer must not be able to redefine canonical issuance identity.

---

# 9. Final Sequence From Current State

AI Provenance V1 and Verifiable Attribute History V2 are now closed.

```text
AI PROVENANCE V1
    CLOSED
      ↓
VAH-01
    Canonical writer identity / historical authorization
    CLOSED
      ↓
VAH-02
    Multi-node canonical reconciliation
    CLOSED
      ↓
VAH-03
    Human / sensor / device writer foundation
    CLOSED
      ↓
VAH-04
    Sensor batching / confirmed event-feed materialization
    CLOSED
      ↓
VAH V2 FINAL CLOSEOUT
    CLOSED / FROZEN
      ↓
RETURN TO CORE TRU ROADMAP
```

## VAH V2   FINAL CLOSEOUT   CLOSED

**Status: CLOSED / FROZEN**

TRU Verifiable Attribute History V2 completed its final integrated closeout on September 2, 2026.

Closed generations:

- VAH-01   Canonical Writer Registry / Historical Authorization
- VAH-02   Multi-Node Canonical Epoch Reconciliation
- VAH-03   Human / Sensor / Device Writer Foundation
- VAH-04   Sensor Batching / Event Feed Architecture

Final closeout proof:

- authoritative post-VAH-04D source ledger   PASS;
- all 12 VAH-owned durable namespaces use strict double-SHA verification   PASS;
- complete VAH-01 through VAH-04 DEV regression set   PASS;
- production `tru_advanced` build   PASS;
- final authoritative source-hash recheck   PASS;
- RPC/P2P external-writer ingress   NOT ENABLED;
- RPC/P2P sensor ingress   NOT ENABLED;
- automatic sensor/external-writer wallet spending   NO;
- automatic re-anchor after canonical-chain disagreement   NO;
- automatic fork/reorg activation   NO;
- chain reset   NO.

The first attempted final closeout stopped safely at its source-freeze gate because a mutable staging copy of `README_TOKEN_EVOLVE.md` had been used to construct the expected ledger instead of the authoritative VAH-04D postimage.

No repo/product state was mutated by that failed closeout.

The corrected closeout used the authoritative VAH-04D source ledger and passed completely.

### Known VAH V2 deferrals / coverage boundaries

`TOKEN:EVOLUTION:` checksum migration remains:

**DEFERRED / DELIBERATE**

`TOKEN:EVOLUTION:` predates the VAH-owned strict namespaces. It must not be blindly converted. Any future conversion requires an explicit inventory of historical records and an offline-verified migration/compatibility policy.

Cross-generation composition coverage:

**ROTATE-AFTER-MATERIALIZE   NOT COVERED AS A DEDICATED INTEGRATED FIXTURE**

The final VAH V2 closeout executed the complete individual VAH-01 through VAH-04 DEV regression matrices sequentially. It did not introduce a dedicated composition fixture exercising authorization rotation/revocation after already-materialized external-writer or sensor state.

This is a known regression-coverage deferral and does not reopen the individually proven VAH-01 through VAH-04 foundations.

Future regression work should explicitly exercise:

1. AUTHORIZE writer;
2. materialize valid external-writer/sensor state;
3. ROTATE or REVOKE writer;
4. verify historical materialized state remains historically valid;
5. verify superseded/revoked writer cannot authorize new state at later epochs;
6. restart and repeat historical/current authorization verification.

**TRU VERIFIED AUTHORIZED HISTORY V2 = CLOSED / FROZEN**

---
HTLC-01   Canonical HTLC Primitive   COMPLETE

The canonical atomic-swap contract primitive is complete.

Frozen TRU-SWAP-V1 rules:

secret is exactly 32 random bytes;
commitment is HASH160(secret);
refund uses Unix timestamp CLTV semantics;
V1 refund timestamp domain remains bounded below 0x80000000;
claim and refund role keys are distinct;
canonical logical HTLC structure is exactly 103 bytes;
claim path requires the valid preimage plus claim signature;
refund path requires timeout maturity plus refund signature;
incorrect secret is rejected;
pre-maturity refund is rejected.

TRU and BSTY use the same canonical logical HTLC format but NOT generally
the same script bytes because chain-local claim/refund public keys and
chain-local refund timestamps differ.

TRU V1 funding identity:

bare 103-byte HTLC output;
no fabricated TRU contract address;
funding target is the exact script output.

BSTY V1 funding identity:

the 103-byte redeemScript is wrapped in P2SH;
funding target is the canonical BSTY P2SH address/scriptPubKey.
SWAP-00   Cross-Chain Protocol Freeze   COMPLETE

Completed protocol decisions:

trust-minimized atomic swap;
no wrapped-token minting;
no custodian;
no replicated private key;
same secret/hash commitment across both chains;
first-funded leg receives the longer refund window;
second-funded leg receives the shorter refund window;
claim reveals the secret;
opposite party observes that secret and claims the other chain;
either properly constructed leg can eventually refund after timeout;
after timeout a still-unspent claim branch remains valid, so timing races
must be handled explicitly.

SWAP-00A / SWAP-00B / SWAP-00C protocol foundation is CLOSED.

SWAP-A   TRU Swap RPC + Durable Record Foundation   COMPLETE

Implemented:

htlcgeneratesecret
htlccreate
htlcclaim
htlcrefund
swaprecordcreate
swaprecordget
swaprecordlist
swaprecordtransition

Security boundary:

protected by dedicated TRU_SWAP_RPC_TOKEN;
native TRU cookie authentication remains outside that inner token;
browser never receives the TRU swap RPC token;
no raw private-key RPC;
persistent swap state never stores the atomic-swap preimage.

Durable swap identity uses canonical deterministic hashing and stores
chain IDs, funding order, HASH160 commitment, role public keys, amounts,
refund times and evidence.

SWAP-B   TRU ↔ BSTY Cross-Chain Engine   COMPLETE

Completed:

canonical chain-specific HTLC derivation;
TRU bare-script funding support;
BSTY P2SH funding support;
BSTY wallet signer isolation;
BSTY claim/refund transaction construction;
TRU claim/refund integration;
secret extraction/observation;
confirmation policy;
persistent funding/claim/refund evidence;
resolution state machine.

Final resolution semantics:

SETTLED
both funded legs confirmed claimed;
REFUNDED
every funded leg confirmed refunded;
RESOLVED_MIXED
two funded legs resolve with one confirmed claim and one confirmed refund.

Terminal resolution requires durable evidence and the configured minimum
confirmation count.

Known production debt is intentionally deferred to SWAP-C / AGENT work.

SWAP-C   Funded E2E / Crash / Adversarial Closeout   PENDING

SWAP-C must close before meaningful mainnet value is used.

SWAP-C01   TRU_FIRST Happy Path

Prove:

fresh secret;
fresh role keys;
TRU funds canonical bare HTLC;
required confirmations;
BSTY funds canonical P2SH HTLC;
required confirmations;
BSTY claim exposes secret;
TRU observes secret;
TRU claim succeeds;
durable state becomes SETTLED.
SWAP-C02   Dual Refund

Prove both funded legs recover safely without a claim.

Expected terminal state:

REFUNDED

SWAP-C03   Mixed Resolution

Prove one funded leg confirmed claimed and the other confirmed refunded.

Expected terminal state:

RESOLVED_MIXED

SWAP-C04   Reverse Funding Order

Repeat the complete lifecycle with:

BSTY_FIRST

and prove the longer/shorter timeout relationship is correctly reversed.

SWAP-C05   Broadcast / Restart Reconciliation

Close the current broadcast-before-persistence weakness.

Requirements:

durable operation intent;
deterministic transaction identity where possible;
restart-safe transaction discovery;
no duplicate funding on retry;
no duplicate claim/refund broadcast;
reconcile broadcast-success/persistence-failure;
reconcile persistence-success/process-crash;
idempotent retry behavior.
SWAP-C06   Reorg / Confirmation Regression

Test:

funding confirmation rollback;
claim confirmation rollback;
refund confirmation rollback;
confirmation threshold recovery;
competing spend detection;
unexpected funding-outpoint spend;
stale transaction evidence;
restart while confirmation state changes.
SWAP-C07   Adverse Timing

Test boundaries around:

first/second funding delays;
second-chain failure;
secret exposure near refund time;
refund/claim race;
offline participant;
watcher restart;
minimum safety gap enforcement.
LOCAL AGENT DELIVERY SERIES
AGENT-01A   Local HTTP Adapter Foundation   COMPLETE

Runtime-proven on gw878.

Properties:

listens only on 127.0.0.1:8645;
no firewall opening;
exact-origin CORS;
browser-safe response filtering;
TRU and BSTY capability probing;
local board storage;
contract descriptor generation;
verification endpoints;
protected browser/agent boundary.

Mutating swap endpoints intentionally remain fail-closed.

AGENT-01A1   Manual Pairing Alignment   COMPLETE

Completed:

canonical tru_bsty_swap_engine.py import path;
engine import selftest;
exact-source pre/post hash gates;
manual pairing token generated locally;
pairing token not exposed from /v1/health;
protected endpoint returns pairing-required behavior;
website successfully distinguishes:
no agent;
agent present but unpaired;
paired/live agent.

Runtime proof:

public market.html reached the private local agent through local loopback;
manual pairing succeeded;
demonstration board was replaced by:
Live board   Connected to Local TRU Swap Board;
real empty local board rendered successfully.
AGENT-01A2   One-Click Pairing + Automatic Reconnect   NEXT

Remove manual token copy/paste from normal user experience.

Required behavior:

FIRST USE:

website → detect local agent → user approves local connection once → paired

NORMAL USE:

website → detect existing trusted local pairing → reconnect automatically

Requirements:

no pairing credential returned by unauthenticated /v1/health;
no TRU_SWAP_RPC_TOKEN in browser;
no wallet/core RPC credentials in browser;
one-time/limited pairing handshake;
explicit local-user consent on first authorization;
persistent browser/agent trust record;
revoke/reset pairing support;
agent restart behavior defined;
stale pairing automatically renegotiated;
paired UI hides/replaces Enter token;
no silent authorization of an unknown website origin.
AGENT-01B   Per-Swap TRU Key + Destination Allocator   PENDING

Production swaps must stop reusing fixed development role children.

Allocate a fresh private deterministic key set per swap:

TRU claim role;
TRU refund role;
TRU claim destination;
TRU refund destination.

Requirements:

monotonically unique allocator;
crash-safe index persistence;
no private derivation metadata sent to browser;
no key reuse between swaps;
destination preview before funding;
destination immutable after exits are armed;
explicit disarm/rebuild flow if destination changes.

BSTY receives equivalent wallet-derived destination handling.

AGENT-01C   Persistent Watcher / Exit Recovery   PENDING

Implement unattended observation for:

TRU funding;
BSTY funding;
BSTY claim/preimage publication;
TRU claim/preimage publication;
refund maturity;
claim/refund confirmations;
outpoint spend;
restart recovery.

Only after this is proven may the UI promise that the user can close the page
while the swap continues safely.

Implement persistent exit authorization/templates only after their exact
sighash and destination semantics are regression proven.

AGENT-01D   Safe Funding / Idempotent Reconciliation   PENDING

Replace the current fail-closed /v1/fund.

Requirements:

browser identifies the user's actual funding leg;
no implicit default to TRU;
verify funding order;
verify exact script/address target;
verify exact amount;
classify malformed funding;
broadcast once;
persist/reconcile transaction identity;
restart-safe confirmation tracking;
repeated button press cannot double-fund;
state transition occurs only from verified chain evidence.

Malformed funding classification should distinguish:

OVERFUNDED
UNDERFUNDED
WRONG_SCRIPT
WRONG_TIMELOCK
WRONG_HASH
WRONG_CLAIM_KEY
WRONG_REFUND_KEY

Future recovery semantics:

FUNDING_INVALID → recovery/refund → RECOVERED

This is distinct from normal REFUNDED.

MARKET DELIVERY SERIES
MARKET-01   Public Intent-Only Swap Board   PENDING

Move public discovery/advert coordination behind the normal public web/API
infrastructure.

Cloudflare may front this PUBLIC board service.

A public advert may contain only intent information such as:

give chain;
get chain;
give amount;
requested amount;
timing policy;
confirmation policy;
expiry;
advert identifier.

A public advert must NOT contain:

atomic-swap preimage;
secret hash generated for a live swap;
role private keys;
role public keys for a not-yet-created live swap;
wallet destinations;
wallet RPC credentials;
agent pairing credentials;
TRU_SWAP_RPC_TOKEN.

The public board and the local wallet-control agent remain separate security
domains.

MARKET-02   Atomic Reserve + Swap Mint   PENDING

board/take remains fail-closed until reserve and mint become one safe workflow.

Required behavior:

ACTIVE advert → atomic reservation → fresh swap material → participant exchange

Failure before successful mint must not silently destroy the advert.

Include:

reservation expiry;
fund-or-forfeit timeout;
taker abandonment recovery;
duplicate-take exclusion;
simultaneous-taker race test;
no secret/key material in advert;
capability check before reservation;
poster/taker perspective registration;
fresh H and role keys only after successful reservation.
DISTRIBUTABLE AGENT / NO-PYTHON SERIES
PKG-01   Portable Agent Runtime   PENDING

Users must not be required to install Python or manually activate a venv.

Package:

local HTTP agent;
cross-chain swap engine;
signer helper;
required runtime libraries;
configuration/bootstrap logic.

Resolve executable discovery for:

TRU wallet/node/CLI;
BSTY wallet/node/CLI.

No secret is compiled into the package.

The local service continues binding only:

127.0.0.1:8645

PKG-02   Native Installers   PENDING

Produce user-facing packages:

Windows installer / .exe;
macOS signed .app / .dmg;
Linux binary with .deb and/or AppImage packaging.

User experience:

install → start TRU Swap Agent → open marketplace

No:

Python installation;
venv commands;
shell activation;
manual source checkout.
PKG-03   Autostart / Tray / Lifecycle   PENDING

Implement:

start at login option;
background service mode;
tray/menu-bar status;
local-agent health;
TRU connection status;
BSTY connection status;
reconnect;
pairing reset;
stop/restart;
log location;
version display.

Possible UI:

TRU Swap Agent
● Agent running
● TRU connected
● BSTY connected

Open Marketplace
Open Swap
Reset Website Pairing
Restart Agent
Stop Agent
LOCAL / REMOTE NODE CONNECTIVITY
AGENT-NET-01   No-Manual-SSH Runtime Topology   PENDING

Production users should NOT need SSH.

Primary production model:

browser + TRU Swap Agent + user's wallet/node runtime on same computer

Therefore:

tokenizedrealutility.com → user's 127.0.0.1:8645

requires no LAN relationship with TRU infrastructure and works from anywhere
the user has Internet access.

Developer / advanced remote-node model:

private authenticated transport only;
no public exposure of agent port 8645;
no public wallet/core RPC;
optional private VPN;
optional persistent SSH tunnel;
automatic reconnect;
host-key verification;
key-based authentication;
no password embedded in scripts.

For the current Mac → gw878 development topology:

replace manually entered ssh -N -L with a persistent private tunnel;
macOS launchd may maintain the tunnel automatically;
if outside the home LAN, use a private VPN/routable private address;
do not expose 127.0.0.1:8645 through Cloudflare.
FINAL WEB / UX ALIGNMENT
UI-SWAP-01   Production Browser Contract   PENDING

Complete:

connected/unpaired/offline states;
automatic reconnect;
pairing approval;
remove stale Enter token while paired;
show correct TRU bare-script semantics;
show BSTY P2SH address semantics;
show destination addresses separately from funding targets;
exact amount warnings;
chain-specific QR policy;
fail-closed button behavior;
visible backend 501/error explanation;
no unhandled funding promise rejection;
capability-derived controls.

TRU bare contract outputs must never be displayed as fake TRU addresses or
encoded in an ordinary address QR.

SWAP-FINAL   RELEASE / SECURITY CLOSEOUT   PENDING

Close only after:

all SWAP-C E2E paths pass;
fresh per-swap keys/destinations pass;
persistent watcher recovery passes;
idempotent funding passes;
public board carries intent only;
atomic advert reserve/mint passes;
executable packaging passes;
browser pairing/reconnect passes;
restart after browser close passes;
agent restart passes;
node restart passes;
machine restart passes;
stale/invalid pairing tests pass;
wrong-origin CORS tests pass;
loopback-only bind is verified;
no Cloudflare/public route to wallet-control agent exists;
secret/private-key/RPC-token leakage audit passes;
no meaningful mainnet value is used until full closeout.

Final target architecture:

PUBLIC:
Cloudflare → tokenizedrealutility.com
→ swap.html / market.html
→ intent-only board/coordinator

LOCAL USER MACHINE:
browser → 127.0.0.1:8645
→ TRU Swap Agent
→ local/private TRU + BSTY wallet/node adapters

PRIVATE MATERIAL:
private keys / swap RPC token / signing / claim / refund
never cross into the public web tier.
---

# 10. Current Status Summary

```text
AI / PROVENANCE FOUNDATION
────────────────────────────────────────────────────────────
TOKEN-AI-01A                 ████████████████████ CLOSED
TOKEN-AI-01B / 01B2R         ████████████████████ CLOSED
TOKEN-AI-01C                 ████████████████████ CLOSED
TOKEN-AI-02A                 ████████████████████ CLOSED
TOKEN-AI-02B                 ████████████████████ CLOSED/HARDENED
TOKEN-AI-02B2                ████████████████████ CLOSED
TOKEN-AI-02B3                ████████████████████ CLOSED
TOKEN-AI-02C                 ████████████████████ CLOSED
TOKEN-AI-02D family          ████████████████████ CLOSED
CR-01A                       ████████████████████ CLOSED
CR-01B                       ████████████████████ CLOSED
TOKEN-AI-03A                 ████████████████████ CLOSED
TOKEN-AI-03A1                ████████████████████ CLOSED
TOKEN-AI-03A2                ████████████████████ CLOSED
TOKEN-AI-03B                 ████████████████████ CLOSED
TOKEN-AI-03C                 ████████████████████ CLOSED
TOKEN-AI-03D                 ████████████████████ CLOSED
AI PROVENANCE V1             ████████████████████ FINAL CLOSEOUT


VAH-01   IDENTITY / AUTHORIZATION
────────────────────────────────────────────────────────────
VAH-01A Writer identity      ████████████████████ CLOSED
VAH-01B Capability registry  ████████████████████ CLOSED
VAH-01C Field matrix         ████████████████████ CLOSED
VAH-01D Signed auth history  ████████████████████ CLOSED
VAH-01E Historical verifier  ████████████████████ CLOSED
VAH-01F Adversarial harness  ████████████████████ CLOSED
VAH-01G Documentation        ████████████████████ CLOSED


VAH-02   MULTI-NODE RECONCILIATION
────────────────────────────────────────────────────────────
VAH-02A Canonical election   ████████████████████ CLOSED
VAH-02B Chain observation    ████████████████████ CLOSED
VAH-02C Durable restart      ████████████████████ CLOSED
VAH-02C.1 Checksum repair    ████████████████████ HARDENED
VAH-02D Strong V2 binding    ████████████████████ CLOSED
VAH-02 FOUNDATION            ████████████████████ CLOSED


VAH-03   HUMAN / SENSOR / DEVICE WRITERS
────────────────────────────────────────────────────────────
VAH-03A  Writer authorization/admission     ████████████████████ CLOSED
VAH-03B  Durable bounded staging            ████████████████████ CLOSED
VAH-03C Production V2 anchor                ████████████████████ CLOSED
VAH-03D Canonical external                  ████████████████████ CLOSED
VAH-03 CLOSEOUT                             ████████████████████ CLOSED


VAH-04   SENSOR BATCHING / EVENT FEEDS
────────────────────────────────────────────────────────────
VAH-04A  Canonical sensor batch + Merkle commitments     ████████████████████ CLOSED
VAH-04B  Durable accumulator / sequence / replay         ████████████████████ CLOSED
VAH-04C.1 Anchor crash-safety + txid-object hardening    ████████████████████ CLOSED
VAH-04D   Recovery/materialization/adversarial closeout  ████████████████████ CLOSED
VAH V2 FINAL CLOSEOUT                                    ████████████████████ CLOSED
```

The immediate rule remains:
The immediate rule is now:

> VAH V2 is CLOSED / FROZEN. Do not add new VAH V2 behavior unless a demonstrated defect requires reopening a specific closed invariant. New feature work returns to the core TRU roadmap; deferred VAH composition coverage belongs in the permanent regression suite.

## Master progress

```text
UI / CONTRACT VAULT
UI-30A  ████████████████████ COMPLETE
UI-30B  ████████████████████ COMPLETE
UI-30C  ████████████████████ COMPLETE*
         * tiny fullscreen SYNC-lane ownership repair remains

SMART CONTRACT HARDENING / ACTIVATION
SC-17   ████████████████████ COMPLETE   Voting
SC-18   ████████████████████ COMPLETE   Oracle
SC-19   ████████████████████ COMPLETE   Hash Lock
SC-20   ████████████████████ COMPLETE   Contract execution
SC-21A  ████████████████████ COMPLETE   Explorer Classification
SC-21B  ████████████████████ COMPLETE   RPC / Base58 Parity
SC-22   ████████████████████ COMPLETE   Script / Relay Policy Cleanup

CONTRACT APPLICATIONS
────────────────────────────────────────────────────────────
MS-01       ████████████████████ COMPLETE   Multisig / Escrow
HTLC-01     ████████████████████ COMPLETE   canonical TRU HTLC / Atomic Swap


TRU ↔ EXTERNAL CHAIN ATOMIC SWAP STACK
────────────────────────────────────────────────────────────
SWAP-00     ████████████████████ COMPLETE   frozen cross-chain protocol
SWAP-A      ████████████████████ COMPLETE   TRU swap RPC/state foundation
SWAP-B      ████████████████████ COMPLETE   TRU↔BSTY engine + resolution state

AGENT-01A   ████████████████████ COMPLETE   loopback HTTP adapter
AGENT-01A1  ████████████████████ COMPLETE   canonical path + manual-pairing alignment
AGENT-01A2  ████████████████████ COMPLETE   HttpOnly pairing / automatic reconnect
AGENT-01A2.1████████████████████ COMPLETE  Cloudflare Access credential/CORS repair

AGENT-01A2.1 ████████████████████ CLOSED
             Cloudflare Access / CORS / HttpOnly remote browser path
             live browser runtime proof completed in SWAP-FRESH-01B3C

SWAP-FRESH-01A
            ████████████████████ CLOSED
            deterministic per-swap TRU claim/refund allocator
            authenticated RPC
            runtime replay/separation proof complete
            no private material exported

SWAP-FRESH-01B1A   ████████████████████ CLOSED
SWAP-FRESH-01B2    ████████████████████ CLOSED
SWAP-FRESH-01B3A   ████████████████████ CLOSED
SWAP-FRESH-01B3B   ████████████████████ CLOSED
SWAP-FRESH-01B3C   ████████████████████ CLOSED
SWAP-FRESH-01B3D   ████████████████████ CLOSED
SWAP-FRESH-01B4A   ████████████████████ CLOSED
SWAP-FRESH-01B4B   ████████████████████ CLOSED
SWAP-FRESH-01B4C   ████████████████████ CLOSED
SWAP-FRESH-01B4D   ████████████████████ CLOSED

GROUP-1     ████████████████████ CLOSED  TRU reservation + broadcast safety
GROUP-2    ████████████████████ CLOSED   BSTY full prebroadcast safety
GROUP-3     ████████████████████ CLOSED   dual-chain funding activation
GROUP-4     ████████████████████ CLOSED   watcher / claim / refund recovery
GROUP-5     ████████████████████ CLOSED   tiny real funded swap
TRU CRASH-SAFE FUNDING RECONCILIATION ████████████████████ CLOSED
SWAP-C     ████████████████████ CLOSED  TRU - BSTY funded E2E / adversarial closeout

AGENT-01B   ████████████████████ CLOSED
            fresh per-swap key integration
            two-party V2 offer / accept / finalize
            canonical swap creation
            browser + Cloudflare Access runtime proof
            adversarial/replay proof
            restart durability proof
AGENT-01C   ████████████████████ CLOSED
            persistent watcher / exit recovery
            durable ARMED / ACTION_STARTED / HOLD_UNKNOWN / COMPLETE
            claim/refund recovery
            restart durability
            mined-outcome reconciliation
            no blind retry

AGENT-01D   ████████████████████ CLOSED
            idempotent funding / crash reconciliation
            TRU reservation barrier
            BSTY prebroadcast barrier
            exact prepared-byte replay
            dual-chain funding activation
            RECORDED journal recovery
            funded real-money E2E proof


TRU MARKET / PRICE DISCOVERY
────────────────────────────────────────────────────────────
MARKET-01A  ████████████████████ COMPLETE   public TRU/BSTY intent book

MARKET-01B  ████████████████████ CLOSED
            canonical BSTY-per-TRU bid/ask pricing
            exact rational ordering
            live public proof complete
            LAST correctly unavailable

MARKET-01C0 ████████████████████ CLOSED
            durable atomic reservation substrate
            race-safe single reservation
            timeout/release/bind semantics
            existing adverts preserved
            public Take still disabled


POST-MULTINODE REORG HARDENING
────────────────────────────────────────────
REORG-TX-01     ████████████████████ CLOSED -  explicit CONFIRMED / MEMPOOL / SIDECHAIN / CONFLICTED / NOT_FOUND

REORG-TX-01A    ████████████████████ CLOSED -  real Group-05 runtime compatibility

REORG-SWAP-01A  ████████████████████ CLOSED -  pure reorg recovery policy - fresh funding after reorg forbidden - blind rebroadcast forbidden

REORG-SWAP-01B  ████████████████████ CLOSED -  durable reorg observation overlay

REORG-SWAP-01C  ████████████████████ CLOSED  - reconciler integration

REORG-EXIT-01   ████████████████████ CLOSED  - permanent-exposure invariant, four-way exit observation, no canonical rewind

REORG-EXIT-01A   ████████████████████ CLOSED -  claim/refund reorg recover  permanent-preimage-exposure invariant

REORG-EXIT-01B  ████████████████████ CLOSED

CANONICAL SOURCE / GITHUB RELEASE   ████████████████████ CLOSED  -  TWO-NODE RUNTIME PROOF

MARKET WEB 
────────────────────────────────────────────────────────────
MARKET-MASTER-01 through MARKET-01C  ██████████░░░░░░░░░░ IN PROGRESS Public Take + atomic reservation + fresh swap binding NO automatic funding

MARKET-MASTER-02 though MARKET-01G  ░░░░░░░░░░░░░░░░░░░░ PENDING   confirmed trade tape + OHLC/volume/VWAP/spread/depth + public read-only data API + historical candles

MARKET-MASTER-03 through MARKET-01H + MARKET-02  ░░░░░░░░░░░░░░░░░░░░ PENDING   integrity controls + duplicate/self-trade protection + production TRU/BSTY market closeout

MARKET-03A  ░░░░░░░░░░░░░░░░░░░░ PENDING   CoinGecko market integration package
MARKET-03B  ░░░░░░░░░░░░░░░░░░░░ PENDING   CoinMarketCap market integration package
MARKET-03C  ░░░░░░░░░░░░░░░░░░░░ PENDING   TRU cryptoasset listing package

MARKET PAIR EXPANSION
────────────────────────────────────────────────────────────
PAIR-01      ░░░░░░░░░░░░░░░░░░░░ PENDING   TRU/BSTY   first native atomic-swap market
PAIR-02      ░░░░░░░░░░░░░░░░░░░░ PENDING   TRU/BTC   stronger external price reference
PAIR-03      ░░░░░░░░░░░░░░░░░░░░ PENDING   TRU/USDT   direct USD-denominated reference


DISTRIBUTABLE SWAP AGENT
────────────────────────────────────────────────────────────
PKG-01       ░░░░░░░░░░░░░░░░░░░░ PENDING   portable agent runtime
PKG-02       ░░░░░░░░░░░░░░░░░░░░ PENDING   Windows / macOS / Linux installers
PKG-03       ░░░░░░░░░░░░░░░░░░░░ PENDING   autostart / tray / signed updates

SWAP-FINAL   ░░░░░░░░░░░░░░░░░░░░ PENDING   production security + release closeout

TRU/BSTY   ░░░░░░░░░░░░░░░░░░░░ PENDING   ← first native cross-chain market
TRU/BTC    ░░░░░░░░░░░░░░░░░░░░ PENDING   ← much stronger external reference
TRU/USDT   ░░░░░░░░░░░░░░░░░░░░ PENDING   ← strongest simple USD reference if/when practical

TOKENS 
────────────────────────────────────────────────────────────
TOKEN-AI ████████████████████ AI Token Evolution 
NFT-01   ░░░░░░░░░░░░░░░░░░░░ Royalty NFT
GOV-01   ░░░░░░░░░░░░░░░░░░░░ Treasury Governance
VEST-01  ░░░░░░░░░░░░░░░░░░░░ Vesting Plan
POE-01   ░░░░░░░░░░░░░░░░░░░░ Proof of Existence
INH-01   ░░░░░░░░░░░░░░░░░░░░ Inheritance Vault
TOK-01   ░░░░░░░░░░░░░░░░░░░░ Batch Distribution

BRIDGE GENERATION
────────────────────────────────────────────────────────────
BR-23    ░░░░░░░░░░░░░░░░░░░░ Bitcoin / generic external-chain framework
BR-24    ░░░░░░░░░░░░░░░░░░░░ GlobalBoost-Y bridge migration
BR-25    ░░░░░░░░░░░░░░░░░░░░ Solana adapter
BR-26    ░░░░░░░░░░░░░░░░░░░░ User-defined / custom bridge interface

FINAL CONTRACT INTEGRATION
────────────────────────────────────────────────────────────
INT-27   ░░░░░░░░░░░░░░░░░░░░ Wallet / compiler / forms parity
INT-28   ░░░░░░░░░░░░░░░░░░░░ Full contract regression / consensus suite

REMAINING CORE PRODUCTION HARDENING
────────────────────────────────────────────────────────────
SEC-14   ░░░░░░░░░░░░░░░░░░░░ Wallet encryption
SEC-17   ░░░░░░░░░░░░░░░░░░░░ Regtest / consensus / fuzz suite
NET-05   ░░░░░░░░░░░░░░░░░░░░ RPC / website gateway deployment
SEC-18   ░░░░░░░░░░░░░░░░░░░░ Hard-fork commitment cleanup
IDX-19   ░░░░░░░░░░░░░░░░░░░░ Header-only block index

VAH V2 KNOWN DEFERRALS / PERMANENT REGRESSION COVERAGE
────────────────────────────────────────────────────────────
VAH-COMP-01 ░░░░░░░░░░░░░░░░░░░░ Rotate/revoke-after-materialize composition fixture   NOT COVERED
VAH-MIG-01  ░░░░░░░░░░░░░░░░░░░░ TOKEN:EVOLUTION legacy checksum inventory/migration   DEFERRED / DELIBERATE


NODE-REAL-01   Independent Multi-Node Divergence / Convergence Proof   CLOSED / RUNTIME PROVEN

Completed runtime proof:

- Two independent TRU nodes operated concurrently.
- Real TRU mainnet P2P connectivity established.
- Initial block synchronization completed.
- Both nodes reached identical active-chain height and best-tip identity.
- Independent competing-chain conditions were exercised.
- Side-chain / competing-branch handling operated through the production network path.
- Strict cumulative `ChainWork256` winner policy was exercised.
- Equal-work behavior remained first-seen / no reorganization.
- Strict greater-work branch became the canonical winner.
- Automatic reorganization executed on the losing node.
- Losing active branch was retained as indexed side-chain history.
- Both nodes converged to the exact same canonical active tip.
- Post-convergence restart preserved the canonical result.
- No chain reset was required.

Final result:

`NODE_REAL_01=TWO_NODE_RUNTIME_PASS`

`MULTINODE_DIVERGENCE=PASS`

`STRICT_GREATER_WORK_WINNER=PASS`

`AUTOMATIC_REORG=PASS`

`POST_REORG_CONVERGENCE=PASS`

`NODE_REAL_01=CLOSED`


CURRENT PRIORITY AFTER VAH V2 CLOSEOUT
1. SEC-17      Permanent regtest / consensus / fuzz coverage
2. SEC-18      Hard-fork commitment cleanup
3. IDX-19      Header-only block index

---

# Current Reorganization Safety Status

**Automatic/network reorganization is still intentionally DISABLED.**

Completed foundations:

- Patch 09 authoritative 256-bit chainwork   DONE
- Patch 08A bounded side-chain indexing   DONE
- Patch 08B.1 durable U4 undo journals   DONE
- Patch 08B.2 authenticated journal reader + safe single-tip disconnect   DONE
- Patch 08B.3 isolated stateful fork validation   DONE
- Patch 08B.3a LevelDB registry lifetime / locking   DONE
- Patch 08B.3b sandbox hygiene / performance   DONE
- Patch 08B.3T controlled runtime execution tests   DONE
- Patch 08B.4A durable reorg preflight / PREPARED state machine   DONE
- Patch 08B.4A.1 save-chain-state durability/lifetime fix   DONE
- Patch 08B.4A.1a deterministic shutdown   DONE
- Patch 08B.4A.1b signal-safe CLI shutdown   DONE
- Patch 08B.4B controlled multi-block reorg executor   DONE
- Patch 08B.4C authenticated crash recovery   DONE
- Patch 08B.4D.1 runtime stale-side pruning   DONE
- Patch 08B.4D.2 work-aware side-pool admission / eviction   DONE
- Patch 08B.4D.3 U4 retention / pruning   DONE
- Patch 08B.4D.4 positive candidate-validation cache   DONE
- Patch 08B.4D.5 immutable-only negative candidate cache   DONE
- Patch 08B.4D.6 per-source validation budget   DONE
- Patch 08B.4D.7 global validation budget   DONE
- Patch 08B.4D.8 final regression / crash / adversarial / gate-OFF closeout   DONE
- Patch 08B.4D.9 Step 1 activation-surface inventory   DONE
- Patch 08B.4D.9 Step 2 executor/API deep dive   DONE
- Patch 08B.4D.9A production gate + fail-stop shutdown plumbing   DONE / CLOSED

Current canonical 9A source:

- `src/blockchain.h`
  `febbca9b8b4f784099a00e72bac6ef5f9ef6e7de4bd495d7b6f7ee8f0d4dbcc8`
- `src/blockchain.cpp`
  `b0f7b7e9f1be93908fd1a70dc9d466dd56fed734330df9b8daaadd647874637d`
- `src/main.cpp`
  `494c1e48aad2e4961f790fce92593585535bc977cce3c4e9841799ffb1849aba`

Current canonical 9A builds:

- DEV `tru_advanced`
  `c51612efc772e5c9c76d4485f0af328048316b7ec82f5b23f1289b0f53ab6a0f`
- PROD `tru_advanced`
  `a3540c8ef6d96f4ec5eb44f63dd5ec0ab882e4eefd26fabe17058be1b1c3d8fc`
- CLI
  `bd4dd49ba9ddc81dc2284273d1c7a3b35e73e1d993b80160f8885062f0b26c1d`

Current activation state:

- Production gate exists   YES
- Gate default   OFF
- Wrong magic   OFF
- Exact magic may arm policy   YES
- Canonical incoming-side automatic PREPARE/EXECUTE wiring   NO
- Legacy `handleChainReorganization()`   FAIL-CLOSED
- DEV-only manual reorg surfaces in PROD   ABSENT
- Host runtime/recovery gates after tests   OFF
- Mempool resurrection after reorg   DEFERRED

Current development stage:

- **Patch 16A.0 confirmed-state-only token indexing   CURRENT BLOCKER**
- **Unconfirmed token-state containment runtime proof   NEXT AFTER 16A.0 DRY RUN/APPLY**
- **Token-U4 issuance + transfer proofs   required immediately afterward**
- **08B.4D.9B corrected v1a   paused until those token gates close**
- **08B.4D.9B shadow mode   required before live automatic execution**
- **08B.4D.9C startup reconciliation   pending**

Token review disposition:

- Active normal block-application token writes appear statically routed through the transaction batch → `mainBatch` → U4 path.
- A separate active `issuetoken` RPC path currently writes confirmed token metadata/index keys directly after finding the newly created transaction in chain **or mempool**; this bypass must be removed before the U4 proof is meaningful.
- The runtime unconfirmed-state containment proof and Token-U4 proof are both required before live automatic reorganization.
- Separate Patch 16 tracks the confirmed token-hardening debt:
  - unconditional metadata-signature stub;
  - unsafe/off-path direct mutation helpers;
  - 32-bit token identifiers;
  - unchecked amount arithmetic;
  - malformed-state fail-open behavior;
  - explicit token consensus/reorg regression coverage.

Safety rule:

**DO NOT ENABLE AUTOMATIC ACTIVE FORK PROMOTION until Patch 16A.0 confirmed-state containment, unconfirmed token-state runtime proof, Token-U4 issuance/transfer proofs, corrected 9B review, shadow mode, real strict-winner runtime tests, 9C startup reconciliation, and explicit final 08B.4D.9 approval all pass.**





That is the copy/paste roadmap block.

---

# 2. Where we actually are right now

I would describe the current state as **protocol/engine largely done, product delivery about halfway through**.

| Layer | Status |
|---|---|
| Canonical TRU HTLC | ✅ Closed |
| BSTY P2SH HTLC mechanics | ✅ Proven |
| Same-secret cross-chain protocol | ✅ Frozen |
| TRU SWAP RPC/state | ✅ Closed |
| Cross-chain engine | ✅ Built |
| `SETTLED / REFUNDED / RESOLVED_MIXED` resolution | ✅ Built |
| Local agent install | ✅ |
| Agent isolated Python venv | ✅ |
| Live TRU connectivity | ✅ |
| Live BSTY connectivity | ✅ |
| Browser → local agent | ✅ |
| CORS/local-network path | ✅ |
| Manual browser pairing | ✅ |
| Real Live Board rendered | ✅ |
| Agent publicly exposed | **NO   correct** |
| Live funding from website | 🔒 Deliberately disabled |
| `board/take` | 🔒 Deliberately disabled |
| offer create/import | 🔒 Deliberately disabled |
| persistent exits/watcher | 🔒 Deliberately disabled |
| fresh TRU keys per swap | ❌ Not yet |
| automatic pairing | ❌ Not yet |
| packaged executable | ❌ Not yet |
| public shared marketplace | ❌ Not yet |
| no-SSH user deployment | ❌ Not yet |
| full funded SWAP-C matrix | ⏳ Waiting / pending |

The important milestone from today is real: **the public website successfully found and paired with the real local agent and rendered the actual Local TRU Swap Board.** That is no longer a mock-only architecture.

The roadmap itself already treats `HTLC-01` as complete in the master progress section, despite the stale older detailed `PENDING` block. :contentReference[oaicite:3]{index=3} :contentReference[oaicite:4]{index=4}

---

## What I would do next, in exact order

Because you're currently waiting on the BSTY side for the funded atomic-swap work, we have useful work that **doesn't touch the frozen swap engine or move coins**:

```text
NOW
 │
 ├── AGENT-01A2
 │     One-click pairing + automatic reconnect
 │
 ├── UI-SWAP-01A
 │     paired-status cleanup / remove Enter token when live
 │
 ├── AGENT-NET-01A
 │     automatic private Mac→gw878 tunnel for your development machine
 │
 ├── PKG-01A
 │     portable-runtime audit
 │     determine exactly what must ship with executable
 │
 └── WAIT FOR BSTY READINESS
          ↓
       SWAP-C01
          ↓
       SWAP-C02
          ↓
       SWAP-C03
          ↓
       SWAP-C04
          ↓
       SWAP-C05/C06/C07
          ↓
       AGENT-01B/C/D
          ↓
       MARKET-01/02
          ↓
       PKG-01/02/03
          ↓
       SWAP-FINAL

