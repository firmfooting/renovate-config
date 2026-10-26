<h1>
<picture>
  <source media="(prefers-color-scheme: dark)"
    srcset="lockups/renovate-config_dark.svg">
  <img alt="renovate-config, by firmfooting"
    src="lockups/renovate-config.svg"
    height="56">
</picture>
</h1>

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
| `extends` | `config:best-practices` | The preset the Renovate maintainers recommend. It builds on `config:recommended` and adds digest pinning for Docker images and GitHub Actions, config migration pull requests, dev-dependency pinning, abandoned-package detection and a three-day wait on npm packages, which this file raises to 14 days. Current Renovate releases also add weekly lock file maintenance. |
| `extends` | `:dependencyDashboard` | One issue per repository that lists every pending and held update. It is already part of `config:recommended`. Naming it here keeps it on if that preset changes. |
| `semanticCommits` | `"enabled"` | Commit messages and pull request titles carry Conventional Commit prefixes. |
| `automergeStrategy` | `"squash"` | Each automerged update lands as a single commit. |
| `platformAutomerge` | `true` | Renovate hands the merge to GitHub's own auto-merge, which waits on the repository's required checks. |
| `prConcurrentLimit` | `5` | At most five Renovate pull requests are open in a repository at once. |
| `labels` | `["dependencies"]` | Every Renovate pull request is labelled `dependencies`. |
| `minimumReleaseAge` | `"14 days"` | The figure Renovate documents for automerging third-party dependencies. It gives registries time to pull a malicious release before it merges itself. |
| npm `minor` and `patch`: `minimumReleaseAge` | `"14 days"` | Brings npm back in line with everything else. Without this rule npm would wait three days: see below. |
| Majors: `minimumReleaseAge` | `"30 days"` | A major is where an API disappears or a default flips. A person reads it, so the longer wait costs nobody anything. |
| Majors: `automerge` | `false` | A major always waits for a person. A green suite does not reliably catch a removed API, a flipped default or a broken peer range. |
| `minor`, `patch`, `digest`, `pin`, `pinDigest`, `bump`, `lockFileMaintenance`: `automerge` | `true` | These merge themselves once CI is green. |
| Source `https://github.com/firmfooting/**`: `minimumReleaseAge` | `null` | This organisation's own releases (the shared workflows and every package whose source URL is a firmfooting repository) are tags cut and reviewed here, so a bump is offered, and below a major merged, as soon as CI is green. A wait would hold every dependent repository on the previous release for two weeks. A package with no source URL in its registry metadata is not matched, so publish one. |

Every update but this organisation's own releases waits 14 days, npm included.
`config:best-practices` extends `security:minimumReleaseAgeNpm`, which sets
npm's wait to `"3 days"` in a package rule, and a package rule overrides a
top-level setting. The npm rule
in this file comes after the preset's, so it wins and puts npm back at 14
days. It names only `minor` and `patch`: the preset deliberately clears the
wait for update types that carry no release timestamp, such as `pin`, `bump`
and `lockFileMaintenance`, because a wait on those would hold them forever.
Majors, npm included, wait 30 days under the majors rule.

Any held update can be requested early from the Dependency Dashboard.

## Security fixes skip the wait

When a dependency has a known vulnerability, Renovate raises the fix at once:
no 14-day or 30-day wait, no schedule, and no Dependency Dashboard approval.
Its commit message and title end in `[SECURITY]`. This is Renovate's default
`vulnerabilityAlerts` behaviour, which this file keeps. A fix below a major
then merges itself once CI is green, like any other minor or patch. A fix that
needs a major still waits for a person.

It applies at every severity, not only critical. Renovate applies these
settings with `force`, which overrides every package rule, so they cannot be
limited to one severity.

It needs a source of alerts. On GitHub, Renovate reads the repository's
Dependabot alerts, so turn Dependabot alerts on under *Settings → Security*
in each repository. Renovate never sees a vulnerability that GitHub has not
raised an alert for.

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

## Checks

| Check | Where it runs |
| --- | --- |
| JSON, YAML and merge-conflict hygiene, `renovate-config-validator`, markdownlint, zizmor (offline) | `.pre-commit-config.yaml`, through prek, locally and in the `prek` workflow |
| zizmor (online) | `zizmor.yml` |
| Conventional PR title | `pr-title.yml` |
| LF line endings | `line-endings.yml` |

Every workflow calls the shared definition in
[firmfooting/.github](https://github.com/firmfooting/.github). GitHub Actions do
not run on this private repository, so run `uvx prek run --all-files` before
pushing; it is the same command the `prek` workflow runs.

## Changing the policy

`github>firmfooting/renovate-config` names no tag, so Renovate reads
`default.json` from this repository's default branch each time it runs. A
commit merged to `main` reaches every repository that extends the preset,
directly or through an organisation's own `renovate-config`, on that
repository's next Renovate run. Nothing is released and nothing downstream
needs a change.

A mistake reaches every repository the same way. Before merging a change:

- Run the checks: `uvx prek run --all-files`. They include
  `renovate-config-validator` over `default.json`, since a preset that fails
  to parse stops Renovate in every repository that extends it. Install them
  as a git hook once with `uvx prek install`.
- Read `packageRules` in order. Where two rules set the same option for the
  same update, the later one wins. Rules from extended presets come first,
  then the rules in this file, then each repository's own.
- Check the automerge list. Adding an update type there makes it merge itself
  in every repository with green checks, including those with no required
  checks.
- Remember that `config:best-practices` is not pinned either. When Renovate
  changes what it includes, every repository picks that up too. Its npm wait,
  overridden above, is one such effect, so read the Renovate release notes for
  changes to it.
- Leave local decisions out of this file. A rule that only one repository
  needs belongs in that repository's `renovate.json`.

---

<sub>Part of <a href="https://github.com/firmfooting">firmfooting</a>: safe,
plain tooling for M365 and SharePoint operators.</sub>
