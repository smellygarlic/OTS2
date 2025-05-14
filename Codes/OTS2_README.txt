=========================================
OTS2 Dual-Channel Time Delay Analysis Pipeline
=========================================

This pipeline performs full dual-channel colocalization analysis from XML-tracked spots, performs dark-region filtering on RICM images, and generates time-delay visualizations and marker-centered temporal patch views.

-----------------------------------------
Features
-----------------------------------------
- XML import from TrackMate for both channels
- Filtering for ROI, FOI, early-activation, revived and overlapping tracks
- Track pairing via colocalization
- Histogram analysis of time-delay differences
- Single-exponential decay fitting with R² calculation
- RICM-based dark region classification
- Marker patch extraction with color-coded visualization

-----------------------------------------
Pipeline Execution Flow
-----------------------------------------

1. Load tracking XMLs and raw TIFF image
2. Apply filtering to remove poor or redundant tracks
3. Find colocalized track pairs (green-red)
4. Histogram and fit time delay between matched tracks
5. Identify tracks within dark regions of RICM signal
6. Extract patch markers around colocalized events

-----------------------------------------
Function Hierarchy
-----------------------------------------

run_full_analysis()
├── main_data_loading()
│   ├── load_tracks_xml()
│   └── tracks_to_dataframe()
├── run_main_filtering()
│   ├── compute_first_and_last_appearance()
│   ├── apply_roi_foi_filter()
│   ├── apply_already_activated_filter()
│   ├── apply_revived_filter_matlab()
│   └── apply_overlapping_filter_matlab()
├── find_colocalized_pairs_matlab()
├── main_time_delay_fitting()
│   ├── fit_time_delay_exp1decay()
│   │   ├── exp1decay_func()
│   │   └── compute_r_squared()
│   ├── plot_time_delay_exp1decay()
│   └── plot_time_delay_exp1decay_inverse_x()
├── main_analysis_after_dark_pairs()
│   ├── load_multistack()
│   ├── analyze_dark_pairs()
│   │   └── visualize_dark_pairs()
│   └── main_time_delay_fitting()
└── making_mark_start()

marker_df_2
├── patch generation per frame
└── merging patch tables

plot_patch_df_grid_grouped
├── render_single_row
└── center_channel_table

combine_figures_to_one_subplot
└── render_to_numpy

-----------------------------------------
Execution Code
-----------------------------------------

# Run full colocalization and dark-region pipeline
df_mark = run_full_analysis(
    criInter=0.6,
    criIntra=0.6,
    criOverlap=2.5,
    criBlink=2.0
)

# Generate patches for visualization
patch_df = marker_df_2(multistack, df_mark, df_mark)
figures = plot_patch_df_grid_grouped(patch_df)
combine_figures_to_one_subplot(figures)

-----------------------------------------
Requirements
-----------------------------------------

Python:
- numpy
- pandas
- tifffile
- scikit-image
- opencv-python
- matplotlib
- lmfit
- tqdm

Others:
- TrackMate-generated XMLs
- RICM-containing multichannel TIFF stack (shape: [x, y, frame, channel])

-----------------------------------------
Notes
-----------------------------------------

- RICM is assumed to be channel 0 (can be modified)
- Time delay is calculated as T_green - T_red
- Frame interval is used for converting delay to seconds (default 1.0)
- Patch extraction covers ±10 frames around the marker

