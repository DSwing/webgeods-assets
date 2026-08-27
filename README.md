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
- `webr/` — (planned) WebR core + R packages, same rationale.

Files are fetched by the `WebGeoDS.Runtime`/`WebGeoDS.Python`/`WebGeoDS.R`
JS modules in the main [webgeods](https://github.com/DSwing) project by
pointing `indexURL`/`baseUrl` here instead of the engines' own CDNs.
