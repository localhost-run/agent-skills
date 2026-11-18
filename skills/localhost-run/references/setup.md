# Setting up localhost.run

Walk the user through these steps once. At the end, `tunnel verify` saves their domain and key, and `tunnel start` just works.

Ask one question at a time, and wait for the user where a step happens in their browser.

If the user says partway through that they'd rather not set up an account, that's their call: `tunnel start <port> --temporary` gives them a temporary address (see "A temporary address" in `SKILL.md`). They can come back to this guide any time.

## 1. A key

The key comes first: it's what the tunnel signs in with, and the quickstart link in step 2 adds it to the user's account.

Run `tunnel keys` to list the user's SSH keys.

- **One key:** suggest it, and check the user is happy to use it for localhost.run.
- **Several keys:** list them (path, type and fingerprint) and ask which to use. If one is already on their localhost.run account, that's the one; if they don't know, any will do, since step 2 or 3 adds it.
- **None, or they'd rather keep localhost.run separate:** offer to create a key just for it:
  ```
  tunnel new-key
  ```
  This creates `~/.ssh/localhost_run` (and `.pub`), refuses to overwrite anything, and has no passphrase, because an agent can't type one. Ask before running it. If the user wants a passphrase, they can create the key themselves with `ssh-keygen -t ed25519 -f ~/.ssh/localhost_run`, and must then run `ssh-add ~/.ssh/localhost_run` before each session.

Never show, copy or move a private key. Only `.pub` files are shared.

## 2. A domain

Ask: **"Do you already have a domain on localhost.run?"**

- **No (the quickstart):** a **managed domain**. localhost.run provides it, so there's no DNS to set up.
  1. Ask what name they'd like: short, lowercase letters, digits and hyphens (e.g. their name or the project's).
  2. Make the link, which carries the name and the key's public half:
     ```
     tunnel claim-url <name> --key <key path>
     ```
  3. Give the user the link to open in their browser. Explain that they'll sign in or create an account there, choose a plan if they need one, and claim the domain; the key is added to their account at the same time. Opening it again is harmless.
  4. Ask them to come back with the domain they ended up with, exactly as the admin site shows it.
- **Yes:** ask for it, exactly as it appears at https://admin.localhost.run/. This includes custom domains the user has already set up there (their own domain, with DNS done). Their key may not be on the account yet: step 3 will say so, and you can then show them the `.pub` file's contents to paste into their account at https://admin.localhost.run/.

Don't offer to set up a custom domain from scratch here. It needs DNS changes, and the user does it in the admin site. If they ask, point them to https://localhost.run/docs/custom-domains, and come back to this step once it's done.

## 3. Verify and save

```
tunnel verify --domain <domain> --key <key path>
```

| Exit | Result | What to do |
|---|---|---|
| 0 | It signed in, the domain is on the account, and both are saved. | Done. Offer to start a tunnel. |
| 3 | Key refused: the key isn't on any localhost.run account. | Quickstart: check the claim finished in the browser. Existing domain: have the user add the `.pub` file's contents at https://admin.localhost.run/. If the key has a passphrase, run `ssh-add` on it first. Then verify again. |
| 2 | Denied: the key signed in, but the domain isn't on that account. | Show the message. Check the spelling, that the claim finished, and that it's the same account the key is on. |
| 4 | No answer. | Check the network, then try again. |

The settings go to `~/.config/localhost-run/agent.conf` (`domain=` and `key=`). They're for this user, not the project, so every project uses the same domain unless `--domain` says otherwise.
