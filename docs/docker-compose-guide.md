# CISC 375 Docker Compose Guide

Fall 2026 DevOps Team

## Background

In order to test Team Y's site with Team X's framework without depending on the live GitHub Pages site, DevOps team has set up Docker Compose for the framework and webpage repositories. Docker runs programs in containers: isolated environments that carry their own copy of everything a program needs, built from a recipe called a Dockerfile. Docker Compose starts several containers from a single file, `compose.yaml`, and connects them. With one command, anyone can build the site and the tests from whichever code they choose, run every test against that copy of the site, and view the results in a browser. Because each container fixes its versions, a run should behave the same on a laptop, the class's Proxmox VM, or a GitHub Actions runner. This guide provides an overview of the setup, how to choose which code gets tested, and how to run it. It covers only the web side of the project; NVDA runs only on Windows and is not part of this setup.

## Current Status

This setup is meant to run on the class's Proxmox VM, along with our own GitHub Actions runners, so that tests run automatically and anyone should be able to open the latest report in their browser. The VM isn't ready yet, so for now there is no shared copy running. Until it is, you can run the setup on your own machine whenever you want to test something, as described below.

The Dockerfiles also haven't been merged into either team repository's `main` yet, so for now they only exist on each team repository's `devops-integrate-docker/master` branch. To use a local copy, check out that branch first (`git switch devops-integrate-docker/master`); to build from GitHub, use `FW=devops-integrate-docker/master` or `WS=devops-integrate-docker/master` wherever this guide leaves them empty. DevOps updates that branch from `main`, so it may be a few commits behind the latest code.

## Structure

There are exactly TWO containers in this setup:

1. **webpage:** Builds Team Y's site with Node 20 (the same version its GitHub Pages workflow uses) and serves it with nginx, a small web server. The site is served under `/nvda-dynamic-testing-webpage/`, because the `homepage` field in the site's `package.json` hard-codes that path into every link in the build. The tests reach it at `http://webpage/nvda-dynamic-testing-webpage/` over a private network that Compose creates; it can also be viewed at `http://localhost:8080`.
2. **tests:** Holds Team X's framework and the Chromium browser, using Microsoft's official Playwright image. Its version, set at the top of the framework's Dockerfile, must match the framework's Playwright version, and the build stops if it doesn't. It waits until the webpage container answers a health check, runs every test once in CI mode, then serves the HTML report at `http://localhost:9323`.

The files for this setup are split across THREE repositories. Anything that deals with both team repositories lives in the devtools repository, and anything only one repository needs stays in that repository:

1. **nvda-dynamic-testing-devtools:** `compose.yaml`, which defines both containers, plus these guides.
2. **nvda-dynamic-testing-framework:** Its own `Dockerfile`, unique to this repository, which builds the tests container (plus a small `.dockerignore` listing files Docker should skip). The Dockerfile also writes the small helper files the container needs (a wrapper config that points the tests at the site to test, and the script that runs the tests and serves the report), each under a comment explaining why it exists.
3. **nvda-dynamic-testing-webpage:** Its own `Dockerfile`, unique to this repository, which builds the webpage container and also writes nginx's settings (plus a small `.dockerignore`).

To use your local copies of the team repositories, clone them into a folder named `.clone` inside the devtools repository, keeping their default folder names, and run `docker compose up` from the devtools folder. Git ignores `.clone`, so the team repositories can never be committed to devtools by accident:

```
nvda-dynamic-testing-devtools/        compose.yaml and these guides (run commands here)
  .clone/                             ignored by Git
    nvda-dynamic-testing-framework/   its own Dockerfile, builds the tests container
    nvda-dynamic-testing-webpage/     its own Dockerfile, builds the webpage container
```

From the devtools folder, these commands clone both:

```bash
git clone https://github.com/UD-CISC-375/nvda-dynamic-testing-framework.git .clone/nvda-dynamic-testing-framework
git clone https://github.com/UD-CISC-375/nvda-dynamic-testing-webpage.git .clone/nvda-dynamic-testing-webpage
```

Folders whose names start with a dot are hidden by default in Finder and in `ls` (use `ls -a` to see them), but editors such as VS Code show them.

A team repository only has to be cloned if you want to use your local copy of it; otherwise it can come from GitHub (see Sources).

Nothing on the private network can be reached from outside except the two published ports: 9323 for the report and 8080 for the tested site. Once the Proxmox VM's networking is set up, anyone who can reach the VM should be able to open `http://<VM address>:9323`.

## Sources

Each container is built from a source chosen by an environment variable: `FW` for the framework and `WS` for the webpage. Each variable has exactly THREE states:

1. **Not set:** Builds from your local copy of that repository, including uncommitted changes. Local copies must be in the devtools repository's `.clone` folder, with their default folder names.
2. **Set but empty (`FW=`):** Builds from the repository's default branch on GitHub (see Current Status).
3. **Set to a name (`FW=<name>`):** Builds from that branch on GitHub. The name can also be a pull request as it would look once merged (`refs/pull/<number>/merge`) or a full commit hash.

When a variable names GitHub, Docker clones that branch into its own temporary storage and builds from it, so the repository does not need to be cloned on your machine. The Dockerfile comes from the same source as the code, so a branch can only be built if it includes that repository's Dockerfile. `FW` and `WS` are independent, so any combination works. They are variables rather than flags such as `--fw`, because `docker compose` rejects any flag it doesn't already know.

Some examples, each run from the devtools repository's folder:

1. **`docker compose up`:** Builds both the framework and the webpage from your local copies. Use this when you are changing both, or when you have both cloned and want to test exactly what is on your machine.
2. **`WS= docker compose up`:** Builds the framework from your local copy and the webpage from its default branch on GitHub. This is the usual command for Team X: you test your framework changes against the current site, and you don't need the webpage cloned.
3. **`FW= docker compose up`:** Builds the webpage from your local copy and the framework from its default branch on GitHub. This is the usual command for Team Y: you test your site changes with the current tests before they go live, and you don't need the framework cloned.
4. **`FW=refs/pull/151/merge WS= docker compose up`:** Builds the framework as pull request #151 would look once merged, and the webpage from its default branch. Use this to check someone's pull request without cloning anything.
5. **`FW=devops-integrate-docker/master WS=devops-integrate-docker/master docker compose up`:** Builds both from the `devops-integrate-docker/master` branch on GitHub, with nothing cloned but devtools. This one works today, before the Dockerfiles reach `main`.

For example, say a member of Team Y changes the accordion page and wants to check it before it goes live. They clone the webpage repository into the devtools repository's `.clone` folder and run `FW= docker compose up` from the devtools folder. Docker builds the site from their local copy and the framework from its default branch on GitHub, runs every test against their version of the site, and the report at `http://localhost:9323` shows whether anything broke, before anything is pushed.

## Usage

You need Docker with Compose (Docker Desktop on a laptop). The first run downloads about 4 GB, mostly the Playwright image, and later runs reuse it. From the devtools repository's folder:

1. **Start:** Run `docker compose up`, with any settings in front of it. When the log says `Serving HTML report`, open `http://localhost:9323` for the report and `http://localhost:8080` for the tested site.
2. **Stop:** Press Ctrl+C, then run `docker compose down`.
3. **Keep settings:** To avoid typing a setting every time, create a file named `.env` next to `compose.yaml` with one setting per line, such as `FW=`. `.env` is ignored by Git, so it stays personal. Note that `FW=` in `.env` means the default branch; to use your local copy, leave the line out. In PowerShell, set a variable with `$env:FW="<name>"` before running `docker compose up`.

The available settings are:

1. **`FW` / `WS`:** Where the framework and the webpage come from, as described above. Default: not set (local).
2. **`BASE_URL`:** Which site the tests visit. Default: the webpage container. Set it to the GitHub Pages address and run `docker compose up --no-deps tests` to test the live site instead.
3. **`SERVE_REPORT`:** `1` keeps the report up after the run; `0` exits with the test result instead, which is what GitHub Actions needs (run `docker compose up --exit-code-from tests`). Default: `1`.
4. **`REPORT_PORT` / `WEBPAGE_PORT`:** The ports for the report and the tested site. Default: `9323` and `8080`.

If something goes wrong, three error messages have simple causes:

1. **`unable to prepare context: path ".../.clone/..." not found`:** A repository you're using locally isn't in the `.clone` folder, or its folder has a different name.
2. **`open Dockerfile: no such file`:** That branch doesn't include the Dockerfile yet.
3. **`repository does not contain ref`:** The branch name is misspelled or doesn't exist.
