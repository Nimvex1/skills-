---
name: smart-contract-security-audit
description: Evidence-gated smart-contract and blockchain security auditing for EVM/Solidity (proxies, ERC-4626 vaults, strategies, staking/rewards, AMMs, Uniswap v4 hooks, lending, permits/Permit2, governance, bridges, multisig, escrow/factory/quest payout systems) and Sui/Move (capability and object authority, entry boundaries, node/execution, verifier, framework invariants, DeepBook-style DEX, key-server, bridge). Covers compiler and EVM-target verification, upgrade and storage-layout diffs, executable grep sweeps, coverage-ledger and multi-lens recall passes, real-exploit PoC replication, Foundry PoCs with negative controls, false-positive elimination, OWASP SC Top 10 classification, and audit-grade reporting. Use for full-repo audits, PR/diff audits, single-module reviews, and adversarial verification of someone else's finding.
version: 4.1.0
---

# Smart Contract Security Audit

Companion skills: `legacy-solidity-audit-harness` (solc < 0.8, get a runnable
Foundry substrate first), `foundry-invariant-gap-audit` (a green invariant suite is
a *map of what is not tested*). Use those instead of re-deriving their ground.

---

## 0. Contents

| § | Section | Use it for |
|---|---|---|
| 1 | Prime directive and scoring | framing |
| 2 | Mode selection | the very first decision |
| 3 | The four modes in detail | scoping |
| 4 | Operating rules | non-negotiable, always |
| 5 | Phase 0 — Recon and baseline | commands you actually run |
| 6 | Phase 1 — Trust model | the tables you must produce |
| 7 | Phase 2 — Invariant ledger | write before hunting |
| 8 | Phase 3 — Coverage ledger | **the recall engine** |
| 9 | Phase 4 — Multi-lens fan-out | parallel passes and merge |
| 10 | Phase 5 — Hypothesis generation | breaking each invariant |
| 11 | Phase 6 — Validation and PoC | executing the exploit |
| 12 | Phase 7 — False-positive gate | one gate, hard mapping |
| 13 | Phase 8 — Self-adversarial review | disproving yourself |
| 14 | Phase 9 — Fix and negative control | proving the fix works |
| 15 | Phase 10 — Severity and confidence | anchored to capital at risk |
| 16 | Reporting | format and discipline |
| 17 | Grep sweep | executable pattern list |
| 18 | Tool cheatsheet | commands, not tool names |
| 19 | PoC recipes | working Foundry code |
| 20 | Upgrades and storage layout | diff mode core |
| 21 | EVM domain playbooks | per-subsystem attack classes |
| 22 | Move / Sui playbooks | object-capability model |
| 23 | Escrow, factory, quest payouts | multi-token custody |
| 24 | Version matrix — dependency and compiler | verified advisories |
| 25 | Cross-contract reasoning | the six graphs |
| 26 | Economic exploit analysis | is it worth doing |
| 27 | Context and budget discipline | stop conditions |
| 28 | Appendices | catalogs and final checklist |
| 29 | References | sources |

---

## 1. Prime directive and scoring

Your job is not to produce findings. Your job is to find **real, externally
meaningful security failures** and prove them.

Optimize for:

```
real impact x exploitability x confidence / false positives
```

Rules that follow from that:

- **Recall is free; precision is not.** A hypothesis costs one paragraph. A
  false-positive finding costs credibility on every real finding in the report.
  Generate broadly, kill ruthlessly.
- **Never inflate severity.** Never hide uncertainty. Never present a
  hypothesis as a confirmed vulnerability.
- If you cannot demonstrate reachability, attacker control, invariant
  violation, impact, and reproducibility — downgrade to observation, question,
  or discard. Do not delete silently: log it in the discarded-candidate list so
  the user can see what you killed and why.
- **Absence of a finding is a claim.** "No High/Critical issues" requires the
  coverage ledger in §8 to be complete. Without it, say "analysis incomplete".

## 2. Mode selection

Decide the mode before reading any logic. The mode determines the file set, the
budget, and what counts as done.

| Signal from the user | Mode |
|---|---|
| "audit this repo" / "full audit" / no version target | **A — Full** |
| a PR, branch, commit range, release, upgrade, "review my changes" | **B — Diff** |
| a named contract/function/module, "why is X exploitable" | **C — Module** |
| a pasted finding, another auditor's report, a contest submission | **D — Verify** |

Rules:

- Default to **B** when a version boundary exists (upgradable protocol,
  monorepo with a diff, "v2"). Most production findings live in the diff, and
  diff mode buys 3-5x more depth per line of budget than full mode.
- Default to **A** only when the user genuinely wants the whole surface.
  Even then, triage: rank files by custody/mint/upgrade surface (§5.2) and
  audit rank 1 fully before reading rank 5 at all.
- **C** and **D** still require §7 (invariant) and §12 (gate). There is no
  shortcut that skips the gate.

## 3. The four modes in detail

### Mode A — Full repository

In scope: every production contract that holds value, moves value, gates
access, or determines accounting. Out of scope by default: mocks, test helpers,
`script/`, `test/`, vendored `lib/` (except where a *modified* vendored copy is
suspect — see §24.4), generated code, interfaces with no implementation.

Done means: coverage ledger complete (§8), every in-scope external function
carries a status, all Critical/High surfaces have an invariant + a kill-or-keep
verdict.

### Mode B — Diff

1. Establish the base. `git status --short`, `git diff <base>...HEAD --stat`,
   `git log --oneline <base>..HEAD`. If the tree is dirty, say so — a dirty tree
   means the diff you audit is not the diff that ships.
2. Read the full diff, not the stat. Then read the **whole** function that
   changed, plus its callers and callees. A one-line change in a modifier
   inherits the blast radius of every function carrying it.
3. Always run §20 (storage layout) if any proxy/implementation/beacon/`init`
   appears anywhere in the diff.
4. For each changed function, answer: *what state did the old version rely on
   that the new version no longer guarantees?*
5. The pre-change code is evidence of intent. A removed `require` is a finding
   candidate; a removed modifier is stronger.

### Mode C — Module

Deep on one subsystem, but still: build the trust model for that subsystem
(§6), write its invariants (§7), and then sweep the *entire repository* for
other call sites of the same pattern. Most "isolated" bugs are the same bug in
four places, and finding all four is the difference between Low and High.

### Mode D — Verify (adversarial review of a submitted finding)

Your default posture is **hostile**. Assume the reporter is wrong until the
code forces otherwise. Steps:

1. Restate the claimed path in your own words, from the code, before reading
   their PoC.
2. Find the check that defeats it. If you cannot name a specific
   defense that holds, the finding is real — say so plainly.
3. Reproduce it. A submitted PoC that does not run in this repo is a finding
   about the *report*, not the code; say exactly which line fails and why.
4. Check reachability assumptions independently: is the caller really
   permissionless? Is the state really reachable? Is the impact really
   quantified?
5. Re-rate severity from scratch. Reporters over-rate; do not inherit their
   number.
6. If it holds, still report it in the §16 format with your own proof, and add
   the reachability caveats the reporter missed.

## 4. Operating rules

1. **Read the code before concluding anything.** Never infer behavior from a
   name, a NatSpec comment, or a file name. NatSpec is frequently aspirational.
2. **Quote exact locations**: `path/to/File.sol:FunctionName:line`. If line
   numbers are unavailable, say so.
3. **Never invent code, lines, roles, or state.** If a file is missing, ask for
   it or mark the audit incomplete — do not assume its contents.
4. **Label every claim**: `Confirmed` / `Likely` / `Hypothesis` / `Observation`
   / `Question`. `Confirmed` requires a PoC or a mechanical proof.
5. **No exploitation claim without a call path.** A suspicious line is not a
   finding until you trace who reaches it and what state changes.
6. **Track assumptions explicitly.** Each one is tested or marked unverified.
   Any unverified assumption caps confidence at Medium.
7. **Prefer a 20-line PoC to 400 words of argument.** Argument is not evidence.
8. **Challenge every finding against §12 and §13 before it leaves your hands.**
9. **Report uncertainty honestly**, with a reason for the confidence level.
10. **Never report as security**: style, naming, gas cost, missing comments,
    missing events, "would be nicer as", or centralization that has no concrete
    exploitable boundary failure. Centralization becomes a finding only when it
    yields an escalation or an unauthorized asset path.
11. **Distrust the test suite.** Green tests are evidence about tested paths
    only. §5.4 tells you how to mine them for what they skip.
12. **Distrust your own memory of known exploits.** A CVE number in this skill
    is a starting point, not evidence. Confirm against §24 and the live
    advisory database before citing.

## 5. Phase 0 — Recon and baseline

### 5.1 Establish what you are looking at

```bash
# identity + scope
git rev-parse HEAD; git status --short; git log --oneline -20
find . -name '*.sol' -not -path './lib/*' -not -path './test/*' | wc -l

# toolchain
cat foundry.toml 2>/dev/null; cat hardhat.config.* 2>/dev/null | head -60
cat package.json | jq '.dependencies,.devDependencies,.scripts'
cat Move.toml 2>/dev/null

# exact dependency versions (never trust semver ranges for security)
jq -r '.dependencies["@openzeppelin/contracts"]' package.json
git submodule status; cat soldeer.lock 2>/dev/null
```

Record and report: languages, compiler versions per file (`pragma solidity`),
framework, dependency versions, upgrade/deploy scripts, CI config, generated
code, external interfaces, and any off-chain component that influences on-chain
security (relayers, keepers, signers, sequencer, orderer, off-chain signer
services).

**A CI profile with no matching `[profile.<name>]` in `foundry.toml` means tests
run unoptimized while production ships optimized. Re-measure any gas-dependent
claim under the production profile before reporting it.**

**Then verify the toolchain itself.** Correct source compiled by a buggy compiler
is still a vulnerable contract, and this check is missing from most audit scopes.
Collect the exact compiler build, the `evm_version`/`evmVersion` target, the
optimizer and `viaIR` flags, and the lockfile dependency SHAs — then check them
against the advisory tables in §24.6 before you trust code that merely *looks*
correct:

```bash
rg -oN 'pragma solidity\s+[^;]+' -g '*.sol' --glob '!lib/**' | sort -u   # range, not version
rg -n 'evm_version|evmVersion|optimizer|via_ir|viaIR' foundry.toml hardhat.config.* 2>/dev/null
```

Four questions, all mandatory: which compiler build is pinned, does it carry a
known bug, does the EVM target match the chain that will actually run the code,
and does the deployed bytecode recompile from the source you are reviewing?
(§24.6.)

### 5.2 Triage: find the money before reading the math

Rank every in-scope contract by blast radius, highest first:

1. Anything that holds or moves assets (vault, escrow, bridge, pool, treasury).
2. Anything that mints, burns, or sets supply.
3. Upgrade / admin / role / pause authority.
4. Signature, permit, message validation.
5. Accounting (shares, rewards, debt, collateral, balances).
6. Oracle, pricing, TWAP, decimal conversion.
7. Arbitrary-call, delegatecall, batch, multicall, callback surfaces.
8. Governance, timelock, voting.
9. Pure view/helper code.

Audit rank 1 completely before reading rank 8. Most audits die of context
exhaustion spent on the wrong files; this ordering prevents it.

### 5.3 Get a runnable baseline

```bash
forge build
forge test --no-match-test "invariant" --summary     # unit baseline; record counts
forge test --match-path "test/invariant/*" -vv      # fuzz baseline; record run count
sui move test; cargo test -p sui-core                # Sui / node
```

Record exact pass/fail counts. A **committed failing test is an objective,
reproducible fact and one of the highest-value inputs you will get** — read
what it asserts before deciding it is stale.

If the build will not run, do not silently switch to reasoning-only. Say the
audit is degraded, and either fix the environment (see
`foundry-invariant-gap-audit` §0 for the four common bootstrap failures) or
pivot to Mode B diff review, which needs no build.

### 5.4 Mine the test suite for what it does *not* cover

The single cheapest high-yield step in this skill. Grep the tests:

```bash
rg -n '@skip|it.skip|describe.skip|test.skip|vm\.assume' test/ | head -50
rg -n '^\s*return;' -A2 test/ | rg -n 'assume|skip'        # early-return guards
rg -n 'assert(True|False)\(true\)|assert\(true\)|vm\.assume\(false\)' test/
rg -n 'deal\(|mintTo\(|vm\.prank' test/ | wc -l            # mock-heavy tests hide real flows
```

Then read the invariant **handlers** — the contract of allowed actions. Every
function the handlers never call is untested by construction, and untested
regions are where survivors live. Enumerate handler surface, compare against
the real entry-point census (§8), and write PoCs in the delta.

Also check whether the tests test the *real* token or a mock that behaves
differently (no fee-on-transfer, no rebasing, always returns true, 18 decimals).
A mock that always returns true silently invalidates every accounting assertion
that depends on transfer success.

### 5.5 Historical-bug mining

```bash
rg -ril 'audit|security|postmortem|vulnerability' --glob '!lib/**' --glob '*.md'
git log --oneline --all | rg -i 'fix|revert|sec|patch|audit|vuln'
rg -n 'TODO|FIXME|HACK|XXX|@audit|do not remove|for review' -g '*.sol'
rg -n 'disabled|reverted|bypass|legacy|old_|_v1|deprecated' -g '*.sol'
```

Three signals to chase: (a) a security fix in history tells you the class of
bug this codebase attracts — audit its siblings; (b) reviewer breadcrumbs
(`TODO`, commented-out checks) are near-bug locations; (c) a `lib/` fork that
differs from upstream is a deliberate modification and must be diffed
(§24.4). Check the repo's own published audits if present, and the protocol's
public incident history, before claiming novelty.

## 6. Phase 1 — Trust model

Produce these four artifacts before any hunting. If you cannot fill a table,
you do not yet understand the system.

### 6.1 Trust boundaries

List every boundary that code crosses: on-chain/off-chain, proxy/implementation,
vault/strategy, strategy/external protocol, bridge source/destination, factory
/instance, user-supplied address / hardcoded address, calldata / storage,
module / shared state, relayer / protocol, keeper / contract.

### 6.2 Asset and liability map

| Asset | Held by | Moved by | Backing invariant |
|---|---|---|---|
| e.g. USDC | `Vault` | deposit/withdraw/`Strategy.withdraw` | `sum(claims) <= redeemable` |

For every asset: who can move it, under what authorization, and which accounting
variable represents the claim against it.

### 6.3 Privilege matrix

For every privileged action, record the check *and* how the authority is
resolved:

| Action | Check | Authority resolved from | Can it escalate? | Timelock |
|---|---|---|---|---|
| `upgradeTo` | `onlyUpgrader` | `AccessControl` role, admin is `DEFAULT_ADMIN_ROLE` | yes, see §24.3 | none |

Never accept a role name as evidence. Resolve it: who holds it, who can grant
it, and who administers the role that grants it. Recurse until you reach an
account or a timelock. A role graph with a cycle or an unadministered root is
itself a finding.

### 6.4 State machines and call graph

For each subsystem write the lifecycle explicitly, e.g.

```
lock -> checkpoint -> slope/bias -> epoch -> transfer -> attach/vote -> detach -> withdraw
```

Then the call graph. State machines are where bugs hide: illegal transitions,
transitions reachable from an unintended state, and time/epoch boundaries.
Enumerate, for each edge, the precondition that guards it and whether that
precondition is *re-checked* on every path that reaches it.

### 6.5 Security assumptions

List each assumption and mark it tested / unverified / contradicted. Common
ones worth challenging every time: oracle honest and fresh; admin trusted; token
is a standard ERC20 (no fee-on-transfer, no rebasing, no callbacks, no
surprising `decimals`); strategy cannot reenter; validator set honest; keeper
is not adversarial; off-chain signer is not malicious or replayable; the code
being audited is the code that is deployed.

## 7. Phase 2 — Invariant ledger

Write invariants **before** hunting. For each: the expression, the state
variables it spans, and every transition that could break it. The ledger is
what makes the audit auditable — and it is the input to the PoC.

A useful invariant is *quantitative* and *falsifiable*. `onlyOwner` is not an
invariant. `mint(amount)` is callable only by a role that cannot be granted to
an arbitrary address is.

Catalog: §28.1. Domain-specific ones: §21.

Procedure:

1. For each subsystem, write 2-5 invariants. Prefer conservation laws.
2. For each invariant, list the transitions that could violate it (every
   external function that touches any variable in the expression, plus every
   callback and hook).
3. That pairing `(invariant, violating transition)` is your task list for §10.
   Do not skip to generic bug-class scanning before this list exists.

## 8. Phase 3 — Coverage ledger

This is the recall engine. Skipping it is why audits miss the obvious.

Enumerate the surface mechanically, then force a verdict per function.

```bash
# surface census
forge inspect <Contract> methods
forge inspect <Contract> storage-layout
slither . --print human-summary 2>/dev/null
rg -n '^\s*(function|public|external)' -g '*.sol' --glob '!lib/**' --glob '!test/**'
```

Build one row per external/public function:

| # | Location | Caller | Asset/state effect | Invariant touched | Status | Verdict |
|---|---|---|---|---|---|---|
| 1 | `Vault.sol:withdraw:212` | any | −assets(user) −shares(user) | I1 solvency | analyzed | candidate V-1 |

Allowed statuses, and **only** these:

- `analyzed-safe` — you traced it and it holds
- `analyzed-candidate` — feeds a candidate (§5, §10)
- `out-of-scope` — with a reason
- `unread` — **not acceptable for a finished audit**

Rules:

- The ledger is append-only evidence, not a mental note. Write it as a table in
  your working output.
- `unread` rows at delivery time ⇒ the report must be titled *partial audit*
  and must name what was not read.
- Reconcile the census against the invariant ledger: every invariant in §7 must
  have at least one function row referencing it, and every `analyzed-candidate`
  must reference a candidate id.
- Report the funnel in the summary: functions in scope / analyzed / candidates /
  confirmed / observations. This is how the user judges coverage, and it is
  falsifiable in a way "we found nothing" is not.

For large repos, batch the census: one file per row group, never more than ~10
files per reading pass, and never read a file you have not already entered in
the ledger.

## 9. Phase 4 — Multi-lens fan-out

Single-pass reading has a blind spot that is structural: once you form a model
of a function, you stop seeing new things in it. Independent passes with
*different questions* recover most of that loss. Use subagents when the host
provides them; otherwise simulate the passes sequentially with an explicit
"reset your model of this function" step between lenses.

### 9.1 The lenses

Dispatch each lens over the same file set, with a fresh context:

| Lens | Sole question | Typical yield |
|---|---|---|
| L1 Authority | who can reach this, and can any role escalate? | init/upgrade/role/allowlist bugs |
| L2 Accounting | does this preserve the conservation law, at every rounding? | share/reward/debt/fee bugs |
| L3 Control flow | what state is live during external calls and callbacks? | reentrancy, read-only reentrancy, ordering |
| L4 Messages | what exactly is signed, by whom, valid where, how many times? | permit/EIP-712/replay/cross-chain |
| L5 Economics | can a rational actor profit, and how much does it cost to try? | MEV, sandwich, oracle, rounding profit |
| L6 Lifecycle | is this transition legal from *every* reachable state and time? | epoch/timestamp/state-machine bugs |
| L7 Domain | which subsystem playbook (§21/§22/§23) applies, and what is its known bug list? | domain-specific |

Give each lens the coverage ledger rows for its files so it knows the surface,
but tell it to form its **own** model — do not hand it other lenses'
conclusions, or you have just serialized the pass you were trying to
parallelize.

### 9.2 Candidate return schema (mandatory)

```
id:            L1-3
location:      src/Vault.sol:withdraw:212
invariant:     I1  sum(user claims) <= redeemable assets
attacker:      arbitrary caller / privileged role / keeper / relayer / contract callback
preconditions: vault has >= 1 strategy; strategy paused=false; ...
call_path:     user -> Vault.withdraw -> Splitter._withdrawFromStrategy -> Strategy.withdraw
effect:        totalAssets() unchanged while shares burned   [state delta]
impact:        extract up to N of asset; repeatable; capital C
proof:         poc | trace | reasoning-only        <-- 'reasoning-only' caps confidence at Low
why_checks_fail: lossTolerance is checked against invested value, not against
                 realized withdrawal, and never after the transfer
confidence:    high | medium | low
```

`proof: reasoning-only` is not a finding. It is an observation. Enforce this.

### 9.3 Merge and dedup

1. Cluster candidates by `(invariant, primary location)`. Same bug found by
   three lenses is one finding with three confirmations.
2. Rank within a cluster: `poc` > `trace` > `reasoning-only`; then by
   quantified impact.
3. Count **every call site** of the same broken pattern before assigning
   severity. One vulnerable `withdraw` in a vault reachable by all users is a
   different severity from one in a test-only helper.
4. Feed the merged set into §12. A cluster whose members disagree about
   reachability is a `Question`, not a finding.
5. Report the funnel: raw candidates -> after merge -> after gate -> confirmed.

## 10. Phase 5 — Hypothesis generation

For each `(invariant, violating transition)` pair from §7, ask in this order.
Stop at the first "no".

1. **Reachability** — can an attacker actually invoke this path? Who is the
   caller, and can they be that caller? If it needs a role, can they obtain it
   (§6.3)?
2. **Attacker control** — which inputs are attacker-chosen? If every relevant
   input is validated and bound to storage, there is no bug here.
3. **Minimal precondition** — the smallest state that makes the path live.
   "A strategy must be registered and paused" is a precondition; "the protocol
   must be used as designed" is a hope.
4. **The delta** — exactly which variable moves by how much, in which direction,
   and who ends up holding the value.
5. **The profit or damage** — in the asset, not in "impact". If you cannot name
   a number, it is an observation.
6. **Repeatability and cost** — §26.

Then run the generic sweep (§17) and the tool sweep (§18) as a *separate* pass.
Generic sweeps find what your model does not expect; the model finds what
sweeps cannot see. Both are required; neither substitutes for the other.

## 11. Phase 6 — Validation and PoC

Build the smallest thing that fails if the vulnerability is real. Order of
preference:

1. **Executable Foundry test** (Solidity) — `test_exploit()` asserting an
   economic or state delta.
2. **Invariant/fuzz test with a handler** — best when the bug is a class, and
   when you want the search rather than the instance.
3. **Move unit test / Rust test** for Sui, node, and execution code.
4. **State-delta trace** — a numbered transaction sequence with before/after
   storage values, when execution is impossible. Acceptable, but `proof:
   trace` and it caps confidence at Medium unless every step is directly
   readable from the code.

A PoC is only strong if it asserts a *quantity*, not a revert. See §19 for
templates. A PoC that merely shows "the call does not revert" proves nothing.

**Before building a PoC from scratch, check whether someone already built it.**
For every bug class you suspect, find a real exploit of that class and adapt its
harness — you inherit the fork setup, the interfaces, and the assertion style,
and you stop guessing at the attack sequence. [DeFiHackLabs] (§29) holds thousands
of them, indexed by date and root cause:

```bash
forge test --contracts src/test/<YYYY-MM>/<NAME>_exp.sol -vvv   # reproduce a known class
# root-cause index: DeFiHackLabs-Incident-Explorer/rootcause_data.json
```

The taxonomy in that index is also a recall aid: *Logic Flaw, Business Logic
Flaw, Bad Reflection, Access Control, Unsafe Math, Faulty Oracle, Incorrect
dividends calculation, Arbitrary External Call, unverified external call, lack
of input validation, slippage protection, unsafecast, storage-slot collision,
invalid-signature-verification, weak randomness, unprotected function.* Walk it
against your invariants (§7) — each label is a hypothesis you have not yet
considered. Replicating a historical exploit is `proof: poc` for the *class*;
you still owe the instance-specific delta for this repo.

## 12. Phase 7 — False-positive gate

One gate. Answer all twelve. Any **no** in the hard set forces a downgrade.

| # | Question | Fail means |
|---|---|---|
| 1 | Is the vulnerable path actually reachable by the claimed actor? | discard |
| 2 | Is every precondition permissionless or realistically obtainable? | downgrade |
| 3 | Does the bad state survive every check on the path? | discard |
| 4 | Does an invariant, cap, timelock, pause, cooldown, or trusted role already prevent it? | discard |
| 5 | Does another contract absorb, correct, or cap the effect? | discard |
| 6 | Is the claimed effect *quantified* in an asset or a state delta? | observation |
| 7 | Is the result only theoretical (no execution, no trace, no number)? | observation |
| 8 | Does it work on the deployed execution model, not just isolated code? | discard |
| 9 | Can I reproduce it here, now, in this repo? | observation |
| 10 | Is the attacker economically motivated and able (§26)? | downgrade severity |
| 11 | Is the affected code actually deployed and reachable, not a test helper or orphan? | discard |
| 12 | Does the fix already exist in another layer? | discard |

Hard set: **1, 3, 8, 9, 11**. Any *no* ⇒ not a finding.

Never report:

- style, naming, or missing comments
- gas inefficiency, or gas as a vulnerability in itself
- centralization without a concrete escalation or asset path
- "could be dangerous" scenarios with no execution path
- third-party issues that cannot reach this system's state
- tool or compiler warnings with no demonstrated impact
- unreachable code paths
- assumptions that contradict the actual code
- a missing check whose absence you cannot tie to a state an attacker controls

## 13. Phase 8 — Self-adversarial review

Before a candidate becomes a finding, actively try to kill it. Write the
attempt down — a candidate you could not attack is usually a candidate you did
not understand.

- Which existing check did I read past? Re-read the modifier, the `require`,
  the override, the library function.
- Whose cooperation does my attack need? If it needs an honest admin, an honest
  keeper, or a non-compromised dependency, it is an observation.
- Is the attack atomic, or does it need a sandwich, a flash loan, a victim, a
  specific block, a specific oracle update, or a specific epoch boundary?
- Is it economically viable after gas, fees, and slippage?
- Is there a cap, a timelock, a delay, a nonce, a rate limit, or a pause that
  makes the window too small?
- Does the "vulnerability" only manifest if the protocol is used in a way its
  own docs forbid?
- Would a well-known firm have already found this? If the code is
  battle-tested, that is prior evidence against — raise your bar and look for
  the *new* thing: the diff, the integration, the novel composition.
- Does the impact survive the *real* numbers — with real TVL, real liquidity,
  real thresholds?

If you cannot attack it, the honest classification is `Likely` or `Likely
(observation)`. Say which.

## 14. Phase 9 — Fix and negative control

Every recommended fix must come with a **negative control**: the same PoC, run
against the patched code, must fail. State this explicitly.

```
PoC on unpatched: PASS (attacker profit 12.4e18)
PoC on patched:   FAIL (reverts: "insufficient assets"; profit 0)
```

Without the negative control, a fix is a suggestion, not a remediation. It also
catches the two most common bad fixes: a fix that stops the exploit but bricks
legitimate use, and a fix that moves the check without actually binding the
value.

Fix guidance:

- The smallest change that closes the invariant violation, in the place where
  the value is bound — not at every call site.
- Bind checks to the *economic* value, not to a re-derived or recomputed copy
  of it.
- Prefer `nonReentrant`/pull over push for external calls, but only if it does
  not break the accounting; a reentrancy guard bolted onto a vault that was
  already read-only-reentrant needs the read fixed too.
- When proposing an architectural change, say so explicitly and give the
  smallest version of it that is safe.
- Check the fix against every other call site of the same pattern (§9.3).

## 15. Phase 10 — Severity and confidence

### 15.1 Capital at risk (required before assigning severity)

State, per finding: the asset, the amount reachable in the deployed state, the
fraction of protocol TVL or of the affected pool, who bears the loss, and
whether the loss is recoverable. A finding without this is an observation.

### 15.2 Severity

| Severity | Criteria |
|---|---|
| Critical | Direct, permissionless, protocol-wide loss of funds; unrestricted upgrade or initialization takeover; forged mint/release; arbitrary asset extraction with no capital at risk. |
| High | Large or meaningful loss of user/protocol funds; privilege escalation to a fund-moving role; permanent takeover of a critical component; broken accounting that yields reliable profit; permanent or protocol-wide freeze. |
| Medium | Constrained loss; griefing with a bounded cost; partial freeze; a broken invariant that needs a non-trivial precondition. |
| Low | Limited impact; a hard-to-reach edge case; a missing defense-in-depth check with no current exploit. |
| Info | No security impact. Observation, hardening, or documentation. |

Adjust by exploitability: permissionless and cheap ⇒ as stated. Requires a
privileged or compromised role ⇒ cap at High unless the role is
plausibly reachable. Requires enormous capital ⇒ cap one level and say why.
Not profitable after gas ⇒ do not report above Low.

**HIGH+ mode** (when the user asks for Critical/High only): report only those
two, and route everything else to an explicit "below the requested threshold"
list so nothing is silently dropped.

### 15.3 Confidence

| Level | Criteria |
|---|---|
| High | Executed PoC in this repo, or a mechanically proven chain with all values traced. |
| Medium | Complete code path, but the PoC is partial, or exactly one assumption is unverified. |
| Low | Plausible path, missing key evidence, privileged actor required, or economic viability unclear. |

`Low` items are observations, not findings, unless the user explicitly asked for
a hypothesis list. Any unverified assumption caps at Medium. `proof:
reasoning-only` caps at Low.

### 15.4 Classification — OWASP Smart Contract Top 10:2026

Tag every reported finding with its OWASP SC category (§29). It gives the report
a citable, current vocabulary and it corrects a weighting error older audit
methodologies carry: **reentrancy is not the main event any more.**

| # | Category | 2025 signal |
|---|---|---|
| SC01 | Access Control | highest dollar loss, ~30 incidents |
| SC02 | Business Logic | highest *incident count*, ~58 — exploit logic, not syntax |
| SC03 | Price Oracle Manipulation | includes spot-price and stale-round reads |
| SC04 | Flash Loan–Facilitated | an amplifier, not a root cause — name the root cause too |
| SC05 | Lack of Input Validation | bounds, casting, unvalidated callbacks |
| SC06 | Unchecked External Calls | return values ignored, swallowed failures |
| SC07 | Arithmetic Errors | **not** overflow: wrong rounding direction, divide-before-multiply, precision loss. Stays in range, compiles clean, still wrong |
| SC08 | Reentrancy | **demoted to #8.** Now mostly *cross-contract* and *read-only*, not same-function |
| SC09 | Integer Overflow/Underflow | largest dollar total in 2025 (~29% of all losses) — but almost entirely **non-EVM** (Cetus, §22.10). Solidity 0.8 closed the EVM half |
| SC10 | Proxy & Upgradeability | newer category; uninitialized implementations, layout breaks |

Ordering consequence for your sweep: **access control** and **business logic**
outrank reentrancy, which is why §9 already runs Authority (L1) and Accounting
(L2) before Control flow (L3). Keep that order, and add SC07 rounding as an
explicit pass over every share/debt/reward formula — it is the category
Solidity's checked arithmetic does *nothing* to prevent. SC07 and SC09 were
split apart in 2026 precisely because both produced large 2025 losses with
different root causes.

Where a finding spans two, tag the root cause, not the amplifier.

## 16. Reporting

### 16.1 Structure

```
[SEVERITY] Title that states the consequence, not the category
  "Unrestricted upgrade via uninitialized implementation drains all vault funds"
  not "Reentrancy in withdraw"

Location
  src/Vault.sol:withdraw:212-247
  (list every call site of the same pattern)

Severity: High          Confidence: High
OWASP: SC01:2026 (Access Control)
Capital at risk: 1,240,000 USDC (62% of vault TVL at commit <sha>)

Summary
  One paragraph: what an attacker does and what they get.

Root cause
  The specific invariant or authorization failure. Name the invariant id from §7.

Attack path
  1. attacker calls Vault.withdraw(0, ...) with strategy paused
  2. Splitter routes to Strategy.withdraw which transfers to msg.sender
  3. totalAssets() is not reduced, so the share price rises
  4. attacker redeems the inflated shares
  Net: +X assets, repeatable, atomic, capital 0

Proof of concept
  test/test_VaultExploit.t.sol:test_withdraw_drains_via_stale_totalAssets
  forge test --mt test_withdraw_drains_via_stale_totalAssets -vvv
  [output showing the balance delta]
  Negative control: same test on the patched Vault reverts at line 88.

Why existing checks fail
  Name the checks that look protective and show precisely why they do not
  stop this. Do not skip this section; it is the reader's fastest route to
  trusting the finding.

Fix
  Smallest change, in the place the value is bound, plus the negative control.

Assumptions / unknowns
  Every assumption, marked verified or not.
```

### 16.2 Discipline

- No hedged non-claims. Never write "this may allow theft", "an attacker could
  potentially", or "likely exploitable" without immediately following with the
  exact condition and the evidence.
- Prefer: *"An unprivileged caller can invoke `X` with `Y`. Because `Z` is not
  bound to `owner`, the caller moves `N` from `A` to `B`. The included test
  reproduces the delta."*
- Keep the report in five separate buckets: **Confirmed findings** /
  **Observations** / **Questions for the team** / **Below threshold** (with a
  one-line reason each) / **Coverage & limitations** (the §8 funnel, plus
  anything unread, missing, or unverifiable).
- Never bury a Medium under a High, and never let an unproven candidate inflate
  the count. If you have three confirmed and forty killed candidates, say so —
  it is evidence of rigor.
- If you found nothing, deliver the coverage table and the invariants. "No
  issues found" is a weak deliverable; "these 42 functions were analyzed against
  these 9 invariants, here is the table" is a defensible one.

## 17. Grep sweep

Run these over production sources (`--glob '!lib/**' --glob '!test/**'`) as a
separate pass from invariant work. Each pattern is a *pointer*, never a finding.

```bash
S="--glob !lib/** --glob !test/** --glob !script/** -n"

rg $S '\bdelegatecall\b|\.functionCall|multicall\(|batch\(' *.sol **/*.sol      # arbitrary control flow
rg $S '\btx\.origin\b'                                                            # auth confusion
rg $S 'block\.(timestamp|number|hash)|getBlockTimestamp|blockhash\('               # time assumptions
rg $S '\.(call|staticcall|send)\s*(\{|\()' | rg -v '\.(call|staticcall)\{'        # unchecked low-level
rg $S 'selfdestruct|suicide'                                                      # pre-1559 assumptions
rg $S 'ecrecover|ECDSA\.|SignatureChecker|recover\(|toEthSignedMessageHash'       # signature surfaces
rg $S 'permit\(|DOMAIN_SEPARATOR|nonces\[|EIP712|typehash'                       # signed-message surfaces
rg $S 'nonReentrant|whenNotPaused|onlyOwner|onlyRole|requiresAuth'               # which guards exist
rg $S 'unchecked\s*\{'                                                            # unchecked arithmetic
rg $S 'assembly\s*\{|type\(uint|int256\(uint|uint256\(int'                       # low-level and cast risk
rg $S '1e18|10\*\*18|1e6|1e27|WAD|RAY|RAY_RATIO|PRECISION|decimals\('            # decimal and precision
rg $S '\.transfer\(|\.approve\(' | rg -v 'SafeERC20|safeTransfer|safeApprove'      # unsafe ERC20 usage
rg $S 'onERC721Received|onERC1155Received|transferAndCall|tokenReceived|callback' # callback entry points
rg $S 'latestRoundData|updatedAt|answerRoundId|staleHeartbeat|sequencer|price0|getAmountOut|consult'  # oracle surfaces
rg $S 'flashLoan|flashSwap|onFlashLoan|callback'                                 # atomic-manipulation surfaces
rg $S 'rewardPerToken|earned|rewardDebt|accReward|unclaimed|checkpoint|totalSupply' # reward accounting
rg $S 'INITIALIZABLE_STORAGE|constant\s+\w*_STORAGE|ERC7201|keccak256\(abi\.encode'  # storage-slot collisions
rg $S 'constructor\s*\(' -A3 | rg -v '_disableInitializers|_getInitializableStorage'  # SCWE-092 candidates
rg $S 'DEFAULT_ADMIN_ROLE|_setRoleAdmin|grantRole|renounceRole|renounceOwnership'   # role graph
rg $S 'CREATE2|create2|new .*salt|computeAddress'                                # deterministic deployment
rg $S 'oracle|rewardsStrategy|setPool|setController' -n                           # mutable trust roots
```

Then the questions that turn pointers into findings:

- **Contracts with external calls but no `nonReentrant`** — cross-reference the
  guard list against the call list; the set difference is your reentrancy
  candidate list.
- **`External *` parameters that reach `transfer`/`approve`/`call` without a
  whitelist binding** — the set difference is your unauthorized-asset-movement
  candidate list.
- **Every `/` followed later in the expression by a `*`** (or vice versa) in a
  share/debt/reward/fee formula — order of operations is SC07 arithmetic, and
  checked arithmetic will not flag it. List each site with its rounding
  direction and who it favours.

## 18. Tool cheatsheet

Tools generate candidates. You prove them. Never report a tool finding you have
not traced.

```bash
# build + storage
forge build --build-info                      # needed for storage-layout diffs
forge inspect <C> storage-layout > /tmp/old.json
forge inspect <C> abi > /tmp/old.json
forge inspect <C> methods

# upgrade diff (see §20)
forge inspect <C> storage-layout --pretty

# static analysis, one detector at a time when chasing a class
slither . --print human-summary
slither . --detect reentrancy-eth,uninitialized-storage,suicidal,controlled-delegatecall
slither-check-upgradeability <path/to/impl.sol> <ContractName> --proxy <Proxy>
semgrep --config=slither sol/ --severity high

# tests + profiling
forge test -vvv --gas-report
forge coverage --report summary
forge test --match-path "test/invariant/*" -vv

# fork reality checks (does the deployed system actually have this state?)
cast call <addr> "totalAssets()(uint256)" --rpc-url $RPC --block <n>
cast code <addr> --rpc-url $RPC | head -c 200
cast storage <addr> <slot> --rpc-url $RPC

# Sui
sui move test; sui move build --dump-bytecode-as-base64
cargo test -p sui-types; cargo clippy -p sui-core
```

Fork checks are underused and high value: before claiming "vault has zero
strategies so the bug is live", read the actual deployed state at the actual
block. Half of "critical" findings evaporate here, and the other half get
quantified.

## 19. PoC recipes

### 19.1 Economic assertion (the default shape)

```solidity
function test_withdraw_inflates_share_price() public {
    // arrange
    uint256 vBefore = vault.totalAssets();
    uint256 aBefore = asset.balanceOf(address(this));
    uint256 sBefore = vault.balanceOf(address(this));

    // act — exactly the attack sequence
    vault.deposit(1000e18, address(this));
    vm.prank(attacker);
    vault.withdraw(0, address(attacker), address(attacker));

    // assert a QUANTITY, and the direction that matters
    assertEq(vault.totalAssets(), vBefore, "totalAssets must fall with withdrawal");
    assertLe(asset.balanceOf(address(attacker)), aBefore + 1 wei, "no profit");
    // and the invariant that was supposed to hold:
    assertLe(vault.convertToAssets(vault.balanceOf(address(this))), sBefore);
}
```

Never `assertGt(attackerAfter, before)` alone — that is satisfied by any
rounding, and it makes the PoC unfalsifiable. Assert the *invariant* and the
*profit*, then state in the report the magnitude and whether it repeats.

### 19.2 Invariant/fuzz test with a handler (for bug classes)

```solidity
contract Handler {
    IVault vault; IERC20 asset; address actor;
    uint256 constant SEED = 1e18;

    function deposit(uint256 amt) public {
        amt = bound(amt, 0, 100e18);
        asset.prank(actor); asset.approve(address(vault), amt);
        vault.deposit(amt, actor);
    }
    function withdraw(uint256 shares) public {
        vault.redeem(bound(shares, 0, vault.maxRedeem(actor)), actor, actor);
    }
    function donate(uint256 amt) public {            // attacker-influenced share price
        amt = bound(amt, SEED, 1_000_000e18);
        asset.prank(actor); asset.transfer(address(vault), amt);
    }
    function warp(uint256 t) public { vault.warp(bound(t, block.timestamp, block.timestamp + 30 days)); }
}

function invariant_solvency() public view {
    // handler surface must cover the real entry points; every gap is an untested bug
    assertLe(vault.convertToAssets(vault.totalSupply()), vault.totalAssets() + 1);
}
```

Always include the attacker's own actions (`donate`, `warp`, `feeOnTransfer`)
as handlers. An invariant suite without an attacker-action handler proves
nothing about adversarial states.

### 19.3 Negative control

```bash
# 1. prove it on the current tree
forge test --mt test_exploit -vvv
# 2. apply the proposed fix to a scratch branch
# 3. prove the SAME test now fails
forge test --mt test_exploit -vvv   # expect revert, and expect the revert reason to be the fix
```

Report both outputs. This is the difference between a remediation and a wish.

### 19.4 State-delta trace (non-Solidity / non-runnable)

Number the transactions, and for each give the exact storage writes:

```
T1  attacker.call(Vault.withdraw{value:0}(0, A, A))
    Vault.totalAssets:      1_000_000e18 -> 1_000_000e18   (unchanged  <-- the bug)
    Strategy.paused:        false -> false
    Strategy.totalDebt:     500_000e18 -> 0
    attacker.asset.balance: 0 -> 0
T2  attacker.call(Vault.redeem(2_000e18, attacker, attacker))
    Vault.totalSupply:      20_000e18 -> 18_000e18
    attacker.asset.balance: 0 -> 200e18
Net attacker profit: 200e18 of asset, capital 0, atomic in one block
```

Cite `File:Function:line` for every state write. If a value cannot be traced to
a line, the finding is `proof: trace` at best.

## 20. Upgrades and storage layout

Run this whenever the diff touches an implementation, proxy, beacon, `init*`,
or upgrade authority. Most upgrade incidents are layout or authority, not logic.

### 20.1 Layout diff

```bash
forge build --build-info
forge inspect <Contract> storage-layout --pretty > new.txt   # on the patched tree
# compare against the base commit
git stash            # or: git worktree add ../base <base-sha>
(cd ../base && forge inspect <Contract> storage-layout --pretty > ../old.txt)
diff -u ../old.txt new.txt
```

For hardhat: `hardhat compile` then compare `artifacts/build-info/*.json`
storageLayout entries, or use `@openzeppelin/hardhat-upgrades` validation.

Read the diff for:

- **reordered / removed / renamed variables** — any of these silently reinterprets
  storage. Removing a variable *without* keeping the slot (or shrinking a `uint`
  and reusing the freed space) is Critical on the next upgrade.
- **type widening or narrowing** — `uint128` -> `uint256` shifts every
  subsequent variable by one slot.
- **inheritance order changes** — reordering base contracts reorders storage.
- **gap consumption** — the new implementation's `__gap` must be at least as
  large as the base's, and the total layout must be unchanged.
- **packed-slot changes** — reordering two `uint128`s in one slot is silent
  corruption, not a compile error.
- **mapping/array changes** — `[]` -> `[5]` changes the encoding of subsequent
  writes; `mapping` -> array breaks all lookups.
- **enum/struct changes** — enum width and struct member reordering.

### 20.2 Authorization migration

- Where do the admin/role lives? In the **proxy** storage, or in the
  implementation? An upgrade that changes *who* `onlyOwner` resolves to has
  changed the trust model, even if the code compiles.
- Is the new implementation's `_authorizeUpgrade` actually access-controlled,
  and does the new version of `owner()`/`admin()` still read the slot the proxy
  populated?
- Can the old admin still act? Can the new admin be a contract that no one
  controls, or `address(0)`?
- Does the upgrade path bypass, shrink, or skip the timelock? Is there a second,
  ungated upgrade route (`_setImplementation`, direct storage write, beacon
  swap)?
- Transparent proxy: can the admin accidentally hit an implementation function,
  and can an attacker force the proxy into "admin called" mode?

### 20.3 Upgrade simulation

Walk it explicitly and check each step:

```
old impl -> new impl -> existing proxy storage -> first privileged call -> first user action
```

At the "first privileged call" step, verify the *caller* of that call still
satisfies the new implementation's authorization. A very common real bug: the
new implementation renamed a modifier from `onlyAdmin` to `onlyRole(ADMIN)`
where `ADMIN`'s admin is now a different role, and the old admin silently loses
control — or a new role is granted to the deployer.

At the "first user action" step, verify the *old* stored values are still
interpreted the same way. Balances, allowances, debt, epochs, flags, addresses,
and packed fields are all candidates.

### 20.4 Implementation hygiene

- `constructor() { _disableInitializers(); }` on **every** upgradeable
  implementation (SCWE-092). This is the single most common upgradeable-contract
  finding, and it is Critical when the implementation holds authority.
  Verify it is present, not merely that `Initializable` is inherited.
- Constructor arguments that the proxy path will not receive — remember the
  constructor runs once, in the *implementation*, not the proxy.
- `selfdestruct` or `delegatecall` in an implementation breaks every future
  upgrade. Search explicitly.
- `immutable` in an implementation is a proxy footgun: it is set from the
  implementation's constructor and shared by all proxies pointing at it.
- Newly added `initializer`-like external functions that can be called
  post-upgrade.
- Interface-compatible but behaviour-changing: a new implementation may
  reinterpret a return value, change decimals, change the asset, or change
  which controller is authoritative. Interface checks prove nothing.

## 21. EVM domain playbooks

### 21.1 Initialization and roles

- Can the implementation be initialized directly? (See §20.4.)
- Can proxy initialization be front-run? Who deploys the proxy — a factory, a
  script, a deployer EOA?
- Is `init` idempotent? Can a second `init` be reached through a different
  function that writes the same slot?
- `DEFAULT_ADMIN_ROLE` is the admin of every role unless `_setRoleAdmin` says
  otherwise. Trace the full chain `DEFAULT_ADMIN_ROLE -> X -> Y` and confirm no
  node can escalate upward. A factory that passes `msg.sender` as
  `DEFAULT_ADMIN_ROLE` hands every creator a path to `UPGRADER_ROLE`.
- Role `admin` can be `address(0)`, which freezes administration permanently —
  a liveness finding.
- `renounceRole` / `renounceOwnership` with no two-step transfer strands
  ownership; with a two-step transfer, verify the pending owner cannot be
  griefed and the nomination cannot be overwritten by a third party.
- Custom `access control` (OZ v5 `AccessManager` / `AccessControlDefaultAdminRules`)
  behaves differently from v4 `AccessControl`. Check the major version and read
  the actual source, not the doc for the version you remember.

### 21.2 ERC-4626 vaults

Invariants: `sum(user economic claims) <= redeemable assets + authorized
liabilities`; `convertToAssets(convertToShares(x)) <= x` (protocol-favouring
rounding); `totalAssets() == idle + invested - recognized losses - fees`.

Check all of: `deposit/mint/withdraw/redeem`, all four `preview*` functions,
all four `max*` functions, and that `preview*` agrees with execution.

Attack classes, in rough order of historical yield:

- **Inflation / first-depositor** — empty or near-empty vault, attacker seeds a
  minimal amount, donates to move the price, victim rounds to zero. Check
  whether `_decimalsOffset` / virtual shares are actually set, and whether the
  vault can ever be emptied to zero supply. A *donation* attack is different
  from a *share-price* attack; check both.
- **Stale `totalAssets`** — `totalAssets()` reads a cached value, or ignores
  idle balance, or double-counts a strategy's reported value against its
  on-chain balance.
- **Fee double counting** — the same performance fee charged in `preview*` and
  again in the settlement path, or charged on assets that were never withdrawn.
- **Withdrawal caps / queue bypass** — `maxWithdraw`/`maxRedeem` are advisory;
  find the path that ignores them, or the second withdrawal of the same request.
- **Receiver/owner mismatch** — the `receiver` argument is validated but the
  `owner` is spent from, or `caller` is confused with `owner`.
- **Zero-share / zero-asset calls** — `deposit(0)`, `redeem(0)`, `mint(0)`;
  guards that `return` early instead of reverting can leave accounting
  half-applied.
- **Unusual `decimals`**, fee-on-transfer assets, rebasing assets: an ERC-4626
  vault that assumes 18 decimals or exact transfers.

### 21.3 Strategy, splitter, insurance, loss

Flow: `vault -> splitter -> strategy -> pool -> rewards -> strategy ->
splitter -> vault`.

- **Registration** — asset equality, splitter equality, controller equality,
  proxy validity, uniqueness, timelock, capacity, paused state, and whether
  "registered" matches the strategy's *actual* state. A registered strategy
  that was upgraded into something else is a trust break.
- **Fund flow** — can a strategy keep funds it should have returned? Can it send
  them anywhere but the splitter? Can the operator make arbitrary external
  calls? Are reward tokens trusted, and can a reward token be confused with the
  underlying?
- **Accounting** — compare, per asset: on-chain balance, reported invested
  value, expected withdrawal, actual withdrawal, reported USD, price used.
  For every conversion check decimals, staleness, zero price, multiply-before-
  divide, precision loss, and denominator consistency.
- **Loss control** — for any `lossTolerance`: what exactly is the denominator,
  what is the reference balance, which direction is the comparison, what is the
  attacker's maximum influence on the baseline, and can the same loss be claimed
  more than once? A loss tolerance measured against *invested* value while the
  attacker can move assets out first is a drain, not a tolerance.
- **Rebalance** — the `lowest APR -> withdraw -> compute loss -> claim insurance
  -> highest APR -> invest` sequence. Attacker influence over APR, stale APR
  history, sort instability, zero-capacity or paused strategies being selected,
  partial withdrawals, tolerance amplification, and reentrancy across strategy
  callbacks.
- **Insurance as a subsystem** — who initializes it; are the vault and asset
  bindings immutable in practice; can the vault move arbitrary amounts; can
  compensation exceed measured loss; can loss be manufactured; can one loss be
  claimed repeatedly; can a strategy cause loss and later recover while
  insurance stays drained; are fees and compensation double-counted; can
  insurance be swapped or detached.
- **Emergency exit** — deserves the same authorization analysis as a normal
  path. Prove the intended operator, the destination, that the destination is
  not attacker-controlled, that loss checks are not bypassed in a way that
  triggers unjustified payouts, and that "emergency" cannot become a permanent
  theft primitive.

### 21.4 Staking, rewards, and emissions

Invariants: `total rewards paid == sum(accounted) + unclaimed`; `rewardPerToken`
is monotone non-decreasing; `unclaimed(a) == earned(a) - claimed(a)`; a user
can never claim twice for the same period.

Check: `rewardPerToken` update ordering (it must be written *before* the
balance change, or the staker loses a block of rewards); `rewardPerTokenStored`
vs last-update-time bookkeeping; division-before-multiplication truncation
that systematically favours the protocol and is *amplified* by small stakes;
`rewardDebt`/`userRewardPerTokenPaid` reset on stake/unstake; cliff handling;
boost/boosted-ratio math; fee-on-transfer of the reward token on claim;
`earned()` vs the actual `getReward()` amount (they must agree); emergency
withdrawal skipping the accounting update; multiple staking entry points
updating the same accumulator; and a `notifyRewardAmount` that raises
`rewardRate` without correcting for the elapsed-but-unaccounted period.

### 21.5 AMMs and concentrated liquidity

Check: constant-product `getAmountOut` rounding direction and its use of
`reserve0/reserve1` ordering; fee application order (fee on input before the
curve, not after); `k` monotonicity; spot price vs execution price; TWAP
manipulability inside a single transaction; liquidity add/remove at
non-current ticks; `slot0`/tick bitmap boundary errors; swap crossing
multiple ticks with a wrong cumulative-fee delta; `liquidityNet` sign
convention; fee growth rounding (`feeGrowthGlobal` truncating so that LPs
lose dust forever, and whether the residual is recoverable); limit-order /
partial-fill settlement; oracle `observe` over a manipulated range; and
protocol-fee accrual that can be claimed twice.

### 21.6 Lending and liquidation

Check: collateral factor and liquidation threshold monotonicity; health factor
at the exact boundary (rounding decides who gets liquidated); liquidation
close-factor and reserve factor; whether `liquidate` can be front-run into a
bad state; seize-token decimals; bad-debt socialization and who absorbs it;
interest accrual and index rounding; oracle staleness vs liquidation delay;
`repayBorrow` / `borrow` with a stale-but-updated index; borrow caps per
collateral; and whether a paused market still allows repayment.

### 21.7 Permits, signatures, and EIP-712

For every signature-accepting path, answer all six:

1. **What** is hashed — is it the full domain separator plus the struct hash,
   including nonce and deadline?
2. **Who** signed it — is the recovered address compared to a *storage* owner,
   and is the zero address rejected (`ecrecover` returns `0` on failure, and
   `address(0)` must never be a valid owner)?
3. **Where** it is valid — `chainId` in the domain, or a contract address that
   can be replayed on a fork/chain-clone with the same signer?
4. **How many times** — nonce consumption before the external call, and nonce
   consumed on *failure* paths (which strands funds or re-enables replay).
5. **Which form** — 65-byte vs 64-byte EIP-2098 compact; malleability matters
   if replay protection marks the *signature* as used instead of the message or
   nonce (CVE-2022-35961, §24).
6. **What else can be swapped** — permit for an arbitrary `spender`, a
   `value`/`deadline` that is not in the signed struct, `v/r/s` reuse across
   different function selectors, and `permit` on a token whose
   `nonces` mapping is not incremented (DAI-style permit requires a separate
   `permit(holder, spender, nonce, expiry, allowed, v, r, s)` with an
   off-by-one nonce domain — an off-by-one there is a replay).

Also: gasless approvals (`permit2`/universal router), signature verification in a
loop (see the ERC-165 DoS in §24), and whether the relayer is trusted with the
spender choice.

**Permit2 specifically.** Permit2 is two contracts with two *different* nonce
models, and conflating them is the common finding. Establish which path an
integrator uses:

- `AllowanceTransfer` — a **standing** per `(owner, token, spender)` allowance
  with its own packed amount/expiration/incrementing nonce, sitting inside the
  Permit2 contract. It survives the transaction. Check the expiration is actually
  set, that `lockdown`/revoke paths exist, and that a stale allowance does not
  outlive its purpose. Note the token-approval prerequisite: users must give
  Permit2 itself an ERC-20 approval, usually infinite — so a Permit2 exposure is
  at least as wide as that approval.
- `SignatureTransfer` — a **one-shot**, unordered bitmap nonce bound to a single
  recipient and amount; permission dies with the transaction.

Common bugs: the EIP-712 **witness** (or any extra signed field) not bound into
the struct, so a signature can be re-pointed at a different action; a deadline
checked outside the hashed struct; nonce consumed only on the success path; and
the classic permit front-running race between `approve` and `permit`.

### 21.8 Tokens and non-standard ERC-20s

Assume nothing about a token. Verify: `decimals()` is what the accounting
assumes; `transfer` returns a bool (or use SafeERC20); no fee-on-transfer
(the code that credits the nominal amount instead of the received amount is
wrong); no rebasing (both positive and negative); no callback hooks
(`ERC777` `tokensToSend`/`tokensReceived`, `ERC1363`); `approve` race (does the
code require zero-first?); no transfer restrictions (blacklists, pausable
transfers, `onlyWhitelisted`); no balance limits; no `permit` surprises. For
ERC-721/1155: `onERC721Received` reentrancy in `safeTransferFrom` loops,
`_beforeTokenTransfer` hooks, ERC1155 `ids` array length vs `amounts` length
mismatch, and `ERC721Consecutive` batch-balance accounting.

### 21.9 Oracles

Check: staleness bound actually enforced against `updatedAt`, not just
`block.timestamp`; the sequencer-uptime feed is consulted on L2 (a stale L1
answer after a sequencer outage is the classic post-incident finding); chainlink
`answerRoundId` / round completeness; decimals and sign conventions between
feeds; whether the same price source is used for both the action and the
valuation (a pool reading its own spot price is not an oracle); TWAP window and
whether it can be moved inside one transaction; a spot price that a flash loan
can move; zero or negative price handling; and whether a revert in the oracle
call bricks withdrawals (a liveness finding that is usually High).

### 21.10 Governance and timelock

For each privileged action, verify the `propose -> delay -> execute` chain:
the delay cannot be shortened; the operation hash cannot be swapped; a stale or
overwritten announcement cannot be replayed; a cancelled operation is really
cancelled; two different operation types cannot be confused; the timestamp
cannot be misused; and a proposer cannot be its own executor when the protocol
requires separation. Check `quorum` changes against already-defeated proposals
(§24), and whether `GovernorSettings`/`GovernorCountingSimple` overrides were
preserved.

### 21.11 Multicall, batch, and account abstraction

`multicall`/`batch` re-enters the same contract with `msg.sender` unchanged,
which defeats per-call intent checks, reentrancy guards keyed on a per-call
flag, and nonce-per-call schemes. Check ERC-4337: `validateUserOp` reentrancy
(entry point reentry), missing `sender` checks, `signature` malleability in the
userOp, dead-gas / no-op operations to grief the bundler, storage collision
across the account and its modules, and paymaster-validity windows.

### 21.12 NFTs and vote-escrow NFTs

Flow: `lock -> checkpoint -> slope/bias -> epoch -> transfer -> attach/vote ->
detach -> withdraw`.

Check: lock-time monotonicity (extending a lock must not reset decay); maximum
lock enforcement; checkpoint correctness at `block.timestamp` boundaries; slope
change propagation to every lock; `merge`/`split` conservation (assert the
equations explicitly in a test); approval semantics vs attachment restrictions;
transfer whitelists (a whitelisted contract that can move another user's NFT,
bypass attachment, create temporary ownership, or swing voting power inside one
transaction); flash-transfer; and delegated operator scope. For `veNFT`,
supply conservation across mint/burn/split/merge and the decay integral.

### 21.13 Uniswap v4 and hook-based AMMs

v4 changes the substrate: a singleton `PoolManager`, **flash accounting**
(deltas accrue in transient storage and settle only at the end of the
transaction, which makes every intra-tx flash loan free), ERC-6909 claim
balances, and **hooks** — arbitrary code invoked at `before/afterSwap`,
`before/afterAddLiquidity`, `before/afterRemoveLiquidity`,
`before/afterDonate`, `before/afterInitialize`.

Invariants: `settled balances == zero at end of transaction`;
`sum of claims <= assets held`; every value a hook returns is *accounted*,
not merely observed.

Attack classes:

- **Permission bitmap is the hook's address.** The `PoolManager` derives
  permissions from the low bits of the deployed hook address, so the address is
  the capability. A deployment whose actual address encodes *different* flags
  than the code assumes gives you a hook with the wrong powers — including
  unwanted `*ReturnDelta` rights. Recompute the expected bitmap from the
  deployed address and diff it against `getHookPermissions()`; treat salt
  grinding as a deployment-pipeline check, not a code review detail.
- **Hook return data overruns the scratch space.** `beforeSwap` returns
  `(bytes4, BeforeSwapDelta, uint24)` — 96 bytes ABI-encoded, past the 64-byte
  scratch region. A hook (or a hand-rolled dispatcher) that writes there corrupts
  the free-memory pointer. Verify any assembly near a hook return.
- **Delta returns as extraction.** `beforeSwapReturnDelta` / `afterSwapReturnDelta`
  let a hook take or give value. An unbalanced return is a direct fund-movement
  path; check that every delta the hook returns is backed by a transfer it made.
- **Reentrancy in three directions.** Any external call from a hook reopens the
  whole environment: back into the *same* pool, into a *different* pool routed
  through the *same* hook (shared storage, shared assumptions), and — because v4
  supports native ETH — straight back into the `PoolManager` or the hook itself
  during a transfer. NonReentrant guards keyed to one pool do not cover the
  other two.
- **Flash accounting confusion.** Balances read mid-transaction are unsettled
  and may be *negative*. Code that reads pool state between `unlock` and settle
  and treats it as final will misprice. Never assume an intermediate read is an
  end-of-tx truth.
- **Custom-curve math.** Hooks that zero out or replace the pool's amount
  (`BaseCustomAccounting`-style) must bound their own curve; accounting drift
  here is the Bunni-class failure.
- **Hook mutability after init.** The pool records its hook at initialization,
  but the hook contract may itself be upgradeable — so the pool's behavior can
  change under existing LPs. Ask which contracts in the chain are upgradeable
  and whether hook upgrades are timelocked and observable.
- **Non-standard tokens.** Fee-on-transfer, rebasing, ERC-777 callbacks,
  pausable/blacklist, and unusual decimals all break the assumptions the standard
  hook examples are written against. Re-check accounting under each.
- **ERC-6909 allowances.** Standing claims inside the `PoolManager carry their
  own allowance/infinite-flag semantics — audit them like ERC-20 approvals.

## 22. Move / Sui playbooks

Do not import EVM mental models. Sui is object-centric: authority is *possession
of an object*, not a storage check. The whole model is the threat model.

### 22.1 Capability and resource analysis

Enumerate every capability, admin cap, witness, and `key`/`seal` authority. For
each: who creates it, who can copy it (a `copy` on a capability is a critical
finding), who destroys it, whether it can be wrapped or unwrapped, and whether
it can be transferred to an untrusted address. Then ask: can an attacker
manufacture a *structurally identical* type via a different module, satisfy a
generic function with an unintended capability, or obtain a second authority
object from a constructor.

`public struct` with a public field is not an authority unless a signature
consumes it. `public(friend)` widens the authority set to a module list — audit
that list like an allowlist.

### 22.2 Object rules

For every mutable object: owner, who may mutate, can it become `shared`, can it
be transferred while logically attached or while borrowed, can the same object
appear twice in one transaction, and are object-id/version assumptions enforced
(the runtime gives you versioned reads — using a stale or mismatched version is
a real bug class). Check `object::delete` reachability, `receive` on shared
objects, dynamic-field key collisions, and wrapping/unwrapping that changes the
type identity of an asset.

### 22.3 Entry-point analysis

For every `entry` and `public` function, list: sender-controlled values, mutable
objects, shared objects, type parameters, capabilities, clock/epoch reads, and
container sizes. Trace each input to a privileged state change. Watch for:
`public` non-entry functions that a non-entry function can call (visibility
rules differ from EVM), missing `Signer` checks, `ctx.sender()` vs the
transaction sender, and vector/map iteration that a caller can grow.

### 22.4 Move Prover and conservation

For coin/payment logic, prove `inputs + authorized minting = outputs +
authorized burning + fees` and treat any mismatch as a prime candidate. Use
`sui move test`, the Move Prover, and `##[test]` abort-code assertions; assert
the resulting object set, not just that it did not abort.

### 22.5 Node, execution, and consensus

Audit the boundary between transaction construction, signature validation,
authority checks, object locking, consensus ordering, execution, effects,
certificates, checkpoints, epochs, and state sync. Look for: signature
bypass; identity confusion; stale-epoch acceptance; replay across epochs;
object-lock bypass; inconsistent validation between transaction and effects;
inconsistency between fast and fallback validation paths; malformed object
state accepted after deserialization; `unwrap`/`panic` reachable from
network-controlled input; state divergence; double execution; incorrect
committee membership; and expensive validation as a liveness attack.

A node-layer issue is High/Critical only with a demonstrated
consistency, liveness, or asset-security consequence. Say which.

### 22.6 Verifier and framework

Audit every rule that turns bytecode or transaction data into accepted
execution: type safety, ability constraints, bytecode validity, visibility
rules, entry-function restrictions, upgrade compatibility, signer checks,
reference safety, object safety, and the ordering of invariant enforcement.

The prime exploit pattern is a **verifier/runtime mismatch**:

```
verifier accepts X  ->  runtime interprets X as Y  ->  authority gained or state corrupted
```

Cross-check the fast path against the fallback path; divergent validation
between them is the bug. When testing a framework primitive, audit its
downstream consequences, not the primitive.

### 22.7 Bridge

`source event -> proof -> message -> quorum -> destination mint/release`.
Audit: replay protection; domain separation (chain id, bridge id, deploy
epoch); nonce and message-id uniqueness; validator-set and epoch transitions;
threshold and signature aggregation; source/destination binding; token mapping
and decimal conversion; mint/burn conservation; message ordering; duplicate
delivery; partial execution and retry semantics; upgrade authority; and
emergency pause plus recovery. Quantify: forged mint, double release,
unauthorized withdrawal, stuck funds, validator takeover, replay.

### 22.8 DEX (DeepBook-style orderbook)

State machine `place -> match -> cancel -> settle -> withdraw`, with
`free balance + locked balance = user balance`.

Check order-id uniqueness and reuse; owner authorization; reservation
accounting vs free balance; partial fill accounting; price-time priority;
self-trade rules; cancel/match races; maker/taker fee application and rounding;
tick and lot size; min quantity; quote/base decimal mismatch; market-order
sweep edge cases; oracle and liquidation interactions; and stale orders after
an object/capability change.

Adversarial sequences to actually test:
`place -> partial fill -> cancel`; `place -> cancel -> reuse id`;
`place A -> place B -> match -> withdraw`; `many small orders -> rounding
accumulation`; `crossed orders -> exact boundary price`.

### 22.9 Key server / cryptographic services

Authorization to request keys; policy evaluation; identity binding;
capability leakage; anti-replay; nonce and session handling; domain separation;
policy bypass; threshold assumptions; secret exposure; downgrade paths;
malformed requests; cross-tenant isolation; audit-log integrity; and
availability attacks that block key release. Never claim a primitive is broken —
claim the exact misuse.

### 22.10 Hand-rolled arithmetic and shift bounds

Move aborts on overflowing `+` and `*`, but **left shift `<<` truncates
silently**. Every hand-rolled checked-shift wrapper is therefore a boundary
you must re-derive by hand, and it is the single most expensive bug class in
the Move ecosystem.

**Cetus, 22 May 2025, ~$223M — largest DeFi loss of 2025.** The shared
`integer-mate` library's guard used

```move
// VULNERABLE
if (n > (0xFFFFFFFFFFFFFFFF << 192)) { (0, true) } else { ((n << 64), false) }
// correct: reject any n >= 1 << 192
```

A single wrong constant let values `>= 2^192` through a `<< 64` that truncated;
`get_delta_a` then priced enormous liquidity at ~1 token. The code had been
live since October 2023 and **passed three professional audits** — the bug was
in a dependency, not in the protocol's own files.

Rules extracted:

- Re-derive every shift/width bound from the *type width*, never from a mask
  someone wrote. A mask of the wrong width is indistinguishable from a correct
  one in review.
- In Move, `x << k` where the high bits do not fit is **silent**, not an abort.
  Treat `<<` as an unchecked operation regardless of what `+`/`*` do.
- **Audit third-party libraries with your own budget.** Cetus's failure was a
  `lib/` dependency, which is where auditors relax. Diff it against upstream
  (§24.4) and read its math, not just your call sites.
- Audits passing is not evidence of absence for boundary arithmetic — it is
  evidence that three reviewers did not find this one instance.

## 23. Escrow, factory, and quest payouts

Lifecycle: `factory init -> escrow create -> token whitelist -> activity ->
reward distribution -> completion -> admin withdrawal`.

Audit these separately from generic access control; the interesting bugs live in
the *combination*.

- **Factory trust boundaries** — factory admin vs per-quest escrow admin vs the
  quest/CUBE contract vs the escrow instance vs treasury vs whitelisted tokens
  vs payout recipient. For every privileged path, decide whether authority is
  global or scoped to one quest, and test the exact precedence of
  `per-quest admin OR global admin`.
- **CREATE2 / deterministic deployment** — salt-namespace uniqueness (is the
  quest id in the salt? is the deployer in the salt?); whether
  `questId -> escrow` can be replaced; constructor arguments correct and
  immutable where intended; whether an attacker can predict, pre-deploy, or
  collide; and whether factory state is written consistently with the address
  actually deployed. Never treat deterministic deployment as a security
  guarantee by itself.
- **Per-quest admin** — can an unauthorized account set the admin; can it be
  `address(0)`; can an admin reassign or upgrade itself; do stale privileges
  survive quest replacement or migration; does the global admin create an
  intended or an accidental escape hatch.
- **Token whitelist** — correct escrow selected; zero-address handling; token
  contract validation; removal cannot strand funds or open an unintended
  withdrawal path; whitelist state consistent between factory and escrow; and
  an attacker cannot substitute a token address.
- **Activity gating** — if withdrawals are blocked while a quest is active,
  prove the factory and the authoritative quest contract agree on quest
  identity, active/inactive state, completion, pause/cancel, and reactivation.
  A local `isQuestActive(questId)` is meaningful only if the underlying state
  cannot be spoofed.
- **Multi-token payout routing** — audit every branch independently: native,
  ERC20, ERC721, ERC1155. Check recipient authorization, amount authorization,
  `token/tokenId` binding, type confusion, zero-value behavior, reserve
  tracking, and payout-vs-withdrawal separation. For ERC721 prove the selected
  token id is owned by the intended escrow. For ERC1155 prove
  `(token, id, amount)` stay bound across the whole path.
- **Fees (`rakeBps` and friends)** — denominator and maximum, who controls it,
  no double application, receiver gets the intended net amount, destination not
  attacker-controlled; test `0`, `1`, `max`, `MAX+1`. Trace the arithmetic into
  the escrow, not just the factory parameter.
- **CUBE-only distribution** — if only the canonical quest contract may
  distribute: is the immutable really the intended instance, can a proxy
  silently replace the trust root, can quest ids be confused across
  environments, and — the important one — can a caller *inside* that trust
  boundary be induced to authorize an attacker-chosen asset transfer? A caller
  check is not a payout authorization.
- **Admin withdrawal** — treat as a high-risk custody primitive. Only the
  intended authority; active quests cannot be drained; recipient authorized;
  every token branch safe; withdrawing token A cannot affect token B's reserve;
  ERC721/1155 ids cannot be redirected; and the documented threat model matches
  the actual global-admin power.

Reserve invariants per asset, which the transitions above must be searched
against:

```
nativeReserve   >= committedNativePayouts   + authorizedExcess
erc20Reserve[t] >= committedErc20Payouts[t] + authorizedExcess
erc1155Reserve[t][id] >= committed1155Payouts[t][id] + authorizedExcess
for every committed ERC721 id: owned by escrow until a valid payout/withdrawal
```

Always compare `factory.escrows[questId]` and `factory.escrowAdmin[questId]`
against the deployed instance's real authorization model. Any desynchronization
between the two is a priority finding.

## 24. Version matrix — dependency and compiler

OZ Contracts is inherited as **source**, so a vendored or modified copy is
*your* code and inherits *your* bugs. Audit usage with the same rigor as custom
code.

**This table is a starting point, not current.** Before citing any row, confirm
against the live advisory list for the exact version in the lockfile:
<https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories>

| Advisory | Component | Affected | Patched | Mechanism |
|---|---|---|---|---|
| CVE-2022-35961 | `ECDSA` | `>=4.1.0 <4.7.3` | 4.7.3 | `recover(bytes)`/`tryRecover(bytes)` accept EIP-2098 compact sigs, so a signature can be replayed in a different form — only matters if reuse protection marks the *signature* used instead of the message/nonce. |
| CVE-2022-31170 | `ERC165Checker` | `>=4.0.0 <4.7.1` | 4.7.1 | `supportsInterface` can **revert** instead of returning `false` when a target returns a non-0/1 value; breaks callers that treat lack of support as non-fatal. |
| CVE-2022-31198 | `GovernorVotesQuorumFraction` | `>=4.3.0 <4.7.2` | 4.7.2 | Lowering the quorum can make **already-defeated** proposals executable. Timelock quorum changes. |
| CVE-2021-39167 / 39168 | `TimelockController` | `3.3.0–<3.4.2`, `4.0.0–<4.3.1` | 3.4.2 / 4.3.1 | An executor-role holder can reset the delay to 0 and escalate to full control of the timelock. |
| CVE-2024-27094 | `Base64` | `>=4.5.0 <4.9.6`, `5.0.0-rc.0–<5.0.2` | 4.9.6 / 5.0.2 | `encode` can read past the input buffer when its length is not a multiple of 3, leaking up to 4 bits of adjacent memory into the output. |
| CVE-2025-54070 | `Bytes.lastIndexOf` | `>=5.2.0 <5.4.0` | 5.4.0 | `lastIndexOf(bytes,byte,uint256)` reads uninitialized memory when `buffer.length == 0` and `pos != type(uint256).max`; can OOG or return an out-of-range index. |
| GHSA-93hq-5wgc-jc82 | `ERC721Consecutive` | `>=4.8.0 <4.8.3` | 4.8.3 | Incorrect `balanceOf` update for a batch of size 1. |
| SCWE-092 | `Initializable` | design, not version | — | Implementation missing `constructor { _disableInitializers(); }` is directly initializable/takeoverable. |

Also check the advisories this table does not list — it is not exhaustive, and
new ones are published regularly.

### 24.1 Version resolution

Read the *lockfile*, not the semver range: `package-lock.json` / `yarn.lock` /
`lib/openzeppelin-contracts` submodule SHA / `soldeer.lock`. A repo on
`^4.8.0` may have 4.9.6 installed. Then confirm the *vendored source* matches
upstream:

```bash
jq -r '.dependencies["@openzeppelin/contracts"]' package.json
git -C lib/openzeppelin-contracts rev-parse HEAD     # submodule
diff -r lib/openzeppelin-contracts/contracts/access/AccessControl.sol <upstream>/AccessControl.sol
```

### 24.2 Upgrade-version check

Per §20: `_disableInitializers()` present in every implementation constructor;
`initializer`/`onlyInitializing` on all init functions; no double-init path; no
frontrunnable init; layout append-only; gaps correct; no constructor reliance;
no `selfdestruct`/`delegatecall` in an implementation; `__gap` sized to absorb
the additions. A transparent-proxy admin must not be able to call implementation
functions.

### 24.3 Access-control check

`DEFAULT_ADMIN_ROLE` is the admin of every role unless `_setRoleAdmin` overrides
it. Confirm the holder is trusted after deployment (factories frequently leave
`msg.sender` as `DEFAULT_ADMIN_ROLE`, handing every creator an
`UPGRADER_ROLE` grant path). Confirm critical roles have explicit admins,
grant/revoke are themselves gated, no role can escalate to a more privileged
one, and renouncing admin is either impossible or deliberately managed. On OZ
v5 `AccessManager` / `AccessControlDefaultAdminRules`, re-derive the model — the
semantics differ from v4.

### 24.4 Inheritance audit ("my functions should follow my checks")

OZ functions carry explicit checks *and* implicit preconditions. Overrides
routinely preserve the function and drop the check.

```bash
rg -n 'function .* (public|external)' -A2 -g '*.sol' | rg -B2 'override'
# list every override of a non-virtual OZ function, then diff its body against upstream
```

For each override, diff against upstream and verify the override preserves the
checks, calls `super` where required, keeps modifiers (`onlyOwner`,
`nonReentrant`, `whenNotPaused`), and does not use hooks
(`_beforeTokenTransfer` / `_afterTokenTransfer` /
`_update` / `_checkRole`) in a way that reintroduces external calls or stale
accounting.

### 24.5 Modular delegatecall / Diamond storage

Plugins executing in one storage context collide on fixed slots. OZ's
namespaced-storage pattern uses ERC-7201
(`keccak256(abi.encode(uint256(keccak256("openzeppelin.storage.X")) - 1)) & ~bytes32(uint256(0xff))`),
which is collision-resistant; older versions used a single shared
`INITIALIZABLE_STORAGE` slot that does not survive multiple modules. Check that
every module uses a distinct namespace and that each upgradeable base has an
adequately sized gap.

### 24.6 Compiler and EVM-target advisories

The compiler is a dependency. Verify it with the same suspicion as OZ.

**Solidity.** Check the lockfile/pinned `solc` build against the official known-bugs
list: <https://docs.soliditylang.org/en/latest/bugs.html> (and
`docs/bugs_by_version.json` for per-version filtering). The maintainers' own
severity rule is the audit rule: *"It is highly suspicious if a contract was
compiled with a compiler that contains a known bug and the contract was created
at a time where a newer compiler version containing a fix was already released."*
Note that `viaIR` and the EVM version are *conditions* on many of these bugs —
the same source can be safe under one codegen path and vulnerable under the other,
so record both.

**Vyper.** Correct Vyper source with a buggy compiler is still a drainable
contract, and this class has a confirmed seven-figure track record:

| Advisory | Versions | Failure | Consequence |
|---|---|---|---|
| CVE-2023-39363 (GHSA-5824-cm3x-3c38) | 0.2.15, 0.2.16, 0.3.0 — fixed in **0.3.1** | `@nonreentrant` named locks allocated **per function, not per key**, so two functions sharing a key never exclude each other | Curve pools, 30 Jul 2023, ~$73M. The decorator did nothing; ~460 deployed contracts were affected |
| CVE-2023-32675 (GHSA-vxmm-cwh2-q762) | **< 0.3.8** — fixed in 0.3.8 | global `calldatasize` check removed; funds can be sent to a nonpayable default function | default-function payable bypass |

Check `pragma`/`vyper` version in each `.vy`, then confirm against
<https://github.com/vyperlang/vyper/security/advisories> — later Vyper compiler
CVEs exist and this table does not list them.

**EVM target vs. chain.** solc >= 0.8.20 defaults the EVM target to Shanghai,
emitting `PUSH0`. Deploy that on a chain without `PUSH0` and the deployment
fails or the contract is dead on arrival — a liveness finding, not a nit.
Confirm `evm_version` in `foundry.toml`/`hardhat.config` matches the *actual*
deployment chain, and that the explorer's verified source recompiles to the
deployed bytecode under the stated optimizer/`viaIR`/EVM settings. If it does
not, you are auditing a file the chain is not running.

## 25. Cross-contract reasoning

The highest-impact bugs are frequently the ones no single file reveals. Build
all six graphs, then look for asymmetries:

1. **Call graph** — who calls whom, with what arguments, in what order.
2. **Authority graph** — which contract decides that another is trusted, and is
   that decision revocable by the trusted party?
3. **Asset graph** — where the tokens actually sit at every step, including
   inside the external protocol.
4. **State-dependency graph** — which variable in A gates security in B. Then
   ask what happens if A is upgraded or paused.
5. **Upgrade graph** — can A replace B, can B affect A, and is the cycle
   reachable from a permissionless path?
6. **Initialization graph** — can a partially initialized component be used?
   Can component A be initialized twice, or skipped, by going through B?

**Failure-mode sweep.** For every downstream call, enumerate what happens when
it: reverts; returns zero; returns malformed data; returns stale data; consumes
partial state; or mutates state before calling back. The code that handles "it
reverts" is often absent, and the code that assumes "it returns valid data" is
where the extraction is.

**Asymmetry check.** For each pair (A trusts B): does B validate everything A
assumes? Any assumption A makes that B does not enforce is a finding.

## 26. Economic exploit analysis

Required for every candidate, and the difference between a report and a
speculation. Compute and state:

- **Capital required** — the attacker's own funds, flash-borrowed funds, or
  victim liquidity. Zero-capital extraction is Critical regardless of size.
- **Gas** — and whether it exceeds the profit.
- **Profit, in the asset, net** — after gas, slippage, and fees.
- **Repeatability** — once, N times, or unbounded. A one-shot extraction of the
  whole pool is more severe than a repeatable drip of 1 wei.
- **Atomicity** — one transaction, or does it need a sandwich, a victim
  transaction, a price move, or a specific block?
- **Prerequisites** — victim liquidity, a specific oracle update, a state that
  occurs naturally (vs one the attacker must create), a seedable market.
- **Detection and reversibility** — does the protocol notice? is the loss
  recoverable? is there a social recovery or an upgrade that reverses it?
- **Attacker position** — can the attacker profit *without* the victim, by
  manipulating the accounting rather than the assets?

If profit is negative, or only positive under implausible capital, or only in a
market that does not yet exist: say so, and downgrade. If the effect is a
protocol insolvency or a permanent freeze rather than a theft, state that
separately — insolvency is often the *worse* outcome and is rated separately
from "attacker profit".

## 27. Context and budget discipline

A long audit that runs out of context reports the easy half. Budget explicitly.

- **Batch, don't skim.** ~10 files per reading pass, ledger updated between
  passes. Never read a file you have not already committed to the ledger.
- **Reuse.** When subagents are available, dispatch per-lens (§9) — a lens that
  returns only candidates, not full file transcripts, is what keeps the main
  context free.
- **Escalate by rank.** Finish §5.2 rank 1-2 completely before starting rank 4.
  Rank 1 has produced the large majority of historical losses.
- **Stop conditions** — stop and report when: the coverage ledger has no
  `unread` rows in the top two ranks; the marginal return on a new lens is
  candidates that all die at §12; the build cannot be made runnable and
  reasoning-only evidence is all that remains; or you are being asked for
  depth you cannot deliver in the remaining budget.
- **Report limitations honestly.** "Read 42 of 61 contracts; the escrow
  subsystem was not analyzed because `EscrowFactory.sol` was not provided" is a
  professional deliverable. Silence is not.
- **Never let the report imply coverage you did not achieve** (§8 funnel, §16.2).

## 28. Appendices

### 28.1 Invariant catalog

| Domain | Invariant |
|---|---|
| Custody | user funds move only to authorized recipients under authorized conditions |
| Mint/burn | `mintedSupply <= authorizedMint + priorSupply - burnedSupply` |
| Shares | `sum(userClaims) <= redeemableAssets + authorizedLiabilities` |
| Vault | `totalAssets == idle + invested - recognizedLosses - fees` |
| Strategy | `reportedAssets ~= idle + invested`, within declared tolerance |
| Loss/insurance | `coveredLoss <= independentlyMeasuredLoss`; each loss claimed at most once |
| Rewards | `sum(paid) + sum(unclaimed) == totalRewards`; `rewardPerToken` monotone non-decreasing |
| Staking | `staked[a] == balanceOf(a) - unstaked[a]`; `unclaimed == earned - claimed` |
| Bridge | `destinationRelease <= validSourceLock`, and each message consumed at most once |
| Oracle | every price used in a value-bearing decision is within its declared staleness bound |
| Governance | no role can escalate to a more privileged role without explicit authorization |
| Timelock | no privileged action is executable before its delay, and the delay is not reducible by a non-admin |
| Initialization | the implementation cannot be initialized directly; only the proxy initializes, exactly once |
| Orderbook | `freeBalance + lockedBalance == userBalance` |
| Escrow | `reserve[asset] >= committedPayouts[asset] + authorizedExcess` |
| Lending | a position is liquidatable **iff** its health factor is below the threshold, and only then |
| AMM | the invariant `k` (or `L`) is monotone except in the fee direction the pool charges |
| Signature | a signed message is accepted at most once, from the intended signer, on the intended chain, before its deadline |
| Move capability | a privileged transition requires possession of the specific authority object; no two authorities exist |
| Move object | a value can be consumed at most once; type identity is preserved across wrap/unwrap |
| Consensus/execution | effects are exactly the deterministic function of the ordered, validated transactions |

### 28.2 Exploit primitive catalog

Unauthorized initialization · upgrade takeover · role/admin escalation · storage
collision · storage-layout reinterpretation · signature replay · compact-signature
malleability · nonce reuse or off-by-one nonce · missing domain/chain separation
· stale-nonce acceptance · oracle manipulation · stale oracle acceptance ·
TWAP manipulation inside one transaction · rounding extraction · first-depositor
inflation · donation-driven share-price move · read-only reentrancy ·
cross-function reentrancy · callback state confusion · arbitrary external call ·
`delegatecall` misuse · `tx.origin` auth · unchecked low-level call return ·
fee-on-transfer miscrediting · rebasing misaccounting · decimal mismatch ·
precision loss / multiply-before-divide · truncate-in-favour-of-caller
amplification · unchecked arithmetic · unbounded loop DoS · forced ETH /
deposit accounting · `CREATE2` address confusion · missing salt namespace ·
signature loop DoS · permit-to-arbitrary-spender · EIP-712 struct omission ·
reentrancy via ERC-777/721/1155 hooks · liquidation rounding at the boundary ·
TWAP/spot as an oracle · vesting/cliff bypass · queue/withdrawal-cap bypass ·
emergency-path privilege abuse · guardian/pause capture · missing
`_disableInitializers` · `DEFAULT_ADMIN_ROLE` escalation · timelock executor
escalation · quorum retroactivity · capability forgery · object version misuse ·
shared-object ordering dependence · epoch replay · verifier/runtime mismatch ·
double execution · bridge message replay · bridge validator takeover ·
orderbook reservation mismatch · escrow reserve drain · multi-token payout
type confusion · creator-residual-admin in factories · compiler known-bug in the
pinned build · `PUSH0`/EVM target unsupported on the deployment chain · verified
source not matching deployed bytecode · hook permission-bitmap mismatch · hook
return-data free-memory-pointer corruption · read of unsettled flash-accounting
state · Move silent left-shift truncation in a hand-rolled bound check.

### 28.3 Severity calibration

**Critical** — permissionless upgrade or init takeover; forged bridge mint;
permissionless full-pool extraction; capability forgeable to mint; an
implementation directly initializable into a role that moves funds.

**High** — large or meaningful fund loss; escalation to a fund-moving role;
reliable profit from broken accounting; a governance timelock that can be
bypassed; a protocol-wide or permanent freeze; a bridge that accepts a replayed
message for a large amount.

**Medium** — constrained or bounded loss; griefing that blocks a subset of
users' withdrawals at a cost; partial freeze; a broken invariant needing a
non-trivial precondition; a signature-accepting loop with unbounded input.

**Low** — a defense-in-depth gap with no current exploit; a hard-to-reach edge
case; a minor integrity issue requiring a trusted actor; a library-version
advisory with no reachable call path.

**Info** — no security impact. Hardening, documentation, or a design note.

### 28.4 Final delivery checklist

```
[ ] Mode and scope stated, with the file list in and out of scope
[ ] Coverage ledger complete; funnel reported; no `unread` in the top ranks
[ ] Trust model, asset map, and privilege matrix produced
[ ] Invariant ledger written BEFORE hunting, and every finding cites one
[ ] Every external function in scope has a status
[ ] Multi-lens pass done (subagents or explicit sequential reset)
[ ] Grep sweep and tool sweep run as separate passes
[ ] Test-suite-mining pass done (skips, assumptions, vacuous asserts, mocks)
[ ] Historical mining done (git log for security fixes, breadcrumbs, prior audits)
[ ] Every finding has: location, attacker, preconditions, call path, violated
    invariant, quantified impact, PoC or trace, why-checks-fail, fix,
    confidence, assumptions
[ ] Every finding has a negative control, or says why it cannot have one
[ ] Capital at risk stated per finding
[ ] False-positive gate (§12) applied; hard-set items all pass
[ ] Self-adversarial review attempted and written down per finding
[ ] Confirmed / Observations / Questions / Below-threshold separated
[ ] Severity not inflated; confidence not inflated
[ ] No style, gas, or centralization reported as a security issue
[ ] OpenZeppelin versions resolved from the lockfile and checked against advisories
[ ] Compiler build, EVM target, and optimizer/viaIR flags recorded and checked
    against the known-bugs list; bytecode recompiles to what is deployed (§24.6)
[ ] Every finding tagged with its OWASP SC category (§15.4)
[ ] Storage layout diffed if the diff touches a proxy or implementation
[ ] Limitations stated: what was unread, missing, or unverifiable
```

## 29. References

**Advisories and classifications**

- OWASP Smart Contract Top 10:2026 (findings taxonomy, §15.4) —
  <https://owasp.org/www-project-smart-contract-top-10/>
- OpenZeppelin security advisories —
  <https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories>
- Solidity known-bugs list + `bugs_by_version.json` (§24.6) —
  <https://docs.soliditylang.org/en/latest/bugs.html>
- Vyper security advisories (§24.6) —
  <https://github.com/vyperlang/vyper/security/advisories>
- OWASP Smart Contract Weakness Classification — <https://scwe.owasp.org/>
- SWC registry — <https://swcregistry.io/>
- GitHub Advisory Database (npm scope) — <https://github.com/advisories?query=openzeppelin>

**Real-exploit replication (§11)**

- DeFiHackLabs — reproduce historical exploits with Foundry; thousands of
  PoCs, `forge test --contracts src/test/<YYYY-MM>/<NAME>_exp.sol -vvv` —
  <https://github.com/SunWeb3Sec/DeFiHackLabs>
- DeFiHackLabs Incident Explorer — root-cause index
  (`rootcause_data.json`) for taxonomy-driven recall —
  <https://github.com/SunWeb3Sec/DeFiHackLabs-Incident-Explorer>
- SCWE tag taxonomy and de-duplicated incident loss data underlying §15.4 —
  <https://scwe.owasp.org/>

**Protocol-specific**

- Uniswap v4 security framework (hooks, flash accounting, permission bitmap) —
  <https://developers.uniswap.org/docs/protocols/v4/security>
- Uniswap Permit2 (AllowanceTransfer vs SignatureTransfer, two nonce models) —
  <https://developers.uniswap.org/docs/protocols/permit2/overview>
- OpenZeppelin Uniswap v4 Core audit (hook return-data memory layout, native-ETH
  reentrancy, hook upgradeability after pool init) —
  <https://www.openzeppelin.com/news/uniswap-v4-core-audit>
- Cetus post-mortems — the `checked_shlw` constant error behind the $223M Move
  loss (§22.10): <https://dedaub.com/blog/the-cetus-amm-200m-hack-how-a-flawed-overflow-check-led-to-catastrophic-loss/>
  and <https://www.halborn.com/blog/post/explained-the-cetus-hack-may-2025>

**Method references**

- Solidity docs (inheritance, `delegatecall`, ABI) — <https://docs.soliditylang.org/>
- EIP-1967 proxy slots — <https://eips.ethereum.org/EIPS/eip-1967>
- EIP-1822 UUPS — <https://eips.ethereum.org/EIPS/eip-1822>
- EIP-712 typed structured data — <https://eips.ethereum.org/EIPS/eip-712>
- EIP-2612 `permit` — <https://eips.ethereum.org/EIPS/eip-2612>
- EIP-2098 compact signatures — <https://eips.ethereum.org/EIPS/eip-2098>
- EIP-4337 account abstraction — <https://eips.ethereum.org/EIPS/eip-4337>
- ERC-7201 namespaced storage — <https://eips.ethereum.org/EIPS/eip-7201>
- ERC-4626 tokenized vaults — <https://eips.ethereum.org/EIPS/eip-4626>
- Foundry book (forge, invariant testing, `inspect`) — <https://getfoundry.sh/>
- Slither detectors — <https://github.com/crytic/slither/wiki/Detector-Documentation>
- Trail of Bits smart contract security wiki — <https://github.com/ConsenSys/smart-contract-best-practices>
- Solidity security considerations — <https://docs.soliditylang.org/en/latest/security-considerations.html>

**Architecture references (the codebases this skill's domain playbooks were shaped around)**

- Tetu Contracts V2 (proxy/controller-gated upgrades, strategy/splitter, ERC-4626
  vault, insurance, vote-escrow) — <https://github.com/tetu-io/tetu-contracts-v2>
- Sui (node, core, types, execution adapter, verifier, framework, bridge) —
  <https://github.com/MystenLabs/sui>
- DeepBook V3 — <https://github.com/MystenLabs/deepbookv3>
- SEAL key server — <https://github.com/MystenLabs/seal>
