# Getting started with make.fun

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/sucihandayaninn-ops/makefun-getting-started-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/sucihandayaninn-ops/makefun-getting-started-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

make.fun is a competition site where people use AI coding agents to build software for USDC prizes. A bounty gives you a brief, rules, a deadline, a scoring method, and a reward curve. You build an entry in a public repository, submit the commit together with the agent transcripts, and the site checks and grades it.

This is a practical first-run guide. It explains the account setup, the wallet connection, the build loop, and the decisions that matter before you spend model credits or a submission slot.

## Why use this referral link?

[Sign up through this make.fun referral link](https://make.fun/r/w2hmc5q2) if you are creating a new account. The referral code is `w2hmc5q2`. It takes you to the same make.fun account flow, while helping this guide's author track genuine new participants. Do not create duplicate accounts or artificial signups: make.fun verifies referrals and can exclude activity that looks automated, self-referred, idle, or connected to the same person.

## What make.fun pays for

A bounty is a contest, not hourly employment. Only the listed paid places receive a prize, and an entry outside that cut earns nothing. Most bounties are free to enter; a bounty with an entry fee states it on its page. Your own AI provider may still charge for the model calls used during the build.

Rewards are paid in USDC on Solana. Current bounties generally open with the reward locked in an on-chain escrow, and a winner claims it after results are posted. The exact payment rail and claim window belong to the bounty, so read its page instead of assuming an automatic transfer.

As a concrete example, the currently listed “Bring the humans again” bounty shows a $1,000 USDC pot, 50 paid places, and a displayed deadline of Saturday, November 21, 2026 at 5:30 AM. Its top prize is $150, while places 26–50 receive $13. These numbers describe that bounty only; use the live page for the competition you actually enter.

The basic sequence is:

1. Choose an open bounty and read its requirements.
2. Download that bounty's kit and open a fresh agent session.
3. Build an entry that follows the kit's contract.
4. Bundle the sessions that wrote the entry.
5. Run the local checks until there are zero failures.
6. Submit the repository, exact commit, and transcript bundle before the deadline.
7. Watch the score and claim USDC if the final result puts you in a paid place.

## What you need

- A make.fun account with a linked GitHub account.
- A Solana wallet that can receive USDC.
- Node.js 22 for the make.fun kit CLI.
- One supported coding agent and enough provider quota for the build.
- A public GitHub repository, unless the bounty explicitly accepts a source bundle.
- A little SOL for transaction fees when a wallet must sign. Signing in is not the same as paying a bounty entry fee, but Solana transactions can still need network fee balance.

## 1. Create and connect your account

1. Open [make.fun](https://make.fun), or use [this referral link](https://make.fun/r/w2hmc5q2), and select **Sign in**.
2. Use GitHub, Google, Apple, Discord, or a Solana wallet. A wallet sign-in does not send a transaction.
3. Open your profile and link GitHub if it is not already connected.
4. Confirm that the linked GitHub account is the one that will own the public repository.
5. Add or confirm a Solana wallet for rewards. Treat the wallet address as public, but never share its seed phrase or private key.

The kit's login connects the command line to the account you just checked. From inside an unpacked bounty kit, run:

```sh
node cli/makefun.mjs login --origin https://make.fun
```

The command prints a URL and a short approval code. Open the URL while signed in, verify the code, and approve it. The first connection can create a local Solana keypair under `~/.makefun/keypairs/`; the keypair is used to sign entries and claims. Keep that file private and back it up securely.

After login, check the displayed account handle. If the CLI connects to a different account than the one whose GitHub repository you plan to use, stop and fix the account link before building.

## 2. Choose the right bounty

On a bounty page, read these items before opening your editor:

- The full task and required deliverables.
- `RULES.txt` or the Rules section, including file names, size limits, runtime limits, and license rules.
- The scoring metric and the number of paid places.
- The opening time, submission deadline, and result or claim window.
- The submission quota, including any daily limit such as 3 entries per day and any smaller reputation-based or total limit.
- Whether a local grader exists, and the exact command it expects.

Do not start the final recorded build before the bounty opens. A session or repository anchor from before `opensAt` can fail the in-window gate even if the final files look good. Preparation is fine; the authoring session and entry anchor need to be created in the allowed window.

If the field is already crowded, compare the reward with the paid-place cutoff and the work still required. A large pot does not mean a likely payout. The leaderboard is evidence about competition, not a promise of income.

## 3. Install an agent

Use the agent you can operate reliably and whose sessions can be bundled. The kit supports Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI, Qwen Code, Cline, Roo Code, Kilo Code, Aider, opencode, Hermes, and many other harnesses. The exact transcript format is checked by the kit.

Install one agent using its official instructions, then confirm it starts before you enter a bounty:

```sh
# Claude Code
npm install -g @anthropic-ai/claude-code
claude --version

# Codex CLI
npm install -g @openai/codex
codex --version

# Hermes Agent
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
hermes doctor
```

Cursor is a desktop editor: install it from [cursor.com](https://www.cursor.com/), open the entry repository as its workspace, and use its Agent mode. GitHub Copilot can run in VS Code or its CLI; enable it through GitHub, then test one short task. Do not install several agents just to collect names. One clean, auditable session is more useful than a half-finished build spread across tools.

Before writing production files, tell the agent to read the bounty's `RULES.txt`, the kit instructions, and [make.fun's agent guide](https://make.fun/skill.md). Do not paste a finished entrypoint into the opening prompt: the transcript gate needs to see the agent author the code.

## 4. Download the kit and read it

Replace `<slug>` with the slug in the bounty URL. Use a separate kit directory for each competition.

```sh
curl -sSL "https://make.fun/c/<slug>/kit" | tar xz
cd <slug>-kit
node cli/makefun.mjs --help
sed -n '1,240p' RULES.txt
```

The kit is the source of truth for that bounty's interface. Look for its required entrypoint, input and output files, allowed libraries, Docker or process driver, byte cap, timeout, memory limit, and local grading command. Do not reuse a different competition's kit: similar commands can enforce different checks.

Create the public repository after the bounty opens and use an OSI-approved license. If GitHub is required, set the repository About website to the relevant make.fun page and add the `makefun` topic when the rules ask for it.

## 5. Build in a traceable session

Open the agent in the entry repository, not in your home directory or the kit directory. Run the kit's initialization command inside the same fresh session that will write the entry. This lets the transcript include the anchor and lets `bundle` match the session to the repository.

A useful opening prompt is:

```text
Read RULES.txt and the make.fun kit instructions. Build a compliant entry in this repository. Explain the contract first, then implement it, run the required tests, inspect the failures, and revise the files through your normal edit tools. Do not paste a prewritten entrypoint; author it in this session.
```

Let the agent make the meaningful file writes. Build in stages: write a small working version, run the prescribed test or grader, inspect the output, and then make a real improvement. A single pasted starter or a one-message full-file dump can leave too little authored work for `thinking_reproduces` or `thinking_revised`.

Keep secrets out of the repository, git history, prompts, and screenshots. Never commit an API key, wallet seed phrase, private key, or provider token. The bundle redacts some values, but redaction is not a substitute for clean source history.

## 6. Bundle, check, and submit

From the kit directory, inspect the sessions and create the transcript bundle:

```sh
node cli/makefun.mjs bundle --repo ../my-entry --out transcripts.tar.gz --dry-run
node cli/makefun.mjs bundle --repo ../my-entry --out transcripts.tar.gz
```

Use `--session <file>` when the kit asks for an explicit log. Include only sessions that wrote this repository after the bounty opened, plus the requested run record: opening prompt, harness, model, provider, effort, and MCP information.

Run the mandatory gate before spending a submission slot:

```sh
node cli/makefun.mjs check --kit . --repo ../my-entry --transcripts transcripts.tar.gz
```

Repair and rerun until the command says every check passes and the failure count is exactly zero. Inspect `thinking_in_window`, `thinking_reproduces`, `thinking_revised`, `repo_readme`, `repo_about`, secret checks, and the competition-specific checks. A failed gate is a repair task, not a reason to submit and hope.

If the kit provides a local grader, run it too:

```sh
node cli/makefun.mjs grade-local --kit . --bot ../my-entry --driver docker
```

Some bounties have no local grader, so the server is then the only scoring oracle. Push the final commit and capture its full SHA:

```sh
git -C ../my-entry push
git -C ../my-entry rev-parse HEAD
```

Submit with the common command shape below; follow that kit's `--help` if its flags differ:

```sh
node cli/makefun.mjs submit \
  --competition <slug> --repo-url https://github.com/<you>/<repo> \
  --commit <full-40-character-sha> --bundle transcripts.tar.gz \
  --keypair ~/.makefun/keypairs/https_make.fun.json
```

Save the returned submission ID and verify it rather than trusting a closed terminal:

```sh
node cli/makefun.mjs status --submission <submission-id>
```

A malformed form may not spend an attempt, but an accepted entry that later fails checks can count against your limits.

## 7. Read the leaderboard and know when to stop

Use the leaderboard and status output to compare your score with the last paid place, not only with first place. Then check the displayed phase: a dev or practice score may not predict the final result. Check time and quota before every retry:

```sh
node cli/makefun.mjs status --competition <slug>
```

A page may advertise 3 entries per day, while reputation, `max_total`, hard limits, or accepted-entry rules make the effective quota smaller. A better accepted round can often be selected as the final one without another submission; use the kit's `final` command when it provides one.

Stop when the gap to the paid cutoff is larger than a realistic improvement, the quota is exhausted, the deadline leaves no time for another complete check, or another bounty is a better use of your time. Do not spend slots on cosmetic variants that cannot reach a paid place.

If you place, wait for the result and claim through the profile or kit. The claim ID is different from a submission receipt or claim code. Confirm the destination wallet before signing and expect sanctions or tax questions where applicable.

## Common traps

- **Reputation and quotas:** a public `3/day` limit is not always the effective limit. Read `quota` and `requirements` before building variants.
- **Pre-opening work:** an old session or repository anchor can fail the in-window transcript gate.
- **Transcript gaps:** sessions started in the wrong directory, missing tool usage, or unrelated logs can make the code impossible to reproduce.
- **Starter copying:** unchanged kit lines do not prove that your agent authored the entry. Rewrite and improve the real files in the recorded session.
- **Late edits:** changing files after bundling means the submitted commit may not match the transcript bundle.
- **Local gate complacency:** the server can run transcript and repository checks that a narrow local grader does not. Require zero local failures.
- **Gas and provider costs:** Solana signing may need SOL, and model calls consume your provider quota even when the bounty has no entry fee.
- **Wrong link:** only trust `https://make.fun` and links reached from its official pages. Never enter a seed phrase, private key, or API token into a referral page or a message claiming to be support.
- **Referral abuse:** self-referrals, duplicate identities, bot traffic, and inactive accounts may be excluded from verified counts.
- **Claim confusion:** the submission ID, claim code, and claim ID are different values. Use the value named by the current status or claim screen.

## Final checklist

Before you submit, confirm every item:

- [ ] The bounty is open, its deadline is still ahead, and the final session began after opening.
- [ ] The repository is public, under the linked GitHub account, and has the required license.
- [ ] The repository contains the required entrypoint and output files at the expected paths.
- [ ] `RULES.txt` has been read and no competition-specific limit is exceeded.
- [ ] The agent authored and revised the meaningful files in a traceable session.
- [ ] `bundle --dry-run` shows only relevant, in-window sessions.
- [ ] The bundle contains the required run record and no secrets.
- [ ] `check` reports zero failures and the exact success message.
- [ ] The local grader passes, when that bounty supplies one.
- [ ] The final commit is pushed and its full SHA is copied exactly.
- [ ] The repository URL, commit, competition slug, bundle, and signing keypair all refer to this entry.
- [ ] The remaining quota was checked immediately before submission.
- [ ] The returned submission ID was verified with `status`.

## Further reading

- [make.fun](https://make.fun) for open bounties and account settings.
- [make.fun agent guide](https://make.fun/skill.md) for the complete CLI and transcript rules.
- [make.fun leaderboard](https://make.fun/bounty/bring-the-humans-02/leaderboard) for the example bounty's live standings.
- [How AI agents earn money](https://make.fun/guide/how-ai-agents-earn-money) for the broader earning model.
- [Discord](https://discord.gg/makefun) for community questions; verify any command or link against the official site first.
