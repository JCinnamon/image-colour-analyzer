# Batch Image Analyzer — Vis-O-Matic

A browser-based tool for measuring colour across a whole set of images at once. It clusters each image into its dominant colours, reports colour values in HSV, CIELAB and LCH, computes a suite of whole-image and spatial metrics, reads EXIF (including GPS), and exports everything as CSV. It handles ordinary flat photos as well as **360° / wide-angle imagery** (equirectangular, fisheye, and azimuthal / tiny-planet projections), with solid-angle weighting, projection conversion, and polar spatial analysis. The tool is subject-agnostic — axes are labelled generically in degrees, so it is not specific to skies. Nothing is uploaded — analysis runs locally in your browser.

**[Live demo →](https://www.jonathancinnamon.com/image-colour-analyzer/)**

Part of [Vis-O-Matic](https://www.jonathancinnamon.com/vis-o-matic/), alongside [Image Cutter](https://www.jonathancinnamon.com/image-cutter/) and [Image Visualizer](https://www.jonathancinnamon.com/image-colour-visualizer/).

---

## Features

- **Batch colour analysis** — drop in a folder or a list of URLs; each image is downsampled to an analysis resolution you set, then measured locally.
- **K-means clustering** — k-means++ initialisation with a fixed seed for reproducibility; 2–20 dominant colours per image, each reported in HSV, CIELAB and LCH with its pixel share.
- **Three levels of granularity** — cluster-level, image-level, and optional pixel-level tables, plus a hue histogram.
- **EXIF passthrough** — GPS, capture date, camera and orientation read from JPEGs and from PNGs carrying an eXIf chunk, including cut-outs exported by [Image Cutter](https://www.jonathancinnamon.com/image-cutter/) with GPS/EXIF preserved.
- **Transparency handling** — composite transparent areas onto white, black or grey, or analyse opaque pixels only.
- **360° / wide-angle mode** — equirectangular, fisheye, stereographic/tiny-planet, Lambert equal-area and equidistant-azimuthal projections, each with its correct solid-angle weighting; convert between projections (export as a ZIP), and run spatial analysis in Cartesian or polar coordinates.
- **CSV exports** for every table, with consistent `SPACE_CHANNEL_STAT` column names; **per-image PNG** visualizations chosen from a dropdown on each result.

---

## Metrics

**Colour central tendency and spread (image level).** Every channel is reported as both a mean and a median — RGB (R/G/B), HSV (H/S/V), CIELAB (L\*/a\*/b\*) and LCH (L/C/H) — with standard deviation and IQR for saturation and chroma, and circular variance for hue. Hue means and medians are computed circularly. Medians resist outliers such as the sun or residual obstructions, so a mean/median divergence flags contamination.

**Column naming.** All CSVs use a consistent `SPACE_CHANNEL_STAT` scheme. The image summary uses names like `HSV_V_mean`, `HSV_H_median`, `LAB_L_mean`, `LCH_C_median`; the per-colour tables (cluster, pixel), which hold a single colour, use `HSV_H`, `HSV_S`, `HSV_V`, `LAB_L`, `LAB_A`, `LAB_B`, `LCH_L`, `LCH_C`, `LCH_H`. Identity columns (filename, gps_lat, …) and scalar descriptors (colourfulness, redness, cct, …) keep their plain names.

**Scene descriptors.** Colourfulness (Hasler–Süsstrunk), a redness index (R−B)/(R+B), correlated colour temperature (CCT, via McCamy), RMS contrast, a dark-channel haze estimate, warm/cool/neutral pixel proportions, hue entropy, and hue circular variance.

**Colour spaces.** HSV separates hue, saturation and brightness; CIELAB/LCH are perceptually oriented, so numerical distances track perceived colour difference more faithfully. LAB/LCH values appear in the cluster, pixel and image tables when the **LAB/LCH** option is ticked (on by default); untick it to omit all LAB/LCH columns from every CSV.

**Per-pixel descriptors.** The pixel-level table also carries the descriptors that are defined at a single pixel: `redness`, `cct`, `dark_channel`, and a `hue_class` label (warm / cool / neutral). Set-level metrics that have no per-pixel value — colourfulness, RMS contrast, hue entropy, Moran's I — remain image- or grid-level only.

---

## Spatial structure (optional)

Tick **Spatial structure** to add, using the pixel positions the tool already has:

- **Row and column profiles** — mean colour and metrics per image row and per column (CSV). Captures gradients in any orientation.
- **Grid / tile table** — an N×N grid (size configurable) with per-cell colour and metrics (CSV), for spatial heat-maps and region comparison.
- **Cluster spatial signature** — centroid (absolute and relative) and spatial spread of each dominant colour, added to the cluster table.
- **Moran's I** — spatial autocorrelation (rook contiguity) of the chosen channels, in the image summary with a companion `morans_i_channel` column: one index of how spatially organised the colour is, from clustered (positive) to salt-and-pepper (negative). Tick the channels to compute in the Spatial-structure panel (L\*, C\*ab, saturation, value, a\*, b\*, R/G/B, redness, CCT, colourfulness, RMS contrast, dark-channel); the opponent axes a\*/b\* are the natural way to capture hue's spatial structure (b\* tracks a blue-to-yellow transition). The grid-heatmap **view** has its own dropdown and can display any of these channels regardless of which were chosen for Moran's I.

Because GPS and capture date are exported with each image, sun position and other solar-geometry analyses can be computed downstream without re-running the tool.

---

## Per-image visualizations

Each result carries a **Visualizations** dropdown that opens one view below the card and downloads it as **PNG**:

- **Palette** — the image's dominant-colour strip, sized by pixel share.
- **Cluster-region image** — the image recoloured to its k dominant clusters.
- **Grid image** — the N×N spatial grid rendered as mean-colour cells (with Spatial structure on).
- **Grid heatmap** — the same grid coloured by a channel you pick from a dropdown offering all channels (L\*, C\*ab, a\*, b\*, saturation, value, R/G/B, redness, CCT, colourfulness, RMS contrast, dark-channel), with a colour-scale legend and the cell's Moran's I in the title.
- **Row/column profile** — the mean-colour gradient along each axis (with Spatial structure on).
- **H/S/V histogram** — hue, saturation or value distribution with gradient-filled bars, matching the dataset histogram, chosen with the channel toggle.

An **Original image** view is also available. Each view carries a title plus an info line (projection · coordinates · statistic) and axis labels in degrees. When the analysis used **polar** spatial mode, the grid and profile views render as a true ring/wedge diagram (radius = angle from the projection centre, wedge = azimuth). All views download as crisp PNG tagged at **300 dpi**; which views are offered is chosen in the Visualization config section.

## Dataset visualizations

Across the whole set, the Dataset visualizations panel offers a summed hue/saturation/value histogram, an **aggregate palette** (every image's dominant colours combined and sorted by hue), and a **metric-distribution** view — a histogram of any image-level metric or Moran's I value across all images, with the mean and median marked. Each downloads as a 300 dpi PNG.

## Dataset spatial composites (metrics by x,y across the whole set)

When Spatial structure is on, the Dataset panel also aggregates every image's grid into one common frame, so you can see spatial pattern across the *dataset* rather than a single image. The composites (each a 300 dpi PNG): a **composite grid** of a chosen channel (b\*, C\*, L\*, redness, CCT, …) where you pick the per-position **measure** — mean, median, SD (variability), or **local Moran's I** (spatial-association map); a **vertical** and a **horizontal colour profile** (mean or median colour per bin); and a **cluster-centroid density** map. A **geom** toggle draws the grid and profiles either as a rectangle or, for disk/azimuthal frames, as a true **polar** diagram (rings = radial angle, wedges = azimuth) — it defaults to polar automatically when the analysis used polar spatial mode.

Because frames are often **not perfectly registered**, two alignment controls sit beside the composite. **Vertical** normalises each image's vertical axis either end-to-end using the opaque-mask extent (absorbs pitch / height shifts — the default) or over a *fixed range*. **Azimuth** rotates each frame to a common reference before aggregating: *none* (stationary / already-registered sets), *brightest column* (metadata-free, e.g. a sun proxy), *compass N* (from the GPS-track heading), or *sun* (solar azimuth from GPS + timestamp, for cross-time comparison). The panel reports how many images it could align. The vertical (radial) profile is rotation-invariant, so it stays valid even with azimuth alignment off.

## 360° imagery mode (projection, weighting & conversion)

An **Imagery mode** selector at the top of Configuration switches between **Standard** (flat photos, no weighting) and **360° / wide-angle**. In 360° mode you set:

- **Input projection** — equirectangular, fisheye (equidistant / equisolid), stereographic / tiny-planet, Lambert azimuthal equal-area, or equidistant azimuthal. Each carries the correct **solid-angle weighting** so pixel counts don't bias the result: equirectangular uses cos(elevation) (set the crop's top/bottom elevation, e.g. 90° zenith → 0° horizon); the disk projections use their matching radial weight (equidistant ∝ sinθ/θ, stereographic ∝ cos⁴(θ/2), equal-area/Lambert/equisolid = flat 1). Weighting flows through means, medians, spread, colourfulness, warm/cool/neutral, histograms, cluster percentages, and the spatial grid.
- **Analysis frame** — analyse *natively* in the input projection with matched weighting, or **convert to another projection first** (e.g. Lambert equal-area, where naive pixel stats are already unbiased) before analysis.
- **Spatial mode** — **Cartesian** (x = azimuth, y = radial angle; Moran's I wraps horizontally for 360°) or **polar** (rings = angle from the projection centre, wedges = azimuth; Moran's I wraps around the azimuth). Polar suits fisheye / tiny-planet / azimuthal frames, where a heading change is just a rotation of the disk. The polar grid scales its wedge count with radius so the centre isn't over-subdivided, and empty wedges borrow the nearest filled one so the disk renders solid.
- **Convert & export** — enabled once you choose *Convert first*: reproject every queued image from the input projection to the chosen target and download them together as a single **ZIP** of 300 dpi PNGs (a progress indicator runs, since decoding takes a moment to start). A **live preview** beside the controls shows the first image in the chosen frame — the full 2:1 panorama for equirectangular, or the disk for azimuthal targets.

Reprojection is a spatial remap only: it never changes a pixel's colour, but it changes how much area each part of the scene occupies, so the weighting (or an equal-area target) is what keeps the statistics honest. Reprojection from a 360° source wraps correctly at the 0°/360° seam. **Generic by design:** axes are labelled in degrees taken from the projection — a disk from its centre (0°) to its edge angle, an equirectangular crop from its top to bottom elevation — so any 360°/wide-angle subject reads correctly, not only skies. Leave Imagery mode on **Standard** for ordinary photos.

## Results interface

The results are split into two tabs to keep a large run navigable. The **Data** tab holds the CSV downloads and a **summary-statistics** panel: pick an analysis level (Image / Cluster / Pixel / Spatial), toggle which metrics to show, and read descriptive statistics — n, mean, median, SD, min, max — computed across the whole dataset, with an optional **group-by capture date** so conditions captured on different days are summarised separately. The **Visualization** tab holds the dataset visualization panel (histograms, palettes, distributions, and the spatial composites) and the individual image results, now listed as collapsible cards with expand-all / collapse-all controls.

## Configuration & columns

Options are grouped into collapsible panels (Columns, Histogram table, Spatial structure, Visualization). Under **Columns** you choose which colour spaces (RGB/HSV/LAB/LCH), statistics (mean, median, std dev, range, IQR), and scene metrics to export, with Core / Full / Minimal presets and per-row "all" toggles — so CSVs stay as wide or as lean as you need. Column names follow a consistent `SPACE_CHANNEL_STAT` scheme.

---

## Usage

1. Open the tool in a browser — no installation, no account, no server.
2. Add local images (or a folder), or paste image URLs.
3. Pick the **Imagery mode** (Standard, or 360° / wide-angle with its projection settings). Set analysis precision, cluster count, and transparency handling; tick the tables and options you want (LAB/LCH, EXIF, hue histogram, pixel-level, spatial structure) and set the grid and pixel-subsample sizes.
4. Click **Analyse Images**.
5. In the **Data** tab, use the CSV download buttons and read the summary-statistics tables; in the **Visualization** tab, open the dataset composites and each result's **Visualizations** dropdown to view and download per-image PNGs.

A typical research sequence runs Image Cutter → Batch Image Analyzer → Image Visualizer, handing folders of cut-outs and CSVs from one tool to the next; each tool also works on its own.

---

## Notes & limitations

- Images are downsampled to the chosen analysis resolution before measurement, which sets the resolution of the pixel-level and spatial outputs.
- Clustering is k-means with a fixed seed, so results are reproducible; treat cluster boundaries as an instrument reading, not ground truth.
- The dark-channel metric is a lightweight per-pixel version of the dark-channel prior and is most meaningful on full-frame scenes rather than tightly cropped subjects.
- Metrics describe apparent colour as recorded, not radiometrically calibrated values.
- In polar mode the innermost rings are genuinely coarser in azimuth (angular detail near the centre is unresolved by design) — the wedge count scales with radius to avoid empty cells.
- Converting between projections resamples pixels (bilinear); prefer *native + matched weighting* to preserve resolution, or *convert-first to Lambert equal-area* when you want unbiased naive statistics. Local Moran's I in the composite grid is a rook-neighbour LISA on the mean field (a local-association estimate, not a p-valued hotspot test).

---

## Privacy

No image ever leaves your computer. There is no server, no account and no upload step. The tool works offline.

---

## Citation

> Cinnamon, J. (2026). *Batch Image Analyzer* [Computer software]. Vis-O-Matic. https://www.jonathancinnamon.com/image-colour-analyzer/

---

## License

MIT. Free to use and adapt with attribution.

---

*Jonathan Cinnamon · [jcinnamon.github.io](https://jcinnamon.github.io/) · UBC Geography*
