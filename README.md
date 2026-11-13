# LoomG2pro Multimodal Samples: Quality Assessment and Release Notes

[Chinese version](README.zh-CN.md)

**Status: An inspected subset with known missing values. The complete delivery has not passed acceptance checks.**

This package documents file integrity and multimodal quality checks on the received LoomG2pro samples. The source archive contains 147 episodes. The current assessment covers 120 locally readable episodes, totaling 77,311 aligned frame indices at a delivered rate of 30 Hz. These findings do not establish that all 147 episodes have been repaired or accepted for use.

## Scope

| Category | Episodes inspected |
|---|---:|
| Apple | 6 |
| Banana | 58 |
| Block | 50 |
| Cup | 6 |
| **Total** | **120** |

Each episode includes head RGB, stereo grayscale, left and right wrist video, depth, bilateral tactile responses, bilateral hand motion capture, head IMU orientation, world-space trajectories, and a shared timeline. See [episode_inventory.csv](reports/episode_inventory.csv) for the inspected subset. Episodes absent from this inventory are outside the current assessment scope.

## Work completed

- Compared SHA-256 hashes of all 15 source archive volumes with the sender's checksum table and tested the complete archive with the official 7-Zip command-line tool.
- Calculated CRC32 checksums for existing local files that could be matched to archive entries.
- Inspected HDF5 array shapes, frame counts, non-finite values, orientation quaternions, video metadata, shared timelines, and tactile statistics for the 120 episodes.
- Sampled five depth frames per episode and decoded the first, middle, and last frames of each video.
- Generated quality reports, missing-value details, and a preview from the actual data. This assessment did not impute, trim, or otherwise modify the sensor arrays.

## Findings and limitations

| Component | Finding | Implication |
|---|---|---|
| File integrity | 3,294 local archive files matched their archived CRC32 checksums; one depth file did not | The mismatched file is excluded from the inspected subset. A matching checksum does not establish semantic correctness. |
| Camera video | All 600 videos reported frame counts matching episode metadata; first, middle, and last frames decoded successfully | Full-frame visual quality and physical synchronization across modalities remain unverified. |
| Depth | Across the five-frame samples per episode, nonzero pixel fractions ranged from 95.1% to 96.8% | Zero is the declared invalid value. A nonzero value does not establish depth accuracy. |
| Hand motion capture | No NaN/Inf values were found; node quaternion normalization and wrist/root consistency checks passed | Joint positions, kinematic accuracy, and image projection accuracy remain unverified. |
| Head IMU | Frame 0 was NaN in every inspected episode; no non-finite values were found in the remaining IMU frames | Apply validity masks rather than treating missing values as valid orientations. |
| World-space trajectories | Head and hand trajectories contain NaN values, including gaps within episodes | Preserve missing-value masks. These arrays are not complete continuous trajectory supervision. |
| Tactile | ADC arrays were readable; no samples reached 255, and some channels remained constant throughout an episode | Values are ADC responses, not calibrated force or pressure. Constant channels may reflect a lack of contact and do not alone establish sensor failure. |
| Timeline | Frame indices match episode lengths, with master timestamps increasing by approximately 1/30 second | The timeline contains only aligned frame indices and master timestamps, without original per-sensor timestamps. Acquisition offsets cannot be independently verified. |

### Examples of missing world-space data

- `banana/ep_0028`: 79 of 699 frames contain non-finite left-hand world coordinates (**11.3%**).
- `block/ep_0001`: 33 of 650 frames contain non-finite right-hand world coordinates (**5.1%**).
- Across all 77,311 frames, head position is missing in 159 frames, left-hand world coordinates in 262 frames, and right-hand world coordinates in 209 frames. These counts are calculated separately by field and may overlap; they must not be added together as a count of distinct invalid frames.

The source documentation specifies NaN as the representation for missing hand world positions. The affected files passed CRC32 checks, so these missing values are present in the received arrays rather than being a consequence of the observed extraction truncation.

## Archive integrity issues and excluded data

The local SHA-256 hashes for volumes `.004` and `.012` do not match the sender's checksum table. The other 13 volumes match. Exact values are recorded in [source_archive_checksums.csv](reports/source_archive_checksums.csv). A mismatch establishes inconsistency with the supplied table, but does not by itself distinguish transfer damage, a version difference, or an error in the table.

The official 7-Zip archive test returned exit code 2 and reported data errors for 6,933 files. The first error was:

```text
data/cup/ep_0019/head_depth/head_depth.h5
```

The extracted file contains 58,720,256 bytes, compared with an expected size of 113,601,192 bytes, and is truncated.

Compared with the archive listing, the current extraction is missing 9,663 files and contains one file with an incorrect size. These counts include player runtime and configuration files, not only sensor data. They do not correspond one-to-one with the archive test's error count.

**Archive volumes are not independent modality or episode groups.** Matching checksums for the other 13 volumes do not establish that all data associated with those volumes has independently passed acceptance checks. Twenty-seven cup episodes are excluded from the inspected subset: `ep_0019` and 26 other episodes that were not fully extracted.

Before expanding the inspected subset, obtain source-confirmed archive volumes, repeat the complete archive test, extract the data again, and rerun the quality assessment.

## Recommended use

1. Use the episode inventory to define the current inspected subset.
2. Build finite-value masks for each required field. For multimodal supervision, intersect the validity masks of the required modalities.
3. Handle quaternion conventions explicitly: motion capture uses `w,x,y,z`; IMU orientation and world-space head orientation use `x,y,z,w`.
4. Depth is stored as `uint16` in millimeters, with zero indicating invalid data. World-space positions are in meters.
5. Retain tactile values as ADC responses; do not label them as force in newtons or calibrated pressure.
6. If interpolation, frame removal, resampling, or recalibration is performed later, preserve the original arrays, processing parameters, and validity masks, and regenerate the quality reports. None of these operations was performed in this assessment.

Action-label accuracy, temporal offsets across modalities, spatial calibration, depth error against ground truth, and downstream training performance remain unverified. No training/validation/test split was created, and leakage across splits was not assessed.

## Preview and reports

![Actual data from apple/ep_0001 at a shared delivered frame index](assets/multimodal_preview.jpg)

The preview uses the delivered frame index and is not evidence of acquisition synchronization accuracy. Tactile curves show mean ADC across channels per frame. The left and right curves are scaled separately; use their labels to compare absolute values.

- [Episode inventory and missing-frame counts](reports/episode_inventory.csv)
- [Detailed assessment statistics](reports/multimodal_audit.json)
- [Missing-frame details by field](reports/missing_data_details.json)
- [Complete archive test log](reports/archive_test.log)

## Distribution and permissions

This package currently contains documentation, reports, and a preview. It does not include the sensor data and has not been published. The intended GitHub account or organization is [CarusAI](https://github.com/CarusAI); the specific repository, dataset download location, and redistribution terms have not yet been specified. This document does not grant redistribution rights for the source dataset or its player software.

Large HDF5 files, videos, and source archive volumes should be distributed through a separate dataset location or an appropriate Git LFS configuration, accompanied by manifests and checksums. GitHub blocks individual files larger than 100 MiB in ordinary Git repositories; see the [official GitHub documentation](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github).
