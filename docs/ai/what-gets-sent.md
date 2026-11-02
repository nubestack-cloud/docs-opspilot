# What gets sent

Only redacted session content leaves the workstation when you use an AI feature, and only
from sessions where AI is on. This page sets out exactly what a provider or a connected
assistant receives, what stays on the machine, and what OpsPilot sends for licensing.

## What a provider receives

- recent scrollback from the session you asked about, after redaction
- the text of commands that have run
- the output of those commands, after redaction
- your question, as you typed it
- on a Local Console tab, the name of your operating system and shell, so the model
  proposes commands in the right syntax

The scrollback is bounded — a diagnosis request collects the most recent 8,000
characters of the session buffer, not the whole history — and an attached file is
clipped at 40,000 characters before it is redacted and attached.

Your question is not redacted, because it is your own words. Do not paste a secret into
it.

## What a provider never receives

- **Your credentials.** SSH passwords and private keys live in the connection store,
  encrypted through the OS credential store. No AI path reads them.
- **Your connection list.** Not from a provider path at all. An assistant can ask for
  the connections you have explicitly marked openable, and gets a name, host, port and
  type — never a credential.
- **Your private keys or license keys.** Including a `-----BEGIN ... PRIVATE KEY-----`
  block or an OpsPilot license key printed to the terminal — the built-in **Private key
  blocks** and **NubeStack license keys** categories are on by default.
- **Anything from a session with AI off.** **Enable AI** is per connection, and a session
  with it off is invisible to every provider and every assistant. So is a session the
  license keeps AI off for, such as a connection outside the trial's 2 AI places. The
  tools re-check that at the point of use rather than trusting a cached list. Set the
  toggle deliberately per connection and check it before you connect.

## The single choke point

Every route from a session to an AI destination passes through the same local redaction
step before anything is sent. It is an architectural property of the product rather than
a policy someone has to remember to apply.

Session content can reach a model by six routes, and all of them redact first.

| Route | What is redacted |
|---|---|
| A diagnosis request | Session scrollback collected for terminal analysis |
| A typed question | Session scrollback behind the question you asked |
| A command you approved | The output it printed, before it feeds the next turn |
| A file you attach | A local or remote file attached to a question |
| Scrollback read by an assistant | Scrollback requested through the read-only tool |
| A file read by an assistant | A file read over SFTP through the read-only tool |

The last four are the ones most often assumed to be exempt. In particular:

- **The output of an executed command is redacted too.** When a command runs, OpsPilot
  captures what it printed, redacts that, and only then feeds it back into the next
  turn. A secret that was not on screen when you asked the question does not get a free
  pass because the command printed it.
- **A file read by an assistant goes through the same layer.** A `.env` file read over
  SFTP by Claude Desktop arrives with its secrets already stripped.

Terminal output also has its escape sequences stripped before it is redacted: colours,
cursor moves, window titles and shell-integration markers. That keeps a colour code in
the middle of a key from hiding it from a pattern, and it removes the invisible marker
recent systemd versions print around every command, which carries the host's machine ID,
boot ID and hostname. Files are sent as their own contents after redaction, without this
step.

Which patterns run is set by the Data Handling profile resolved for that connection —
connection, then group, then Default. A connection with no profile assigned resolves to
Default; content with no connection context at all, such as a file you picked from your
own machine, also resolves to Default. A missing profile degrades to Default rather than
skipping redaction.

## One policy for every AI path

The same policy applies whether the destination is your own API key or a connected
Claude Desktop. It is not a per-integration setting, and no integration can opt out of
it: an assistant's requests are redacted by the same step, with the same profile
resolution, as the panel's.

Enabling an external assistant therefore does not widen the data surface. It changes who
is asking; it does not change what they can be told.

## The data flow

<figure class="opspilot-diagram">
<div class="opspilot-diagram__viewport" role="region" aria-label="Session context and command approval diagram" tabindex="0">
--8<-- "data-flow.svg"
</div>
<figcaption>Terminal content is cleaned and redacted before AI access. Proposals return through Command Safety; High risk commands always need a click.</figcaption>
</figure>

Credentials, private keys, the connection store and the contents of AI-disabled sessions
never leave the machine. With a local model and an offline license file or a deployment
license, nothing does.

## What online activation sends

Licensing is separate from AI, and sends no session content at all. OpsPilot makes a
licensing connection only if you activate a license online (or IT sets a key in
`policy.json`), to `license.nubestack.com` over HTTPS:

| When | What it sends |
|---|---|
| Activation, once | The product, the license key, a one-way device fingerprint, the device label you choose, the platform and the app version; and, for a key set by IT, a note saying so |
| A quick check every 5 minutes while the computer is in use, and a few seconds after start, wake and unlock | The product, an activation ID, the fingerprint, the app version, a timestamp, a random value used once and a proof made with the device's activation secret. The license server does not record it |
| A renewal about once a day | The same as a quick check. The server records when this device last renewed |

It never sends host names, user names, terminal content or usage data, and OpsPilot
never stores your license key. The free trial, offline activation and deployment
licenses make no licensing connection at all, and IT can forbid online activation with
`"networkActivation": "disabled"` in `policy.json`.

For an air-gapped workstation, both halves must stay local: a [local model](offline-ollama.md)
for the AI, and offline activation or a deployment license for the license. See
[Activate OpsPilot](../licensing/activation.md).

## Third-party retention

What a provider does with a request after it arrives is that provider's policy, not
OpsPilot's, and those policies change. Published terms for the Anthropic API as of
mid-2026, for example, described a default retention window, stated that API data is not
used for model training by default, and offered zero-data-retention and HIPAA-with-BAA
arrangements to qualifying customers.

That is one provider at one point in time. Read the current terms and data-processing
agreement for the provider you configure before you rely on any of it. If the answer has
to be that nothing is retained anywhere because nothing is sent, use
[a local model](offline-ollama.md).

Retention terms are the second line of defence. Redaction is the first, and it runs
before the request exists.

## See also

- [Data Handling profiles](../safety/data-handling-profiles.md) — choosing which
  patterns run where
- [Redaction categories](../reference/redaction-categories.md) — the built-in
  categories and their defaults
- [Security model](../safety/security-model.md) — the full threat model
- [Offline with Ollama](offline-ollama.md) — the configuration with no egress at all
