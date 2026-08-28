# webgeods-assets

Vendored engine assets for [webgeods.com](https://webgeods.com), served via
GitHub Pages instead of third-party CDNs (jsdelivr, webr.r-wasm.org) — part
of a plan to remove runtime dependencies on external services that could
change or disappear without notice.

## Contents

- `pyodide/v0.29.4/` — Pyodide core (`pyodide.mjs`, `pyodide.asm.js/.wasm`,
  `python_stdlib.zip`) plus `pyodide-lock.json` and package wheels, laid out
  flat in the same directory exactly like jsdelivr's own `full/` folder —
  this is what lets `pyodide.loadPackage(["geopandas"])` keep working
  unmodified, resolving names through `pyodide-lock.json` against this host
  instead of jsdelivr. Only 16 wheels are actually present here (the full
  dependency closure for `geopandas`: geopandas, shapely, fiona, pyproj,
  pandas, numpy, and their transitive dependencies — sizes verified by
  downloading each one, not estimated); `pyodide-lock.json` itself is
  the complete upstream index, so adding another package later is just
  dropping its wheel in this same folder, no lock-file editing needed.
  Also includes `jinja2`, `pydantic`, `pydantic_core`, `markupsafe`,
  `typing_extensions`, `typing_inspection`, `annotated_types` (Pyodide's
  own index, resolved via `resolve-py-deps.mjs` alongside geopandas) and
  `micropip` itself, plus three wheels NOT in Pyodide's index — fetched
  directly from PyPI and vendored here since micropip has no local-index
  entry for them: `maplibre-0.3.6`, `branca-0.8.2`,
  `eval_type_backport-0.4.0` (the Python↔MapLibre bridge — see
  shared/python.js's MICROPIP_PACKAGES and shared/map.js's
  `WebGeoDS.Map.fromPython()`). `resolve-py-deps.mjs` had its own bug
  fixed alongside this addition: it compared package names without PEP
  503 normalization, so `pydantic`'s dependency on `pydantic_core`
  (underscore) silently failed to match the lock file's `pydantic-core`
  (hyphen) key and was dropped with no warning — caught by diffing
  against a real `micropip.install("maplibre")` network capture.
- `webr/v0.6.0/` — WebR core (`webr.mjs`, `webr-worker.js`, `R.js/.wasm`,
  `libRblas.so`, `libRlapack.so`) plus the `vfs/` lazy-loaded library data
  files webR itself requests at runtime (locale/translation data, proj/
  udunits data needed by spatial packages, grDevices font metrics) —
  mirrors webR's `baseUrl` option target 1:1, captured from real network
  requests, not guessed. **Not currently used by the site** (see v0.4.3
  below) — kept vendored in case v0.6.0 becomes usable again.
- `webr/v0.4.3/` — same core file set, for webR v0.4.3 (R 4.4.2). This is
  the version the site actually points `baseUrl` at. Pinned here instead
  of the newer v0.6.0 because `terra` (a hard `Imports` of `mapgl`) fails
  to load its namespace on webR 0.6.0/0.5.4 — open upstream bug
  r-wasm/webr#621 (terra 1.9-27 imports 18 additional PROJ symbols that
  don't resolve in that WASM binding; terra 1.8-42, what 0.4.3 installs,
  is unaffected). Revert to a newer webR once that issue is closed.
- `webr/repo/` — mirrors webR's `repoUrl` option target: `PACKAGES.rds`
  plus R package closures, one per R/emscripten ABI path:
  - `bin/emscripten/contrib/4.6/` (R 4.6.0, for v0.6.0) — `sf` +
    `geojsonsf` closure, 13 packages: sf, geojsonsf, units, s2, wk,
    classInt, e1071, proxy, class, MASS, DBI, Rcpp, KernSmooth.
  - `bin/emscripten/contrib/4.4/` (R 4.4.2, for v0.4.3, **currently
    used**) — `sf` + `geojsonsf` + `mapgl` closure, 52 packages. Notably
    does NOT include `webr` as a downloadable package: the webR-patched
    `httpuv` lists `webr` in `Imports`, but that's webR's own JS-interop
    shim (`Remotes: webr=github::r-wasm/webr/packages/webr` in its
    DESCRIPTION) — baked into every R.wasm image at build time, not the
    unrelated CRAN package of the same name. `resolve-r-deps.R` treats
    `webr` as always-present for this reason (see its comments) — an
    earlier version of that script got this wrong and produced a
    154-package closure by resolving `webr` against the CRAN-mirrored
    package instead, pulling in moonBook/car/lme4/forecast/etc.
    Verified against a real network capture of `webr.installPackages()`,
    which never fetches a `webr_*.tgz`.

Files are fetched by the `WebGeoDS.Runtime`/`WebGeoDS.Python`/`WebGeoDS.R`
JS modules in the main [webgeods](https://github.com/DSwing) project by
pointing `indexURL`/`baseUrl` here instead of the engines' own CDNs.
