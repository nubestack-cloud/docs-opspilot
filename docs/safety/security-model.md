# Security model

This page is written for the reviewer who has to sign OpsPilot off. It states the
principles the product is built on, the path data takes, what it stores and where, what
it sends for licensing, and what it keeps of a session.

## Principles

1. **The AI proposes, the human disposes.** The model has one output channel: a proposal.
   OpsPilot's own code is the only thing that executes against a session, and it does so
   only when you approve the proposal or when it falls in a class you pre-authorised in a
   Command Safety profile. A dangerous command always needs your click. This is
   architectural. There is no setting, provider, assistant, prompt or command-line flag
   that gives a model a shell, and the product reports it as **Always enforced** rather
   than as a toggle.
2. **Redact before egress.** Terminal content reaches no AI provider or assistant without
   passing through the local redactor first, on one entry point shared by every path.
   Content with no connection context resolves to the Default profile rather than
   bypassing redaction.
3. **Least exposure.** AI access is scoped per connection by the **Enable AI** toggle, and
   the off state is meaningful: a connection with it off is invisible to every model and
   every assistant. Set it deliberately per connection rather than assuming a default —
   the seeded **Local** connection ships with AI off, but the new-connection dialog does
   not open in that state, so verify the toggle before connecting. Letting an external
   assistant *open* a session is a separate and stricter permission again — **Let AI
   Assistant open this session**, off by default and additionally gated by a master switch
   in **Settings → AI Assistants**.
4. **Local by default.** No telemetry, no account and no cloud dependency in the core
   product. Settings, connections, groups, environments and both kinds of profile are
   files on the workstation. Licenses are checked on your computer, and online activation
   is optional. Terminal scrollback is held in memory, and OpsPilot never persists it of
   its own accord — the only exceptions are the explicit, user-initiated **Save terminal
   output** and **Print terminal output** actions on a session's tab menu.
5. **Nothing on the target host.** OpsPilot installs no agent, daemon, sidecar or runtime
   on the hosts it works against, and needs no inbound route or outbound firewall rule
   for them. It operates over the access you already have.

## Data flow

<figure class="opspilot-diagram">
<div class="opspilot-diagram__viewport" role="region" aria-label="Session context and command approval diagram" tabindex="0">
--8<-- "data-flow.svg"
</div>
<figcaption>Terminal content is cleaned and redacted before AI access. Proposals return through Command Safety; High risk commands always need a click.</figcaption>
</figure>

The loop at the bottom is easy to miss on a first reading: the output of an approved
command is stripped and redacted again before it becomes context for the following turn.

## What never leaves the machine

Credentials, private keys, the connection store, the contents of AI-disabled sessions,
and — with a local model and an offline license file or a deployment license —
everything. Your license key is sent only once, when you activate online, and OpsPilot
does not store it. The target hosts never need internet access; only the workstation
does, and only when the configured provider is a remote one or you activate online.

!!! note "Private or VPN-only is not air-gapped"
    With a cloud provider, the workstation still reaches the internet. That is **private
    or VPN-only** operation. A genuinely air-gapped deployment needs a local model — see
    [Offline with Ollama](../ai/offline-ollama.md) — **and** a license that needs no
    network: offline activation or a deployment license, see
    [Activate OpsPilot](../licensing/activation.md).

## Threat model

The threat model does not assume the AI provider is malicious. It assumes terminal output
is sensitive by default and that engineers are fallible by default, and builds controls
for both. The risks it covers for a terminal-session-to-AI pipeline:

| Risk | Control in the shipped product |
|---|---|
| **Credentials in output** — passwords echoed to screen, keys in an `env` dump, tokens in a `cat` of a config file, private key contents, NubeStack license keys | The six credential redaction categories, on by default, applied after escape sequences are stripped |
| **Customer-identifying data** — internal hostnames, IP ranges, tenant or project names that reveal who a client is or how their estate is laid out | The five structural categories, opt-in per profile, plus custom patterns |
| **Regulated data** — anything carrying a data-residency or sector obligation | Custom patterns, a production Data Handling profile, or a local model so nothing leaves the machine |
| **Destructive commands** — a suggestion that is technically correct and catastrophic if approved without being read | The approval gate, the three tiers, the dangerous-pattern list, a click that is always required for a dangerous command, and the typed reason |
| **Loose rules in the wrong place** — a permissive Command Safety profile on a production group | Per-profile rules with a plain-language summary, a confirmation before looser rules are saved, and the composer chip showing each tab's rule |
| **Credential-at-rest risk** — credentials stored carelessly by the tool itself | OS-native credential storage, described below |
| **MITM on SSH** — silently trusting an unknown host key | The network path: run OpsPilot over a VPN or a trusted management network |
| **Third-party data handling** — what a provider does with what is sent, and for how long | Redaction first; provider choice, including a local model, second |
| **Licensing traffic** — what a license check could reveal | No licensing connection at all unless you activate online; the content listed below; offline activation and deployment licenses for sites where nothing may leave |

!!! warning "Telnet and RSH are unencrypted"
    Credentials and session content travel in clear text. Both are supported because
    switch and appliance consoles still require them; use them only on trusted
    management networks. See
    [Connection types](../connections/connection-types.md).

## The five-stage pipeline

| Stage | What happens | Key control |
|---|---|---|
| 1. Terminal session | Raw output is captured in the application | Nothing is sent anywhere yet, and it stays in memory |
| 2. Local redaction | Escape sequences are stripped, then output is scrubbed **before** it is added to AI context | Runs entirely on-device, no network call |
| 3. Provider or assistant | Redacted context is sent for diagnosis and a suggested command | Provider is yours to choose, including a local one |
| 4. Approval gate | The suggestion is shown as a card | What it takes to run is set by the connection's Command Safety profile: from a click on every command to unattended runs of anything the profile does not call dangerous. A dangerous command always needs a click |
| 5. Execution | OpsPilot's own code types the command into the real session, and its output is redacted again on the way back | The same single redaction entry point |

## Licensing traffic

OpsPilot sends no telemetry. The only connection it makes on its own is for licensing,
and only after you activate a license online (or IT sets a key in `policy.json`). It goes
over HTTPS to `license.nubestack.com`:

- **when you activate:** the product, the license key (once; OpsPilot never stores or
  logs it), a one-way device fingerprint, the device label you chose, the platform and
  the app version, and — when IT set the key in `policy.json` — a note saying so
- **a quick check** every 5 minutes while the computer is in use, and a few seconds after
  start, wake and unlock: it asks only whether this computer is still licensed, and the
  license server does not record it
- **a renewal** of the 30-day online license about once a day, with a download of the
  list of revoked licenses; the server records when this device last renewed, and your
  organisation's admins see that date in the portal

Checks and renewals send the product, an activation ID, the fingerprint, the app
version, a timestamp, a random value used once and a proof made with the device's
activation secret, instead of the key. Licensing never sends terminal content, host names, user names or
usage data.

Only answers signed by NubeStack's keys, which are built into OpsPilot, change the license
state. A fake or intercepted server cannot grant a license, and an unsigned answer never
removes one. The free trial, offline activation and deployment licenses make no licensing
connection at all, and IT can forbid it on managed machines with
`"networkActivation": "disabled"` in `policy.json`. See
[Licensing for IT](../licensing/for-it.md).

## Session records

Session transcripts and terminal scrollback are held in memory for the life of the
session. OpsPilot does not write them to disk, and the two exports available are
**Save terminal output** and **Print terminal output** on a session's tab menu, which
capture the scrollback text. If your estate needs a durable execution record, capture it
from the host side or from your own session-recording arrangements.

## Credential storage

SSH passwords, private keys and passphrases are encrypted with the operating system's
own encryption — the Keychain on macOS, the Data Protection API on Windows, the desktop
keyring on Linux — and never written to a plaintext configuration file. The connection dialog says so directly on its
**Save connection** toggle: *credentials encrypted, never plaintext*.

Details to record in a review:

- **There is no plaintext fallback.** If OS encryption is unavailable, storing the
  credential fails with an error rather than silently downgrading.
- **The OpenAI tunnel's runtime key is held separately**, encrypted the same way and
  decrypted only into the tunnel process. The tunnel additionally refuses to start when
  the platform's only available encryption backend is the plaintext one.
- **The license activation secret** — the per-device secret that signs online renewals —
  follows the same rule. It is stored only when OS encryption is available, and not with
  Linux's plaintext fallback; without it, online activation is off and says why, while
  offline activation and deployment licenses keep working. The license key itself is
  never stored, and neither the key nor the secret is logged. License state lives in a
  `licensing` folder in OpsPilot's application data folder, not in `settings.json`.

!!! note "AI provider API keys are held differently from connection credentials"
    Provider API keys are stored in the application's own settings file on the
    workstation, not in the OS credential store that holds connection credentials. Treat
    a provider key as you would any other credential in a configuration file: scope what
    it can reach, rotate it on the provider's side when needed, and take particular care
    on a shared or multi-user workstation.

    The **Clear** button in **Settings → Security** removes saved AI provider
    configuration, including those keys. Connection credentials are cleared by deleting
    the connections that hold them.

    If your review needs inference credentials held to the same standard as SSH
    credentials, the options are a local model, which needs no key at all, or a
    self-hosted endpoint on your own network. Both are covered in
    [Offline with Ollama](../ai/offline-ollama.md) and
    [Self-hosted & enterprise](../ai/self-hosted.md).

!!! warning "RDP passwords on Linux are visible to local process listing"
    On Linux, an RDP password is passed to the FreeRDP client on its command line. It is
    never written to disk, but anything that can list the user's processes on that
    workstation can read it. Treat a shared or multi-user Linux workstation as unsuitable
    for saved RDP credentials.

## The machine-wide policy folder

IT can place `policy.json` and license files in a machine-wide folder that applies to
every user of the computer (`%ProgramData%\NubeStack\OpsPilot` on Windows). Because it is
trusted for everyone, only administrators may write to it. On Windows, where any user can
create folders under `ProgramData`, the installer creates the folder and, on every
install, makes `%ProgramData%\NubeStack` owned by Administrators, with full control for
SYSTEM and Administrators and read-only access for Users. Otherwise a standard user on a
shared host could plant a `policy.json` pointing other users' license keys at a server of
their choosing. **Settings → License** names any license server set by policy, so a
redirected server is visible.

## One OpsPilot per user

One OpsPilot runs per operating-system account, so a second copy cannot double the
licensing limits. Starting it again brings the running window to the front. On a shared
jump host, another user cannot stop your OpsPilot from starting by holding the same name
first. If the check itself fails, OpsPilot starts anyway.

Release builds also refuse a debugger attaching to the running app, and detached terminal
and remote desktop windows offer no developer tools.

## Third-party provider handling

If you use a remote provider, its own terms govern what happens to the redacted text you
send, and those terms — retention windows, whether inputs are used for training, and the
contractual mechanisms available to change either — are the provider's own and change
over time. Check the current terms for the provider you configure, and treat them as the
second line of defence rather than the first: local redaction is what runs before
anything is sent.

## Reporting a vulnerability

Email NubeStack support at support@nubestack.com. Security reports are handled directly
by the NubeStack engineering team.

## See also

- [Hardening checklist](hardening.md) — the same material as a pre-deployment gate
- [Data Handling profiles](data-handling-profiles.md) — the redaction layer in detail
- [Approvals & auto-run](approvals.md) — principle 1, in practice
- [Licensing for IT](../licensing/for-it.md) — the policy folder, `policy.json` and
  deployment licenses
