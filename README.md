# localhost.run skills for coding agents

```
npx skills add localhost-run/agent-skills
```

Then ask your agent: **"Put my app on localhost.run."**

---

Your coding agent can build and run an app, but it can't show it to anyone. This skill gives it a public HTTPS address for whatever it's running locally. Your domain stays the same every time, so you can bookmark it, paste it into a webhook setting once, or send it to a client, and it keeps working session after session.

- **Nothing to install** apart from the skill. It uses `ssh`, `bash` and `curl`, which your machine already has.
- **Works with your agent:** Claude Code, Codex, Gemini CLI, OpenCode, Cursor, GitHub Copilot and anything else that reads [Agent Skills](https://agentskills.io).
- **Sets itself up:** the first time, the skill makes an SSH key just for localhost.run and walks you through setting up a domain. One that localhost.run provides takes a minute and needs no DNS setup.
- **Tells you what's happening:** the agent checks whether your app is answering, reconnects if the connection drops and, when you're working on a webhook, can see each request that arrives.

## What you need

- A localhost.run account with a stable domain. If you don't have one, the skill links you to set one up at [admin.localhost.run](https://admin.localhost.run).
- macOS, Linux or WSL, with `ssh`, `bash` and `curl`.

## Choosing your domain

The skill connects your app to **your** domain, so the address is one you know in advance and it never changes.

- **Your default domain** is the one you set up first. Just ask, and your app goes there:
  > Put my app on localhost.run.
- **A different domain, just this once:** name it.
  > Put the API on api-dev.example.com.

  The agent runs `tunnel start <port> --domain api-dev.example.com`. Any domain on your account works, including managed domains and your own (custom) domains.
- **A different default from now on:**
  > Use api-dev.example.com for localhost.run from now on.

  The agent checks that the domain is on your account and saves it (`tunnel verify --domain …`).
- **A domain you don't have yet:** ask for one, and the agent links you to [set one up](https://admin.localhost.run/domains/add): one localhost.run provides takes a minute and needs no DNS. Or name your own domain, such as `dev.example.com`: if it isn't set up yet, the agent gives you the link to the page that sets it up, including the DNS record it needs ([how custom domains work](https://localhost.run/docs/custom-domains)).

A domain only works with a key on the same account. If something isn't ready, localhost.run says why and usually gives a link to the page in your account that fixes it: the agent passes it on, waits for you, and connects again.

**Just need a quick link?** Tell the agent you want a temporary address, and it skips setup: `tunnel start <port> --temporary` gives you a free address picked by localhost.run. Free addresses change periodically to prevent abuse, so these links stop working after a while. That's fine for a quick look, but not for something you'll bookmark, share, or give to a webhook. Set up a stable domain whenever you're ready, and the address stays the same.

## Install

**Any agent** (the [`skills`](https://skills.sh) installer finds the agents you have and installs into each):

```
npx skills add localhost-run/agent-skills
```

Add `-g` to install it for all your projects rather than just this one. To choose agents yourself, add `-a`, e.g. `-a claude-code`, `-a codex`, `-a gemini-cli`, `-a opencode` or `-a cursor`.

**Without Node:** clone this repo and copy `skills/localhost-run` into your agent's skills folder: `.agents/skills/` in a project works for most agents, and `.claude/skills/` for Claude Code.

```
git clone https://github.com/localhost-run/agent-skills
mkdir -p .agents/skills && cp -r agent-skills/skills/localhost-run .agents/skills/
```

Start a new agent session afterwards, so it picks up the skill.

## Example workflows

Every example works the same way in any agent: just ask. The prompts below are only suggestions.

### First time: setting up

> **You:** I want to share my app with localhost.run.
>
> **Agent:** You haven't set up localhost.run yet. Setting it up takes about a minute, and gives you an address that stays the same every time. Shall we?
>
> **You:** Yes.
>
> **Agent:** I've made an SSH key just for localhost.run, in `~/.local/share/localhost-run/`. It can't be used for anything else, and you can remove it from your account any time. Do you already have a domain you'd like to use, either your own or one on your localhost.run account? Or shall I give you a link to set one up?
>
> **You:** Set one up.
>
> **Agent:** Open https://admin.localhost.run/domains/add, sign in or create an account, and pick a domain. Tell me the domain when it's set up. After that there's one more link, to connect this skill to your account.
>
> **You:** Done, it's `…`.
>
> **Agent:** Open this link to connect this skill to your localhost.run account, and check the page shows the fingerprint `SHA256:…` before you approve: `https://admin.localhost.run/ssh-keys/add?key=…`. Tell me when you've done it.
>
> **You:** Done.
>
> **Agent:** Connected: the key signs in and the domain is yours. Saved.

### Putting it on a specific domain

> **You:** Run the docs site and put it on docs-preview.example.com.
>
> **Agent:** The docs site is running on port 4000.
> `connected: https://docs-preview.example.com -> localhost:4000`
> `public: https://docs-preview.example.com/ answers HTTP 200`
> It's live at https://docs-preview.example.com. Your default domain is still the one you set up, so other projects aren't affected.

### Claude Code: sharing work in progress

> **You:** Start the dev server and give me a link I can send to the client.
>
> **Agent:** The app is running on port 5173.
> `connected: https://your-app.example.com -> localhost:5173`
> `local: localhost:5173 answers HTTP 200`
> `public: https://your-app.example.com/ answers HTTP 200`
> Here's the link: https://your-app.example.com. Anyone with it can see the app while the tunnel is up. Should I stop it when we're done?

### Codex: developing a webhook

> **You:** Add a Stripe webhook handler at `/webhooks/stripe`, put the app on localhost.run, and tell me what URL to give Stripe.
>
> **Agent:** The handler is in, and the tunnel is recording requests so I can see the webhook arrive. In the Stripe dashboard, set the endpoint to `https://your-app.example.com/webhooks/stripe`. Send a test event when you're ready.
>
> **You:** Sent.
>
> **Agent:** It arrived: `request: POST /webhooks/stripe -> 400 (12 ms)`. The handler rejected the signature, because the webhook secret isn't in `.env`. Let me fix that.

Your domain doesn't change, so you set the webhook up in Stripe once, and it works in every later session.

### Gemini CLI: testing on your phone

> **You:** I need to check the layout on my phone.
>
> **Agent:** Open https://your-app.example.com on your phone. I'll watch the requests: … your phone loaded `/`, `/app.css` and `/app.js`, all 200.

### OpenCode: "it's not working"

> **You:** The client says the link is broken.
>
> **Agent:** `local: nothing is listening on localhost:3000`. The tunnel is fine, but the dev server stopped (it crashed on a syntax error in `routes.ts`). Fixed and restarted: `https://your-app.example.com/` answers HTTP 200 again.

### Cursor: finishing up

> **You:** That's it for today.
>
> **Agent:** I've stopped the tunnel to https://your-app.example.com, so the app isn't public anymore. It'll be at the same address next time.

## What's inside

```
skills/localhost-run/
├── SKILL.md              when and how the agent uses localhost.run
├── references/setup.md   first-time setup: key, domain, verify
├── scripts/tunnel        the commands the agent runs
└── scripts/known_hosts   localhost.run's SSH host key
```

`scripts/tunnel` also works by hand:

| Command | What it does |
|---|---|
| `tunnel start <port>` | Connect your domain to `localhost:<port>`, and keep it connected |
| `tunnel start <port> --domain D` | The same, on another of your domains |
| `tunnel start <port> --hours N` | The same, stopping by itself after N hours instead of 8 |
| `tunnel start <port> --requests` | The same, and record each request, for `tunnel events` |
| `tunnel start <port> --temporary` | The same, on a temporary address, with no setup |
| `tunnel status` | Each tunnel's state, its port, and whether the app is answering |
| `tunnel check` | Is the app answering, locally and through your URL? |
| `tunnel events [N]` | Recent events, including each request and its status if `--requests` is on |
| `tunnel stop` | Disconnect |
| `tunnel verify --domain D` | Connect to a domain with the key and save it as the default, or show the link that fixes it |
| `tunnel key` | The skill's own SSH key: creates it the first time, then shows it |

## Safety and privacy

- **Your app is public while the tunnel is up.** Anyone with the URL can reach it. The agent tells you when it starts a tunnel, and stops it when you're done. If it's forgotten, it stops by itself after 8 hours anyway, and nothing ever starts it again on its own.
- **The skill doesn't touch your SSH keys.** It never reads `~/.ssh`. It makes its own key, `~/.local/share/localhost-run/id_ed25519`, used only for localhost.run, which you can remove from your account at any time. Its private half never leaves your machine. To add it to your account, localhost.run gives the agent a link that adds the key once you sign in, check its fingerprint and approve. The agent only shows you links to admin.localhost.run, and only a key link for its own key's fingerprint. It has no passphrase, because your agent can't type one; that's why it's a separate key that unlocks nothing else. The skill has no option for using another key, so it can't be pointed at yours.
- **Your SSH setup isn't involved.** The skill's `ssh` reads no `ssh_config` (yours or the system's) and doesn't use your ssh-agent, so nothing you've configured for other hosts applies to it. If your network only allows SSH through a proxy set up in `~/.ssh/config`, the skill won't connect.
- **The connection is checked against a pinned host key.** localhost.run's server key ships with the skill, in `scripts/known_hosts`, and the skill connects only if the server presents that exact key, the first time too. It never writes to your `~/.ssh/known_hosts`.
- **Visitors' requests are only recorded when needed.** Request paths are written by whoever visits your URL, so they could contain text aimed at your agent. The agent turns recording on only for tasks like webhook debugging. It keeps paths encoded, drops query strings, and treats them as data, never instructions.
- **Settings:** your domain is saved in `~/.config/localhost-run/agent.conf`. The skill's key is in `~/.local/share/localhost-run/` (never in `~/.config`, which people often sync to dotfiles repos). Tunnel state and recent events are in `~/.local/state/localhost-run/`.
- **Read it first:** skills run with your agent's permissions. The whole skill is four short files in this repo.

## More

- [localhost.run docs](https://localhost.run/docs/)
- [Your account and domains](https://admin.localhost.run)
- Using localhost.run without an agent: `ssh -R 80:localhost:8080 localhost.run`

## Licence

[MIT](LICENSE)
