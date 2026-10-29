# Contributing to the OpsPilot documentation

Read `CLAUDE.md` first — it holds the canonical sources, the source-file map and
the product invariants you must not contradict. This file is about *how to
write*: voice, page shape, conventions and the checks a change has to pass.

## Who this documentation is for

Engineers evaluating, installing, operating or governing OpsPilot: DevOps, SRE,
network, platform and development engineers, plus the security and compliance
reviewers who have to sign the thing off. Assume the reader is technically
fluent, is short on time, and has read a lot of documentation that wasted it.

Write for someone who wants to get a real task done, and who will notice if you
overstate what the product does.

## Voice

- **Plain, declarative prose.** Say what happens, in what order, and what the
  reader has to decide. Second person for instructions.
- **British spelling** — organise, organisation, colour, behaviour, sanitise — with
  one exception: **"license" for both noun and verb**, because the product and the
  subscription site use it ("Settings → License", "Import license file", "license
  key"). Never write "licence".
- **Sentence case for all headings**, including table headers.
- **No marketing register.** No "seamlessly", "effortlessly", "powerful",
  "cutting-edge", "revolutionary", "game-changing", "robust", "leverage",
  "unlock", "empower", "supercharge". No exclamation marks.
- **Lead with the answer.** The first paragraph of a page says what the page is
  for and what the reader will be able to do. Do not open with history or a
  restatement of the heading.
- **Concrete over abstract.** A real command, a real field name, a real file
  path, a real number — or nothing.
- **Bold is for UI labels and settings**, not for emphasis of ordinary prose:
  **Settings → AI Providers**, **Auto-run safe commands**, **List VMs**.
- Keep lines wrapped at roughly 90 characters. Diffs in this repo should be
  readable.

## What this is not

This is **user and product documentation**. It is not a developer guide, an
engineering notebook, a design rationale or a commentary on its own sources. Five
habits break that, and all five are easy to slip into. Each is a defect; fix it
on sight.

### 1. Never write about the documentation, or about its sources

The reader does not know or care that a user guide, a design document or a source
tree exists. Writing about them breaks the illusion that they are reading a
product manual, and it reads as a machine narrating its inputs.

| Do not write | Write |
|---|---|
| "The sources for this documentation do not record a release date, so none is stated here." | *Nothing.* Omit the date. |
| "The user guide says X, but the product actually does Y." | "The product does Y." |
| "This page says which files those are." | *Nothing.* Just say which files. |
| "The sources do not say when the macOS installer ships, so this page does not give a date." | "macOS installers are not on the download page yet; email support@nubestack.com to hear when they are." |
| "and so does this page" / "this documentation does not call it one" | *Nothing.* |
| "The guide records this as an open item." | *Nothing*, or state the behaviour plainly. |

Never name "the guide", "the sources", "this page", "this documentation" or
"the product guide" in body text. A page may refer to *other pages* by their
title, which is different.

### 2. No source code, file paths or symbol names

A user cannot open `renderer.js`. Citing it is engineering evidence, not
documentation, and it makes a product page read like a code review.

| Do not write | Write |
|---|---|
| "`matchDangerousPattern()` lower-cases the command and asks whether it contains it" plus a JavaScript block | "Matching is case-insensitive: `rm -rf` matches `sudo rm -rf /var/tmp/x`." |
| "The timeout is `PROPOSAL_TIMEOUT_MS = 5 * 60 * 1000`." | "A proposal expires after five minutes." |
| "Source of truth: `src/command-safety-defaults.js`." | *Nothing.* |
| "`src/connection-store.js` refuses to save rather than downgrading" | "If the operating system cannot encrypt the credential, saving fails rather than storing it in the clear." |
| "the module-level default is `connAiEnabled = true`" | Describe what the reader sees in the interface. |

**There are no exceptions.** No page names a source file, a function, a constant,
an internal identifier, a line number, a repository, a branch or a build script.
Not in body text, not in a table, not in an admonition, not as a "source of
truth" line under a heading. If a fact is true, state it as behaviour; the reader
takes it on the same trust they extend to any other product manual.

That includes the **Architecture** page. Architecture is about trust boundaries,
process separation and where data goes — all of which can be described without a
module inventory. Naming a *technology* is fine where it is a fact the reader
benefits from (it is an Electron application; the terminal is xterm.js; the
editor is Monaco; RDP uses FreeRDP on Windows and Linux). Naming a file is not.

Configuration files a *user* interacts with — `settings.json`,
`connections.json`, `credentials.enc.json` — are legitimate, because the user can
see them, back them up and restore them. Command-line tools a user runs
(`ollama pull`, `ssh`, `xfreerdp`) are legitimate for the same reason.

### 3. No editorial voice, and no essays

Write the product's behaviour, not an opinion about it. Drop the knowing asides,
the little dramas and the self-congratulation.

| Do not write | Write |
|---|---|
| "An approval gate that has become muscle memory is not a control; it is a formality with a button." | "Approving many commands in a row makes it easy to stop reading them." |
| "a written record of why you did it at 3am, produced at the moment you still knew why" | "The justification is recorded with the command." |
| "the on-call network engineer at 3am should be able to add `drop table` without debugging a regex" | "Plain substrings are quicker to write and review than regular expressions." |
| "That is the design, not a gap." | *Nothing.* |
| "Two honest details:" / "Hardware and the honest caveat" | "Two things to note:" or just the facts. |
| "This is the part worth internalising on day one." | *Nothing.* Lead with the fact. |
| "the class where a command was correct and the window was not" | "It prevents running a correct command on the wrong host." |
| "This page is meant to be used, not read." | *Nothing.* |
| "which is the only version of this that survives an incident at 02:00" | *Nothing.* |

Banned constructions: "worth noticing", "worth internalising", "worth saying
plainly", "honest caveat", "the honest truth", "that is the design, not a gap",
"not a defect", "rather than hiding it", "deserves a reviewer's attention", "this
chapter is the product", "say it precisely", any "at 3am" / "at 02:00" framing,
and rhetorical questions.

Also avoid the tic of counting things in prose — "Two properties worth knowing",
"Three failures are worth knowing in advance", "Two consequences, and they are
the reason". Just make the list.

### 4. Headings name a subject; they do not argue a case

A heading is a label a reader scans and a link they land on, not a sentence
making a point. Use a noun phrase. If a heading contains a verb asserting
something good about the product, or an aside after a comma or dash, it is wrong.

| Do not write | Write |
|---|---|
| "Auto-run exists, is genuinely useful, and ships off" | "Auto-run" |
| "Three risk tiers, and the dangerous list can only get stricter" | "Risk tiers" |
| "Secrets are removed before anything leaves your machine" | "Local redaction" |
| "Bring your own AI — ten providers, including one that never phones home" | "Supported providers" |
| "The AI is never given a shell" | "The execution boundary" |
| "It brings AI into environments AI cannot reach" | "Disconnected and VPN-only estates" |
| "One workbench instead of six tools" | "Connection types in one window" |
| "It closes the developer loop" | "Deploying and verifying a change" |
| "Or bring no AI budget at all" | "Using an existing subscription" |
| "Why auto-run exists, and why it ships off" | "When to enable auto-run" |
| "Hardware and the honest caveat" | "Hardware" |
| "`open_session` is pre-authorised, and doubly gated" | "`open_session`" |
| "The two AI modes, and why they are separate" | "The two AI modes" |

Two deliberate exceptions:

- **Troubleshooting headings are symptoms**, phrased as the reader would search
  for them: "SSH will not connect", "The ChatGPT tunnel will not start". Keep
  those exactly as they are.
- **Numbered walkthrough steps** may take the form "Create a connection" under a
  numbered sequence.

"What X does not do" headings are acceptable only where the content is genuinely
scoping — what installation does *not* change on the machine, what a backup does
*not* contain. Where the content is a list of missing features, both the heading
and the section should go.

### 5. Document capabilities, not shortcomings

Set expectations where a reader would otherwise be misled, then move on. Do not
catalogue what is missing, and never structure a page around it.

**Legitimate, keep:** scoping what a feature covers ("Auto-run applies only to
read-only commands"); a prerequisite ("RDP on Linux needs `xfreerdp` installed");
a protocol property ("Telnet and RSH are unencrypted"); a real constraint on a
workflow ("a High risk command needs a typed justification at the workstation").
A **Known limitations** list in release notes is also legitimate — that is what
release notes are for — but keep each entry to one neutral sentence.

**Not legitimate:** a section comparing intended design against shipped
behaviour; "do not plan a control around this"; speculation about what a future
version might do; anything phrased as a defect, a gap, an oversight or something
"under review"; and any passage whose subject is the product falling short.

State a boundary once, in the place the reader needs it, in the positive where
possible. "Capture an execution record from your own session-recording
arrangements" is documentation. "There is no persistent audit log, so plan around
this gap" is a complaint.

## Page shape

Every page:

1. Starts with a single `# Heading` in sentence case, matching its nav label
   closely enough that a reader does not have to reconcile two names.
2. Follows with one to three sentences of orientation — what this is, who needs
   it, what it assumes you have already done.
3. Uses `##` for major sections and `###` sparingly below that. Do not go
   deeper than `###`.
4. Ends with a **See also** section of two to four relative links to the pages a
   reader would genuinely want next. Not a sitemap — a short, deliberate
   handoff.

Section `index.md` pages are landing pages, not dumping grounds: a short
paragraph on what the section covers, then a brief annotated list of its pages
saying why you would read each one. Keep them under about 60 lines.

Reference pages (`reference/*.md`) are the opposite of narrative: lead with the
table, keep prose to the minimum needed to make the table unambiguous, and say
where in the product each value is set or read.

Target lengths, as a rough guide rather than a rule: a reference page may be
almost all table; a narrative page is usually 60–200 lines. If a page is
running past ~250 lines, it probably wants splitting — raise it rather than
silently restructuring the nav.

## Conventions

### Links

Internal links are **relative paths to the `.md` file**, because `--strict`
validates them:

```markdown
See [Command Safety profiles](../safety/command-safety-profiles.md).
See [the glossary](glossary.md).
```

Never link to a built URL path (`/safety/command-safety-profiles/`), and never
link to a heading anchor in another file unless you have confirmed the heading
exists.

### Images

Screenshots live in `docs/assets/images/` and are referenced relatively:

```markdown
![Settings → AI Providers with the active provider at the top and the others listed as Not set up](../assets/images/26-settings-ai-providers.png)
_The AI Providers page. The provider in use sits at the top marked **Active**; open
another to enter its details, then **Test Connection** before **Save & Activate**._
```

- Alt text describes what is in the image, for someone who cannot see it.
- The italic line directly under an image is styled as a caption by
  `extra.css`. Write it with underscores (`_…_`), on the line **immediately after**
  the image, with no blank line between them: the caption style only applies when
  the two share a paragraph. Inside a numbered list, indent both lines four spaces.
  Use it to say what the reader should notice — not to repeat the alt text.
- Where a step needs a screenshot that does not exist yet, leave a comment on its
  own line, `<!-- screenshot: what the image should show -->`, instead of a broken
  reference.
- Each screenshot has an owning page (see the table below). Reuse an image on a
  second page only when it genuinely earns its place there.

| Image | Used on |
|---|---|
| `11-rdp-connection.png` | `connections/remote-desktop.md` |
| `12-kvm-console.png` | `connections/hypervisor-consoles.md` |
| `13-openstack-console.png` | `connections/hypervisor-consoles.md` |
| `14-trial-first-launch.png` | `connections/organising.md`, `getting-started/first-run.md`, `getting-started/quickstart.md`, `licensing/trial-and-limits.md` |
| `15-new-connection.png` | `connections/adding-connections.md`, `getting-started/quickstart.md` |
| `16-ssh-session.png` | `ai/using-the-ai-panel.md`, `connections/adding-connections.md`, `getting-started/quickstart.md` |
| `17-ai-readonly-proposal.png` | `ai/using-the-ai-panel.md`, `getting-started/quickstart.md` |
| `18-ai-low-risk-proposal.png` | `ai/using-the-ai-panel.md`, `safety/risk-tiers.md` |
| `19-ai-summary.png` | `ai/using-the-ai-panel.md`, `index.md`, `overview/what-is-opspilot.md` |
| `20-ai-high-risk.png` | `ai/using-the-ai-panel.md`, `getting-started/quickstart.md`, `safety/approvals.md`, `safety/risk-tiers.md` |
| `21-trial-ai-dialog.png` | `connections/organising.md`, `licensing/trial-and-limits.md` |
| `22-session-limit.png` | `licensing/trial-and-limits.md`, `operations/troubleshooting.md`, `workspace/sessions.md` |
| `23-trial-ai-off-tab.png` | `licensing/trial-and-limits.md`, `workspace/sessions.md` |
| `24-license-trial.png` | `getting-started/quickstart.md`, `licensing/activation.md`, `licensing/trial-and-limits.md`, `reference/settings-map.md` |
| `25-license-activate-offline.png` | `licensing/activation.md` |
| `26-settings-ai-providers.png` | `ai/providers.md`, `getting-started/quickstart.md` |
| `27-settings-ai-assistants.png` | `ai/assistants.md` |
| `28-settings-command-safety.png` | `getting-started/quickstart.md`, `reference/settings-map.md`, `safety/approvals.md` |
| `29-command-safety-profile-editor.png` | `reference/dangerous-patterns.md`, `safety/command-safety-profiles.md` |
| `30-data-handling-editor.png` | `reference/redaction-categories.md`, `safety/data-handling-profiles.md` |
| `31-settings-about.png` | `about/support.md`, `reference/settings-map.md` |
| `32-locked-connections.png` | `connections/organising.md`, `licensing/trial-and-limits.md`, `operations/troubleshooting.md` |
| `34-connection-menu.png` | `connections/organising.md` |
| `35-explorer-editor.png` | `workspace/files-and-transfers.md` |
| `36-port-forward.png` | `workspace/tools.md` |
| `37-system-monitor.png` | `workspace/tools.md` |
| `38-snippets.png` | `workspace/tools.md` |
| `39-tools.png` | `workspace/tools.md` |
| `40-insights.png` | `workspace/tools.md` |
| `41-license-device-limit.png` | `licensing/activation.md`, `operations/troubleshooting.md` |
| `42-license-licensed.png` | `licensing/activation.md`, `operations/backup-and-upgrade.md`, `reference/settings-map.md` |
| `43-licensed-main.png` | `licensing/trial-and-limits.md` |
| `44-portal-overview.png` | `licensing/subscribe.md` |
| `45-portal-seats.png` | `licensing/subscribe.md` |
| `46-portal-devices.png` | `licensing/activation.md` |
| `47-portal-offline.png` | `licensing/activation.md`, `licensing/for-it.md` |
| `48-portal-offline-done.png` | `licensing/activation.md` |
| `49-portal-deployments.png` | `licensing/for-it.md` |
| `50-portal-billing.png` | `licensing/subscribe.md` |
| `51-portal-seat.png` | `licensing/subscribe.md` |
| `52-site-pricing.png` | `licensing/subscribe.md` |
| `53-site-download.png` | `getting-started/install.md` |
| `54-ai-assistants-connector-on.png` | `ai/assistants.md` |
| `55-ai-assistants-claude-connected.png` | `ai/assistants.md` |
| `56-claude-desktop-connectors.png` | `ai/assistants.md` |
| `57-claude-desktop-session-check.png` | `ai/assistants.md` |
| `58-claude-desktop-list-refused.png` | `ai/assistants.md` |
| `59-claude-desktop-recent-output.png` | `ai/assistants.md` |

`11` to `13` came from the product repository's own `docs/images/`; `14` onwards were taken
for the 0.5.1 refresh from an isolated copy of OpsPilot with demo connections only (demo
hosts on `127.0.0.x`, a local test AI endpoint) and from a local copy of the NubeStack
portal with demo data. Never take a screenshot that shows a real customer's or your own
connections, hosts, keys or device codes. New screenshots continue from the next free
number; add the row here in the same change.

### Risk-tier badges

`extra.css` provides badge markup for the three tiers. Use it in prose and
tables where a tier is the subject:

```markdown
<span class="tier tier-readonly">Read-only</span>
<span class="tier tier-low">Low risk</span>
<span class="tier tier-high">High risk</span>
```

### Admonitions

Available via `pymdownx.details` and `admonition`. Use them sparingly — a page
of admonitions has no emphasis left.

```markdown
!!! note "Private or VPN-only, not air-gapped"
    With a cloud provider your workstation still reaches the internet.

!!! warning "Telnet and RSH are unencrypted"
    Credentials and session content travel in clear text.

??? example "A starting pattern list"
    Collapsed by default — good for long lists and sample configuration.
```

Reserve `!!! danger` for things that can destroy data or expose credentials.

### Code blocks

Always label the language — `bash`, `yaml`, `json`, `text`. Use `text` for the
ASCII diagrams carried over from the guide. Do not put a `$` prompt in front of
shell commands; the copy button copies it too.

### Tables

Tables are the right format for connection types, providers, settings, tools,
categories and tiers, and this documentation uses a lot of them. Keep column
headers in sentence case, keep the leftmost column the thing being looked up,
and do not let a table exceed about five columns.

### Calls to action

Every page gets **Download** and **Subscribe** buttons from the theme override in
`overrides/`. Do not add your own call-to-action blocks; link
`https://subscription.nubestack.com/download` or
`https://subscription.nubestack.com/opspilot` in prose where a reader needs them. To
hide the buttons on one page, set `hide_cta: true` in its front matter:

```markdown
---
hide_cta: true
---
```

### Mermaid

`mermaid` fenced blocks are configured and render. Prefer the plain-text ASCII
diagrams from the source guide where they already exist and read well — they are
part of the product's voice. Use Mermaid for a flow the sources do not already
draw.

## Terminology

Use these exactly. The distinctions are load-bearing.

| Use | Not |
|---|---|
| AI Providers (you supply a key or sign-in) | "the AI", when you mean the provider mode |
| AI Assistants (an external app over MCP) | "integrations", "plugins" |
| proposal | "the command the AI ran", "AI action" |
| approval gate | "confirmation dialog" |
| Read-only / Low risk / High risk | "safe", "unsafe", "medium" |
| Command Safety profile | "safety settings", "policy" |
| Data Handling profile | "redaction settings" |
| dangerous pattern | "blocklist", "blacklist" |
| environment (a colour-coded tag) | "workspace" |
| group (a folder of connections that carries policy) | "folder" |
| session | "connection", when you mean an open one |
| connection (a saved definition) | "host", when you mean the saved entry |
| private / VPN-only | "air-gapped", unless a local model and offline activation or a deployment license are in use |
| workstation | "client", "agent" |
| redaction | "masking", "filtering", "anonymisation" |
| license (noun and verb), license key, license file | "licence", "serial", "token" |
| seat (one person, up to 2 devices) | "user licence", "named user" |
| activate online / activate offline | "register", "sign in to OpsPilot" |
| request code (goes out) / license file (comes back) | "activation code" |
| device code (six characters such as `R59-EFG`) | "device ID", "fingerprint" |
| deployment license (one file for the whole organisation) | "site licence", "volume key" |
| sessions open at once | "concurrent connections", when you mean the limit |
| whoever manages licenses in your organisation | "your administrator", "your NubeStack administrator" |

OpsPilot installs nothing on target hosts. Never write "agent", "daemon" or
"sidecar" in a way that suggests it does.

## Before you commit

```bash
. .venv/bin/activate
mkdocs build --strict
```

Then check your change against this list:

- [ ] `mkdocs build --strict` passes with no warnings.
- [ ] Every page you touched is in the `nav` in `mkdocs.yml`.
- [ ] Every internal link is a relative `.md` path that resolves.
- [ ] No stub admonition left behind on a page you were meant to write.
- [ ] Every factual claim traces to a source in `CLAUDE.md`'s source map — no
      invented paths, flags, versions, prices, shortcuts or URLs.
- [ ] None of the product invariants in `CLAUDE.md` is contradicted.
- [ ] No page adds its own Download or Subscribe block (the theme adds them).
- [ ] Limitations stated plainly rather than omitted.
- [ ] Terminology table followed.
- [ ] Images have alt text and a caption, and are referenced relatively.
- [ ] Page ends with a **See also** block.
- [ ] No marketing register, no invented benchmarks, no competitor comparisons.

And against the five rules in [What this is not](#what-this-is-not):

- [ ] Nothing refers to this documentation, to "the guide", or to "the sources".
- [ ] No source file, function, constant, internal identifier or repository is
      named anywhere.
- [ ] No editorial asides, no "worth noticing", no "at 3am", no counting things
      in prose before listing them.
- [ ] Every heading names a subject rather than arguing a case — except a
      troubleshooting symptom or a numbered walkthrough step.
- [ ] No section catalogues missing features or compares intent against shipped
      behaviour. Boundaries stated once, where the reader needs them.

## Adding a page

1. Add the file under the right section directory.
2. Add a nav entry in `mkdocs.yml` in the same change — `--strict` will fail
   otherwise.
3. Link to it from its section `index.md` and from at least one related page's
   **See also**, so it is reachable by a reader who is not using the nav.
