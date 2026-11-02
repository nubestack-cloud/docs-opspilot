# Remote & mobile operation

Your workstation stays on the VPN. You do not. From the ChatGPT mobile or web app
you can list sessions, read output and propose commands against an estate with no
internet connection of its own. This page covers the tunnel that makes that
possible, what it is suited to, and what to weigh before enabling it.

The tunnel needs an active OpsPilot free trial or subscription. Without one, **Settings
→ AI Assistants** says so under **ChatGPT Web / Work**; your tunnel settings are kept,
and the tunnel starts again once OpsPilot is licensed. See
[Free trial and limits](../licensing/trial-and-limits.md).

## What the tunnel connects

The workstation becomes a governed bridge into the isolated network. The servers
gain no connectivity; the workstation already had it, and the tunnel lets a
hosted ChatGPT chat reach the MCP connector that remains bound to loopback on
that workstation.

<figure class="opspilot-diagram">
<div class="opspilot-diagram__viewport" role="region" aria-label="Remote and mobile access diagram" tabindex="0">
--8<-- "remote-access.svg"
</div>
<figcaption>The tunnel reaches the workstation's loopback connector. Command Safety rules and approval stay at the workstation.</figcaption>
</figure>

Approval still happens at the workstation, through the same execution boundary
and the same Command Safety profiles as every other AI path.

## Setting up the secure tunnel

You need an **OpenAI tunnel ID** and a **runtime API key** from OpenAI, and
OpenAI's official `tunnel-client` binary on the workstation.

1. In **Settings → AI Assistants**, enable the connector.
2. On the **ChatGPT Web / Work** row, click **Configure**.
3. Enter your **Tunnel ID** — it begins `tunnel_` — and the **Runtime API key**.
   The key is stored with OS encryption and is only ever decrypted into the
   tunnel child process. It is never written to `settings.json` and never
   appears in a command line.
4. Point OpsPilot at the `tunnel-client` executable, or leave `tunnel-client` if
   it is already on your `PATH`. This is under **Advanced settings**, and
   OpsPilot auto-detects it when it can.
5. Click **Configure & Start**. If it does not come up, click **Run Doctor**.

Once it is running, add OpsPilot in ChatGPT's plugin settings using the same
tunnel ID. The local MCP URL and access token are handled automatically and are
never shown in the panel — there is nothing to copy.

**Run Doctor** invokes the tunnel client's own diagnostic and shows its output
in the panel. OpsPilot also runs it automatically as part of starting the
tunnel, so a failed start reports which stage failed rather than just failing.
Anything the tunnel client prints has your keys and tokens stripped out of it
before it is displayed.

### What the tunnel is allowed to reach

OpsPilot checks the forwarding destination before the tunnel client starts. It
has to be OpsPilot's own MCP endpoint on the loopback interface of the
workstation, using the expected scheme and path, and carrying a valid access
token. A destination that fails any part of that check is refused at that
point, and the error names the only destination the tunnel may forward to. The
tunnel ID is checked the same way, against its expected `tunnel_` prefix, so
bad input fails at configuration time rather than at runtime.

**No public listener is created.** The tunnel client makes an outbound
connection; the only listener involved is its own health endpoint, which OpsPilot
starts on `127.0.0.1` with an ephemeral port and refuses to talk to if it comes
up on any non-local address.

## Realistic expectations

**Works well from a phone:** checking service status, reading logs, triaging an
alert, confirming a deployment landed, answering "is it actually down".

**Deliberately does not work well from a phone:** destructive change. A
<span class="tier tier-high">High risk</span> command always needs a click at the
machine — with a typed reason, if the profile asks for one — and proposals expire
after five minutes. If you are not there, it does not run.

What else runs while you are away is set by the Command Safety profile of each
connection the tunnel can reach. Under **Ask me every time**, nothing runs until
someone clicks at the workstation. Under **Only read-only commands**, read-only checks
run by themselves, which is what makes triage from a phone practical. Under
**Everything except dangerous ones**, a command that changes something runs too, with
nobody watching.

What is on offer is remote *triage* — finding out what is wrong, from anywhere.
A destructive change stays a deliberate act at the workstation.

## Security considerations

Your workstation becomes a bridge into the isolated network whenever it is on and
the tunnel is up. Treat it accordingly:

- Set the **idle lock** in **Settings → Security**. Lock on minimise is
  available too.
- Stop the tunnel when you are not using it.
- Keep **Allow opening/reconnecting sessions** off unless you need it — with it
  on, an assistant can open a session with no click.
- Check the Command Safety profile of every connection the tunnel can reach. Keep
  **Run commands without asking me** at **Ask me every time** or **Only read-only
  commands** there, and **What counts as a dangerous command** at **The AI's warning
  and my list**.
- Leave **Enable AI** off entirely on your most sensitive connections. A
  connection with it off is invisible to the tunnel as it is to everything else.
- Treat the runtime API key as a credential with real reach, and rotate it the
  way you would any other.
- Confirm the configuration is acceptable to your security team *before* you
  enable it.

!!! warning "This configuration will not be acceptable everywhere"
    For many regulated environments a standing bridge from a hosted chat service
    into an isolated network will not be approved. The rest of OpsPilot —
    including the AI panel with a local model — works exactly the same with the
    tunnel switched off and never configured.

## See also

- [AI Assistants (MCP)](assistants.md) — the connector, the four assistants and
  the six tools
- [Approvals & auto-run](../safety/approvals.md) — the rules for what runs without
  a click, and the five-minute expiry
- [Security model](../safety/security-model.md) — the whole threat model in one
  place
- [Troubleshooting](../operations/troubleshooting.md) — when the tunnel does not
  come up
