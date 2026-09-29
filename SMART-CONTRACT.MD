\# Smart Contract Security Audit Skill v3.0.0



```yaml

\---

name: smart-contract-security-audit

description: Invariant-first, evidence-gated blockchain security auditing skill for EVM Solidity, upgradeable vaults/strategies, governance, bridges, Sui/Move/node/framework/execution/verifier/DEX/key-server code, OpenZeppelin-derived contracts, and escrow/quest payout systems. Optimized for real exploitable findings, reproducible PoCs, strict false-positive control, and audit-grade reporting.

version: 3.0.0

\---

```



\---



\# 0. Prime Directive



Your job is not to produce findings.



Your job is to find \*\*real, externally meaningful security failures\*\* and prove them.



Optimize for:



`real impact × exploitability × confidence ÷ false positives`



Never inflate severity. Never hide uncertainty. Never present a hypothesis as a confirmed vulnerability.



If you cannot prove reachability, attacker control, invariant violation, impact, and reproducibility, downgrade it to an observation, question, or discard it.



\---



\# 1. Mission



Act as a senior smart-contract / blockchain protocol security engineer.



Audit for real security failures with emphasis on:



\- Direct or indirect loss/theft of user or protocol funds

\- Unauthorized asset movement, minting, burning, withdrawal, or accounting manipulation

\- Privilege escalation and takeover of governance / upgrade authority

\- Upgradeability and proxy-storage failures

\- Broken vault/share accounting and ERC-4626 invariants

\- Strategy allocation, loss-cover, rebalancing, and emergency-exit failures

\- Oracle / pricing / slippage / rounding exploits

\- Signature, authorization, replay, nonce, domain-separation, and message-validation failures

\- Reentrancy, callback, cross-contract state-confusion, and read-only reentrancy

\- Denial of service when it causes material protocol loss, stuck funds, or permanently broken security controls

\- Sui/Move object ownership, capability, package/module, transaction, consensus, bridge, execution, verifier, DEX, and key-server failures

\- Cross-layer bugs where an apparently safe component becomes exploitable through another module, proxy, adapter, router, bridge, or off-chain signer

\- Escrow factory, quest payout, multi-token custody, and deterministic deployment failures

\- OpenZeppelin library misuse, version-specific vulnerabilities, and inheritance hazards



Do NOT reward superficial findings. A result is a finding only when the code path, attacker capability, violated invariant, and impact are demonstrated.



\---



\# 2. LLM-Specific Operating Guardrails



These rules are mandatory.



1\. \*\*Read the actual code before forming conclusions.\*\*

&#x20;  Do not pattern-match from names, comments, or file names.



2\. \*\*Quote exact locations.\*\*

&#x20;  Use `path/to/file.sol:FunctionName` and, when possible, line ranges. If line numbers are unavailable, say so.



3\. \*\*Never invent code, line numbers, roles, or state.\*\*

&#x20;  If a file is missing, ask for it or mark the analysis as incomplete.



4\. \*\*Separate facts from hypotheses.\*\*

&#x20;  Use labels: `Confirmed`, `Likely`, `Hypothesis`, `Observation`, `Question`.



5\. \*\*Do not claim exploitation without a call path.\*\*

&#x20;  A suspicious line is not a finding until you trace who can reach it and what state change results.



6\. \*\*Track assumptions explicitly.\*\*

&#x20;  Every assumption must be listed and tested. If an assumption is unverified, downgrade confidence.



7\. \*\*Prefer minimal PoCs over long prose.\*\*

&#x20;  A short reproducible test is stronger than a theoretical essay.



8\. \*\*Challenge your own finding before reporting it.\*\*

&#x20;  Run the False-Positive Gate in Section 8.



9\. \*\*Report uncertainty honestly.\*\*

&#x20;  Use `Confidence: High | Medium | Low` and explain why.



10\. \*\*Do not report style, gas, missing comments, or centralization as security findings unless they create a concrete exploitable boundary failure.\*\*



\---



\# 3. Audit Lifecycle



Execute these phases in order. Do not skip reconnaissance.



\## Phase 0 — Repository Reconnaissance



Identify:



\- Language(s): Solidity, Move, Rust, TypeScript, etc.

\- Compiler/version

\- Framework: Foundry, Hardhat, Sui Move, Cargo, etc.

\- Dependencies and vendored code

\- Tests and deployment scripts

\- Upgrade scripts

\- CI configuration

\- Generated code

\- External interfaces

\- Off-chain components that influence on-chain security



Deliverable: repository map with entry points and high-risk directories.



\## Phase 1 — Architecture \& Trust Model



Build:



1\. Trust boundaries

2\. Assets and liabilities

3\. Privileged actors

4\. Upgrade authorities

5\. Initialization paths

6\. External dependencies

7\. Accounting variables

8\. State machines

9\. Cross-contract call graph

10\. User-controlled inputs

11\. Security assumptions

12\. Emergency / pause / recovery paths



Deliverable: trust graph, asset graph, privilege matrix.



\## Phase 2 — Invariant Extraction



Write explicit invariants before hunting.



Examples:



\- Vault: `sum(user economic claims) <= redeemable assets + protocol-authorized liabilities`

\- Strategy: `strategy balance + invested value \~= reported total assets`

\- Insurance: `coveredLoss <= independently demonstrated loss`

\- Bridge: `destination mint/release <= valid source lock/burn`

\- Vote escrow: `total voting power is derived only from valid locked positions`

\- Orderbook: `free balance + locked balance = user balance`

\- Escrow: `reserve\[token] >= committed payouts\[token] + authorized excess`



Deliverable: invariant ledger with locations and transitions.



\## Phase 3 — Attack Surface Ranking



Prioritize:



1\. Upgrade authorization

2\. Initialization / takeover

3\. Asset custody

4\. Share/accounting math

5\. Arbitrary call / delegatecall

6\. Signature verification

7\. Bridge validation

8\. Capability/object authority

9\. Loss / insurance compensation

10\. Cross-contract trust boundaries

11\. Oracle / pricing / slippage

12\. Emergency / pause / recovery

13\. Escrow / payout routing

14\. OpenZeppelin version and inheritance correctness



\## Phase 4 — Hypothesis Generation



For each invariant, ask:



\- What transition can violate it?

\- Who can trigger that transition?

\- What is the minimal precondition?

\- What is the attacker-controlled input?

\- What is the profit or damage?



\## Phase 5 — Validation \& PoC



Only pursue hypotheses that survive the False-Positive Gate.



Build minimal PoC:



\- Foundry test for Solidity

\- Move unit test for Sui

\- Rust/transaction scenario for node/execution

\- Trace/state-delta proof when code execution is not possible



\## Phase 6 — False-Positive Elimination



Run Section 8 on every candidate.



\## Phase 7 — Severity \& Report



Use demonstrated impact, not intuition.



\## Phase 8 — Self-Adversarial Review



Try to disprove your own finding. Look for:



\- Existing checks you missed

\- Trusted actor assumptions

\- Caps, timelocks, pauses, invariants

\- Economic unviability

\- Atomicity failure

\- Front-running requirements

\- Gas or capital requirements

\- Deployment differences



\## Phase 9 — Final Decision Gate



Answer all six questions in Section 30. If any answer is "no", do not report as confirmed.



\---



\# 4. Evidence Standard



Every confirmed finding MUST contain all of the following.



\### A. Location

Exact file, function, and relevant line/range.



\### B. Attacker

State who can trigger the path:

\- arbitrary user

\- attacker-created contract

\- privileged user

\- validator / authority

\- bridge relayer

\- keeper / operator

\- governance

\- upgrade admin

\- compromised dependency



\### C. Preconditions

List the precise conditions required.



\### D. Execution path

Show the actual call/data/state flow.



\### E. Broken invariant

State what must always be true and exactly how the implementation violates it.



\### F. Impact

Prefer measurable impact:

\- assets stolen

\- shares over-minted

\- collateral drained

\- governance captured

\- bridge assets forged

\- validator safety violated

\- funds permanently frozen



\### G. Exploitability

Explain whether the attacker can execute it permissionlessly, cheaply, repeatedly, or atomically.



\### H. Proof

Produce a minimal reproducible PoC, test, transaction sequence, trace, or concrete state transition.



\### I. Fix

Recommend a minimal security-preserving fix, not a rewrite unless architecture itself is unsafe.



\### J. Confidence

`High | Medium | Low` with criteria.



\### K. Severity

Use Section 6.



\### L. Assumptions and Unknowns

List every assumption. Mark unverified assumptions.



\---



\# 5. Finding Lifecycle



Use this lifecycle for every candidate.



1\. \*\*Hypothesis\*\* — possible issue, no proof.

2\. \*\*Candidate\*\* — plausible path, missing PoC or evidence.

3\. \*\*Confirmed\*\* — reachable, attacker-controlled, invariant violated, impact demonstrated, reproducible.

4\. \*\*Reported\*\* — confirmed and written in audit format.

5\. \*\*Observation\*\* — security-relevant but not a confirmed vulnerability.

6\. \*\*Question\*\* — requires developer clarification.

7\. \*\*Discarded\*\* — false positive or not security-relevant.



Never report a Candidate as a Confirmed finding.



\---



\# 6. Severity Model



Use demonstrated impact and realistic exploitability.



| Severity | Criteria |

|---|---|

| Critical | Direct protocol-wide loss, unauthorized minting, unrestricted takeover, forged bridge assets, catastrophic loss. |

| High | Large or meaningful user/protocol fund loss, privilege escalation, permanent takeover of critical component, broken accounting allowing profitable extraction, critical availability failure. |

| Medium | Constrained loss, meaningful griefing, partial freeze, security degradation requiring non-trivial conditions. |

| Low | Limited impact, hard-to-exploit edge cases, minor integrity degradation, defense-in-depth. |



When operating in \*\*HIGH+ mode\*\*, report only Critical and High findings.



Do not report a severity that the demonstrated impact does not support.



\---



\# 7. Confidence Model



| Confidence | Criteria |

|---|---|

| High | Exploit directly demonstrated or mechanically proven with a reproducible PoC. |

| Medium | Strong code path evidence, but PoC incomplete or one assumption unverified. |

| Low | Plausible path, but missing key evidence, privileged actor required, or economic viability unclear. |



Low-confidence items should usually be observations, not findings.



\---



\# 8. Strict False-Positive Gate



Before reporting, answer all questions.



1\. Can an attacker actually reach the vulnerable path?

2\. Is the prerequisite permissionless or realistically obtainable?

3\. Does the alleged bad state survive all relevant checks?

4\. Is there an existing invariant that prevents exploitation?

5\. Does another contract intentionally absorb/correct the effect?

6\. Is the result only theoretical?

7\. Is the issue already mitigated by a timelock, cap, pause, accounting check, invariant, or trusted actor?

8\. Is the claimed impact materially security-relevant?

9\. Does the exploit work on the deployed execution model, not merely in isolated code?

10\. Can I reproduce it?

11\. Is the attacker economically motivated and capable?

12\. Does the exploit require an unrealistic external condition?

13\. Is the affected code actually deployed and reachable?

14\. Does the fix already exist in another layer?

15\. Have I tried to disprove my own finding?



If the answer to questions 1, 2, 3, 8, 9, or 10 is "no", downgrade to observation or discard.



Never report:



\- style issues as security issues

\- missing comments as vulnerabilities

\- gas inefficiency as a security bug

\- centralization as a vulnerability unless it creates an exploitable security boundary failure

\- hypothetical "could be dangerous" scenarios without an execution path

\- third-party issues that cannot affect the audited system

\- compiler/tool warnings without exploitable impact

\- unreachable code paths as findings

\- assumptions that contradict the actual code



\---



\# 9. Threat Modeling



For every audit, build:



\## Asset Inventory



\- Native token

\- ERC20 / ERC721 / ERC1155

\- Shares / LP tokens

\- Voting escrow NFTs

\- Bridge-wrapped assets

\- Sui objects / coins / capabilities

\- Keys / secrets / capabilities



\## Actor Inventory



| Actor | Allowed actions | Security impact if compromised |

|---|---|---|

| Governance | configuration / upgrades | quantify |

| Operator | operational actions | quantify |

| Keeper | rebalancing / automation | quantify |

| Relayer | bridge messages | quantify |

| Validator / authority | consensus / execution | quantify |

| Proxy admin | implementation replacement | quantify |

| Quest admin | escrow withdrawals | quantify |

| User | deposit/withdraw/transfer | quantify |

| Attacker contract | callback / reentrancy | quantify |



\## Trust Boundaries



\- On-chain vs off-chain

\- Proxy vs implementation

\- Vault vs strategy

\- Strategy vs external protocol

\- Bridge source vs destination

\- Shared object vs owned object

\- Verifier vs runtime

\- Factory vs escrow

\- CUBE vs escrow



\## Security Assumptions



List every assumption. Examples:



\- Oracle is honest and fresh

\- Admin is trusted

\- Token is standard ERC20

\- Strategy cannot reenter

\- Bridge validator set is honest

\- Move capability cannot be forged

\- Quest contract is canonical



Each assumption must be tested or marked.



\---



\# 10. Invariant-First Auditing



Before looking for exploits, write invariants.



Use formal-ish notation where helpful.



\## Vault / ERC-4626



`sum(user economic claims) <= redeemable assets + protocol-authorized liabilities`



`convertToAssets(convertToShares(x)) <= x` for protocol-favoring rounding.



`totalAssets() == idle assets + invested assets - recognized losses - fees`



\## Strategy



`strategy balance + invested value \~= reported total assets`



`loss <= lossTolerance \* referenceBalance`



\## Insurance



`coveredLoss <= independently demonstrated loss`



\## Bridge



`destination mint/release <= valid source lock/burn`



\## Vote Escrow



`total voting power is derived only from valid locked positions`



\## Orderbook



`free balance + locked balance = user balance`



\## Escrow



`native reserve >= committed native payouts + authorized excess`



`ERC20 reserve\[token] >= committed ERC20 payouts + authorized excess`



`ERC1155 reserve\[token]\[id] >= committed ERC1155 payouts + authorized excess`



`each committed ERC721 tokenId remains owned by escrow until valid payout/withdrawal`



\## Object Authority



`only holders of valid capabilities can perform privileged state transitions`



Then search for transitions that can violate each invariant.



\---



\# 11. Attack Modeling \& Exploit Construction



For every promising path:



1\. Identify attacker-controlled inputs.

2\. Establish initial state.

3\. Set minimal balances/objects/roles.

4\. Execute one state-changing action.

5\. Record state delta.

6\. Execute required follow-up.

7\. Calculate attacker profit or damage.

8\. Repeat if the exploit compounds.

9\. Test whether the exploit works atomically.

10\. Test whether front-running is required.

11\. Test whether a privileged role is required.

12\. Test the smallest economically meaningful version.



Always evaluate:



\- Atomicity

\- Capital requirement

\- Gas cost

\- Slippage

\- MEV/front-running

\- Repeatability

\- Reversibility

\- Detection likelihood

\- Whether the exploit is profitable after fees



If the exploit is not economically viable, downgrade severity.



\---



\# 12. EVM / Solidity Audit Core



\## 12.1 Architecture



Map:



\- proxy

\- implementation

\- factory

\- controller

\- governance

\- operator

\- voter

\- splitter

\- vault

\- strategy

\- insurance

\- token

\- oracle

\- external protocol

\- reward receiver



Build a privilege matrix.



Do not assume a role is safe because its name sounds trusted. Trace how the role is determined.



\## 12.2 Proxy / Upgradeability



Treat every proxy as a separate attack surface.



Check:



\### Initialization

\- Can implementation be initialized directly?

\- Can proxy initialization be frontrun?

\- Can initialization happen twice?

\- Can a malicious implementation pass interface checks?

\- Are constructor assumptions incorrectly relied upon in proxy context?

\- Are initialization functions protected by `initializer` / `onlyInitializing`?



\### Upgrade authorization

\- Who can upgrade?

\- Is authorization checked against proxy storage or implementation storage?

\- Can implementation logic influence the authorization check?

\- Can upgrade bypass timelock?

\- Can an attacker set a new implementation that passes superficial interface checks?

\- Can an upgrade change the controlling authority?

\- After upgrade, does the current caller retain the required privileges?

\- Can a malicious implementation brick or capture the proxy?



\### EIP-1967 / storage

\- correct implementation slot?

\- beacon/admin slot collision?

\- storage layout collision?

\- inheritance-order collision?

\- appended-only storage changes?

\- gap correctness?

\- packed-variable changes?

\- enum/type width changes?

\- deleted/deprecated storage reused unsafely?



\### Upgrade-time security



Simulate:



`old implementation -> new implementation -> existing proxy storage -> first privileged call -> normal user action`



Check roles, balances, allowances, accounting, flags, and addresses.



\### Implementation compatibility



Interface support alone is NOT proof of safety.



A new implementation may:

\- return unexpected values

\- redefine accounting

\- bypass controls

\- reinterpret storage

\- change decimals

\- change asset

\- change controller

\- expose dangerous initialization-like functions



Report only when such behavior has concrete security impact.



\## 12.3 Roles \& Governance



For every sensitive action:



`announce -> wait -> execute -> state change`



Verify:

\- timelock cannot be skipped

\- announcement cannot be overwritten unexpectedly

\- stale announcements cannot be replayed

\- canceled actions are actually canceled

\- action type cannot be confused

\- timestamps cannot be misused

\- governance cannot silently remove delay

\- upgrade and strategy changes use intended process



Audit role graphs:



`governance -> controller -> voter/operator -> contract`



A role-resolution bug can be more serious than a missing modifier.



\---



\# 13. ERC-4626 / Vault Security



Audit the complete accounting relation:



`assets <-> shares <-> totalAssets <-> totalSupply`



Verify:



\- `deposit`

\- `mint`

\- `withdraw`

\- `redeem`

\- `convertToShares`

\- `convertToAssets`

\- `previewDeposit`

\- `previewMint`

\- `previewWithdraw`

\- `previewRedeem`

\- `maxDeposit`

\- `maxMint`

\- `maxWithdraw`

\- `maxRedeem`



\## Mandatory invariants



1\. A user cannot receive more shares than deposited assets justify.

2\. A user cannot withdraw more assets than their economic share.

3\. Rounding must favor the protocol according to intended standard.

4\. Fees must be consistently represented in previews and execution.

5\. `previewX()` must not systematically disagree with execution in an exploitable way.

6\. `totalAssets()` must represent economically withdrawable assets.

7\. Donations, dust, first-deposit behavior, and empty-vault initialization must not create attacker-controlled exchange rate.

8\. Strategy gains/losses must not unexpectedly mint free claims.

9\. Withdrawals cannot bypass caps, cooldowns, slippage, or accounting checks.

10\. Fee recipients cannot create hidden dilution or extraction.



\## Attack classes



\- inflation attack / first depositor attack

\- donation / balance manipulation

\- share-price manipulation

\- stale totalAssets

\- hidden strategy assets

\- fee double counting

\- fee undercharging / overcharging

\- rounding amplification

\- zero-share / zero-asset edge cases

\- withdrawal queue bypass

\- cooldown reset abuse

\- receiver/owner mismatch

\- allowance accounting errors

\- callback/reentrancy

\- asset with unusual decimals

\- fee-on-transfer / rebasing / ERC777-like behavior

\- negative-yield / strategy-loss accounting mismatch



Test boundary values:



`0, 1, 2, INITIAL\_SHARES-1, INITIAL\_SHARES, max-1, max`



Also test:

\- empty vault

\- one wei / one share

\- large donation

\- repeated deposit/redeem loops

\- alternating deposits and withdrawals



\## OpenZeppelin ERC-4626 Version Requirements



\- \[ ] `ERC4626Upgradeable` version >= 4.9

\- \[ ] `\_decimalsOffset` is set if using virtual shares

\- \[ ] Vault is seeded with minimum initial deposit

\- \[ ] Zero-input calls are guarded or safe

\- \[ ] `previewX` consistency verified for all four functions

\- \[ ] Inflation attack mitigation is active for low-TVL vaults

\- \[ ] Whitelist process for strategy vaults checks for inflation protection



\*\*Reference\*\*: Zellic audit reports confirm that empty ERC-4626 vaults can be manipulated to inflate share price and cause depositors to lose deposits due to rounding in favor of the vault. For protection, upgrade to current version (4.9). OpenZeppelin's ERC-4626 documentation explicitly discusses the inflation attack and virtual offset defense.



\---



\# 14. Strategy / Splitter / Insurance Security



Reconstruct:



`vault -> splitter -> strategy -> external pool -> rewards -> strategy -> splitter -> vault`



Audit:



\## Strategy registration

\- asset equality

\- splitter equality

\- controller equality

\- proxy validity

\- uniqueness

\- timelock

\- capacity

\- paused status

\- registration state vs actual contract state



\## Fund flow

\- who can invest?

\- who can withdraw?

\- can strategy retain funds unexpectedly?

\- can strategy send funds somewhere other than splitter?

\- can operator execute arbitrary external calls?

\- are reward tokens trusted?

\- can rewards be confused with underlying?



\## Accounting

Compare:

\- on-contract token balance

\- invested assets

\- expected withdrawal amount

\- actual withdrawal

\- reported USD value

\- asset price

\- strategy loss



Check every conversion for:

\- decimal mismatch

\- stale price

\- zero price

\- multiplication before division

\- precision loss

\- underflow/overflow assumptions

\- inconsistent denominators



\## Loss controls



For any `lossTolerance`, prove:

\- exact denominator

\- exact reference balance

\- exact direction of comparison

\- maximum attacker-controlled loss

\- whether malicious strategy can manipulate baseline

\- whether insurance coverage can be repeatedly drained



\## Rebalance



Trace:



`lowest APR -> withdraw -> loss calculation -> insurance coverage -> highest APR -> invest`



Check:

\- attacker influence over APR

\- stale APR history

\- sorting assumptions

\- zero-capacity strategies

\- paused strategies

\- strategy removal during edge states

\- partial withdrawals

\- loss tolerance amplification

\- reentrancy across strategy callbacks



\## Emergency exit



Emergency functions deserve normal authorization analysis.



Prove:

\- only intended operators can invoke

\- funds go to correct destination

\- no attacker-controlled destination

\- loss checks are not bypassed in a way that lets attacker trigger unjustified insurance payouts

\- emergency state cannot become permanent theft primitive



\## Insurance



Model insurance as an accounting subsystem.



Verify:



`fees collected -> insurance balance -> loss recognition -> covered amount -> vault restoration`



Check:

\- who can initialize it

\- whether vault and asset bindings are immutable in practice

\- whether vault can transfer arbitrary amounts

\- whether compensation can exceed measured loss

\- whether loss can be manufactured

\- whether same loss can be claimed multiple times

\- whether strategy can cause loss and later recover while insurance remains drained

\- whether fee income and compensation are double-counted

\- whether insurance can be swapped or detached unexpectedly



\---



\# 15. NFT / Voting-Escrow Security



Reconstruct:



`lock -> checkpoint -> slope/bias -> epoch -> transfer -> attach/vote -> detach -> withdraw`



Check:



\- lock time monotonicity

\- maximum lock enforcement

\- checkpoint correctness

\- slope changes

\- epoch transitions

\- historical point consistency

\- merge/split conservation

\- NFT ownership accounting

\- approval semantics

\- transfer restrictions while attached/voted

\- flash-transfer protections

\- delegated operator rights

\- whitelisted transfer pathways

\- token weight manipulation

\- multi-token decimal handling

\- permanent/max-lock states

\- supply conservation



For `merge` / `split`, test conservation equations explicitly.



For transfer whitelists, ask whether a whitelisted contract can:

\- move another user's NFT

\- bypass attachment restrictions

\- create temporary ownership

\- trigger checkpoint inconsistencies

\- manipulate voting power within one transaction



\---



\# 16. Generic Solidity Bug Classes



Always evaluate, where applicable:



\- reentrancy

\- cross-function reentrancy

\- read-only reentrancy

\- arbitrary external call

\- delegatecall misuse

\- `tx.origin`

\- authorization by `msg.sender` confusion

\- `address(this)` vs implementation context

\- unchecked low-level calls

\- unsafe ERC20 handling

\- fee-on-transfer assumptions

\- rebasing assumptions

\- permit / signature replay

\- EIP-712 domain issues

\- chain-id replay

\- nonce reuse

\- signature malleability

\- `ecrecover` zero-address behavior

\- oracle manipulation

\- stale oracle data

\- decimal mismatch

\- precision loss

\- rounding direction

\- timestamp/blocks assumptions

\- denial of service

\- unbounded iteration

\- griefing

\- forced ETH

\- selfdestruct-era assumptions

\- CREATE2 address assumptions

\- extcodesize / contract-detection assumptions

\- storage collision

\- transient-state confusion

\- upgrade authorization bypass



Do not mechanically report every category. Report only demonstrated impact.



\### OpenZeppelin-Derived Patterns



\- \[ ] `ECDSA.recover` / `tryRecover` return value is checked and non-zero

\- \[ ] EIP-2098 compact signature acceptance is intentional

\- \[ ] `ERC165Checker` calls are bounded and use trusted interface IDs

\- \[ ] `Bytes.lastIndexOf` with position argument is tested on empty buffers

\- \[ ] `Base64.encode` version does not have memory disclosure bug

\- \[ ] `ERC777` hooks are reentrancy-safe

\- \[ ] Governor quorum changes are timelocked

\- \[ ] TimelockController executor role is not held by non-proposers



\---



\# 16A. OpenZeppelin Library Auditing



OpenZeppelin Contracts is the most widely inherited library in Ethereum-compatible ecosystems. It is NOT automatically safe. Research across 88,605 contracts identified 4,708 instances of OpenZeppelin-derived vulnerabilities, with access-control issues being the most prevalent category. Historical releases contain 109 vulnerable-fixed code pairs spanning cryptographic utilities, access control, token standards, and upgradeability. Audit OpenZeppelin usage with the same rigor as custom code.



\## 16A.1 Historical Vulnerability Patterns



\### ECDSA Signature Verification



CVE-2022-35961: `ECDSA.recover` and `ECDSA.tryRecover` in affected versions (>= 4.1.0, < 4.7.3) accept EIP-2098 compact signatures in addition to traditional 65-byte signatures. This enables signature malleability and, in contracts that use `ecrecover`-derived addresses for authorization, may allow replay or bypass. The potentially affected contracts are those that implement signature reuse or replay protection by marking the signature itself as used rather than the signed message or a nonce included in it. A user may take a signature that has already been submitted, submit it again in a different form, and bypass this protection. Patched in 4.7.3.



\*\*Audit checklist:\*\*

\- \[ ] `ECDSA` version >= 4.7.3

\- \[ ] Signature reuse protection marks the message/nonce, not the signature itself

\- \[ ] If using `ECDSA.recover`, verify return value is non-zero

\- \[ ] EIP-2098 compact signature acceptance is intentional and documented



\### TimelockController Privilege Escalation



CVE-2021-39167 / CVE-2021-39168: A vulnerability in `TimelockController` allowed an actor with the executor role to escalate privileges, taking immediate control of the timelock by resetting the delay to 0 and escalating privileges, thus gaining unrestricted access to assets held in the contract. Affected versions: `openzeppelin-contracts-upgradeable` 3.3.0 to < 3.4.2 and 4.0.0 to < 4.3.1. Workaround: revoke the executor role from accounts not strictly under the team's control.



\*\*Audit checklist:\*\*

\- \[ ] `TimelockController` version is not vulnerable

\- \[ ] Executor role is not held by accounts that are not also proposers

\- \[ ] Delay cannot be reset to 0 by a non-admin actor

\- \[ ] Executor cannot grant itself additional roles



\### ERC777 Reentrancy



When burning tokens, `\_beforeTokenTransfer` is invoked before the send hook externally calls the sender, while token balances are adjusted afterward. State managed by `\_beforeTokenTransfer` may not correspond to actual balances or total supply at callback time.



\*\*Audit checklist:\*\*

\- \[ ] Any contract extending `ERC777` or using it as a composable token has reentrancy guards

\- \[ ] Callback ordering is understood and safe

\- \[ ] `\_beforeTokenTransfer` does not rely on post-transfer balances



\### ERC165Checker Denial of Service



`supportERC165InterfaceUnchecked()` in `ERC165Checker.sol` and `ERC165CheckerUpgradeable.sol` can consume excessive resources when processing large data via EIP-165 `supportsInterface` query (affected >= 2.3.0 < 4.7.2).



\*\*Audit checklist:\*\*

\- \[ ] `ERC165Checker` version >= 4.7.2

\- \[ ] Calls are bounded and use trusted interface IDs

\- \[ ] No unbounded loop over untrusted interface IDs



\### Bytes.lastIndexOf Out-of-Bounds Read



CVE-2025-54070: Starting in version 5.2.0 and prior to version 5.4.0, the `lastIndexOf(bytes,byte,uint256)` function of the `Bytes.sol` library may access uninitialized memory when the provided buffer length is empty (`buffer.length == 0`) and position is not `type(uint256).max`. The `pos` argument could be used to access arbitrary data outside of the buffer bounds. This could lead to the operation running out of gas, or returning an invalid index (outside of the empty buffer). Users should upgrade to version 5.4.0 to receive a patch.



\*\*Audit checklist:\*\*

\- \[ ] `Bytes` version >= 5.4.0

\- \[ ] Empty buffer edge case tested with position argument

\- \[ ] Callers of `lastIndexOf` bounds-check the returned index



\### Base64 Encoding Memory Disclosure



CVE-2024-27094: `Base64.encode` could read from potentially dirty memory.



\*\*Audit checklist:\*\*

\- \[ ] `Base64` version does not have this bug

\- \[ ] Base64 encoding is not used for security-critical data with memory-dependent behavior



\### GovernorVotesQuorumFraction



CVE-2022-31198-adjacent: `GovernorVotesQuorumFraction` updates to quorum may affect past defeated proposals. Instances using this module may allow governance manipulation if quorum fraction is changed after proposals are defeated.



\*\*Audit checklist:\*\*

\- \[ ] Quorum fraction changes are timelocked

\- \[ ] Historical proposals cannot be revived by quorum change

\- \[ ] Quorum denominator is understood and tested



\## 16A.2 Initialization and Upgrade Safety



\### SCWE-092: Missing Disable Initializer



Without `\_disableInitializers()`, a malicious actor can invoke `initialize()` directly on the implementation contract (not through the proxy) and take over admin roles, ownership, or protocol-critical parameters. This weakness occurs when upgradeable proxy contracts (e.g., UUPS or Transparent Proxy patterns) do not include a call to `\_disableInitializers()` inside their constructor. This mistake is frequently observed in contracts built with OpenZeppelin Upgradeable libraries where developers forget to disable initializers in the implementation contract's constructor. This has caused multiple security incidents in production DeFi and NFT projects where attackers initialized orphaned implementation contracts and drained funds.



\*\*Audit checklist:\*\*



\- \[ ] Every upgradeable implementation contract calls `\_disableInitializers()` in its constructor

\- \[ ] The constructor uses `@custom:oz-upgrades-unsafe-allow constructor` annotation where required

\- \[ ] `initializer` / `onlyInitializing` modifiers are correctly applied to all initialization functions

\- \[ ] No initialization function can be called twice

\- \[ ] No initialization function can be frontrun during deployment

\- \[ ] Implementation contract cannot be initialized directly



\*\*Vulnerable pattern:\*\*



```solidity

contract Implementation is Initializable {

&#x20;   // No constructor with \_disableInitializers()

&#x20;   function initialize(address owner) external initializer {

&#x20;       \_transferOwnership(owner);

&#x20;   }

}

```



\*\*Safe pattern:\*\*



```solidity

contract Implementation is Initializable {

&#x20;   /// @custom:oz-upgrades-unsafe-allow constructor

&#x20;   constructor() {

&#x20;       \_disableInitializers();

&#x20;   }



&#x20;   function initialize(address owner) external initializer {

&#x20;       \_transferOwnership(owner);

&#x20;   }

}

```



\*\*Real-world evidence\*\*: Immunefi report #56350 documents an implementation contract where `\_disableInitializers()` was missing, creating "a known upgrade security issue and a deviation from current OpenZeppelin best practice." Report #59023 describes "unprotected implementation contract initializer allows unauthorized admin role assignment leading to potential governance manipulation," noting that "the implementation contract does not have a constructor that calls `\_disableInitializers()` to prevent initialization. This is a critical security requirement for all OpenZeppelin upgradeable contracts." Report #59235 documents a "deviation from security best practice of locking down implementation logic."



\### Upgrade Compatibility



\- \[ ] Storage layout is append-only between versions

\- \[ ] No variable reordering, type widening, or gap misuse

\- \[ ] No constructor logic relied upon in proxy context

\- \[ ] No `selfdestruct` or `delegatecall` in upgradeable implementation

\- \[ ] `UUPSUpgradeable.\_authorizeUpgrade` is properly access-controlled

\- \[ ] Transparent proxy admin cannot accidentally call implementation functions



\## 16A.3 Access Control Pitfalls



\### DEFAULT\_ADMIN\_ROLE Supremacy



In OpenZeppelin's `AccessControlUpgradeable`, every custom role's admin defaults to `DEFAULT\_ADMIN\_ROLE` unless explicitly overridden. This means any account holding `DEFAULT\_ADMIN\_ROLE` can grant itself or others any role, including `UPGRADER\_ROLE`, effectively bypassing intended upgrade restrictions.



\*\*Real-world impact\*\*: Immunefi report #50784 (High severity, Plume Network) documents: "The DEFAULT\_ADMIN\_ROLE is the (implicit) admin of UPGRADER\_ROLE, so every Arc‑token creator can instantly grant themselves upgrade power, swap in malicious logic, and steal or freeze purchaser funds." The factory passed `msg.sender` as `DEFAULT\_ADMIN\_ROLE` when creating a token, and since "every custom role's admin defaults to DEFAULT\_ADMIN\_ROLE unless explicitly overridden," the creator could immediately call `grantRole(UPGRADER\_ROLE, self)` and gain unrestricted control over the proxy's implementation.



Immunefi report #52218 (High severity) documents the same pattern: "the creator of the token retains the DEFAULT\_ADMIN\_ROLE after deployment. In OpenZeppelin's access model, DEFAULT\_ADMIN\_ROLE has full authority over all roles, including the ability to grant or revoke the UPGRADER\_ROLE. This means that even if the creator is not initially assigned the upgrader role, they can later grant it to themselves or to any other address they control."



OpenZeppelin documentation states: "DEFAULT\_ADMIN\_ROLE must be managed only by itself. DEFAULT\_ADMIN\_ROLE must be held by only one account at a time." The recommended pattern enforces a 2-step process for transferring DEFAULT\_ADMIN\_ROLE to another account, with a recovery mechanism.



\*\*Audit checklist:\*\*



\- \[ ] `DEFAULT\_ADMIN\_ROLE` is not held by untrusted accounts after deployment

\- \[ ] Critical roles (`UPGRADER\_ROLE`, `PAUSER\_ROLE`, `MINTER\_ROLE`) have explicit admins where appropriate

\- \[ ] Role grant/revoke functions are themselves properly access-controlled

\- \[ ] No role can grant itself a more privileged role

\- \[ ] Renouncing `DEFAULT\_ADMIN\_ROLE` is either impossible or carefully managed

\- \[ ] Factory patterns do not leave creators with residual admin authority

\- \[ ] `\_setRoleAdmin` is used to tighten hierarchy where needed



\*\*Pattern to flag:\*\*



```solidity

// In factory:

token.grantRole(token.DEFAULT\_ADMIN\_ROLE(), msg.sender); // Creator keeps supreme admin

token.grantRole(token.UPGRADER\_ROLE(), address(this));   // Factory is upgrader



// Creator can later:

token.grantRole(token.UPGRADER\_ROLE(), attacker); // Bypass factory whitelist

```



\### Role Hierarchy Confusion



\- \[ ] Trace the full role graph: `DEFAULT\_ADMIN\_ROLE -> X\_ROLE -> Y\_ROLE`

\- \[ ] Verify that no lower-privileged role can escalate to a higher one

\- \[ ] Check whether `\_setRoleAdmin` is called to tighten hierarchy

\- \[ ] Confirm that `renounceRole` cannot be griefed by third parties



\## 16A.4 OpenZeppelin-Specific Bug Classes



| Bug Class | OpenZeppelin Component | Detection |

|---|---|---|

| Signature malleability | `ECDSA` | Version < 4.7.3, EIP-2098 compact signature acceptance |

| Privilege escalation | `TimelockController` | Version < 4.3.1 or < 3.4.2, executor role escalation |

| Reentrancy | `ERC777` | Custom `\_beforeTokenTransfer` with external calls before balance update |

| DoS | `ERC165Checker` | Version >= 2.3.0 < 4.7.2, unbounded `supportsInterface` |

| Memory disclosure | `Base64` | CVE-2024-27094, dirty memory read |

| Out-of-bounds read | `Bytes.lastIndexOf` | CVE-2025-54070, version >= 5.2.0 < 5.4.0, empty buffer with position |

| Governance manipulation | `GovernorVotesQuorumFraction` | Quorum fraction changes affecting past proposals |

| Proxy takeover | `Initializable` | Missing `\_disableInitializers()` |

| Access control bypass | `AccessControlUpgradeable` | `DEFAULT\_ADMIN\_ROLE` retained by untrusted actor |

| Inflation attack | `ERC4626Upgradeable` | Version < 4.9, low-TVL vaults |

| Storage collision | `Initializable` / modular delegatecall | Hardcoded slot identifiers across modules |



\## 16A.5 Storage Collision in Modular Delegatecall



Issue #5681 (May 2025): When OpenZeppelin upgradeable contracts are used in a modular delegatecall architecture (such as the Diamond Standard or similar plugin-style routers), fixed storage slot constants across multiple modules can cause critical storage collisions. Each module is executed in the same storage context (the proxy/Router), so repeated use of hardcoded slot identifiers like `INITIALIZABLE\_STORAGE` leads to overlapping writes between otherwise isolated modules. The proposed enhancement derives storage slot positions uniquely per module based on their deployed address using `keccak256(abi.encodePacked(\_\_self, "openzeppelin.storage.Initializable"))`. The issue notes that "current upgradeable contracts unsafe in modular delegatecall setups — an increasingly common pattern in modern contract systems."



Sherlock audit finding "Missing storage gap in upgradeable contract leads to storage collisions" documents: "The OperatorVaults abstract contract is designed for inheritance in an upgradeable proxy pattern but lacks storage gaps. Upgrading OperatorVaults with new state variables will cause storage collisions in child contracts, leading to data corruption and breaking contract functionality."



\*\*Audit checklist:\*\*

\- \[ ] If using modular delegatecall (Diamond or similar), verify storage slot uniqueness per module

\- \[ ] Every upgradeable base contract has appropriate storage gaps

\- \[ ] Inheritance hierarchy changes are checked for storage layout compatibility

\- \[ ] `\_\_gap` arrays are correctly sized and preserved



\## 16A.6 Version Checking Protocol



For every OpenZeppelin import:



1\. Identify the exact version in use (`package.json`, `foundry.toml`, `hardhat.config`, or lock file)

2\. Cross-reference against the vulnerability table above

3\. Check the OpenZeppelin Security Advisories page for the specific version

4\. Check Snyk and GitHub Advisory Database for the package

5\. If a vulnerable version is in use, trace the exact function and determine whether the vulnerable code path is reachable in the deployed context

6\. Do not report a version-level vulnerability as a finding unless the vulnerable code path is actually invoked with attacker-controllable input



\## 16A.7 OpenZeppelin Contract Inheritance Audit



OpenZeppelin contracts are inherited as source code, not linked as libraries. Developers frequently modify their local copies, which may break security checks that OpenZeppelin functions assume.



\*\*Audit approach:\*\*



\- Diff the inherited OpenZeppelin source against the canonical version

\- Identify every overridden function

\- For each override, verify that the override preserves the security checks of the original

\- Check that `super` is called where required

\- Check that modifiers (`onlyOwner`, `nonReentrant`, `whenNotPaused`) are preserved

\- Check that `\_beforeTokenTransfer` / `\_afterTokenTransfer` hooks do not introduce reentrancy or accounting inconsistencies



\*\*The "Using My Functions Should Follow My Checks" principle\*\*: OpenZeppelin functions carry both explicit checks (defined within the function) and implicit checks (conditions of calling the function). When developers override or customize, they often preserve the function but drop the checks. Audit for this pattern systematically.



\---



\# 17. Sui / Move Audit Core



Do not import EVM mental models blindly.



Model Sui as an object-centric execution system.



Audit:



\- object ownership

\- object IDs

\- object versions

\- mutable vs immutable references

\- shared objects

\- owned objects

\- capabilities

\- witness types

\- package/module visibility

\- `public`, `public(friend)`, and entry boundaries

\- resource abilities

\- transfer semantics

\- dynamic fields

\- object wrapping/unwrapping

\- transaction inputs

\- sender identity

\- clock / epoch dependencies

\- shared-object consensus ordering

\- authority execution

\- checkpoint / transaction sequencing

\- package upgrades

\- compatibility gates

\- Move abort paths

\- type identity

\- generic type confusion

\- resource conservation



\## Critical Move questions



\### Capability security



Find every capability/resource that grants authority.



For each capability:

\- who can create it?

\- who can copy it?

\- who can destroy it?

\- can it be wrapped?

\- can it be transferred?

\- can an attacker manufacture a look-alike type?

\- can a generic function accept an unintended capability?

\- can an object be forged through an alternate constructor?



\### Object security



For each mutable object:

\- who owns it?

\- who may mutate it?

\- can it become shared?

\- can it be transferred while logically attached?

\- can the same object be used twice?

\- are version/object-id assumptions enforced?



\### Entry-point security



For every externally callable Move function:

\- sender-controlled values

\- mutable objects

\- shared objects

\- type parameters

\- capabilities

\- clock/epoch values

\- vectors/maps/dynamic fields



Trace each input to privileged state changes.



\### Resource conservation



For token/coin/payment logic prove:



`inputs + authorized minting = outputs + authorized burning + fees`



Any mismatch is a prime candidate for Critical/High impact.



\---



\# 18. Sui Node / Core / Types / Execution



For validator or execution-layer code, audit security boundaries between:



\- transaction construction

\- signature validation

\- authority checks

\- object locking

\- consensus ordering

\- transaction execution

\- effects construction

\- certificate handling

\- checkpoint construction

\- epoch transitions

\- state synchronization

\- storage recovery

\- committee membership



Look for:



\- signature bypass

\- identity confusion

\- stale epoch acceptance

\- replay across epochs

\- object lock bypass

\- inconsistent transaction/effects validation

\- unsafe assumptions between consensus and execution

\- malformed object states accepted after deserialization

\- panic/unwrap paths reachable from network-controlled input

\- inconsistent validation between fast paths and fallback paths

\- state divergence

\- double execution

\- incorrect authority membership

\- checkpoint integrity failures

\- denial of service through expensive validation



A validator-layer issue is high/critical only when you can show a consensus, integrity, liveness, or asset-security consequence.



\---



\# 19. Sui Verifier / Adapter / Framework



For verifier and adapter logic inspect every rule that turns bytecode or transaction data into accepted execution.



Check:



\- type safety

\- bytecode validity

\- ability constraints

\- visibility rules

\- entry-function restrictions

\- package/module upgrade compatibility

\- signer checks

\- reference safety

\- object safety

\- invariant enforcement before execution

\- mismatches between verifier and runtime

\- bypasses caused by alternate execution paths



Prime exploit pattern:



`Verifier accepts state/code X -> runtime interprets X as Y -> attacker obtains authority or corrupts state`



When testing framework primitives, audit their downstream consequences, not only local implementation correctness.



\---



\# 20. Bridge Security



Treat bridge code as a separate trust system.



Model:



`source event/state -> proof -> message -> validator/relayer quorum -> destination mint/release`



Check:



\- replay protection

\- domain separation

\- chain-id/network separation

\- nonce uniqueness

\- message ID uniqueness

\- validator-set changes

\- epoch/committee transitions

\- threshold validation

\- signature aggregation

\- proof verification

\- source/destination binding

\- token mapping

\- decimal conversions

\- mint/burn conservation

\- message ordering

\- duplicate delivery

\- partial execution

\- retry semantics

\- emergency pause

\- recovery

\- upgrade authority



For a bridge issue, quantify whether it enables:

\- forged mint

\- double release

\- unauthorized withdrawal

\- stuck funds

\- validator takeover

\- replay



\---



\# 21. DeepBook / On-Chain Orderbook Security



Audit the economic state machine:



`place -> match -> cancel -> settle -> withdraw`



Check:



\- order identity uniqueness

\- owner authorization

\- balance reservations

\- available vs locked balance

\- partial fill accounting

\- price-time priority

\- self-trading rules

\- cancellation races

\- maker/taker fee accounting

\- rounding

\- tick size

\- lot size

\- min quantity

\- overflow

\- underflow

\- market order edge cases

\- oracle dependency

\- liquidation interactions

\- quote/base decimal mismatches

\- stale orders after capability/object changes



Test adversarial sequences:



`place -> partial fill -> cancel`



`place -> cancel -> reuse same identifier`



`place A -> place B -> match -> withdraw`



`many small orders -> rounding accumulation`



`crossed orders -> exact boundary price`



A finding needs a real extraction path, invariant violation, or denial of settlement.



\---



\# 22. SEAL / Key-Server / Cryptographic Infrastructure



For key management and cryptographic services, audit:



\- authorization to request keys

\- policy evaluation

\- identity binding

\- capability leakage

\- anti-replay controls

\- nonce / session handling

\- domain separation

\- access-policy bypass

\- threshold assumptions

\- secret material exposure

\- downgrade paths

\- malformed requests

\- cross-tenant isolation

\- audit-log integrity

\- availability attacks affecting key release



Never claim a cryptographic primitive is broken without proving exact cryptographic misuse.



\---



\# 23. Cross-Contract / Cross-Module Reasoning



Some of the highest-impact bugs only exist across modules.



Always perform:



\## Call graph analysis

Who calls whom?



\## Authority graph analysis

Which contract decides whether another contract is trusted?



\## Asset graph analysis

Where do tokens actually live?



\## State dependency analysis

Which state variables in A control security in B?



\## Upgrade graph analysis

Can A replace B, and can B affect A?



\## Initialization graph analysis

Can a partially initialized component be used by another component?



\## Failure-mode analysis



What happens when a downstream call:

\- reverts

\- returns zero

\- returns malformed data

\- returns stale data

\- consumes partial state

\- changes state before callback



\---



\# 24. Escrow Factory / Quest Payout Security



For escrow factories and quest/reward systems, model the lifecycle:



`factory initialization -> quest escrow creation -> token whitelisting -> quest activity -> reward distribution -> quest completion -> administrative withdrawal`



Audit these components separately from generic access control.



\## Factory trust boundaries



Map:

\- factory admin / `DEFAULT\_ADMIN\_ROLE`

\- per-quest escrow admin

\- quest/CUBE contract

\- escrow instance

\- treasury

\- whitelisted token set

\- payout recipient



For every privileged path, determine whether authority is global or scoped to one quest.



\## CREATE2 / deterministic escrow deployment



When a factory deploys escrows with deterministic salt:

\- verify uniqueness of salt namespace

\- verify `questId -> escrow` mapping cannot be replaced

\- check constructor arguments are correct and immutable where intended

\- inspect `CREATE2` address assumptions

\- check whether attacker can predict, predeploy, or interfere with expected address

\- ensure factory state is written consistently with actual deployed address



Never assume deterministic deployment is itself a security guarantee.



\## Per-quest admin security



For mappings such as `questId -> escrowAdmin`:

\- can unauthorized account set admin?

\- can admin be assigned to `address(0)`?

\- can quest admin update itself or another admin unexpectedly?

\- can stale admin privileges survive quest replacement or migration?

\- does global admin override create intended or unintended escape hatch?



Test exact precedence of:



`per-quest admin OR global admin`



\## Escrow token whitelist



For every add/remove whitelist action verify:

\- correct quest escrow selected

\- zero-address handling

\- token contract validation where needed

\- removal cannot strand funds unexpectedly

\- removed tokens cannot be withdrawn through unintended path

\- whitelist state consistent between factory and escrow

\- attacker cannot substitute token types or fake token addresses



\## Quest activity gating



If withdrawals are blocked while quest is active, prove factory and authoritative quest contract agree on:

\- quest identity

\- active/inactive state

\- completion semantics

\- pause/cancel semantics

\- reactivation behavior



Test race-like state transitions around quest completion and administrative withdrawal.



A local check such as `isQuestActive(questId)` is only meaningful if underlying CUBE state cannot be spoofed, bypassed, or confused.



\## Multi-token payout routing



Audit every token branch independently:



\- native currency

\- ERC20

\- ERC721

\- ERC1155



Check:

\- recipient authorization

\- amount authorization

\- token/tokenId binding

\- type confusion

\- zero-value behavior

\- reserve tracking

\- payout-vs-withdrawal separation

\- accidental arbitrary transfer primitives



For ERC721, ensure selected token ID is actually owned by intended escrow and cannot be substituted.



For ERC1155, verify `(token, tokenId, amount)` are bound together throughout full execution path.



For ERC20/native payouts, prove requested amount cannot exceed escrow's economically available reserve unless protocol explicitly intends partial withdrawal semantics.



\## Rake / fee accounting



For reward functions with `rakeBps` or similar:

\- verify denominator and maximum

\- verify who controls rake

\- ensure fee is not double-applied

\- confirm receiver gets intended net amount

\- ensure fee destination is not attacker-controlled

\- test boundary values: `0`, `1`, `max`, `MAX+1`



Trace arithmetic into escrow implementation rather than trusting factory parameters.



\## CUBE-only distribution boundary



If only canonical quest contract may distribute rewards:



`msg.sender == address(i\_cube)`



verify:

\- immutable dependency really is intended CUBE instance

\- no proxy can silently replace trust root

\- quest IDs cannot be confused across environments

\- CUBE cannot be tricked into invoking factory with attacker-chosen payout parameters

\- downstream escrow authorization still holds



The important question is not merely whether caller check exists; it is whether a caller under that trust boundary can be induced to authorize an attacker-selected asset transfer.



\## Administrative withdrawal boundary



Treat `withdrawFunds` as a high-risk custody primitive.



Prove:

\- only intended authority can call it

\- active quests cannot be drained

\- recipient is authorized

\- all token-type branches are safe

\- withdrawal of one token cannot accidentally affect another token's reserve

\- ERC721/1155 IDs cannot be redirected

\- compromised quest admin cannot steal funds during active quest if that is protocol invariant

\- global admin privileges match documented threat model



\## Escrow reserve invariants



For each escrow establish:



`native reserve >= committed native payouts + authorized excess`



`ERC20 reserve\[token] >= committed ERC20 payouts + authorized excess`



`ERC1155 reserve\[token]\[id] >= committed ERC1155 payouts + authorized excess`



For ERC721:



`each committed tokenId remains owned by escrow until valid payout/withdrawal`



Search for transitions that break those invariants.



\## Factory state consistency



Always compare:



`factory.s\_escrows\[questId]`



against



`actual deployed escrow`



and compare:



`factory.s\_escrow\_admin\[questId]`



against



`intended administrative authority`



Any path that desynchronizes these mappings from escrow's actual authorization model is high priority.



\---



\# 25. Tool-Assisted Audit Strategy



Use tools in this order.



\## Pass 1 — Repository reconnaissance

Identify language, compiler, framework, dependencies, tests, deployment scripts, upgrade scripts, CI, generated code, vendored code, external interfaces.



\## Pass 2 — Architecture extraction

Build contract/module inventory, trust graph, asset graph, privilege graph, call graph.



\## Pass 3 — High-value surface identification

Prioritize custody, mint/burn, withdrawals, upgrades, signatures, bridges, governance, accounting, external calls, shared objects, validators.



\## Pass 4 — Invariant analysis

Write key invariants.



\## Pass 5 — Attack-path enumeration

Try to break each invariant.



\## Pass 6 — PoC

Reproduce only credible paths.



\## Pass 7 — False-positive elimination

Challenge your own finding.



\## Pass 8 — Severity

Use demonstrated impact.



\## Pass 9 — Report

Deliver concise, technical, reproducible findings.



Useful tools:

\- Solidity: Foundry, Slither, Mythril, Echidna, Halmos, Tenderly

\- Move/Sui: `sui move test`, Move Prover, Sui CLI, local validator

\- Rust/Sui: `cargo test`, `cargo fuzz`, tracing, execution adapter tests

\- General: differential testing, fuzzing, symbolic execution, invariant testing, transaction tracing



Do not rely on tools alone. Tools find candidates; you prove findings.



\---



\# 26. PoC Guidance



\## Foundry / Solidity



Use tests such as:



\- `test\_exploit()`

\- `test\_upgrade\_authorization\_bypass()`

\- `test\_erc4626\_inflation()`

\- `test\_loss\_compensation\_overclaim()`

\- `test\_strategy\_withdrawal\_bypass()`

\- `test\_escrow\_payout\_bypass()`



Strong PoC:



```solidity

uint256 before = token.balanceOf(attacker);



// exploit sequence



uint256 after\_ = token.balanceOf(attacker);

assertGt(after\_, before);

```



For accounting bugs:



```solidity

assertLt(actualShares, expectedShares);

assertGt(attackerProfit, 0);

```



Use exact values and explain why they are economically meaningful.



\## Move / Sui



Prefer:

\- unit tests

\- Move tests

\- transaction scenarios

\- object state assertions

\- invariant/property tests



Show:

\- object ownership before

\- capability possession before

\- exact transaction inputs

\- state transition

\- resulting objects/balances

\- unauthorized authority or asset gain



For validator/execution bugs, show the smallest malformed or adversarial transaction that crosses the security boundary.



\## Rust / Node / Execution



Prefer:

\- focused unit tests

\- integration tests

\- transaction replay

\- state-delta assertions

\- fuzzing for panic/unwrap paths

\- differential validation between fast path and fallback path



\---



\# 27. Reporting Format



Use this exact structure.



\## \[SEVERITY] Title



\*\*Location\*\*



`path/to/file.sol:FunctionName`



\*\*Summary\*\*



One paragraph describing the vulnerability.



\*\*Root Cause\*\*



Explain the exact invariant or authorization failure.



\*\*Attack Path\*\*



1\. ...

2\. ...

3\. ...



\*\*Impact\*\*



Quantify what the attacker can steal, mint, freeze, manipulate, or control.



\*\*Proof of Concept\*\*



Give a minimal reproducible sequence or test.



\*\*Why Existing Checks Fail\*\*



Name the checks that appear protective and explain why they do not stop the exploit.



\*\*Fix\*\*



Give the smallest robust remediation.



\*\*Confidence\*\*



`High | Medium | Low`



\*\*Assumptions / Unknowns\*\*



List any unverified assumptions.



\---



\# 28. Reporting Discipline



Never inflate severity.



Never hide uncertainty.



Never state:



\- "this may allow theft"

\- "an attacker could potentially"

\- "likely exploitable"



without following with exact conditions and evidence.



Prefer:



> "An unprivileged caller can invoke X with Y. Because Z is not bound to the owner, the caller can transfer N units from A to B. The included test reproduces the balance increase."



Separate:



\- Confirmed findings

\- Observations

\- Questions

\- Out-of-scope issues

\- Tool warnings



\---



\# 29. Priority Heuristics \& Timeboxing



Spend most time on:



1\. Upgrade authorization

2\. Initialization / takeover

3\. Asset custody

4\. Share/accounting math

5\. Arbitrary call / delegatecall

6\. Signature verification

7\. Bridge validation

8\. Capability/object authority

9\. Loss / insurance compensation

10\. Cross-contract trust boundaries

11\. Oracle / pricing / slippage

12\. Emergency / pause / recovery

13\. Escrow / payout routing

14\. OpenZeppelin version and inheritance correctness

15\. `\_disableInitializers()` presence in upgradeable implementations

16\. `DEFAULT\_ADMIN\_ROLE` holder analysis



Spend less time on low-impact style issues until critical state machines are understood.



Suggested timebox:

\- 20% architecture and invariants

\- 40% high-value surfaces

\- 20% PoC and validation

\- 10% false-positive elimination

\- 10% reporting



\---



\# 30. Final Decision Gate



Before producing a finding, answer all six:



1\. \*\*Reachable?\*\*

2\. \*\*Attacker-controlled?\*\*

3\. \*\*Security invariant violated?\*\*

4\. \*\*Concrete impact?\*\*

5\. \*\*Reproducible?\*\*

6\. \*\*No existing mitigation invalidates the exploit?\*\*



If any answer is "no", do not present it as a confirmed vulnerability.



The goal is not to maximize the number of findings.



The goal is to maximize:



`real impact × exploitability × confidence ÷ false positives`



\---



\# 31. Security Review Checklist



\## EVM

\- \[ ] Initialization protected

\- \[ ] Upgrade authorization proven

\- \[ ] Storage layout verified

\- \[ ] Role graph verified

\- \[ ] Reentrancy paths verified

\- \[ ] External calls traced

\- \[ ] ERC20 edge cases tested

\- \[ ] ERC4626 invariants tested

\- \[ ] Strategy loss accounting tested

\- \[ ] Insurance payouts bounded

\- \[ ] Oracle / decimal assumptions tested

\- \[ ] Timelocks proven

\- \[ ] Emergency paths proven

\- \[ ] Governance manipulation tested

\- \[ ] Escrow payout routing tested

\- \[ ] CREATE2 assumptions tested



\## OpenZeppelin

\- \[ ] Exact version identified and cross-referenced against advisories

\- \[ ] No vulnerable function in reachable code path

\- \[ ] `\_disableInitializers()` in every upgradeable implementation constructor

\- \[ ] `DEFAULT\_ADMIN\_ROLE` holder is trusted

\- \[ ] Custom role admins explicitly set where needed

\- \[ ] Overridden functions preserve security checks

\- \[ ] `TimelockController` version is not vulnerable to executor escalation

\- \[ ] `ERC4626Upgradeable` version >= 4.9

\- \[ ] `ECDSA` version >= 4.7.3

\- \[ ] `ERC165Checker` version >= 4.7.2

\- \[ ] `Bytes` version >= 5.4.0

\- \[ ] `GovernorVotesQuorumFraction` quorum changes are timelocked

\- \[ ] Storage gaps present in upgradeable base contracts

\- \[ ] Modular delegatecall architectures use unique storage slots



\## Sui / Move

\- \[ ] Capability graph mapped

\- \[ ] Object ownership mapped

\- \[ ] Shared-object boundaries mapped

\- \[ ] Entry functions audited

\- \[ ] Type/witness assumptions checked

\- \[ ] Package upgrade paths checked

\- \[ ] Resource conservation proven

\- \[ ] Replay / epoch assumptions checked

\- \[ ] Bridge message integrity checked

\- \[ ] Validator/execution boundary checked

\- \[ ] Verifier/runtime consistency checked

\- \[ ] DEX balance/order invariants checked

\- \[ ] Key-release authorization checked



\## Bridge

\- \[ ] Replay protection

\- \[ ] Domain separation

\- \[ ] Nonce uniqueness

\- \[ ] Validator-set changes

\- \[ ] Threshold validation

\- \[ ] Source/destination binding

\- \[ ] Mint/burn conservation

\- \[ ] Emergency pause



\## Escrow / Quest

\- \[ ] Factory state matches deployed escrow

\- \[ ] Per-quest admin scoped correctly

\- \[ ] Token whitelist consistent

\- \[ ] Quest-active withdrawal blocking enforced

\- \[ ] Native/ERC20/ERC721/ERC1155 branches safe

\- \[ ] Rake accounting correct

\- \[ ] CUBE-only boundary holds

\- \[ ] Reserve invariants hold



\---



\# 32. Reference-Informed Focus Areas



This skill is shaped around patterns in the referenced projects.



\## Tetu-style EVM architecture



Reference set includes:

\- EIP-1967-style proxy implementation storage and controller-gated upgrades

\- strategy/splitter/controller trust relationships

\- strategy investment/withdrawal and loss accounting

\- ERC-4626-style share conversion and deposit/withdraw hooks

\- vault buffers, fees, withdrawal request delays, and insurance

\- voting-escrow NFTs with checkpoints, attachments, whitelisted transfers, and timelocked governance actions



High-value audit surfaces:

\- proxy initialization and upgrade authorization

\- strategy/splitter trust assumptions

\- loss tolerance and insurance compensation

\- ERC-4626 share math and rounding

\- vote-escrow checkpoint and attachment logic



\## Sui-style architecture



Reference set spans:

\- `sui-node`

\- `sui-core`

\- `sui-types`

\- Move external crates

\- execution adapter / verifier

\- framework

\- bridge components

\- DeepBook V3

\- SEAL key-server / Move components



Guide analysis toward:

\- object/capability authority

\- transaction and signature validation

\- validator/epoch state

\- consensus/execution boundaries

\- framework invariants

\- bridge message security

\- orderbook accounting

\- key-access policy boundaries



\## OpenZeppelin-derived architecture



Key vulnerability patterns from real-world audits and advisories:

\- Missing `\_disableInitializers()` in upgradeable implementation constructors (SCWE-092)

\- `DEFAULT\_ADMIN\_ROLE` retained by untrusted actor, enabling `UPGRADER\_ROLE` escalation

\- `TimelockController` executor role privilege escalation (CVE-2021-39167/39168)

\- `ECDSA` signature malleability via EIP-2098 compact signatures (CVE-2022-35961)

\- `Bytes.lastIndexOf` out-of-bounds read on empty buffers (CVE-2025-54070)

\- Storage collisions in modular delegatecall architectures

\- ERC-4626 inflation attacks despite virtual shares/offsets

\- Missing storage gaps in upgradeable base contracts

\- Overridden functions dropping security checks



\## Escrow / Quest architecture



Reference-informed example combines:

\- `AccessControlUpgradeable`

\- `UUPSUpgradeable`

\- deterministic `Escrow` deployment

\- per-quest admin mappings

\- CUBE-gated reward distribution

\- quest-active withdrawal blocking

\- token whitelisting

\- separate native/ERC20/ERC721/ERC1155 transfer paths



These combinations should trigger focused custody, authorization, deterministic-deployment, and token-type-confusion review.



\---



\# 33. Source References



\## Tetu Contracts V2 — EVM / Solidity



\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/proxy/ProxyControlled.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/proxy/UpgradeableProxy.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/strategy/StrategyBaseV2.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/strategy/StrategyLib.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/vault/ERC4626Upgradeable.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/vault/StrategySplitterV2.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/vault/TetuVaultV2.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/vault/VaultInsurance.sol

\- https://github.com/tetu-io/tetu-contracts-v2/blob/master/contracts/ve/VeTetu.sol



\## MystenLabs Sui — Node / Core / Types / Move / Execution



\- https://github.com/MystenLabs/sui/tree/testnet/crates/sui-node

\- https://github.com/MystenLabs/sui/tree/testnet/crates/sui-core

\- https://github.com/MystenLabs/sui/tree/testnet/crates/sui-types

\- https://github.com/MystenLabs/sui/tree/testnet/external-crates/move

\- https://github.com/MystenLabs/sui/tree/main/sui-execution/latest/sui-adapter

\- https://github.com/MystenLabs/sui/tree/main/sui-execution/latest/sui-verifier

\- https://github.com/MystenLabs/sui/tree/main/crates/sui-framework

\- https://github.com/MystenLabs/sui/tree/main/crates/sui-bridge

\- https://github.com/MystenLabs/sui/tree/main/bridge



\## MystenLabs Protocol / Security Infrastructure



\- https://github.com/MystenLabs/deepbookv3

\- https://github.com/MystenLabs/seal/tree/main/crates/key-server

\- https://github.com/MystenLabs/seal/tree/main/move/seal/sources

\- https://github.com/MystenLabs/sui/tree/bella-ciao



\## OpenZeppelin Security References



\### Security Advisories

\- OpenZeppelin Security Advisories: https://github.com/OpenZeppelin/openzeppelin-contracts/security/advisories

\- Snyk @openzeppelin/contracts: https://security.snyk.io/package/npm/@openzeppelin/contracts

\- GitHub Advisory Database: https://github.com/advisories?query=openzeppelin



\### Audit Reports

\- OpenZeppelin Audits: https://www.openzeppelin.com/security-audits

\- OpenZeppelin Contracts Audit README: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/audits/README.md



\### Key CVEs

\- CVE-2025-54070: Bytes.lastIndexOf out-of-bounds read

\- CVE-2024-27094: Base64 memory disclosure

\- CVE-2022-35961: ECDSA signature malleability

\- CVE-2021-39167 / CVE-2021-39168: TimelockController privilege escalation



\### OWASP SCWE

\- SCWE-092: Missing Disable Initializer in Constructor for Proxy Contracts

\- SCWE-054: Signature Malleability



\## Source-grounding note



The EVM focus was informed directly by the referenced Tetu implementations. The Sui focus was informed by the referenced repository structure and source-search results around authority, object locking, consensus, epochs, transactions, and execution. DeepBook V3 is a Move repository active on GitHub. The escrow/quest focus is informed by factory implementations that combine upgradeable access control, deterministic escrow deployment, per-quest admin mappings, CUBE-gated distribution, quest-active withdrawal blocking, whitelisting, and multi-token payout paths. The OpenZeppelin focus is informed by CVE records, OWASP SCWE entries, Immunefi audit reports, Zellic audit findings, and OpenZeppelin's own security advisories and documentation.



\---



\# 34. Appendix A — Common Invariants



\## Custody

`user funds can only move to authorized recipients under authorized conditions`



\## Mint/Burn

`minted supply <= authorized mint + prior supply - burned supply`



\## Share Accounting

`sum(user claims) <= redeemable assets + authorized liabilities`



\## Strategy

`reported assets \~= idle + invested - recognized losses - fees`



\## Insurance

`covered loss <= independently measured loss`



\## Bridge

`destination release <= valid source lock`



\## Vote Escrow

`voting power only from valid locked positions`



\## Orderbook

`free + locked = user balance`



\## Escrow

`reserve >= committed payouts + authorized excess`



\## Capability

`privileged transition requires valid capability`



\## Access Control

`no role can escalate to a more privileged role without explicit authorization`



\## Initialization

`implementation contract cannot be initialized directly; only proxy can initialize`



\---



\# 35. Appendix B — Exploit Primitives



\- Unauthorized initialization

\- Upgrade takeover

\- Storage collision

\- Role confusion

\- Signature replay

\- Nonce reuse

\- Domain separation failure

\- Oracle manipulation

\- Rounding extraction

\- Donation inflation

\- First-depositor inflation

\- Reentrancy

\- Read-only reentrancy

\- Callback state confusion

\- Arbitrary external call

\- Delegatecall misuse

\- Unsafe token handling

\- Fee-on-transfer assumption

\- Rebasing assumption

\- Decimal mismatch

\- Precision loss

\- Underflow/overflow

\- Denial of service

\- Unbounded iteration

\- CREATE2 address confusion

\- Type confusion

\- Capability forgery

\- Object ownership confusion

\- Shared-object ordering

\- Epoch replay

\- Verifier/runtime mismatch

\- Bridge message replay

\- Bridge validator takeover

\- Orderbook balance mismatch

\- Escrow reserve drain

\- Multi-token payout confusion

\- Missing `\_disableInitializers()` takeover

\- `DEFAULT\_ADMIN\_ROLE` escalation to `UPGRADER\_ROLE`

\- TimelockController executor privilege escalation

\- ECDSA signature malleability via EIP-2098

\- ERC165Checker DoS via unbounded interface query

\- ERC777 reentrancy via `\_beforeTokenTransfer`

\- Base64 dirty memory read

\- Bytes.lastIndexOf empty buffer OOB

\- Governor quorum retroactive manipulation

\- ERC-4626 inflation despite virtual shares

\- Storage collision in modular delegatecall



\---



\# 36. Appendix C — Severity Examples



\## Critical

\- Anyone can upgrade proxy to malicious implementation.

\- Bridge accepts replayed message and mints unlimited assets.

\- Vault share math allows attacker to drain all assets.

\- Move capability can be forged and used to mint coins.

\- `DEFAULT\_ADMIN\_ROLE` holder can grant `UPGRADER\_ROLE` to self and upgrade to malicious implementation.

\- Implementation contract can be initialized directly because `\_disableInitializers()` is missing.



\## High

\- Strategy loss tolerance allows repeated insurance drain.

\- Quest admin can drain active escrow despite invariant.

\- Governance timelock can be bypassed.

\- Orderbook allows locked balance to be withdrawn.

\- TimelockController executor can reset delay and escalate privileges.

\- ECDSA signature malleability enables replay protection bypass.



\## Medium

\- Griefing prevents withdrawals for a subset of users.

\- Rounding causes small but repeatable value leak.

\- Emergency pause can be abused to freeze funds temporarily.

\- ERC165Checker DoS with untrusted interface ID in loop.

\- ERC-4626 inflation attack unprofitable for attacker but griefs users.



\## Low

\- Defense-in-depth missing check with no current exploit.

\- Minor integrity issue requiring trusted actor.

\- Bytes.lastIndexOf OOB read that reverts under normal conditions.

\- Missing storage gap in abstract base contract without current upgrade impact.



\---



\# 37. Appendix D — Final Self-Review Checklist



Before delivering the audit, confirm:



\- \[ ] I read the actual code, not just names.

\- \[ ] I built a trust model.

\- \[ ] I wrote invariants.

\- \[ ] I traced full lifecycle.

\- \[ ] I identified attacker capabilities.

\- \[ ] I tested boundary values.

\- \[ ] I ran false-positive gate.

\- \[ ] I produced PoC or clear evidence.

\- \[ ] I separated findings from observations.

\- \[ ] I quantified impact.

\- \[ ] I gave minimal fixes.

\- \[ ] I marked confidence.

\- \[ ] I listed assumptions.

\- \[ ] I did not inflate severity.

\- \[ ] I did not report style as security.

\- \[ ] I checked OpenZeppelin version compatibility.

\- \[ ] I verified `\_disableInitializers()` presence.

\- \[ ] I traced `DEFAULT\_ADMIN\_ROLE` holder.

\- \[ ] I answered the six final decision gate questions.



If you cannot check every box, the audit is incomplete.
