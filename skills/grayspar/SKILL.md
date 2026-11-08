---
name: grayspar
description: Deploys apps to Grayspar and shares them with chosen people, using the grayspar CLI. Use when the user asks to put an app online, deploy, redeploy, share or invite someone to an app, keep an app's data across deploys, set its environment variables, or read its logs on Grayspar.
license: Apache-2.0
compatibility: Requires Node.js 22 or later to install the Grayspar CLI, and network access.
metadata:
  author: grayspar
  version: "1.0"
---

# Grayspar

Grayspar builds an app from a folder, runs it at its own address, and keeps that address private: only the people the owner shares it with can open it. The user's code is the app; on Grayspar it becomes a project of one or more services. Everything happens through the `grayspar` CLI. Do not search the web for Grayspar; the CLI and this skill are the whole interface.

## Set up

1. Run `grayspar --version`. If the command is missing, tell the user you will install the Grayspar CLI with `npm install -g @grayspar/cli`, wait for their yes, then run it.
2. Run `grayspar whoami`. If it reports no sign-in, ask the user to run `grayspar login` themselves, since it opens their browser. Signing in the first time creates their Grayspar account, so a new user needs nothing else. Wait until they say they are signed in.
3. Before the first command that changes something, such as a deploy or a share, tell the user what you will run and wait for their yes.
4. If a command refuses, show the user the CLI's message as it is. Do not explain it in other words or work around it.

## Before deploying

1. Do not build or install the app locally first. Grayspar finds the app's services, installs their dependencies and builds them itself, from the source as it is.
2. Check whether the app writes files it must keep, such as a SQLite database or uploads. Anything outside a volume is lost on every deploy. If it does, note the folder for the deploy command, and make the app read its location from `GRAYSPAR_VOLUME_PATH` if it does not already.
3. Check each service that answers at a path other than `/`. Grayspar passes every request to that service with the path unchanged: a request for `/<path>/<route>` reaches it as `/<path>/<route>`, not `/<route>`. The service must serve its routes under that path. If a frontend's development proxy removes the path before passing requests on, as Vite's `rewrite` option does, change the service to serve its routes under its path before deploying.
4. Before changing the app's code, tell the user what will change and why, and wait for their yes.

## Deploy

```
Deploy progress:
- [ ] 1. grayspar deploy --yes   (with --volume <service>=<folder> if the app keeps files)
- [ ] 2. grayspar status
```

1. Run `grayspar deploy --yes` in the app's folder. The first deploy creates the project, named after the folder unless `--name <name>` gives another, and links the folder to it in `.grayspar/project.json`; later commands in that folder act on the same project. The output ends with a table of each service, its address and its status.
2. If the app keeps files, name the folder on that same deploy: `grayspar deploy --yes --volume app=/app/data`. The service's first version then starts with the folder kept. A service at the root of the app's folder is named `app`, and a service in a subfolder is named after that folder. A service keeps one volume, so the flag names one folder for each service and repeats only across services. A folder the service already keeps is left as it is, so the same command is safe on every deploy.
3. Run `grayspar status` and confirm each service reads Running.

For a project that is already deployed, `grayspar volume add <folder>` adds a volume and redeploys the service by itself; do not run `grayspar deploy` again for it.

To check the address without a browser, expect a redirect to sign-in: `curl` gets a 302 because the project is private, and that means it works. To see the app itself, the owner opens the address in their browser.

## Without a terminal

Add `--json` to any command when you need to parse the result. A command that needs an answer it cannot ask for refuses instead of waiting, and its JSON names the flag that supplies the answer:

```json
{ "status": "error", "error": "needs_a_path", "message": "…", "supply_with": "--path" }
```

Run the command again with that flag. `--yes` accepts the plan `grayspar deploy` would otherwise ask about.

## Share

- `grayspar access add alice@company.example` lets that person in and emails them the link. They sign in with Google, GitHub or a code sent to that address.
- `grayspar access add @company.example` lets in everyone who proves an address at that company. Several targets fit in one command.
- `grayspar access list` shows the list; `grayspar access remove <target>` takes someone off it.
- `grayspar access set --mode open` lets in anyone with the link, with no sign-in at all; `--mode restricted` returns to the list. An open project's requests carry no identity headers, so keep an app that relies on them restricted.
- `grayspar open-path add /webhooks` lets requests to that path and everything below it through without sign-in, for webhooks and similar callers. Only plain paths can be open.

## What the running app receives

- `PORT`: the port to listen on, on 0.0.0.0, and the port requests go to. Grayspar chooses it, 8080 unless the Dockerfile's `EXPOSE` names another, until the service saves its own with `grayspar env set PORT=3000`. A service with a path that doesn't listen on `PORT` fails to deploy while the running version stays live, so an app that hardcodes its port needs `PORT` set to that port.
- `GRAYSPAR_VOLUME_PATH`: the folder of the service's volume. Grayspar sets it, so `grayspar env list` does not show it.
- On every request to a restricted project, Grayspar names the signed-in visitor in `X-User-Email`, `X-User-Id`, `X-User-Name` (percent-encoded; read it with `decodeURIComponent`) and `X-User-Role`: `owner` or `member` for the project's team, `viewer` for anyone it was shared with. An app needs no login code of its own.
- `grayspar env set KEY=VALUE` saves a variable and redeploys the service by itself. A deploy reads `.env`, `.env.production` and `.env.local`, and with `--yes` sends each variable new to the project, except one whose value points at `localhost` or `127.0.0.1`. The output names each one left out; set its production value with `grayspar env set`. Before `--yes`, check that the `.env` files hold no value meant only for development. A name starting with `RAILPACK_` is refused, since Grayspar sets up the build itself; a language version comes from the project's own files, such as `engines` in `package.json` or `.python-version`.

## Read what happened

- `grayspar logs` shows what the service printed while running; `-f` keeps following; `--build` shows the last build log instead. Use `-s <service>` when the project has several services.
- The line `A new deploy replaced this version.` marks a version a deploy replaced. Lines just before it such as `[ELIFECYCLE] Command failed.` are the app's runtime reporting that stop, not a crash. A crash shows an error or stack trace from the app, with no such line after it.
- Deploying again while a deploy is still building replaces it: the earlier command ends with `A newer deploy of <project> replaced this one`, and the newer deploy carries on. A deploy that has started going live finishes first, and the newer one waits for it.
- `grayspar restart` runs a service again in place, keeping its volume.
