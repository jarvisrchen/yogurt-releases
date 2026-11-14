# Set up Yogurt Cloud

Yogurt Cloud is a webapp that runs on your own Vercel account, with its data in your own Supabase project.
Your Macs push meetings to it, and your phone records and reads them there.
Nobody else, Yogurt included, can see what is in it.

This guide works for a person and for a coding agent such as Claude Code or Codex.
To hand it to an agent, paste: `Set up Yogurt Cloud for me by following https://github.com/jarvisrchen/yogurt-releases/blob/main/cloud/SETUP.md`.

## What you need

- A Yogurt Pro license.
  Yogurt on your Mac shows it under Settings, License, with a button that copies it.
- Node.js 20 or newer (`node --version`).
- A free [Supabase](https://supabase.com) account.
  A free organization holds 2 active projects, and this uses one.
- A free [Vercel](https://vercel.com) account.
- Two tokens, made in your browser:
  - a Supabase access token from [supabase.com/dashboard/account/tokens](https://supabase.com/dashboard/account/tokens), used only while the deploy runs;
  - a Vercel token from [vercel.com/account/settings/tokens](https://vercel.com/account/settings/tokens), which the Cloud keeps so it can update itself.
    Give it the scope of the account or team that will hold the Cloud, and No expiration, because updates stop when it expires.
- An owner email, the one login your Cloud accepts.
  Use your Supabase account's email, because Supabase's built-in mail only sends to members of your Supabase organization, and the login needs its confirmation email.

## Rules for a coding agent

- Never ask for, print, or type the user's tokens, license, Cloud password, or PIN.
  The user types tokens and the license at `deploy.mjs`'s hidden prompts, or exports them in their own shell.
- Never type an LLM or Deepgram API key yourself.
- Ask before installing anything.
- If a step fails, stop and tell the user what failed.
- `deploy.mjs` waits up to 30 minutes for the user to create their login.
  Run it where it can keep running that long, and pass on the Cloud address it prints as soon as it appears.

## 1. Download the bundle

```bash
mkdir -p ~/yogurt-cloud && cd ~/yogurt-cloud
curl -fLO https://github.com/jarvisrchen/yogurt-releases/releases/latest/download/yogurt-cloud.tar.gz
curl -fLO https://github.com/jarvisrchen/yogurt-releases/releases/latest/download/yogurt-cloud.tar.gz.sha256
shasum -a 256 -c yogurt-cloud.tar.gz.sha256
tar xzf yogurt-cloud.tar.gz
cd yogurt-cloud-*/
```

After this first setup, the Cloud [updates itself](#update).

## 2. Optional: chat, Enhance, and recording in a browser

The Android app can use its own keys for these, which stay on the phone.
For chat, Enhance, and recording in a browser or on an iPhone, the Cloud needs keys of its own.
Export any of these in the same shell before step 3, and `deploy.mjs` saves them to your Vercel project, Production only, the keys as Sensitive:

| Variable | Value |
| --- | --- |
| `YOGURT_LLM_BASE_URL` | Any OpenAI-compatible API, such as `https://api.minimax.io/v1` |
| `YOGURT_LLM_MODEL` | A model at that API, such as `MiniMax-M3` |
| `YOGURT_LLM_API_KEY` | That provider's key |
| `YOGURT_DEEPGRAM_API_KEY` | A [Deepgram](https://console.deepgram.com) key with the Member role or higher |

You can also add them later in Vercel, under the project's Settings, Environment Variables, as Sensitive and Production only, then run step 3 again so they take effect.

## 3. Deploy

```bash
node deploy.mjs --owner you@example.com
```

It asks for the Supabase token, the Vercel token, and your license without echoing them, or reads `SUPABASE_ACCESS_TOKEN`, `VERCEL_TOKEN`, and `YOGURT_LICENSE` from the environment.
On the first run it:

1. creates a Vercel project named `yogurt-cloud` (`--name` picks another, `--team` puts it in a Vercel team);
2. creates a Supabase project of the same name (`--org` picks the organization when you have several, `--region` the region, default `us-east-1`), with a random database password;
3. saves the Supabase URL, publishable key, and owner email to the Vercel project, and as Sensitive the license and what updates need: the Vercel token and the database's connection string with that password;
4. applies the database schema;
5. deploys the prebuilt webapp and its functions;
6. points Supabase's login emails at the Cloud's address;
7. prints the **Cloud address**, such as `https://yogurt-cloud.vercel.app`, and waits for you to create the login, as in step 4;
8. closes sign-ups once that login is confirmed, so nobody else can make one.

If the Supabase project has Confirm email turned off, anyone could create the owner's login without the email, so `deploy.mjs` stops before closing sign-ups and says how to turn it on.

Without a license that covers this bundle, the Cloud shows "Yogurt Cloud needs a Pro license" in place of the app, and `deploy.mjs` says why.
A Pro purchase covers every bundle released on or before the day its updates end, so after that, deploy the bundle from an earlier release or renew.

## 4. Create the login

While `deploy.mjs` waits:

1. Open the Cloud address and select Create your Cloud login.
2. Use the owner email and a password of 6 or more characters, then select Create login.
3. Select the link in the confirmation email.
   Supabase sends only 2 emails an hour, so do not retry quickly, and check spam.

`deploy.mjs` then closes sign-ups and finishes.
It waits up to 30 minutes.
If it gives up, the Cloud still works and sign-ups stay open: confirm the login, then run `node deploy.mjs` again with the same `--name` and `--team` to close them.

Sign in at the Cloud address and choose a 4 or 6 digit PIN.
The Cloud now shows "No machine has pushed yet".

## 5. Connect each Mac

In Yogurt on the Mac, open Settings, then Cloud sync, then Connect.
Enter the Cloud address, the login email and password, and a machine name such as "Work laptop".
Select Sync now, or pick an Auto-push interval.

## 6. Set up the phone

**Android:**

1. Install [Yogurt for Android](https://github.com/jarvisrchen/yogurt-releases/releases/latest/download/yogurt-android.apk) and enter the Cloud address.
2. Sign in and type the PIN.
3. For chat and Enhance, add an LLM provider key in the app's Settings, then Model.
4. To record without Deepgram, pick Local in Settings, then Transcription. The audio then stays on the phone.

**iPhone or another browser:** open the Cloud address, add it to the home screen, sign in, and type the PIN.
Chat, Enhance, and recording there need the variables from step 2.

## Update

The Cloud updates itself.
Once a day it checks [yogurt-releases](https://github.com/jarvisrchen/yogurt-releases/releases/latest) for a newer bundle.
When there is one, it checks the download against the release's `.sha256`, applies any schema changes, and deploys the new webapp to your Vercel project with your saved Vercel token.
Your meetings, login, and PIN stay as they are.

- It never installs a bundle that fails its check, an older version, or one your license does not cover.
- If an update fails, the running Cloud stays as it was, and the next day's check tries again.
- Settings, About, in the webapp shows the version the Cloud runs and when it updated, and why the last update failed, if it did.
- A Cloud updates before your Macs do, and each release keeps taking pushes from the Mac release before it.
  A Mac too old for its Cloud says to update Yogurt on that Mac.

A Cloud set up before updates existed starts updating once you run `node deploy.mjs` from a new bundle, as in steps 1 and 3, with the same `--name` and `--team`.
That run gives the Supabase database a new random password, which nothing else used, and saves it for updates.
Running `deploy.mjs` from a newer bundle yourself still works at any time, and it also replaces a Vercel token that expired or that you revoked.

## Where things live

| What | Where |
| --- | --- |
| Meetings, the login, the PIN | Your Supabase project |
| The webapp and its functions | Your Vercel project |
| The Supabase URL, publishable key, owner email | The Vercel project's environment variables |
| LLM and Deepgram keys | The Vercel project's environment variables, Sensitive, Production only |
| Your license | The Vercel project's environment variables, Sensitive, Production only, read by its `/api/license` function and by updates, which install only bundles it covers |
| Your Vercel token | The Vercel project's environment variables as `YOGURT_VERCEL_TOKEN`, Sensitive, Production only, read only by its daily `/api/update` function. It can do anything in the Vercel account or team it is scoped to, and the Cloud uses it only to read its own project and deploy to it. Revoking it in Vercel stops updates and nothing else. |
| The database password | Inside `POSTGRES_URL_NON_POOLING`, the Vercel project's environment variables, Sensitive, Production only, read only by `/api/update` to apply new schemas |
| A random secret, `CRON_SECRET` | The Vercel project's environment variables, Sensitive, so only Vercel's daily cron can start an update |
| Your Supabase token | Nowhere: `deploy.mjs` uses it for one run |
