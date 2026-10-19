# cgra_pnr — notes for Claude sessions

Placer (`thunder/`) and router (`cyclone/`) for the CGRA, used by archipelago / `aha pnr`.
aha `mek` pins this repo's personal branch `thesis` (user rule: commit and push there, never `master`).

## Shared build in /aha

`/aha/cgra_pnr/cyclone` holds the shared router build (pycyclone `.so` + `router` binary) that
every session's `aha pnr` uses. Rebuild it by installing to a temp name and renaming, never in
place under a running PnR.

## Opt-in router search (c16f471, session cfc4174c)

`CYCLONE_REG_SEARCH=1` turns on a register-aware route search in `cyclone/src/route.{cc,hh}`.
Unset, routes are byte-identical to before (checked on a conv_3_3 design).

## CI (GitHub "Linux Wheel Test", `.github/workflows/linux.yml` → `scripts/ci.sh`)

Runs on every push (a commit message with `skip ci` skips it) and by hand (`workflow_dispatch`:
`POST /repos/StanfordAHA/cgra_pnr/actions/workflows/linux.yml/dispatches {"ref": "thesis"}`).
In the stock `stanfordaha/garnet:latest` container (aha master's other repos) it replaces
`/aha/cgra_pnr` with the workflow's checkout, builds the placer and router, `pip install -e`s
thunder and cyclone into the aha venv, then runs `aha map apps/pointwise` and
`aha pnr apps/pointwise --width 4 --height 4`.

- Before 2026-10-18 it had never passed: it fetched the branch inside the container, whose repo
  asks for credentials ("could not read Username"; fixed in 8b6658e by `docker cp` of the
  checkout, as lake's and garnet's CI do), and pip-installed with the image's default pip
  (conda Python 3.14, which this pybind11 cannot build against; c3fda9d uses `/aha/bin/activate`).
  First green run: 39187265953 (c3fda9d, dispatched).
