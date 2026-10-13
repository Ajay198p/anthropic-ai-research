# Anthropic Claude: Integration Research

**Project:** anthropic-ai-research
**Research date:** 13 October 2026
**Scope:** Claude models, GitHub integration, GitLab integration, Eclipse IDE integration, and a Java/Spring Boot use case.

**Status labels used in this document**

| Label | Meaning |
|---|---|
| ✅ **Verified (docs)** | Confirmed against official Anthropic documentation or the official pricing page on the research date. |
| ⚠️ **Third-party** | Comes from a non-Anthropic source. Treat as unverified. |
| 🧪 **Not tested** | Has not been run hands-on by me or the project owner. Needs testing. |

Pricing, plans and beta features change often. Re-check the linked sources before relying on any number.

---

## 1. Executive summary

- **Models.** The current lineup on the official models page is Fable 5.1, Opus 5.5, Sonnet 5.5 and Haiku 5.5. All four have a 1M-token context window and 128K max output. Opus 5.5 is the recommended starting point for most workloads. ✅
- **GitHub.** There are three routes: the Claude.ai GitHub connector (read and discuss, beta), Claude Code on a local checkout, and Claude Code GitHub Actions (`@claude` in issues and PRs). The Actions route needs repository admin access. ✅
- **GitLab.** An official Claude Code GitLab CI/CD integration exists but is **beta and maintained by GitLab**. It runs Claude Code as a CI job. `@claude` mention triggers need extra webhook plumbing that you build yourself. ✅ The setup command used for GitHub does not work with GitLab. ✅
- **Eclipse.** I found no official Anthropic Eclipse plugin. Options are the Claude Code CLI next to Eclipse, an unofficial community plugin, or the separate Eclipse Theia IDE. ✅ (absence of an official plugin) / 🧪 (everything else)
- **Access.** Claude Code needs a Pro, Max, Team, Enterprise or Console account. The free plan does not include it. ✅
- **Privacy.** Training and retention depend on account type. Consumer plans (Free, Pro, Max) have a training opt-in setting with 5-year or 30-day retention. Commercial plans (Team, Enterprise, API) are not used for training by default. ✅
- **Testing.** No hands-on testing has been done yet. Every workflow in this document is 🧪 until the checklist in Section 10 is completed.

---

## 2. Claude model comparison and API pricing

### 2.1 Current models ✅

| | Claude Fable 5.1 | Claude Opus 5.5 | Claude Sonnet 5.5 | Claude Haiku 5.5 |
|---|---|---|---|---|
| **API model ID** | `claude-fable-5-1` | `claude-opus-5-5` | `claude-sonnet-5-5` | `claude-haiku-5-5` |
| **Positioned for** | Demanding reasoning and long-horizon agentic work | Long-running agentic coding and knowledge work | Best mix of speed and intelligence | High-volume, latency-sensitive tasks (classification, extraction, routing) |
| **Relative latency** | Slower | Moderate | Fast | Fastest |
| **Context window** | 1M tokens | 1M tokens | 1M tokens | 1M tokens |
| **Max output** | 128K tokens | 128K tokens | 128K tokens | 128K tokens |
| **Thinking** | Adaptive (always on) | Adaptive (always on) | Adaptive | Adaptive |
| **Default effort** | high | medium | high | medium |
| **Reliable knowledge cutoff** | Jun 2026 | Jun 2026 | Jun 2026 | Jun 2026 |

Source: [Models overview](https://platform.claude.com/docs/en/models/overview). The overview recommends starting with Opus 5.5 for most workloads, and moving to Fable 5.1 when evaluations on Opus 5.5 at higher effort still fall short.

Notes:
- All current models support text and image input, text output, multilingual use, vision and tool use.
- Anthropic also has a higher "Mythos" tier. I did not verify its API availability, and it is not in the overview table. Check [claude.com/pricing](https://claude.com/pricing) for current status.
- Legacy models still listed on the pricing page include Opus 5, Sonnet 5, Fable 5, Opus 4.8, Sonnet 4.6 and Haiku 4.5.

### 2.2 API pricing (USD per million tokens) ✅

| Model | Input | Output | Cache read | Cache write (5-min) |
|---|---|---|---|---|
| Fable 5.1 | $10 | $50 | $0.25 | $12.50 |
| Opus 5.5 | $4 | $20 | $0.20 | $5 |
| Sonnet 5.5 | $2 | $10 | $0.10 | $2.50 |
| Haiku 5.5 (prompts ≤ 100K tokens) | $0.10 | $0.50 | $0.01 | $0.125 |
| Haiku 5.5 (prompts > 100K tokens) | $0.50 | $2.50 | $0.05 | $0.625 |

Other pricing facts from the official pricing page:
- Batch processing gives a 50% discount.
- US-only inference is available at 1.1x pricing.
- Fast mode for Opus 5.5 is offered at 2x standard pricing.

Sources: [Pricing page](https://claude.com/pricing) and [API pricing docs](https://platform.claude.com/docs/en/about-claude/pricing).

---

## 3. GitHub integration

### 3.1 Options at a glance ✅

| Route | What it does | Needs | Beginner difficulty |
|---|---|---|---|
| **A. Claude.ai GitHub connector** | Adds repository content to a chat or Project so Claude can read and discuss it. Beta. | Claude.ai account; GitHub App access for private repos | Easiest |
| **B. Claude Code GitHub Actions** | Runs Claude Code in workflows; `@claude` in issues and PRs can analyze code, implement changes and push commits | Paid plan or API key; **repo admin**; GitHub CLI for quick setup | Moderate |
| **C. Claude Code locally** | Runs on your own checkout of the repo | Claude Code installed; paid plan or Console account | Easy |

### 3.2 Route A: Claude.ai connector 🧪

Steps (from the official help article):
1. In a chat, choose **Add from GitHub** from the menu, or in a Project select **GitHub**.
2. Search for a repository or paste its URL. Authenticate with GitHub if prompted.
3. Use **Configure files** to choose which files Claude reads. Start with a small subset.
4. For a private repository, follow the warning link to the GitHub App. You can grant access yourself (all repos or selected ones) or request it from an organization admin.
5. Click **Sync now** whenever the repository changes.

Sources: [Connector docs](https://claude.com/docs/connectors/github), [Help Center article](https://support.claude.com/en/articles/10167454-use-the-github-integration).

### 3.3 Route B: Claude Code GitHub Actions 🧪

Prerequisites ✅: repository admin access, the GitHub CLI installed and authenticated (`gh auth login`), and Claude Code installed (see Section 8.3).

Quick setup:
1. Open `claude` in the repository folder.
2. Run `/install-github-app` and follow the prompts. It installs the Claude GitHub App.
3. Choose an authentication method. It saves either `ANTHROPIC_API_KEY` (API key) or `CLAUDE_CODE_OAUTH_TOKEN` (subscription token for Pro, Max, Team or Enterprise) as a repository secret.
4. It pushes a branch with the workflow file(s) and opens GitHub with a pull request ready. Create and merge that PR.
5. Test by commenting `@claude explain this project's structure` on an issue or PR.

Manual setup, if the command fails: install the [Claude GitHub App](https://github.com/apps/claude), add the secret in the repository's Actions secrets, and copy the example workflow from the [claude-code-action repository](https://github.com/anthropics/claude-code-action) into `.github/workflows/`.

Troubleshooting checklist ✅: app installed on the repo, workflows enabled, secret present, comment contains `@claude` as a whole word, and the commenter has write access.

Source: [Claude Code GitHub Actions docs](https://code.claude.com/docs/en/github-actions).

### 3.4 Route C: Local Claude Code 🧪

Install Claude Code (Section 8.3), `cd` into your cloned repository, run `claude`, and ask questions about the code.

---

## 4. GitLab integration options

### 4.1 Status ✅

- The official Claude Code GitLab CI/CD integration is **beta**. It is **maintained by GitLab**; support is via a [GitLab issue](https://gitlab.com/gitlab-org/gitlab/-/issues/573776).
- It runs Claude Code in an isolated CI job on your GitLab runners. Results come back as merge requests, so branch protection and approvals still apply.
- Supported providers: Claude API, Amazon Bedrock (OIDC), Google Cloud Agent Platform (Workload Identity Federation).
- `/install-github-app` only works with github.com repositories. It exits if the remote is gitlab.com or bitbucket.org.

### 4.2 Options

| Option | Effort | Notes |
|---|---|---|
| **Manual-trigger CI job** | Low | Add `ANTHROPIC_API_KEY` as a masked variable plus a job in `.gitlab-ci.yml`; run it from **CI/CD → Pipelines**. Recommended starting point. |
| **`@claude` mention triggers** | High | Docs describe it as optional. You add a project webhook for comments and an event listener that calls the pipeline trigger API. You must build the listener. |
| **Bedrock or Google Cloud via OIDC** | High | For enterprise data residency and no stored keys. Requires cloud-side setup. |
| **Local Claude Code on a GitLab checkout** | Low | My inference, not a documented GitLab feature: Claude Code works on any local git checkout. 🧪 |

### 4.3 Beginner steps (manual trigger) 🧪

1. Push your project to GitLab.
2. Create an API key in the [Claude Console](https://platform.claude.com).
3. In GitLab: **Settings → CI/CD → Variables**, add `ANTHROPIC_API_KEY` and mark it **masked**.
4. Add this `.gitlab-ci.yml` (simplified, read-only adaptation of the documented job; **not run**):

```yaml
stages: [ai]

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'   # runs when started manually from the UI
  before_script:
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
    - export PATH="$HOME/.local/bin:$PATH"
  script:
    - claude -p "Review this repo and list potential bugs" --allowedTools "Read"
```

5. Go to **CI/CD → Pipelines → Run pipeline** and read the job log.

The official example also grants write tools and a GitLab MCP server so Claude can open merge requests. Add those only after the read-only version works.

Source: [Claude Code GitLab CI/CD docs](https://code.claude.com/docs/en/gitlab-ci-cd).

---

## 5. Eclipse IDE integration options

### 5.1 What exists

| Option | Official? | Summary | Status |
|---|---|---|---|
| **Claude Code CLI beside Eclipse** (external terminal or Eclipse's Terminal view) | Official CLI | The workaround cited in feature requests. Lacks IDE features such as diffs and selection sharing. | ✅ (workaround cited) / 🧪 (Eclipse Terminal view) |
| **Community plugin "Claude Code for Eclipse IDE (Unofficial)"** | ❌ Unofficial | By eilonwy06. Not affiliated with or endorsed by Anthropic. MIT-licensed. | ⚠️ 🧪 |
| **Eclipse Theia IDE** | Eclipse Foundation | A separate IDE platform (not the Eclipse IDE) with native Claude Code integration. | ⚠️ Not evaluated |
| **JetBrains IDEs (IntelliJ IDEA)** | ✅ Official plugin (beta) | A documented fallback if Eclipse is not mandatory. | ✅ |

Findings:
- No official Anthropic Eclipse plugin was found in Anthropic's docs. An [open feature request](https://github.com/anthropics/claude-code/issues/30914) asks for one. A second request was closed in May 2026, but I could not confirm that it resulted in a plugin. ✅
- The plugin's GitHub README lists: Eclipse 2025-09 or later, Java 21 or later, the Claude Code CLI on `PATH`, and an Anthropic account or API key. It installs through **Help → Install New Software** using a p2 update site (URL in its README). ⚠️
- Its Eclipse Marketplace listing says **Windows** and Eclipse **2026-03**, which conflicts with the README's multi-platform claim. Treat compatibility as unverified. ⚠️
- The README suggests installing the CLI via npm with Node.js 18+. Anthropic's current docs recommend the native installer, and npm requires Node.js 22 or later. The plugin only needs `claude` on `PATH`. ✅

### 5.2 Beginner steps 🧪

**Option 1: CLI beside Eclipse (recommended first)**
1. Install Claude Code (Section 8.3).
2. In Eclipse, open the Terminal view (**Window → Show View → Other → Terminal**) or use any external terminal.
3. Start a terminal in your project folder and run `claude`.

**Option 2: Community plugin (only after Option 1 works)**
1. Confirm prerequisites above, and that `claude --version` works.
2. **Help → Install New Software → Add**, paste the update-site URL from the [plugin README](https://github.com/eilonwy06/claudecode-eclipse-ide), select the feature, and restart Eclipse.
3. Open **Window → Show View → Other → Claude Code**.
4. Review the plugin's security policy first. Keep "Allow bypass permissions mode" **off** (the README says it is off by default).

---

## 6. Java / Spring Boot sample use case 🧪

This example is my own and has **not been run**. It contains three deliberate flaws so you can check whether Claude finds them.

### 6.1 Setup
1. Generate a Maven project at [start.spring.io](https://start.spring.io) (Java, Spring Web, JUnit).
2. Add the class below and push the project to GitHub.

```java
package com.example.shop;

import java.math.BigDecimal;

public class PriceService {
    // Intended: 10% off at 100 or more, 5% off at 50 or more
    public BigDecimal applyDiscount(BigDecimal total) {
        if (total.compareTo(new BigDecimal("100")) > 0) {
            return total.multiply(new BigDecimal("0.90"));
        }
        if (total.compareTo(new BigDecimal("50")) > 0) {
            return total.multiply(new BigDecimal("0.95"));
        }
        return total;
    }
}
```

Planted flaws (answer key):
1. Boundary bug: `> 100` and `> 50` should be `>= 100` and `>= 50`.
2. A `null` total causes a `NullPointerException`.
3. Results are not rounded to 2 decimal places.

### 6.2 Test workflow
1. In `claude`, ask: *"Explain this project's structure."*
2. Ask: *"Review PriceService against the comment in the code and list bugs. Do not edit files."*
3. Ask: *"Write JUnit tests for the boundaries at 50 and 100, plus a null input."*
4. Run `./mvnw test`. Failing tests should expose the flaws.
5. Let Claude propose fixes, review the diff before approving, and re-run the tests.

### 6.3 What to record

| Item | Result |
|---|---|
| Model used | |
| Flaws found (of 3) | |
| Extra or false findings | |
| Tests passed / failed | |
| Usage or cost | |
| Limitations noticed | |

---

## 7. Security and privacy considerations

### 7.1 Training and retention ✅

| Account type | Used for model training? | Retention |
|---|---|---|
| Free, Pro, Max (consumer) | Only if you allow it (setting on claude.ai) | 5 years if allowed; 30 days if not |
| Team, Enterprise, API (commercial) | Not by default under commercial terms | 30 days standard |
| Enterprise with zero data retention | Not by default | Available to qualified accounts only, enabled per organization |

Source: [Claude Code data usage](https://code.claude.com/docs/en/data-usage).

### 7.2 Other points ✅
- **Local transcripts.** Claude Code stores session transcripts locally in plaintext (under `~/.claude/projects/`) for 30 days by default.
- **Feedback commands.** `/feedback`, `/bug` and `/share` send conversation history, including code, to Anthropic.
- **Telemetry.** Operational metrics are sent by default with the Claude API. They exclude code, prompts and file paths. Opt-out variables are documented.
- **Permissions.** Claude Code is read-only by default and asks before editing files or running commands. Writes are limited to the folder where it started. You are responsible for reviewing what you approve.
- **Prompt injection.** The docs describe safeguards, but untrusted content in a repository (issues, comments, files) can still be a risk. Review actions, especially in CI. (General caution; my addition.)
- **GitHub App breadth.** The official app requests a wide permission set and GitHub does not allow accepting a subset. A custom GitHub App with only Contents, Issues and Pull requests is a documented alternative for the Actions workflow.
- **Secrets.** Never commit API keys or tokens. Use GitHub Secrets or masked GitLab CI/CD variables. A subscription OAuth token is tied to the person who generated it.
- **Third-party plugins.** The community Eclipse plugin is not covered by Anthropic's security documentation. Review it before use. ⚠️
- **Company policy.** Confirm that your employer allows sending source code to external AI services before using any route.

Sources: [Data usage](https://code.claude.com/docs/en/data-usage), [Security](https://code.claude.com/docs/en/security), [GitHub Actions docs](https://code.claude.com/docs/en/github-actions).

---

## 8. Costs, limitations and prerequisites

### 8.1 Plans (Claude.ai) ✅

| Plan | Price (from official pricing page) | Claude Code |
|---|---|---|
| Free | $0 | **No** |
| Pro | $17/month billed annually ($20 monthly) | Yes |
| Max | From $100/month (5x or 20x Pro usage) | Yes |
| Team | Standard $20 annual / $25 monthly per seat; Premium $100 annual / $125 monthly per seat | Listed as included; confirm seat-level details before buying |
| Enterprise | $20/seat/month plus usage at API rates | Yes |
| Console (API) | Pay per token (Section 2.2) | Yes |

- Claude Code usage shares one pool with chat usage. Plans have 5-hour and weekly limits, and extra usage can be bought on paid plans.
- Plans and prices are subject to change.

Source: [claude.com/pricing](https://claude.com/pricing).

### 8.2 Cost by integration route ✅

| Route | Cost drivers |
|---|---|
| Claude.ai connector | Counts against your plan's usage |
| GitHub Actions | GitHub Actions minutes plus API tokens (or subscription usage if using an OAuth token) |
| GitLab CI/CD | GitLab runner time plus API tokens |
| Local Claude Code | Subscription usage or API tokens |

Cost controls documented for CI: set `--max-turns`, set job timeouts, write specific requests, keep `CLAUDE.md` short, and limit concurrency.

### 8.3 Installing Claude Code (prerequisites) ✅

- **Supported systems:** macOS 13+, Windows 10 1809+, Ubuntu 20.04+, Debian 10+, Alpine 3.19+; 4 GB+ RAM; internet connection; a supported country.
- **Install:**
  - macOS, Linux, WSL: `curl -fsSL https://claude.ai/install.sh | bash`
  - Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`
  - Windows CMD: `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd`
  - npm is also supported and requires Node.js 22 or later.
- **Verify:** open a new terminal and run `claude --version`. Use `claude doctor` for diagnostics.
- **Sign in:** run `claude` and follow the browser prompts.
- Git for Windows is recommended on native Windows (it enables the Bash tool).

Source: [Advanced setup](https://code.claude.com/docs/en/setup).

### 8.3.1 Other prerequisites
- GitHub Actions route: repository admin, GitHub CLI.
- GitLab route: a GitLab project with CI/CD runners, and an API key (Claude Console) or cloud provider access.
- Eclipse community plugin: Eclipse 2025-09+, Java 21+ (⚠️ per plugin README).

### 8.4 Limitations
- AI-generated code can be wrong and must be reviewed and tested before merging.
- The GitLab integration and the Claude.ai GitHub connector are **beta**.
- No official Eclipse plugin was found.
- The `@claude` mention flow on GitLab requires custom webhook infrastructure.
- Context is large (1M tokens) but not unlimited. The connector requires repositories to fit within the context window.
- Usage limits can interrupt long sessions on subscription plans.
- Model names, prices and features change frequently.

---

## 9. Official source links

**Models and pricing**
- Models overview: https://platform.claude.com/docs/en/models/overview
- API pricing: https://platform.claude.com/docs/en/about-claude/pricing
- Plans and pricing: https://claude.com/pricing

**GitHub**
- Connector docs: https://claude.com/docs/connectors/github
- Help Center article: https://support.claude.com/en/articles/10167454-use-the-github-integration
- Claude Code GitHub Actions: https://code.claude.com/docs/en/github-actions
- Claude GitHub App: https://github.com/apps/claude
- claude-code-action repository: https://github.com/anthropics/claude-code-action

**GitLab**
- Claude Code GitLab CI/CD: https://code.claude.com/docs/en/gitlab-ci-cd
- GitLab support issue: https://gitlab.com/gitlab-org/gitlab/-/issues/573776

**Claude Code, IDEs and security**
- Setup and installation: https://code.claude.com/docs/en/setup
- JetBrains plugin: https://code.claude.com/docs/en/jetbrains
- Data usage: https://code.claude.com/docs/en/data-usage
- Security: https://code.claude.com/docs/en/security

**Non-Anthropic sources (not official)**
- Community Eclipse plugin: https://github.com/eilonwy06/claudecode-eclipse-ide
- Eclipse Marketplace listing: https://marketplace.eclipse.org/content/claude-code-eclipse-ide
- Eclipse plugin feature request: https://github.com/anthropics/claude-code/issues/30914
- Eclipse Theia announcement: https://newsroom.eclipse.org/node/43152

---

## 10. Hands-on testing status and next steps

### 10.1 Testing status

| Item | Status |
|---|---|
| Model IDs and pricing | ✅ Verified against official pages on 13 Oct 2026 |
| Claude.ai GitHub connector steps | 🧪 Not tested |
| `/install-github-app` and `@claude` workflow | 🧪 Not tested |
| GitLab manual-trigger job | 🧪 Not tested (YAML is a simplified adaptation) |
| GitLab `@claude` webhook flow | 🧪 Not tested; not attempted |
| Claude Code in Eclipse's Terminal view | 🧪 Not tested |
| Community Eclipse plugin | 🧪 Not tested; compatibility unclear |
| Spring Boot sample and answer key | 🧪 Not run |
| Cost per run | 🧪 Not measured |

### 10.2 Next steps
1. Install Claude Code and confirm `claude --version` and sign-in work.
2. Create the Spring Boot sample project and run the Section 6 workflow locally. Fill in the results table.
3. Test Route A (Claude.ai connector) on your repository.
4. Test GitHub Actions on a throwaway repository with a spending-limited API key.
5. Test the GitLab manual-trigger job.
6. Test Claude Code in Eclipse's Terminal view; try the community plugin only if you accept the risks in Section 7.
7. Compare Opus 5.5 and Sonnet 5.5 on the same task for quality and cost.
8. Check the Team-plan seat details and your organization's data policy before any work use.
9. Update `claude-models.md` with the current lineup and mark its "Research Status" section accordingly.
10. Write final recommendations once the results are recorded.

---

*Document status: research draft. Facts were verified against the official sources above on 13 October 2026. Nothing in this document has been tested hands-on.*
