# renovate-config

The shared dependency policy for every organisation in this estate. One file,
extended by every repository, so a change to the policy is one commit rather
than a sweep.

## Using it

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>firmfooting/renovate-config"]
}
```

Each organisation also carries its own `<org>/renovate-config` repository whose
`default.json` extends this one. Renovate treats a repository by that name as
the organisation's default preset and folds it into the onboarding pull
request, so a NEW repository picks the policy up without anybody remembering
to add it.

Renovate's own guidance is to extend a shared preset explicitly from each
repository rather than to apply it invisibly through inherited config, because
an explicit `extends` can be traced back by whoever is reading the repository.

## What it sets

| Setting | Value | Why |
| --- | --- | --- |
| `config:best-practices` | | The preset the Renovate maintainers name, rather than `config:recommended`. Brings digest pinning, config migration, dev-dependency pinning and abandoned-package detection. |
| `minimumReleaseAge` | 14 days | The documented figure for automerging a third-party dependency. It gives registries time to pull a malicious release before it merges itself. |
| Majors | 30 days, never automerged | A major is where an API disappears or a default flips. A person reads it, so the longer wait costs nobody anything. |
| Everything else | automerged | Once CI is green. |

## The automerge caveat, stated plainly

Renovate will not automerge a branch with no passing checks, so a repository
without tests merges nothing and this fails safe.

That is not the same as a gate. Only branch protection with required checks
guarantees a check exists at all, and most repositories here have none, several
because they are private on a plan that does not offer it. Where a repository
genuinely matters, give it required checks; do not rely on this preset to
supply a safety it cannot.

## Local rules still win

A repository's own `packageRules` are merged after this preset's, so a local
rule overrides a shared one. That is how, for example, a rule that forbids
automerging a particular vendored binary keeps working under a preset that
automerges non-majors by default. Keep genuinely local decisions local: version
caps, package groupings, schedules and time zones.
