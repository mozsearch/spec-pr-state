# spec-pr-state

Snapshots of the open pull requests' metadata of the spec repositories that
searchfox's `wubspecs` tree summarizes, at `/wubspecs/spec-prs` (see
[mozsearch's scripts/spec-prs/README.md](https://github.com/mozsearch/mozsearch/blob/hyperblame/scripts/spec-prs/README.md)).

An hourly GitHub Action ([snapshot.yml](.github/workflows/snapshot.yml)) runs
mozsearch's `scripts/spec-prs/snapshot.py` with its own `GITHUB_TOKEN`, for
the repositories the tree configures (its `spec-prs.toml` in
[mozsearch-mozilla](https://github.com/mozsearch/mozsearch-mozilla/tree/firefox-disco/wubspecs)).
It publishes the snapshot as the single commit of the `snapshot` branch,
force-pushed each time, so this repository doesn't grow:

    https://raw.githubusercontent.com/mozsearch/spec-pr-state/snapshot/snapshot.json

The tree's syncs read it, so searchfox never handles a GitHub token. A sync
gets everything else, the pull requests' heads and what they change, through
git.

Beside it, the Action publishes the merged pull requests of all of the
tree's specs' repositories (`scripts/spec-prs/merged.py`): their numbers,
merge commits and the issues they close, for the tree's blame popups' links
to commits' pull requests and issues. The tree's indexing downloads it:

    https://raw.githubusercontent.com/mozsearch/spec-pr-state/snapshot/merged.json

Each run asks only for the pull requests updated since the last one's (about
20 queries, 100 points, for 391 repositories). The first run pages through
all of them: on 2026-10-30, 52,281 in 768 queries and 8.5 minutes, in one
run. If a run runs out of time (15 minutes) or points (it leaves 200 for the
next run's open pull requests), the next one continues where it was. If the
step fails, the last `merged.json` is published again.

- **Which repositories:** change the tree's `spec-prs.toml`; the next run
  follows it.
- **Run it now:** the Action's "Run workflow" button (`workflow_dispatch`).
- **Rate limits:** a GraphQL query per 100 pull requests, review decisions
  included (about 90 queries an hour for 85 repositories, of the token's
  1,000). On a 403 or 429, it waits as GitHub says for up to 10 minutes, then
  publishes what it has, with the repositories it didn't finish marked
  `incomplete` (which syncs skip, or for one repository, ask the REST API
  for).
- **Scheduled runs:** GitHub disables a public repository's scheduled
  workflows after 60 days without activity; if the snapshot stops updating,
  re-enable the workflow in the Actions tab.
