# Fork development guide

This guide is for `madhakish/lex-microsoft_teams`. Upstream architecture remains in [CLAUDE.md](../CLAUDE.md); this file is the canonical fork-local validation and automation guidance. Use the checked-out source and [lib/legion/extensions/microsoft_teams/version.rb](../lib/legion/extensions/microsoft_teams/version.rb) for current behavior/version, rather than historical example counts.

Agent entrypoints: [AGENTS.md](../AGENTS.md) and the [review-teams-contracts skill](../.agents/skills/review-teams-contracts/SKILL.md).

## Change routing

- Follow runners in `lib/legion/extensions/microsoft_teams/runners/` through helpers, actors and transport. The spec helper supplies framework stubs and the Definitions DSL; unit success does not prove full Legion boot integration.
- For Graph list runners cover `@odata.nextLink`, page limits, empty pages and string/symbol response keys. Use fake responses and synthetic chat IDs.
- Send/reply definitions in `runners/messages.rb` are elevated and non-idempotent. Review permission guards and metadata alongside payload construction. Debug logs currently interpolate message content; do not run these paths with real messages.
- Actors can poll chats, ingest caches and process messages. Test callbacks directly with doubles; preserve observer-disabled defaults and high-water-mark behavior.
- OAuth PKCE/device flows, refresh tokens, browser callbacks, local Teams cache ingestion and Vault writes require separate explicit live-integration scope.

## Safe validation

No real Teams messages, subscriptions, consent flows, cache reads or polling during validation. No matching local-cache parser/extractor specs were found in the current fork; add synthetic parser tests before changing that path. Use synthetic Graph/Bot fixtures; never copy a personal Teams cache or production token into the repository.

Python 3.9+ is sufficient for the read-only helper; it uses only the standard library:

```sh
python3 scripts/spec_plan.py lib/legion/extensions/microsoft_teams/runners/messages.rb
python3 -m unittest discover -s scripts -p 'test_*.py'
```

Pass repository-relative changed paths to the planner. It returns existing spec candidates, cross-component review notes, unmapped files and stale curated patterns. It never loads Ruby or executes a test. Basename matching is conservative; inspect the result and add relevant integration coverage. It does not establish coverage or make the Ruby suite safe to run.

With Ruby >= 3.4 and dependencies provisioned in an approved disposable environment, inspect the spec helper, run the targeted specs, then `bundle exec rspec` and non-mutating `bundle exec rubocop`. Run `bundle check` before installing missing gems. Dependency installation may fetch packages and execute build hooks. If Ruby, dependencies or isolation are absent, report the blocked command; do not label static checks as passing RSpec. Autocorrection (`rubocop -A`) is an explicit code change, not a prerequisite to reading or reviewing this fork.

## GitHub Actions recommendations (not activated)

Run synthetic Graph/Bot pagination and payload contracts, runner-definition/permission tests and actor deduplication tests. Keep live Microsoft tenant tests disabled by default and outside routine CI; no OAuth tokens on the personal runner.

The checked-in workflows do not verify an available self-hosted runner. Direct jobs in this family use `ubuntu-latest`; reusable jobs call `LegionIO/.github` workflows at mutable `@main`. The inspected upstream workflows use `${{ inputs.runner || 'ubuntu-latest' }}` and these callers do not supply `runner`, so they inherit GitHub-hosted defaults at the reviewed snapshot. Before implementing the requested self-hosted policy, inspect the reusable workflows and the registered runner inventory; choose only verified labels. Do not guess OS/architecture labels or silently fall back to GitHub-hosted runners.

Because this is a public repository, never run unreviewed pull-request code on a personal persistent runner. Prefer an isolated ephemeral runner with no host mounts, production network access or reusable credentials. If only the personal runner exists, keep proposed validation disabled until that isolation is established. A trusted manual dispatch must validate an exact maintainer-reviewed commit; do not use `pull_request_target` to check out and execute a PR head. Minimize token permissions, pin reviewed actions/reusable workflows to immutable SHAs, set timeouts/concurrency, and separate test artifacts from secrets. Release, publishing and infrastructure changes need their own approval. No workflow files are changed by this development overlay.
