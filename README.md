# Bidirekt agent skill

`bidirekt` is an agent skill that writes and updates a service's Bidirekt contract files (`contracts/*.yaml`) from its code, checks them against the broker, and fixes them from a `can-i-deploy` report. It runs only the CLI commands that read and never changes anything in the broker; it ends with the `publish` command for you to run.

It uses only the open Agent Skills format (https://agentskills.io), so it works in any agent that `npx skills` installs to.

## Requirements

- The Bidirekt CLI and a broker it points at: https://bidirekt.com/docs/guides/installation
- Access to https://bidirekt.com/llms.txt. The skill keeps no copy of the contract rules or the CLI commands: it reads them there on every run, and stops when it cannot.

## Install

In the current project:

```sh
npx skills add bidirekt/skills
```

For every project, with `-g`:

```sh
npx skills add bidirekt/skills -g
```

## Update

```sh
npx skills update
```

The skill follows the `main` branch; there are no tags or versions.

## License

Apache License 2.0, in `LICENSE`.
