# ITKR

R package that builds ITK from source and ships its headers + static libs so
downstream packages (ANTsRCore, ANTsR) can `LinkingTo: ITKR`. Nothing imports
ITKR at runtime; it only matters at build time.

## Remotes
- `upstream` = ANTsX/ITKR (stnava/ITKR redirects there); default branch `master`.
- `origin` = muschellij2/ITKR (fork). Work on a branch, push to `origin`,
  open PRs against `ANTsX/ITKR:master`.
- Never commit straight to `master`.

## Building
- `configure` / `configure.win` clone ITK at the commit pinned in `itktag`
  (top of `configure`) and build with cmake. A full build is slow (tens of
  minutes); don't run `R CMD INSTALL` locally just to check a CI/YAML change.
- Changing `itktag` changes every downstream binary; call it out in the PR.

## Releases
- Bump `Version` in DESCRIPTION in its own commit ("Bump version to X.Y.Z"),
  tag it `vX.Y.Z` (annotated, message "ITKR X.Y.Z"), and push the tag to
  `upstream`. Unlike ANTsRCore there is no post-commit hook or
  tag-version workflow — the tag must be created and pushed by hand.
- Pushing a `v*` tag runs `.github/workflows/release-binaries.yml`, which builds
  Windows and macOS (arm64) binaries for oldrel-1, release and devel and
  attaches them to the release, marked latest.
- To add/rebuild binaries for an existing tag:
  `gh workflow run release-binaries.yml -R ANTsX/ITKR -f tag=vX.Y.Z`

## Release asset names are an API
ANTsRCore and ANTsR CI download these from `releases/latest/download/`:
- Windows: `ITKR-<label>.zip` (label = oldrel-1 | release | devel), plus
  `ITKR-<label>-R<ver>.zip` and the legacy `ITKR-<ver>.zip`.
- macOS: `ITKR-macos-<arch>-<label>.tgz`, plus `...-R<ver>.tgz`. R >= 4.7
  builds `.tar.xz` on macOS; the workflow republishes it as `.tgz`.
Don't rename these without updating ANTsRCore's `release-binaries.yml` and
`package-build.yml` in the same change.

## Conventions
- R versions in CI are named by setup-r labels (oldrel-1, release, devel), not
  numbers.
- Match the existing comment style in workflows: short `#` lines that explain why.
