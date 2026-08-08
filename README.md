# Batch Image Analyzer — Vis-O-Matic

A browser-based tool for measuring colour across a whole set of images at once. It clusters each image into its dominant colours, reports colour values in HSV, CIELAB and LCH, computes a suite of whole-image and spatial metrics, reads EXIF (including GPS), and exports everything as CSV. Nothing is uploaded — analysis runs locally in your browser.

**[Live demo →](https://www.jonathancinnamon.com/image-colour-analyzer/)**

Part of [Vis-O-Matic](https://www.jonathancinnamon.com/vis-o-matic/), alongside [Image Cutter](https://www.jonathancinnamon.com/image-cutter/) and [Image Visualizer](https://www.jonathancinnamon.com/image-colour-visualizer/).

---

## Features

- **Batch colour analysis** — drop in a folder or a list of URLs; each image is downsampled to an analysis resolution you set, then measured locally.
- **K-means clustering** — k-means++ initialisation with a fixed seed for reproducibility; 2–20 dominant colours per image, each reported in HSV, CIELAB and LCH with its pixel share.
- **Three levels of granularity** — cluster-level, image-level, and optional pixel-level tables, plus a hue histogram.
- **EXIF passthrough** — GPS, capture date, camera and orientation read from JPEGs and from PNGs carrying an eXIf chunk, including cut-outs exported by [Image Cutter](https://www.jonathancinnamon.com/image-cutter/) with GPS/EXIF preserved.
- **Transparency handling** — composite transparent areas onto white, black or grey, or analyse opaque pixels only.
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
- **Moran's I** — spatial autocorrelation (rook contiguity) of a chosen channel, in the image summary with a companion `morans_i_channel` column: one index of how spatially organised the colour is, from clustered (positive) to salt-and-pepper (negative). A **channel selector** (Lightness L\*, Chroma C\*ab, Saturation, a\*, b\*) sets the channel used for both Moran's I and the grid heatmap; the opponent axes a\*/b\* are the correct way to capture hue's spatial structure (b\* tracks the blue-to-yellow smoke transition).

Because GPS and capture date are exported with each image, sun position and other solar-geometry analyses can be computed downstream without re-running the tool.

---

## Per-image visualizations

Each result carries a **Visualizations** dropdown that opens one view below the card and downloads it as **PNG**:

- **Palette** — the image's dominant-colour strip, sized by pixel share.
- **Cluster-region image** — the image recoloured to its k dominant clusters.
- **Grid image** — the N×N spatial grid rendered as mean-colour cells (with Spatial structure on).
- **Grid heatmap** — the same grid coloured by the selected channel (L\*, C\*ab, saturation, a\* or b\*) with a colour-scale legend and the cell's Moran's I in the title.
- **Row/column profile** — the mean-colour gradient along each axis (with Spatial structure on).
- **H/S/V histogram** — hue, saturation or value distribution with gradient-filled bars, matching the dataset histogram, chosen with the channel toggle.

An **Original image** view is also available. All views carry axis scales and tick marks where relevant, and download as crisp PNG tagged at **300 dpi**. Which views are offered is chosen in the Visualization config section.

## Dataset visualizations

Across the whole set, the Dataset visualizations panel offers a summed hue/saturation/value histogram, an **aggregate palette** (every image's dominant colours combined and sorted by hue), and a **metric-distribution** view — a histogram of any image-level metric or Moran's I value across all images, with the mean and median marked. Each downloads as a 300 dpi PNG.

## Dataset spatial composites (metrics by x,y across the whole set)

When Spatial structure is on, the Dataset panel also aggregates every image's grid into one common frame, so you can see spatial pattern across the *dataset* rather than a single sky. Five composites (each a 300 dpi PNG): a **mean composite grid** (average of a chosen channel — b\*, C\*, L\*, redness, CCT, etc. — at each position), a **variability (SD) grid** (where skies differ most across the set — usually the horizon band), a **mean elevation profile** and a **mean azimuth profile** (each with a ±SD band), and a **cluster-centroid density** map (where dominant colours tend to sit).

Because your frames are usually **not perfectly registered**, two alignment controls sit beside the composite. **Vertical** normalises each image's elevation axis either *horizon→zenith* using the sky-mask extent (absorbs mount pitch/height shifts — the default) or over a *fixed range*. **Azimuth** rotates each frame to a common reference before aggregating: *none* (stationary/already-registered sets), *brightest column* (metadata-free, a sun proxy), *compass N* (from the GPS-track heading), or *sun* (solar azimuth from GPS + timestamp, for cross-time comparison). The panel reports how many images it could align. Elevation profiles are rotation-invariant, so they stay valid even with azimuth alignment off.

## Projection weighting (360° / fisheye)

Equirectangular and fisheye images are not equal-area — pixels over-represent the zenith/poles — so pixel-weighted metrics are biased. The **Projection / weighting** section corrects this by weighting every pixel by the solid angle it truly represents: **equirectangular** applies a cos(elevation) weight (set the crop's top/bottom elevation, e.g. 90° zenith → 0° horizon), and **fisheye** applies the matching radial weight (equidistant corrects; equisolid is already equal-area). Weighting flows through the means, medians, spread, colourfulness, warm/cool/neutral, histograms, cluster percentages, and the spatial grid; Moran's I additionally wraps horizontally for 360° continuity. Leave it on "Flat / none" for ordinary photos.

## Configuration & columns

Options are grouped into collapsible panels (Columns, Histogram table, Spatial structure, Visualization). Under **Columns** you choose which colour spaces (RGB/HSV/LAB/LCH), statistics (mean, median, std dev, range, IQR), and scene metrics to export, with Core / Full / Minimal presets and per-row "all" toggles — so CSVs stay as wide or as lean as you need. Column names follow a consistent `SPACE_CHANNEL_STAT` scheme.

---

## Usage

1. Open the tool in a browser — no installation, no account, no server.
2. Add local images (or a folder), or paste image URLs.
3. Set analysis precision, cluster count, and transparency handling; tick the tables and options you want (LAB/LCH, EXIF, hue histogram, pixel-level, spatial structure) and set the grid and pixel-subsample sizes.
4. Click **Analyse Images**.
5. Use the download buttons above the results for the CSV tables; open the **Visualizations** dropdown on each result to view and download per-image PNGs.

A typical research sequence runs Image Cutter → Batch Image Analyzer → Image Visualizer, handing folders of cut-outs and CSVs from one tool to the next; each tool also works on its own.

---

## Notes & limitations

- Images are downsampled to the chosen analysis resolution before measurement, which sets the resolution of the pixel-level and spatial outputs.
- Clustering is k-means with a fixed seed, so results are reproducible; treat cluster boundaries as an instrument reading, not ground truth.
- The dark-channel metric is a lightweight per-pixel version of the dark-channel prior and is most meaningful on full-frame scenes rather than tightly cropped subjects.
- Metrics describe apparent colour as recorded, not radiometrically calibrated values.

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
