# Batch Image Analyzer — Vis-O-Matic

A browser-based tool for measuring colour in images, singly or in batches. Drop in a folder of imagess; choose how you want colour analyzed; review the distributions and export them as CSV, per-image summaries, or aggregate tables. Nothing is uploaded — analysis runs locally in your browser.

**[Live demo →](https://www.jonathancinnamon.com/image-colour-analyzer/)**

Part of [Vis-O-Matic](https://www.jonathancinnamon.com/vis-o-matic/), alongside [Image Cutter](https://www.jonathancinnamon.com/image-cutter/) and [Image Visualizer](https://www.jonathancinnamon.com/image-colour-visualizer/).

---

## Features

- **In-browser colour analysis** — runs entirely client-side in your browser; images never leave your computer, and no account or server is involved.
- **Batch processing** — drag in a folder of images and measure them in one pass; handles hundreds of files with consistent settings.
- **Flexible colour models** — summarize colour in RGB, HSL, HSV, or LAB space, with options for dominant colour, mean colour, and distributions.
- **Mask and transparency handling** — respect alpha channels and binary masks from Image Cutter, so measurements focus on the segmented region rather than the full frame.
- **Spatial summaries** — divide each image into grids or bands (e.g. sky vs ground) and compute colour for each zone, useful for skyline and smoke analysis.
- **CSV output** — export per-image measurements (mean, median, variance, distribution bins), image metadata, and any spatial splits into a single `measurements.csv`.
- **Project-level aggregates** — compute summary statistics across the whole batch (e.g. average sky hue, distribution of haze intensity) for quick comparison between folders.
- **EXIF / GPS integration** — read EXIF and PNG `eXIf` chunks from source images and pass GPS/location fields into the CSV, so colour measurements can be joined to place and time.

---

## EXIF / GPS integration

Colour analysis often depends on where and when an image was taken. When the source photos carry EXIF metadata (including PNG `eXIf` chunks written by Image Cutter), Batch Image Analyzer reads and includes those fields in the CSV:

- **From each image** — GPS latitude/longitude/altitude, capture datetime, camera make/model, and orientation are extracted where available.
- **Into the measurements CSV** — columns such as `gps_lat`, `gps_lon`, `gps_alt`, `datetime`, `camera_make`, `camera_model`, and `orientation` are added alongside colour fields.

JPEGs, EXIF-bearing PNGs, and other supported formats bring their metadata through automatically; images without EXIF simply leave those columns blank. This lets colour, smoke, and sky measurements be linked to capture context without manual joins, especially in workflows starting from Image Cutter.

---

## Usage

1. Open the tool in a browser — no installation, no account, no server.
2. Drop in an image or a folder (or choose files) and select colour model and summary options.
3. Configure analysis: choose whether to honour transparency/masks, set grid or band splits (e.g. sky vs ground), and pick per-image vs aggregate outputs.
4. Run the analysis to compute colour summaries for each image; review sample results in the interface to confirm the settings behave as expected.
5. Export the results as a `measurements.csv` (and optional per-image tables or project summary CSVs), with EXIF/GPS fields included where available.
6. Use the CSV directly in statistical software, or pass it on to [Image Visualizer](https://www.jonathancinnamon.com/image-colour-visualizer/) for mapping and visual exploration.

A typical research sequence runs Image Cutter → Batch Image Analyzer → Image Visualizer, handing folders of cut-outs and CSVs from one tool to the next. Each tool also works on its own.

---

## Notes & limitations

- The analyser summarizes the colours it sees; it does not judge image quality or "correct" exposure. Treat each measurement as an instrument reading and report the settings used.
- Internally, images may be resized or sampled to keep memory and computation manageable; this preserves overall distributions but may not capture single-pixel features.
- Above roughly a few hundred images the browser may feel sluggish, since everything runs client-side; consider splitting very large collections into batches.
- If masks or transparency are ignored, measurements refer to the whole frame; be explicit in methods sections about whether region-of-interest segmentation was used.

---

## Privacy

No image ever leaves your computer. There is no server, no account, and no upload step. The tool works offline once loaded; web access is needed only to fetch the code initially, which is then cached locally.

---

## Citation

If this is useful in research or teaching, please cite the tool:

> Cinnamon, J. (2026). *Batch Image Analyzer* [Computer software]. Vis-O-Matic. https://www.jonathancinnamon.com/image-colour-analyzer/

---

## License

MIT. Libraries used for image handling, colour conversion, and EXIF parsing carry their own licences, noted in the tool. Free to use and adapt with attribution.

---

*Jonathan Cinnamon · [jcinnamon.github.io](https://jcinnamon.github.io/) · UBC Geography*
