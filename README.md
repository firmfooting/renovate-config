<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/firmfooting/branding/main/lockups/renovate-config_dark.svg">
  <img alt="renovate-config, by firmfooting" src="https://raw.githubusercontent.com/firmfooting/branding/main/lockups/renovate-config.svg" height="56">
</picture>

The shared Renovate dependency policy for every firmfooting-owned organisation.

[![Line endings](https://github.com/firmfooting/renovate-config/actions/workflows/line-endings.yml/badge.svg?branch=main)](https://github.com/firmfooting/renovate-config/actions/workflows/line-endings.yml)
[![Renovate preset](https://img.shields.io/badge/Renovate-preset-2A6B64)](https://docs.renovatebot.com/config-presets/)

One file, [`default.json`](default.json), extended by every repository, so a
change to the policy is one commit rather than a sweep.

## Using it

Extend the preset from the repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>firmfooting/renovate-config"]
}
```

When Renovate onboards a repository, it looks for a repository called
`renovate-config` with a `default.json` in the same organisation. If it finds
one, it suggests that preset as the only entry in the onboarding pull
request's `extends`. A new repository therefore picks the policy up without
anybody remembering to add it.

This repository is that preset for the firmfooting organisation itself. Every
other firmfooting-owned organisation carries its own `<org>/renovate-config`
repository, whose `default.json` extends this one. The policy lives here, and
each organisation's own repository only points at it.

Renovate's own guidance is to extend a shared preset explicitly from each
repository rather than to apply it invisibly through inherited config, because
an explicit `extends` can be traced back by whoever is reading the repository.

## What it sets

| Setting | Value | Why |
| --- | --- | --- |
| `extends` | `config:best-practices` | The preset the Renovate maintainers recommend. It builds on `config:recommended` and adds digest pinning for Docker images and GitHub Actions, config migration pull requests, dev-dependency pinning, abandoned-package detection and a three-day wait on npm packages. Current Renovate releases also add weekly lock file maintenance. |
| `extends` | `:dependencyDashboard` | One issue per repository that lists every pending and held update. It is already part of `config:recommended`. Naming it here keeps it on if that preset changes. |
| `semanticCommits` | `"enabled"` | Commit messages and pull request titles carry Conventional Commit prefixes. |
| `automergeStrategy` | `"squash"` | Each automerged update lands as a single commit. |
| `platformAutomerge` | `true` | Renovate hands the merge to GitHub's own auto-merge, which waits on the repository's required checks. |
| `prConcurrentLimit` | `5` | At most five Renovate pull requests are open in a repository at once. |
| `labels` | `["dependencies"]` | Every Renovate pull request is labelled `dependencies`. |
| `minimumReleaseAge` | `"14 days"` | The figure Renovate documents for automerging third-party dependencies. It gives registries time to pull a malicious release before it merges itself. npm is the exception: see below. |
| Majors: `minimumReleaseAge` | `"30 days"` | A major is where an API disappears or a default flips. A person reads it, so the longer wait costs nobody anything. |
| Majors: `automerge` | `false` | A major always waits for a person. A green suite does not reliably catch a removed API, a flipped default or a broken peer range. |
| `minor`, `patch`, `digest`, `pin`, `pinDigest`, `bump`, `lockFileMaintenance`: `automerge` | `true` | These merge themselves once CI is green. |

For npm packages, updates below a major wait three days.
`config:best-practices` extends `security:minimumReleaseAgeNpm`, which sets
`minimumReleaseAge` to `"3 days"` in a package rule. A package rule overrides
a top-level setting, so that rule beats the top-level `"14 days"` here. The
majors rule in this file comes later, so npm majors still wait 30 days. The
same Renovate preset also drops the wait for npm `pin` updates, because
Renovate cannot check a release age for them.

Any held update can be requested early from the Dependency Dashboard.

## The automerge caveat

Renovate will not automerge a branch with no passing checks, so a repository
without tests merges nothing and this fails safe.

That is not the same as a gate. Only branch protection with required checks
guarantees a check exists at all, and most repositories here have none, several
because they are private on a plan that does not offer it. Where a repository
genuinely matters, give it required checks. Do not rely on this preset to
supply a safety it cannot.

## Local rules still win

A repository's own `packageRules` are merged after this preset's, so a local
rule overrides a shared one. That is how, for example, a rule that forbids
automerging a particular vendored binary keeps working under a preset that
automerges non-majors by default. Keep genuinely local decisions local: version
caps, package groupings, schedules and time zones.

## Changing the policy

`github>firmfooting/renovate-config` names no tag, so Renovate reads
`default.json` from this repository's default branch each time it runs. A
commit merged to `main` reaches every repository that extends the preset,
directly or through an organisation's own `renovate-config`, on that
repository's next Renovate run. Nothing is released and nothing downstream
needs a change.

A mistake reaches every repository the same way. Before merging a change:

- Check that `default.json` is valid JSON and passes Renovate's validator:
  `npx --yes --package renovate -- renovate-config-validator default.json`.
  A preset that fails to parse stops Renovate in every repository that
  extends it.
- Read `packageRules` in order. Where two rules set the same option for the
  same update, the later one wins. Rules from extended presets come first,
  then the rules in this file, then each repository's own.
- Check the automerge list. Adding an update type there makes it merge itself
  in every repository with green checks, including those with no required
  checks.
- Remember that `config:best-practices` is not pinned either. When Renovate
  changes what it includes, every repository picks that up too. The npm wait
  above is one such effect, so read the Renovate release notes for changes to
  it.
- Leave local decisions out of this file. A rule that only one repository
  needs belongs in that repository's `renovate.json`.

---

<sub>Part of <a href="https://github.com/firmfooting">firmfooting</a>: safe, plain tooling for M365 and SharePoint operators.</sub>
