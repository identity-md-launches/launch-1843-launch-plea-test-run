# PLEA Sepolia test run 7 adaptation

This checkout is the requester's final code for the launch: run 6's code (branch `testnet6` @ `6ef22d1`: the run-5 delivery `0f7b870c`, the agreed-clock check in `PleaLaunch.seed`, the appeal band on the lesser share, the Tribunal never holding an excluded seller's plea) plus the run-7 batch and the fixes from the four audit rounds (`audit/FINDINGS.md` R1-* to R4-*, reports in `audit/rounds/`, regressions in `test/AuditRound1.t.sol`). The launch factory deploys exactly the five contracts `launch.json` lists, in that order, with the constructor arguments it lists.

This adaptation audited the imported code against the brief and the launch rules and deploys it **unchanged**: no file under `src/`, `script/` or `test/` and no word of `launch.json` was modified. No deployment, signing or broadcast was performed. Only this file changed (it previously described the run-5 adaptation).

## Changes

None to deployed source, tests, scripts or the manifest.

1. `ADAPTATION.md` (this file): rewritten for run 7. The run-5 text it replaced described fixes that are now part of the imported code and ledgered in `audit/FINDINGS.md` (R5-I-1, R5-I-2, the run-5 build repair), so it no longer listed what this launch changes.

## Audit findings

The import audit supplied with this task reported one finding, `43b7c30e…` (Info): a coverage statement that every `src` contract was read in full and no reproducible defect was found in the run-7 batch or the Tribunal, with the live Sepolia dependencies not reached. It is not a defect and nothing was changed for it. This adaptation re-read the same code and reproduced the audit's verifications (below); it also found no defect. The four audit rounds in `audit/FINDINGS.md` list no `open` finding: every R1-* to R4-* finding is `fixed` (with its commit) or `accepted` by the project owner with a written reason (R2-A3-2, the hold-time dilution through a named buy, listed in `audit/THREAT-MODEL.md` as a known limit). There is nothing left to fix, so no new test was needed; the acceptance test for "every change is tested" is met by the existing 759-test suite, which guards each ledgered fix.

## What was verified against the brief

Read in full: `src/PLEA.sol`, `src/PleaLaunch.sol`, `src/PleaDistributor.sol`, `src/CabalGate.sol`, `src/Seasons.sol`, `src/Tribunal.sol`, `src/Letterbox.sol`, `src/PleaHook.sol`, `src/OracleAttestation.sol`, the two interfaces, `launch.json`, the protected rehearsal tests and the eth-security reference. Each run-7 item and agreed number was located in the code:

- Letters pay no 0.35 IMD auto-sell fee: `Letterbox._deliver` takes `AUTO_SELL_FEE` only when the plea's flags lack `BY_LETTER`; `_requireWorthTheFee` keeps the 0.70 IMD floor for letters and the site's box alike. `LetterTooLong(bytes, 280)` from `_checkLength`. `LETTER_SHARE_BPS = 1_500`.
- Seasons: `SEAT_MULTIPLIER_BPS = 12_000`, `pointsFor` floors `score * 12000 / 10000` when `holdsSeat` is true; `holdsSeat` is a 100,000-gas staticcall to the fourth constructor argument that returns false on no code, a revert or a malformed answer, so it never blocks `record`. `MIN_POINTS = 60`, `TOP = 25`, `shareBps` packed 450 down to 350, `BONUS_SEASONS = 6`, `BONUS_PER_SEASON = 10_000_000e18`, `CLOSE_GRACE = 1 hours`.
- PleaHook: `CASHBACK_BPS 50`, `OWNER_BPS 75`, `PRIZE_BPS 25`, `POOL_BPS 25` (IMD fee 1.75%), `PLEA_BURN_BPS 25` (2.0% in all); `CAP_DECAY_DAYS = 15` with the 900,000 PLEA floor.
- PleaDistributor: `Ownable2Step`; `renounceOwnership` reverts `RootNotSet` while the root is zero; the root is set once, never zero.
- CabalGate: `PASS_MARK = 71`; `_factScoreAt` scores share ≤15%: 16, ≤30%: 9, more: 3; held ≥7 days: 7, ≥3 days: 4, ≥1 day: 2; at a loss: 13, gain ≤50%: 8, ≤200%: 3, more: 0; trend up (over +2%): 9, flat: 5, down: 0 (maximum 45); `_open` reverts `CannotPass(need)` when `71 - fact > 45`; an appeal is banded on the lesser of its own share and the original's. Panel 30, quorum 17, 5-hour seasons (18,000) and the 33-hour dead-man (118,800) are the manifest's sixth and seventh arguments.
- Tribunal: `OBJECTION_WINDOW` and `PENDING_EXPIRY` 20 minutes, `ORACLE_FEE 0.5e18` (minted on Sepolia when the launch's mint flag is set, as the Letterbox's), `STAKE 0.35e18`, `MARGIN 2`, `MAX_OBJECTIONS 2` with `SameObjector`; sustained iff `newScore + 2 < need`; refused, split or 20 minutes unanswered voids it (`_void`, `expire`); `settle` pays a sustained or void stake back and an overruled one to Seasons through `checkpointObjection`, booked to the plea's season; the recorded score is the lowest any panel gave (`cases[].lowest`); `object` refuses the seller, excluded wallets and an excluded seller's plea; no ruling touches the sale or `lastVerdictAt`.
- PLEA: `init` mints 82% to the hook, 6% to Seasons and 12% to the distributor, launcher-only and once; `setLauncher` accepts the deploy-transaction transient flag or PLEA's deployment block, closed by the atomic factory transaction; no mint, pause, freeze, blacklist or seizure exists; the only holder restriction is the Cabal's sell gate, which ends for good at the dead-man.
- Launch rules: every deployed constructor is nonpayable and calls no other contract (PleaLaunch reads only the four contracts deployed before it in the same transaction); every owner argument is `$owner` (`0x4b91…6821`), never `msg.sender` or `tx.origin`; no DELEGATECALL, CALLCODE, SELFDESTRUCT, proxy or initializer; no public or external library function; at most 13 arguments per contract.
- `launch.json` is the requester's manifest and the brief's reference ("exactly as launch.json lists it"); it is kept as it arrived. Its 12th and 13th PleaLaunch arguments equal `keccak256` of the build's Letterbox (`0x3c7cd573…c0235c`) and Tribunal (`0x562d0f3e…755665`) creation code; its notes are 3,180 characters, under the 4,000 cap.

## Verification

All with the repository's own configuration (Solc 0.8.26, Cancun, via-IR, optimizer 200, `bytecode_hash = "none"`) and the verifier's seed `FOUNDRY_FUZZ_SEED=0x2098de5e4458b94d3d9244de82563d528653d0a20bd536067c6a06aacc930687`.

- `forge build --skip test --skip script`: passed in 35 s; the three pre-existing compiler warnings (Tribunal `status` shadowing, Seasons' unused tuple member, CabalGate `_text` mutability) remain and are informational.
- Default profile, three groups of `--match-path`: 201 + 250 + 152 = **603 passed, 0 failed** in 59 suites.
- `FOUNDRY_PROFILE=full`, the invariant, adversarial and fork files in two groups: 74 + 55 = **129 passed, 0 failed, 2 skipped** in 25 suites. The two skips are `ForkMainnet` and `ForkTestnet`, whose setups skip without their RPC URL variables; no live-fork result is claimed. The two forks hold the remaining 25 of the suite's 759 tests.
- Whole suite: 84 suites, 732 run here, 0 failed.
- `test/LaunchManifest.t.sol`: the five-contract launch through IMD's factory charges **13,534,269** gas; with the 22% headroom **16,511,808**, **265,408** under the 16,777,216 cap. `deployLetterbox` 5,575,052 and `deployTribunal` 4,910,592 gas after the launch.
- Runtime sizes from the build: CabalGate **24,493** (83 under EIP-170), PleaHook **24,358**, Letterbox **23,229**, Tribunal **20,116**, Seasons **12,473**, PLEA 4,893, PleaLaunch 3,945, PleaDistributor 1,513.
- Not run: Slither, Mythril and the long fuzz campaigns the eth-security reference names (not provided here). The live Sepolia dependencies (Intake v2, Stacker, pool TestIMD, the oracle-fee test token, the IMD seat NFT) were not reached; the contracts check them on first use, as the launch rules require.

## Open items for the deployer, not this adaptation

- Explorer verification of the five contracts and of the Letterbox, Tribunal and hook deployed afterwards through `PleaLaunch` belong to IMD's deployer and the launch policy.
- After the launch the owner deploys the hook with a mined flag-compatible salt, then the Letterbox and the Tribunal from the committed code, sets the distributor's single Merkle root, and opens trading through `seed` (any time after the hook; anyone from START 1792162800). None of these is a manifest initialization call.
