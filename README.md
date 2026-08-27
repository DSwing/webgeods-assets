# webgeods-assets

Vendored engine assets for [webgeods.com](https://webgeods.com), served via
GitHub Pages instead of third-party CDNs (jsdelivr, webr.r-wasm.org) — part
of a plan to remove runtime dependencies on external services that could
change or disappear without notice.

## Contents

- `pyodide/v0.29.4/` — Pyodide core (`pyodide.mjs`, `pyodide.asm.js/.wasm`,
  `python_stdlib.zip`, `pyodide-lock.json`) plus `packages/`: the full
  dependency closure needed for `geopandas` (16 wheels — geopandas, shapely,
  fiona, pyproj, pandas, numpy, and their transitive dependencies), verified
  against Pyodide's own lock file for this exact version.
- `webr/` — (planned) WebR core + R packages, same rationale.

Files are fetched by the `WebGeoDS.Runtime`/`WebGeoDS.Python`/`WebGeoDS.R`
JS modules in the main [webgeods](https://github.com/DSwing) project via
explicit URLs, bypassing each engine's own remote package index.
