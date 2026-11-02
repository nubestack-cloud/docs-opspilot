---
hide:
  - toc
---

<div class="opspilot-hero" markdown>

<svg class="opspilot-hero-mark" viewBox="8 8 48 48" role="img" aria-label="OpsPilot">
  <rect class="opspilot-mark-frame" x="10" y="10" width="44" height="44" rx="11"/>
  <path class="opspilot-mark-prompt" d="M22 26 L31 33 L22 40"/>
  <rect class="opspilot-mark-cursor" x="34" y="36.5" width="11" height="4" rx="2"/>
</svg>

# OpsPilot

**AI on your terminal — including the terminals AI can't normally reach.**

[Get started](getting-started/quickstart.md){ .md-button .md-button--primary }
[Download](https://subscription.nubestack.com/download){ .md-button }
[Pricing](https://subscription.nubestack.com/opspilot){ .md-button }
[Why OpsPilot](overview/why-opspilot.md){ .md-button }

</div>

![The OpsPilot workspace: saved connections with their AI badges on the left, an SSH session in the centre where a broken nginx configuration was fixed, and the AI panel on the right listing the three commands that ran, each with its risk tier, followed by a summary](assets/images/19-ai-summary.png){ .opspilot-hero-shot }

OpsPilot is a desktop operations workbench — a terminal, a file manager, a
remote-desktop client and an AI assistant in one application. You connect the way
you always have — SSH, RDP, VNC, FTP, S3, serial, or a local shell — and in SSH
and local shell sessions OpsPilot watches the session with you. Ask it a question
and it reads the scrollback, strips the secrets out locally, and brings back an
explanation plus a proposed command.

!!! quote ""
    **The AI helps you operate your infrastructure. It never operates your
    infrastructure itself.**

That command does not run on its own. It appears in front of you, labelled with a
risk tier and showing the exact text you are about to execute. You decide.

## Start here

<div class="grid cards" markdown>

-   :material-compass-outline:{ .lg .middle } **New to OpsPilot**

    ---

    What it is, why it exists and how the loop works.

    [Overview](overview/index.md)

-   :material-download-outline:{ .lg .middle } **Ready to install**

    ---

    Requirements, installers, first run, and a five-minute quickstart.

    [Getting started](getting-started/index.md)

-   :material-key-outline:{ .lg .middle } **Trying and buying**

    ---

    The 15-day free trial, subscribing, and activating online, offline or
    for a whole site.

    [Licensing](licensing/index.md)

-   :material-lan-connect:{ .lg .middle } **Connecting to things**

    ---

    Ten connection types, hypervisor consoles, groups and credentials.

    [Connections](connections/index.md)

-   :material-robot-outline:{ .lg .middle } **Bringing your own AI**

    ---

    Ten providers, a fully offline option, or the subscription you already pay
    for.

    [AI](ai/index.md)

-   :material-shield-check-outline:{ .lg .middle } **Signing it off**

    ---

    The execution boundary, risk tiers, redaction and the security model.

    [Safety & security](safety/index.md)

-   :material-account-group-outline:{ .lg .middle } **Rolling it out**

    ---

    Role workflows, administration, upgrades and troubleshooting.

    [Operate](operations/index.md)

</div>

## What makes it different

Most AI tooling assumes the machine you are working on can reach the internet,
and assumes that giving the model a shell is the point. OpsPilot assumes the
opposite on both counts.

Your servers can stay disconnected: only the workstation needs to reach the AI,
and with a local model not even that. And the model is never given a shell — it
can propose, it cannot execute. Together those two properties make OpsPilot
usable in estates where AI assistance has not been an option: regulated banking,
defence, healthcare, industrial control, classified networks, and production
environments where a model running a command directly would not pass review.

Nothing is installed on the target hosts — no agent, no sidecar, no daemon and no
outbound firewall rule. Any host you can already reach is a host OpsPilot can
work against.

!!! note "Private or VPN-only is not the same as air-gapped"
    With a cloud provider, your *workstation* still talks to the internet — that
    is private or VPN-only operation, and it is what most regulated teams
    actually need. For a genuinely air-gapped deployment, point OpsPilot at a
    local Ollama or self-hosted model, and license it with an offline license
    file or an organisation-wide deployment license. The loop then closes
    entirely: nothing leaves the machine, licensing included.

## The safety model

- **The AI never gets a shell.** Proposals become commands only through
  OpsPilot's own approval path. This is architectural, not a setting.
- **Secrets are removed locally, before anything leaves the machine.** Every
  route to a model redacts first.
- **You decide what runs without asking.** Each Command Safety profile sets
  which commands may run without a click — none, read-only ones, or everything
  except dangerous ones — and a command that counts as
  <span class="tier tier-high">High risk</span> always waits for your click. By
  default your pattern list can only make things stricter: a pattern can
  promote a command, and nothing demotes one the model already flagged.

## See also

- [What OpsPilot is](overview/what-is-opspilot.md) — the longer answer
- [Quickstart](getting-started/quickstart.md) — first connection to first AI
  answer in about five minutes
- [Execution boundary & risk tiers](safety/risk-tiers.md) — read this before
  rolling OpsPilot out to a team
- [Licensing](licensing/index.md) — the free trial, subscribing and activation
