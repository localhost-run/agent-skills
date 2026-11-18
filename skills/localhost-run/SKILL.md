---
name: localhost-run
description: Put a local dev server on the user's stable public HTTPS domain with localhost.run, for previews, sharing work in progress, testing on phones, or receiving webhooks. Use when the user wants a public URL for something running locally, asks to share or preview their local app, needs a webhook endpoint, wants their app on a particular domain, or mentions localhost.run. Also sets up a new user's localhost.run domain and SSH key, or gives them a temporary address if they'd rather skip setup.
license: MIT
---

# localhost.run tunnels

localhost.run gives a local port a public HTTPS URL over SSH. This skill connects the user's **stable domain** (one that stays the same every time) to a port the agent's work is running on, keeps it connected, and shows what's arriving.

All the work is done by `scripts/tunnel` (bash and ssh, nothing to install). Run it by its path in this skill's directory.

## First: is it set up?

Run `scripts/tunnel config`. If it says "not set up", or `start` exits with code 2 or 3, read `references/setup.md` and walk the user through it before going on.

Setting up is the normal path, even when the user just asked for "a link": it takes about a minute, and from then on every link you give them keeps working. When it's not set up, say so and offer to set it up, in one short message, for example:

> You don't have a localhost.run domain yet. Setting one up takes about a minute, and gives you an address that stays the same every time, so links you share and webhook settings keep working. Shall we do that now? (If you only need a quick throwaway link, I can use a temporary address instead.)

### A temporary address: only when the user chooses it

`tunnel start <port> --temporary` needs no account and no key: localhost.run picks the address. Use it only when the user chooses it: they take the throwaway option above, say they don't want an account, or ask for a quick or temporary link themselves. Don't switch to it on your own because setup is a step, or to save time, and don't fall back to it when setup or `start` fails; fix the problem, or ask.

- **Say what it means, once,** when you give the URL: free addresses change periodically to prevent abuse, so the link will stop working at some point, and a stable domain is how to keep it the same.
- **Webhooks:** before using a temporary address for one, point out that the webhook setting will need updating each time the address changes, and offer setup again. Do what the user decides.
- **Don't nag.** After that, use what the user chose. If a later task would clearly be better on a stable domain (a link they're about to send someone, a webhook), you can mention it again, briefly.

## Which domain

- **No domain named:** `start` uses the saved one, from setup.
- **The user names a domain** ("put it on dev.mycompany.com"): `tunnel start <port> --domain dev.mycompany.com`. It must be one of theirs at https://admin.localhost.run/, and on the same account as the saved key. If it's denied, show the message: usually the domain isn't on that account, or isn't set up yet.
- **The user wants a different domain from now on:** run `tunnel verify --domain D --key K` with the new domain (and the saved key, from `tunnel config`). It checks the domain and saves it as the default.
- **A domain they don't have yet:** a managed one is claimed as in `references/setup.md`, step 2. Their own domain (a custom domain) needs DNS set up in the admin site first: point them to https://localhost.run/docs/custom-domains.

## Commands

| Command | What it does |
|---|---|
| `tunnel start <port>` | Connects the saved domain to `localhost:<port>`, replacing any tunnel this skill already has on that domain. Waits until it's live (up to 30 s), then checks the URL. Keeps reconnecting in the background if the connection drops. |
| `tunnel start <port> --temporary` | The same on a temporary address that localhost.run picks, with no setup. Only when the user has chosen it (see above). |
| `tunnel status` | Each domain's state (`connected`, `reconnecting`, `denied`, `key_refused`, `stopped`), its port, and its last message. |
| `tunnel events [N]` | The last N events (default 20), including each HTTP request: method, path, status and time. Use it to check that a webhook or a visitor's request arrived. |
| `tunnel check` | Whether the app answers on its local port, and then through the public URL. Run it when a visitor or webhook gets no answer: a local failure means the app isn't running (or is on another port), not a tunnel problem. Exits 0 when both answer. |
| `tunnel stop` | Disconnects the tunnel. |
| `tunnel verify --domain D --key K` | Checks the key signs in and the domain is on the account, then saves both. Used in setup. |
| `tunnel keys` | Lists the user's public keys with fingerprints. Used in setup. |
| `tunnel new-key [PATH]` | Creates a key just for localhost.run (default `~/.ssh/localhost_run`), never overwriting one. Used in setup. |
| `tunnel claim-url <name> --key K` | The link for claiming a managed domain, which also adds the key to the user's account. Used in setup. |

`start` and `status` also check the local port, so they say at once if the app isn't answering.

`start` exit codes: `0` connected; `2` domain denied (the message says why); `3` key refused; `4` no answer yet (it keeps retrying in the background).

`--domain D` and `--key K` override the saved settings for one command. For a temporary tunnel, pass `--temporary` to `stop`, `status`, `events` and `check` too.

## How to use it

1. **Start the app first,** and note its port. `start` works without it, but warns that nothing is answering.
2. **Run `tunnel start <port>`** (with `--domain` if the user named one), then give the user the URL it prints.
3. **Point the user at the URL** for whatever they need it for: a webhook setting, a phone, a teammate. Use `tunnel events` to confirm requests are arriving, and what the app answered.
4. **Run `tunnel stop`** when the work is done, or when the user asks. Don't leave a tunnel running after the task without saying so.

## Rules

- **It's public.** Anyone with the URL can reach the local app while the tunnel is up. Say so when you start one, and never tunnel something the user hasn't asked to share.
- **A temporary address can change.** If its tunnel reconnects, it may come back on a new address: `status` shows the current one. Tell the user if it changes.
- **One tunnel per domain.** localhost.run shares a domain's traffic across every connection holding it. `start` replaces this skill's own tunnel, but a tunnel the user started elsewhere (another terminal or machine) would still take some of the requests. If requests seem to go missing, ask about that.
- **No speed claims.** Don't tell the user how fast the tunnel is or will be. Paid plans have priority bandwidth, nothing more specific.
- **Keys stay private.** Only ever read, show or copy **public** keys (`.pub`). Never print, move or upload a private key.
- **Denials explain themselves.** When `start` or `verify` is denied, show the user the message as it is: it comes from localhost.run and says what to fix.
