# Repository instructions

**This repository is public.** Nothing internal belongs in it — not in
file content, not in commit messages, not in PR titles/descriptions, not
in code comments. Treat every push here as permanent and world-readable
from the moment it lands.

This repo was split out of `jiminy-sdk`, whose `CLAUDE.md` documents a real
incident (23 Aug 2026) where internal ticket IDs, sprint names, the private
repo's name, a feature codename, and `Claude-Session` URLs leaked into that
public repo's commit history and had to be scrubbed with a full
`git filter-repo` rewrite. The same rules apply here.

## Never include, anywhere in this repo or anything pushed to it

- Internal ticket IDs (e.g. `JIM-076`)
- Internal sprint/project names (e.g. `Sprint 3`)
- The private repo's name, in any casing (`JIMINY-dev`, `Jiminy-dev`)
- Internal feature codenames
- `Claude-Session:` links or any other private session/tooling URL
- A `Co-Authored-By: Claude` trailer
- Internal business/ops detail, pricing, or regulatory strategy

If work here needs to reference or mirror something from the private repo
or from `jiminy-sdk`, describe *what changed and why* in plain terms a
public reader could see, not the internal name for it.

## Commit messages

Do not append a `Co-Authored-By: Claude` trailer or a `Claude-Session:`
link to commits in this repository.
