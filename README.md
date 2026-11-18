# localhost.run skills for coding agents

```
npx skills add localhost-run/agent-skills
```

Then ask your agent: **"Put my app on localhost.run."**

---

Your coding agent can build and run an app, but it can't show it to anyone. This skill gives it a public HTTPS address for whatever it's running locally. Your domain stays the same every time, so you can bookmark it, paste it into a webhook setting once, or send it to a client, and it keeps working session after session.

- **Nothing to install** apart from the skill. It uses `ssh`, `bash` and `curl`, which your machine already has.
- **Works with your agent:** Claude Code, Codex, Gemini CLI, OpenCode, Cursor, GitHub Copilot and anything else that reads [Agent Skills](https://agentskills.io).
- **Sets itself up:** the first time, the skill walks you through choosing an SSH key and claiming a domain. A managed domain takes a minute and needs no DNS setup.
- **Tells you what's happening:** the agent can see each request that arrives, check whether your app is answering, and reconnect if the connection drops.

## What you need

- A localhost.run account with a stable domain. If you don't have one, the skill links you to claim one, ready set up with your SSH key, at [admin.localhost.run](https://admin.localhost.run).
- macOS, Linux or WSL, with `ssh`, `bash` and `curl`.

## Choosing your domain

The skill connects your app to **your** domain, so the address is one you know in advance and it never changes.

- **Your default domain** is the one you set up first. Just ask, and your app goes there:
  > Put my app on localhost.run.
- **A different domain, just this once:** name it.
  > Put the API on api-dev.mycompany.com.

  The agent runs `tunnel start <port> --domain api-dev.mycompany.com`. Any domain on your account works, including managed domains and your own (custom) domains.
- **A different default from now on:**
  > Use api-dev.mycompany.com for localhost.run from now on.

  The agent checks that the domain is on your account and saves it (`tunnel verify --domain … --key …`).
- **A domain you don't have yet:** ask for one, and the agent links you to claim a managed domain, which takes a minute and needs no DNS. To use your own domain, such as `dev.mycompany.com`, set it up in [the admin site](https://admin.localhost.run) first ([how](https://localhost.run/docs/custom-domains)), then name it as above.

A domain only works with an SSH key on the same account. If the agent says a domain was denied, it shows you localhost.run's reason: usually the domain is misspelled, not set up yet, or on another account.

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
> **Agent:** You haven't set up localhost.run yet. Which SSH key should it use? You have `~/.ssh/id_ed25519`, or I can create one just for localhost.run.
>
> **You:** Make a new one.
>
> **Agent:** Created `~/.ssh/localhost_run`. Do you already have a domain on localhost.run?
>
> **You:** No.
>
> **Agent:** What name would you like? … Open this link to claim it: `https://admin.localhost.run/claim?managed=true&name=…`. It adds your new key to your account too. Tell me your domain when you're done.
>
> **You:** Done, it's `…`.
>
> **Agent:** Verified: the key signs in and the domain is yours. Saved.

### Putting it on a specific domain

> **You:** Run the docs site and put it on docs-preview.mycompany.com.
>
> **Agent:** The docs site is running on port 4000.
> `connected: https://docs-preview.mycompany.com -> localhost:4000`
> `public: https://docs-preview.mycompany.com/ answers HTTP 200`
> It's live at https://docs-preview.mycompany.com. Your default domain is still the one you set up, so other projects aren't affected.

### Claude Code: sharing work in progress

> **You:** Start the dev server and give me a link I can send to the client.
>
> **Agent:** The app is running on port 5173.
> `connected: https://your-domain -> localhost:5173`
> `local: localhost:5173 answers HTTP 200`
> `public: https://your-domain/ answers HTTP 200`
> Here's the link: https://your-domain. Anyone with it can see the app while the tunnel is up. Should I stop it when we're done?

### Codex: developing a webhook

> **You:** Add a Stripe webhook handler at `/webhooks/stripe`, put the app on localhost.run, and tell me what URL to give Stripe.
>
> **Agent:** The handler is in. In the Stripe dashboard, set the endpoint to `https://your-domain/webhooks/stripe`. Send a test event when you're ready.
>
> **You:** Sent.
>
> **Agent:** It arrived: `request: POST /webhooks/stripe -> 400 (12 ms)`. The handler rejected the signature, because the webhook secret isn't in `.env`. Let me fix that.

Your domain doesn't change, so you set the webhook up in Stripe once, and it works in every later session.

### Gemini CLI: testing on your phone

> **You:** I need to check the layout on my phone.
>
> **Agent:** Open https://your-domain on your phone. I'll watch the requests: … your phone loaded `/`, `/app.css` and `/app.js`, all 200.

### OpenCode: "it's not working"

> **You:** The client says the link is broken.
>
> **Agent:** `local: nothing is listening on localhost:3000`. The tunnel is fine, but the dev server stopped (it crashed on a syntax error in `routes.ts`). Fixed and restarted: `https://your-domain/` answers HTTP 200 again.

### Cursor: finishing up

> **You:** That's it for today.
>
> **Agent:** I've stopped the tunnel to https://your-domain, so the app isn't public anymore. It'll be at the same address next time.

## What's inside

```
skills/localhost-run/
├── SKILL.md              when and how the agent uses localhost.run
├── references/setup.md   first-time setup: key, domain, verify
└── scripts/tunnel        the commands the agent runs
```

`scripts/tunnel` also works by hand:

| Command | What it does |
|---|---|
| `tunnel start <port>` | Connect your domain to `localhost:<port>`, and keep it connected |
| `tunnel start <port> --domain D` | The same, on another of your domains |
| `tunnel start <port> --temporary` | The same, on a temporary address, with no setup |
| `tunnel status` | Each tunnel's state, its port, and whether the app is answering |
| `tunnel check` | Is the app answering, locally and through your URL? |
| `tunnel events [N]` | Recent events, including each request and its status |
| `tunnel stop` | Disconnect |
| `tunnel verify --domain D --key K` | Check a domain and key work, and save them as the default |
| `tunnel keys` / `new-key` / `claim-url` | Setup helpers |

## Safety and privacy

- **Your app is public while the tunnel is up.** Anyone with the URL can reach it. The agent tells you when it starts a tunnel, and stops it when you're done.
- **Your private key never leaves your machine.** The skill only ever reads and shows the public half (`.pub`). The claim link carries the public key, without its comment.
- **Settings:** your domain and the path of your key are saved in `~/.config/localhost-run/agent.conf`. Tunnel state and recent events are in `~/.local/state/localhost-run/`.
- **Read it first:** skills run with your agent's permissions. The whole skill is three short files in this repo.

## More

- [localhost.run docs](https://localhost.run/docs/)
- [Your account and domains](https://admin.localhost.run)
- Using localhost.run without an agent: `ssh -R 80:localhost:8080 localhost.run`

## Licence

[MIT](LICENSE)
