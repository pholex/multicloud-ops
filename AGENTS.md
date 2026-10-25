# Repository instructions

Before every commit, read `PUBLIC_CONTENT.md`, review the complete staged diff and proposed commit message, and run `python3 scripts/check-public-content.py --staged`. Enable the local hooks with `sh scripts/install-git-hooks.sh`; do not bypass them. Before pushing, scan the outgoing commit range. Report sensitive findings only by location and category. Preserve mandatory third-party license notices and attribution.

Work on a topic branch, commit locally, then merge into main. Push only when explicitly requested.
