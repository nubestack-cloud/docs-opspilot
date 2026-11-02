# Architecture

This page is for an architect or a security reviewer: where the trust boundaries sit,
what crosses them and in which direction, how a proposal becomes a command, and where
configuration and credentials are held. The same boundaries are stated as policy, with
the threat model behind them, in [Security model](../safety/security-model.md).

## Process separation

OpsPilot is an Electron desktop application, split into a privileged layer and an
interface layer that holds no privileges of its own.

The **privileged layer** owns everything that touches a network or a disk: connection
and session handling, requests to AI providers, redaction, the connection and profile
stores, licensing, the connector for external assistants, and the tunnel used for remote
operation. Every connection to a target host and every request to an AI provider
originates there.

Licensing is enforced in the privileged layer too. It verifies licenses against
NubeStack's public keys, which are built into the application and never fetched; it
counts every open session of every kind; and it decides which sessions may have AI. The
interface layer only displays the result, so nothing in it can lift a limit.

The **interface layer** draws the connection tree, session tabs, terminals, the file
explorer, the AI panel and the approval cards. It runs with no module loading and no
filesystem or socket access of its own, and reaches the privileged layer only through a
fixed set of named requests across a narrow bridge; there is no general-purpose channel
between the two. Terminals are rendered with xterm.js, and the built-in file editor is
Monaco. The widest surface the interface can reach is therefore that fixed set of
requests, and a model's surface is narrower still: its reply is data the interface
renders, never a request it makes.

Release builds do not accept a debugger attached to the running application, and
detached terminal and remote-desktop windows offer no developer tools. One OpsPilot runs
per operating-system account: starting a second copy brings the running window to the
front instead, so a second copy cannot double the licensing limits.

## Trust boundaries

| Boundary | What crosses it | Direction |
|---|---|---|
| Interface and privileged layer | Named requests out; session output and results back | Across the fixed request surface only |
| Workstation and target host | Keystrokes, file transfers and protocol traffic | Outbound from the workstation; nothing is installed on the host |
| Workstation and AI provider | Redacted session text and your question out; an explanation and a proposed command back | Outbound requests only; no inbound connection |
| Connected assistant and OpsPilot | Tool calls in over loopback, with a token; redacted output and results back | Inbound on loopback only |
| Workstation and license server (optional) | Only after an online activation: the license key once, with the device label and platform; then a device fingerprint, the app version and a signed proof. Signed licenses and status answers back | Outbound HTTPS to `license.nubestack.com` only |
| IT-managed folder and OpsPilot | `policy.json`, license files and revocation lists placed by administrators | Read by OpsPilot; writable only by administrators |

A target host never initiates anything, never contacts an AI provider and runs no
OpsPilot component. With a cloud provider the *workstation* reaches the internet, which
is private or VPN-only operation rather than air-gapped; a local or self-hosted model
removes that crossing entirely. The license server crossing is optional as well: the
free trial, offline license files and deployment licenses need no network, and IT can
forbid licensing traffic on managed machines. A license server cannot change a license,
because every license and status answer is checked against the built-in keys, and an
unsigned answer never changes the license state.

## Data flow

<figure class="opspilot-diagram">
<div class="opspilot-diagram__viewport" role="region" aria-label="Session context and command approval diagram" tabindex="0">
--8<-- "data-flow.svg"
</div>
<figcaption>Terminal content is cleaned and redacted before AI access. Proposals return through Command Safety; High risk commands always need a click.</figcaption>
</figure>

## The redaction choke point

Terminal content becomes AI context through exactly one redaction step, which runs in
the privileged layer. No path from a session buffer to a provider avoids it, because the
interface never holds both the raw buffer and a way to send it. That step serves every
AI path in both directions of the loop: scrollback for a panel analysis or a question,
the output an approved command produced, a file attached to the composer, and the
scrollback and remote file reads a connected assistant requests.

Terminal output is cleaned before it is redacted: escape sequences and other terminal
control sequences are stripped, leaving the text a person would see on screen. This
matters for redaction, because a secret with a colour code in the middle of it would not
otherwise match a pattern. It also removes the shell-integration markers that recent
systemd versions print around every command, which carry the host's machine ID, boot ID
and hostname. File content — a file attached to the composer, or a remote file an
assistant reads — is redacted but not cleaned, since its bytes are what was asked for.

Resolution cannot fail open. The step resolves the Data Handling profile itself —
connection, else group, else Default — and an unknown connection, a missing connection
reference or a deleted profile all degrade to Default rather than skipping redaction.
The path that carries a multi-step investigation forward holds only material already
redacted at capture time: the output captured at approval, plus the model's prior
explanation, which never contained terminal text.

Custom redaction patterns are real regular expressions, each validated in a separate
worker with a hard timeout before it can be saved, so a pattern that would backtrack
catastrophically is rejected at that point. At match time each pattern is applied
independently, so one failing pattern cannot suppress the ones after it.

## From proposal to command

A model's reply is a proposal: the command text, a required reason, and the model's own
assessment of the command as safe or dangerous. Classification combines that
self-assessment with the dangerous patterns in the resolved Command Safety profile. With
the profile's default, **The AI's warning and my list**, the combination is asymmetric —
a pattern match can promote a command to High risk, and nothing can demote a command the
model has already flagged. With **Only my list**, the patterns alone decide; the model's
warning stays visible on the command as a note, but is not applied.

Each profile also decides what may run without a click (**Run commands without asking
me**: nothing, read-only commands, or everything except dangerous ones) and whether a
dangerous command needs a typed reason (**Make me type a reason for dangerous
commands**, on by default). A dangerous command always needs a click. Whether a command
runs after a click or under a rule set in advance, it reaches the shell through the same
execution path, written by OpsPilot's own code; no provider, assistant or prompt can
produce that step, and the interface reports the boundary as **Always enforced** rather
than as a setting. A proposal from a connected assistant joins the same queue on the
same terms, and expires after five minutes if nobody acts on it.

## The connector for external assistants

External assistants reach OpsPilot through an MCP connector in the privileged layer. It
ships disabled. Enabled, it binds to `127.0.0.1` only — no public listener is ever
opened — and every request must carry the bearer token OpsPilot generated. Loopback
binding keeps other machines out; the token keeps out other processes running as the
same user.

Six tools are exposed. Four are read-only — listing the connections an assistant is
allowed to open, listing open sessions (SSH sessions and local shell tabs, each local
one with its operating system and shell), reading recent output and reading a remote
file over an SSH session's existing transfer channel — and everything they return passes
through the redaction step. Opening a session is available only where session opening is
enabled centrally and per connection. Proposing a command enters the approval queue and
never executes by itself. Authorisation is re-checked on every call rather than trusted
from what the client was told earlier: a session with AI disabled is refused even if the
client still holds its identifier, and opening a session re-checks the per-connection
permission that the connection listing filtered on. [MCP tools](../reference/mcp-tools.md)
has the set in full.

Every tool also checks licensing first. Without an active trial or subscription the
connector keeps running, so a connected assistant gets a reason rather than a broken
connection, but each call is refused. Sessions whose AI the license does not allow are
left out of the session list; locked connections, and during the trial connections
without one of its AI places, are left out of the connection list; and a proposal still
waiting for approval is refused if the license takes AI away from its session.

Applications that accept a local HTTP MCP endpoint reach the connector directly; those
that accept only a local stdio server are served by a generic proxy that forwards
requests verbatim and holds no OpsPilot logic of its own, so tool behaviour, redaction
and approval wiring stay in one place. For remote and mobile operation OpsPilot
supervises OpenAI's own tunnel client and validates its forwarding target: the
destination must be loopback HTTP, on the connector's own path, with a token present, or
the tunnel does not start. The tunnel's runtime API key is encrypted through the OS
credential store and decrypted only into the tunnel process.

## Sessions and protocol handling

Each protocol is handled in the privileged layer behind the same shape — create a
session, hold its buffer, emit data, close it — so the layers above do not branch per
protocol. SSH carries its file-transfer channel on the same connection and can open
local port forwards, Telnet and RSH are raw TCP, and a local shell is a real PTY.
Session scrollback is a bounded in-memory buffer, not a file. The per-session AI state
is held in the privileged layer, which lets the connector enforce "no AI for this
session" without trusting the interface. The same layer keeps one count of every open
session — terminals, local shells, file browsers, remote desktops and consoles, and
programs OpsPilot starts in their own window — which is what the session limit of the
trial and of use without a subscription is checked against. Licensing never closes a
session that is already open.

Hypervisor consoles are rendered in the application over a loopback proxy on an
ephemeral port that forwards through an SSH port forward, so the VM's console port is
never exposed beyond the workstation; a direct VNC host opens your system VNC viewer
instead.

## Configuration and credentials

Configuration is a set of small JSON files in a per-user application data directory,
re-read on each access — which is why a connection's *current* group membership is
honoured even if it changed after the session was opened. Saved connections, groups,
environment tags, both kinds of profile, transfer history and application settings each
have their own file, and the folder can be backed up and restored as a unit. Both
profile kinds keep a Default profile that cannot be deleted, so resolution always has a
fallback, and profiles are replacements rather than additions: a group profile can
legitimately be less strict than Default.

Connection credentials — SSH passwords, private keys and passphrases, OpenStack
passwords and application-credential secrets, and the S3 secret access key — are
encrypted through the OS credential store and held in their own file, separate from the
connection definitions. If the operating system cannot encrypt them, saving fails rather
than writing them in the clear. AI provider API keys take a different path: they are
held as plain text in the application settings file, so where inference credentials must
meet the same standard as connection credentials, use a local or self-hosted model,
which needs no key.

License state — an online activation and any imported license files — is kept in a
`licensing` folder of the application data directory, apart from the settings file the
interface writes. An online activation has a per-device secret that signs its renewals;
it is encrypted through the OS credential store, and where that is not possible (Linux
without an unlocked keyring) online activation is refused rather than storing the
secret in the clear, while offline activation keeps working. The license key itself is
never stored, and a license key that appears in a terminal is redacted before anything
reaches an AI provider.

Machine-wide license files and the IT policy file live in a folder every user of the
machine trusts — `%ProgramData%\NubeStack\OpsPilot` on Windows,
`/Library/Application Support/NubeStack/OpsPilot` on macOS and `/etc/nubestack/opspilot`
on Linux — so only administrators may write it. On Windows, where any user may create
folders under `ProgramData`, the installer creates the folder and on every install
restricts `%ProgramData%\NubeStack` to administrators and the system account, with
read-only access for other users. See [Licensing for IT](../licensing/for-it.md).

## RDP per platform

Every connection type except RDP behaves identically on Windows, macOS and Linux. RDP
differs because embedding another application's window is available on Windows and has
no counterpart on the other two — macOS offers no equivalent, and on Linux Wayland
blocks cross-application embedding.

| Platform | How an RDP session opens |
|---|---|
| Windows | An embedded tab in the OpsPilot window, using the bundled FreeRDP client |
| macOS | A standard RDP connection file handed to Microsoft's Windows App |
| Linux | FreeRDP as an external client window |

The macOS handoff omits the password from the connection file it generates, and the
external client prompts for it. The bundled FreeRDP client is scoped to the Windows
build, so RDP on Linux needs FreeRDP installed on the workstation.

## See also

- [Security model](../safety/security-model.md) — the same boundaries stated as
  policy, with the threat model behind them
- [How it works](how-it-works.md) — the same loop from the operator's side
- [Network requirements](../reference/network-requirements.md) — the ports and
  destinations a firewall reviewer needs
- [AI Assistants (MCP)](../ai/assistants.md) — connecting an external
  application to the connector
