# nvda-dynamic-testing-devtools

For anything that should be able to work across both the framework and webpage repos:

- [nvda-dynamic-testing-framework](https://github.com/UD-CISC-375/nvda-dynamic-testing-framework): Team X's Playwright tests
- [nvda-dynamic-testing-webpage](https://github.com/UD-CISC-375/nvda-dynamic-testing-webpage): Team Y's site of accessible and broken UI patterns

Anything that deals with both repos lives here, and anything only one repo needs stays in that repo.

## Guides

- [Docker Compose guide](docs/docker-compose-guide.md): run the framework's tests against the site, using your local copies or branches from GitHub. Start with its Current Status section.
- [Branching convention guide](docs/branching-convention-guide.md): how branches are structured and named across the class's repos.

## Quick start

With Docker running and the team repos cloned next to this one (on `devops-integrate-docker/master` for now; see the guide's Current Status), run this from this folder:

```bash
docker compose up
```

When the log says `Serving HTML report`, open:

- http://localhost:9323 for the test report
- http://localhost:8080 for the site that was tested

The Docker Compose guide covers the setup, choosing which branches get tested, and common problems.

## Class resources

- [Class shared Google Drive](https://drive.google.com/drive/folders/1n-hi-56tc7gHOR0lVVZaEIHYB5KwKvo2?usp=sharing): team files, including the table with the agreed definition of a meaningful DOM change.
- [Team U research findings](https://docs.google.com/document/d/1CS3vxe0eyLiuNODllZ7G2dtVKJ1hVm7aw8bkK5Ejtzs/edit?usp=sharing): interview notes, NVDA resources, and user stories. Keep these in mind when adding to the repos.
- [Team U request form](https://docs.google.com/forms/d/e/1FAIpQLSeqnPxrcjRk0luBFpw2kYLcLyXafuPjr3Q8t2iQZB7oeRaHzw/viewform?usp=publish-editor): ask Team U to look into something.

## Contributing

Have a tool, script, or guide that works across both repos? Anyone in the class can open a pull request. It needs an approval from DevOps or a team manager; a few files that affect everyone's setup (currently `compose.yaml`) need DevOps. [.github/CODEOWNERS](.github/CODEOWNERS) lists who approves what.
