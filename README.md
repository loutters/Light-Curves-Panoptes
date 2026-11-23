# Light-Curves-Panoptes

A step-by-step Jupyter notebook to build the **light curve** of a star from raw images taken by a
[PANOPTES](https://www.projectpanoptes.org) unit.

The notebook is written for beginners: no prior knowledge of astronomy image processing is needed.
Each step explains what it does, shows a plot to check the result, and ends with a
*"What can go wrong?"* note describing what a bad result looks like and how to fix it.

The example used is **DY Pegasi**, a pulsating star whose brightness changes by about 0.6 magnitude
every 1.75 hours, observed with a single-camera PANOPTES unit at EPFL (Lausanne, Switzerland).


## What you will learn

- How an astronomical image (FITS file) is organised: pixels and header
- How to check that an image is usable (stars, background, colour camera or not)
- How to detect stars and find where the image points in the sky (astrometry)
- How to measure the brightness of a star (aperture photometry)
- How to correct for the atmosphere with comparison stars (differential photometry)
- How to follow the stars over hundreds of images, even when the mount drifts or jumps
- How to judge the quality of a light curve


## Repository contents

| File | Description |
|---|---|
| `projet_courbe.ipynb` | The notebook |
| `README.md` | This file |

The images are **not** stored in this repository (about 7 GB). They are available on Zenodo (see below).


## Installation

The notebook was tested with **Python 3.14.6**.

Install the required libraries:

```
pip install numpy matplotlib scipy astropy astroquery photutils twirl jupyter
```

An **internet connection** is needed while running the notebook: the star name is checked with the
SIMBAD database (section 0) and a star catalog is downloaded from the MAST archive (section 5).


## Get the data

The images are available on Zenodo: **https://doi.org/10.5281/zenodo.24466635**

| File | Content | Size |
|---|---|---|
| `data_pegasi.zip` | 476 images of DY Pegasi, 13 November 2026, 17:48 to 20:10 UTC, 9 s exposures | ~4 GB |
| `test_images.zip` | One image of the same sequence, to try the notebook quickly | ~10 MB |

Download the file(s) and unzip them **in the same folder as the notebook**:

```
Light-Curves-Panoptes/
├── projet_courbe.ipynb
├── README.md
├── data_pegasi/                 <- from data_pegasi.zip
│   ├── 20261113T174833.fits.fz
│   ├── 20261113T174848.fits.fz
│   └── ...
└── test_image/                  <- from test_images.zip
    └── 20261113T174833.fits.fz
```

If you use git, add these lines to a `.gitignore` file so that the images are never committed by mistake:

```
data_pegasi/
test_image/
*.fits.fz
*.zip
```


## How to use the notebook

1. Open `projet_courbe.ipynb` in Jupyter or VS Code.
2. In the **parameters** cell (section 0), set:
   - `DATA_FOLDER = "data_pegasi"` (or `"test_image"` for a quick test)
   - `REFERENCE_FILE = "20261113T174833.fits.fz"`
   - `TARGET_NAME = "DY Peg"`
3. Run the cells **in order**, from top to bottom.
4. After each step, look at the plot or the printed values before moving on.

With `test_image`, sections 1 to 9 work normally. Sections 10 to 12 need the full sequence.

The loop over all the images (section 10) takes several minutes.


## The steps

| # | Section | What it checks |
|---|---|---|
| 0 | Parameters and imports | The star name is recognised by SIMBAD |
| 1 | Open an image and read its header | Image type, date, pointing of the mount |
| 2 | Are there stars in the image? | Background level, number of bright pixels, whole folder overview |
| 3 | Colour camera or not? | Light of one star in the 4 sub-grids of pixels |
| 4 | Detect the stars | No star detected twice |
| 5 | Astrometry | Pixel scale, catalog stars placed on the image |
| 6 | Find the target | Centroid converges on the star |
| 7 | Aperture photometry | Curve of growth, signal-to-noise, saturation |
| 8 | Comparison stars | Isolated stars, no neighbour in the background annuli |
| 9 | Measurement function | Same results as by hand |
| 10 | Loop over all images | Movement of the field, failed measurements |
| 11 | Light curve | Control curve between two comparison stars |
| 12 | Save the results | CSV file |


## Expected result

For DY Pegasi with the `data_pegasi` sequence:

- one full pulsation, with a maximum of brightness about 1.3 hours after the first image;
- an amplitude of about 0.6 magnitude, with a fast rise and a slower decline;
- a scatter of the control curve of about 0.03 magnitude.

The results are saved in `light_curve.csv`.

[Light curve of DY Peg](light_curve.png)


## Using the notebook for another star

The method is general, but some settings were chosen for this data set.

**Always change:**
- `DATA_FOLDER`, `REFERENCE_FILE` and `TARGET_NAME` (section 0);
- `COMP_INDICES`, the comparison stars (section 8): choose them again by looking at the image.

**Always check:**
- the astrometry (section 5): the pixel scale must match the expected value
  (about 11.2 arcsec/pixel for an 85 mm lens), otherwise adjust `FWHM` (section 4) or `N_CATALOG`;
- the brightness of the target: it must not be saturated (peak below ~60000 ADU),
  and stars much fainter than magnitude 12 give a low signal-to-noise ratio in 9 s exposures;
- if the target is fainter than magnitude 10, increase `CATALOG_MAG_MAX` (section 5);
- the aperture and annulus radii (section 7), if the focus or the field is different;
- the colour test (section 3) if you use another camera;
- the offset plot (section 10): it must vary smoothly, without jumping between very different values.


## Troubleshooting

Each section of the notebook ends with a *"What can go wrong?"* note. The most common problems:

- **No stars in the images**: the lens was covered or the roof closed (section 2).
- **Wrong astrometry** (pixel scale far from the expected value): a star detected twice (section 4).
- **Gaps in the light curve**: the mount slipped or jumped; look at the offset plot (section 10).
- **Connection error** when downloading the catalog: the server is busy, run the cell again later.
- **`AttributeError: 'MaskedColumn'`**: convert the catalog columns with `np.array(..., dtype=float)` (section 5).


## Data and citation

The images were taken with the PANOPTES unit at EPFL. If you use them, please cite:

> Outters, L., & Bach, A. (2026). *PANOPTES single-camera unit at EPFL: raw images of the pulsating star
> DY Pegasi (13 November 2026)* [Data set]. Zenodo. https://doi.org/10.5281/zenodo.24466635

The data are distributed under the Creative Commons Attribution 4.0 licence (CC BY 4.0).


## Acknowledgements

This notebook uses [astropy](https://www.astropy.org), [photutils](https://photutils.readthedocs.io),
[astroquery](https://astroquery.readthedocs.io), [twirl](https://github.com/lgrcia/twirl),
[numpy](https://numpy.org), [scipy](https://scipy.org) and [matplotlib](https://matplotlib.org),
the SIMBAD database (CDS, Strasbourg) and the TESS Input Catalog from the MAST archive.