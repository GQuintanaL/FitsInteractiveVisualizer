# FITS Cube 3D Viewer

A single-file, zero-install, browser-based viewer for position–position–velocity (PPV) FITS cubes. It turns the emission of a spectral-line cube into an interactive 3D point cloud, with the **edges of each iso-intensity surface highlighted** so that the kinematic structure (shells, outflows, rings, discs…) is easy to read.

Everything runs locally in your browser: your data never leave your computer.

![screenshot](docs/screenshot.png)
<!-- TODO: add a screenshot (e.g. a CO cube of an evolved star) in docs/screenshot.png -->

**Live demo:** https://GQuintanaL.github.io/FitsInteractiveVisualizer/ <!-- TODO: enable GitHub Pages -->

## Features

- Loads standard FITS cubes (`BITPIX` 8, 16, 32, −32, −64; `BSCALE`/`BZERO` honoured) by file dialog or drag & drop.
- **Real velocity axis**: built from `CRVAL3`/`CRPIX3`/`CDELT3`; frequency axes (`CTYPE3 = FREQ`) are converted to radio-convention velocity using `RESTFRQ`.
- **Adjustable velocity stretch** (0.2×–5×), applied live without recomputing, so the velocity axis is never squeezed.
- Automatic noise estimate (MAD) and emission threshold in units of σ.
- Linear interpolation between channels to obtain a smooth continuum along the velocity axis.
- **Contour enhancement**: points on the boundary of the emission (spatially *and* in velocity) are flagged and drawn larger/brighter (GPU shader, live slider).
- Two colour schemes: solid (structure) and flux gradient.
- Axes with angular offsets (arcsec) and velocities (km/s).

## Quick start

1. Open `index.html` in a modern browser (Chrome, Firefox, Edge, Safari). No build step, no server.
2. Click **Load FITS Cube** (or drop a file on the canvas).
3. Choose the threshold (σ), then click **Generate Continuous Structure**.
4. Use the green slider to stretch the velocity axis.

> The viewer loads [three.js r128](https://threejs.org) from a CDN. For offline use, download `three.min.js` and change the `<script src=...>` line.

## Controls

| Action | Effect |
|---|---|
| Left-drag | Rotate |
| Mouse wheel | Zoom |
| Right-drag | Pan |
| *Reset view* | Restore default camera |

## Parameters

| Parameter | Meaning |
|---|---|
| Velocity axis stretch | Length of the velocity axis relative to the largest spatial dimension (1.0× = equal) |
| Emphasize contours | Extra size of edge points |
| Continuum density (Z) | Number of interpolated sub-channels per real channel |
| Emission threshold | Cut level in units of the estimated noise σ |
| Point size / opacity | Rendering options |

## How it works (short)

1. The FITS header is parsed and a velocity is assigned to every channel.
2. σ is estimated as 1.4826 × MAD from the first/last channels (assumed line-free).
3. Channels are linearly interpolated; every voxel with flux ≥ *k*σ becomes a point.
4. A point is flagged as *edge* if any of its six neighbours (±RA, ±Dec, ±velocity) is below threshold or undefined.
5. Edge points are enlarged in a vertex shader; the velocity axis is scaled on the mesh, so the stretch slider is instantaneous.

See [`paper/main.tex`](paper/main.tex) for a more detailed description.

## Limitations

- Visualisation aid, **not** a quantitative analysis tool: the output is a point cloud, not a meshed isosurface.
- Spatial axes use `CDELT1/2` only (no rotation matrix, no non-trivial projections).
- The entire cube is loaded into memory (as Float32) in the browser; very large cubes may need cropping/binning first. The viewer stops above 6 M points: raise σ or lower the Z density.
- The first/last channels must be (mostly) line-free for the noise estimate to be meaningful.
- Only the first three FITS axes are used (a trailing Stokes axis of length 1 is fine).

## Citation

If you use this software in a publication, please cite it (see [`CITATION.cff`](CITATION.cff); GitHub shows a *Cite this repository* button):

```bibtex
@software{fits3d_viewer,
  author  = {Guillermo Quintana-Lacaci},
  title   = {FITS Cube 3D Viewer},
  year    = {2026},
  url     = {https://github.com/GQuintanaL/FitsInteractiveVisualizer},
  doi     = {10.5281/zenodo.XXXXXXX}
}
```

A short description paper is available on arXiv: *TODO: arXiv number*.

## License

Released under the [MIT License](LICENSE). three.js is © its authors, also MIT-licensed.

## Contributing

Issues and pull requests are welcome, especially: test files with unusual headers, WCS improvements, and performance reports on large cubes.
