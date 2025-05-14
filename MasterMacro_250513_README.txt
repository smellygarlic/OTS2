==============================
ND2 Drift & Dual-Channel Tracking Pipeline
==============================

This pipeline performs preprocessing and tracking for time-lapse multichannel ND2 microscopy data using ImageJ and TrackMate via pyimagej.

----------------------------------------
Features
----------------------------------------
- ND2 loading via Bio-Formats
- TrackMate spot detection and tracking (LoG + LAP)
- XY drift correction using tracked spot median
- Chromatic correction via MATLAB affine matrix
- Dual-channel tracking (Green and Red)
- Background subtraction and TIFF recombination

----------------------------------------
Directory Structure After Processing
----------------------------------------

project/
├── 1.ND2_file/
│   └── sample.nd2
├── 2.Tracks_for_driftc/
│   ├── sample_Tracks.xml
│   ├── sample.tif
│   └── sample_metadata.txt
├── 3.driftc_reg/
│   ├── sample_chan1_drftc.tif
│   ├── sample_chan2_drftc.tif
│   ├── sample_chan3_drftc.tif
│   └── sample_drftc_reg.tif
├── 4.Tracks_for_TimeDelay/
│   ├── sample_drftc_reg_g.xml
│   └── sample_drftc_reg_r.xml
├── 5.Background_Corrected/
│   └── sample_back.tif

----------------------------------------
Pipeline Steps
----------------------------------------

1. TrackMate spot detection from ND2 → _Tracks.xml
2. XY drift estimation from multiple tracks
3. Per-frame drift correction (FFT-based)
4. Chromatic shift correction using MATLAB affine transform
5. Merge corrected grayscale channels into multi-channel TIFF
6. Run dual-channel TrackMate (green/red) for delay or colocalization analysis
7. Background subtraction and final cleaned TIFF generation (optional)

----------------------------------------
Function Hierarchy
----------------------------------------

run_precorrection()
├── run_trackmate_nd2()
├── load_tracks_xml()
├── compute_median_drift_from_tracks()
├── apply_drift_correction_to_tiff()
├── apply_chromatic_registration_from_mat()
└── combine_tif_channels()

run_trackmate_dual_channel_from_config()
└── run_single_channel()

subtract_background_and_merge()

----------------------------------------
Execution Code
----------------------------------------

# Step 1: Run full preprocessing
tif_path = run_precorrection(nd2_path, config, mat_path, ij)
print("Final registered TIFF saved at:", tif_path)

# Step 2: Run dual-channel tracking
run_trackmate_dual_channel_from_config(
    ij,
    tif_path,
    config_green,
    config_red
)

# Step 3: Background subtraction and merging
back_path = subtract_background_and_merge(tif_path, ij=ij)
print("Final background-corrected TIFF saved at:", back_path)

----------------------------------------
Requirements
----------------------------------------

Required Environment:
- Python ≥ 3.8
- Java Runtime Environment (JRE) ≥ 8 (Fiji / ImageJ required)
- MATLAB (only needed to generate the affine transformation matrix, not for runtime)

Python Packages:
- pyimagej         # Interface to ImageJ/Fiji from Python
- scyjava          # Required by pyimagej for Java interaction
- numpy            # Array and numerical operations
- pandas           # For DataFrame-based tracking structures
- tifffile         # Saving multipage TIFFs
- scikit-image     # For affine transformation and image processing

ImageJ/Fiji:
- Download and install Fiji (https://fiji.sc/)
- Make sure the TrackMate plugin is installed and updated
- You must initialize ImageJ with `add_legacy=True` for TrackMate compatibility

MATLAB:
- You must prepare a `.mat` file that contains a 3x3 affine matrix under the key `T`
- This matrix will be used to correct chromatic aberration

Recommended:
- Java heap size: Set to at least 8–16 GB to handle large ND2 stacks
  (configured via: `scyjava.config.add_options("-Xmx16g")`)

Install Python dependencies:
pip install pyimagej scyjava numpy pandas tifffile scikit-image

NOTE:
- On first `imagej.init()`, pyimagej will download Fiji automatically unless a custom path is specified.
- You may want to cache or manually set Fiji path for reproducibility or firewall-limited environments.

----------------------------------------
Notes
----------------------------------------

- Channel indices follow TrackMate (1-based)
- Default frame interval: 2.5 sec
- Output TIFFs are saved in structured directories
- XML tracking results are ready for downstream analysis