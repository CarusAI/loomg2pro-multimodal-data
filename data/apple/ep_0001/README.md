# Apple placement · Episode 0001

The action annotations describe a person placing five apples on a plate. The left hand places the first three apples and the right hand places the final two. This episode contains **1,228 frames at 30 Hz**, or approximately **40.93 seconds** of playback. All paths below are relative to this episode.

![Five camera views and depth at frame 614, with tactile traces across the recording](../../../assets/apple-preview.jpg)

## Replay and timing

Open [the head RGB video](camera/head_RGB.mp4) for the first-person view. Frame indices run from 0 through 1227. Use the same index to select a video frame, a depth image, a tactile vector and a pose sample. The [shared timeline](meta/timeline/sync.parquet) contains `frame_idx` and `master_ts` in seconds. It describes delivered playback timing. Estimating sensor acquisition offsets requires additional timing information.

| Camera stream | Delivered resolution (width × height) |
|---|---:|
| [head_RGB](camera/head_RGB.mp4) | 1280 × 800 |
| [head_gray_left](camera/head_gray_left.mp4) | 1280 × 800 |
| [head_gray_right](camera/head_gray_right.mp4) | 1280 × 800 |
| [wrist_left](camera/wrist_left.mp4) | 1280 × 720 |
| [wrist_right](camera/wrist_right.mp4) | 1280 × 720 |

## Action intervals

Both endpoints are inclusive. For Python slices use `start:end+1`. Frame 0 is outside the annotated intervals. These descriptions identify action phases and have not been independently verified as frame-exact contact events. [Source labels](annotations/auto_actions.json)

| Frames | Annotated action |
|---|---|
| 1–198 | The apples and plate are initially on the table; the left hand moves to the first apple. |
| 199–322 | The left hand grasps the first apple and moves it above the plate. |
| 323–423 | The left hand releases the apple onto the plate, then moves to the second apple. |
| 424–572 | The left hand grasps the second apple and moves it above the plate. |
| 573–653 | The left hand releases the apple onto the plate, then moves to the third apple. |
| 654–801 | The left hand grasps the third apple and moves it above the plate. |
| 802–856 | The left hand releases the apple onto the plate; the right hand moves to the fourth apple. |
| 857–968 | The right hand grasps the apple and moves it above the plate. |
| 969–1015 | The right hand releases the apple onto the plate, then moves to the fifth apple. |
| 1016–1110 | The right hand grasps the fifth apple and moves it above the plate. |
| 1111–1227 | The right hand releases the apple onto the plate, completing the placement. |

## Sensor arrays

The first dimension of every array is the delivered frame index. For the bilateral tactile and motion capture streams, each hand has its own file.

| File | Dataset | Shape per frame | Interpretation |
|---|---|---|---|
| [Head depth](head_depth/head_depth.h5) | `depth` | 800 × 1280 | Unsigned 16-bit optical-axis depth, in millimeters, in the rectified left stereo camera. Zero marks an invalid pixel. |
| [Left tactile](tactile/left.h5), [right tactile](tactile/right.h5) | `adc` | 1132 | Unsigned 8-bit sensor responses in ADC counts. Physical force and pressure require separate calibration. |
| [Left mocap](mocap/left.h5), [right mocap](mocap/right.h5) | `skeleton` | 20 × 7 | SDK model-frame node positions in meters followed by orientation: `px, py, pz, qw, qx, qy, qz`. |
| Same mocap files | `wrist_quat` | 4 | Node 0 orientation, in `w, x, y, z` order. The delivered skeleton already includes the vendor pose calibration. |
| [Head IMU](imu/head_imu.h5) | `aligned/orientation` | 4 | Quaternion in `x, y, z, w` order, rotating from the current IMU frame into the IMU frame at the first raw sample. |

Node 0 is the hand root. The thumb tip is node 3 and the index fingertip is node 7. Use the [20-node diagram](../../../assets/mocap_20_nodes_en.png) to identify other joints. Tactile metadata records channel order and calibration separately for the [left](meta/devices/tactile/left.json) and [right](meta/devices/tactile/right.json) hands.

## World-space motion and image coordinates

[world_motion.h5](world_pose/world_motion.h5) contains the following derived fields. The episode uses a gravity-aligned world frame with positive Z upward. The [field definitions and transforms](meta/world_motion.json) specify the frame for each quantity. Arrays with a two-hand dimension use left, then right.

| Field | Shape per frame | Meaning |
|---|---|---|
| `head_position_m` | 3 | Head stereo left camera position in world coordinates, in meters. |
| `head_quaternion_xyzw` | 4 | Head stereo left camera orientation in world coordinates, in x, y, z, w order. |
| `left_world_m` | 20 × 3 | Positions of 20 left-hand nodes in world coordinates, in meters. |
| `right_world_m` | 20 × 3 | Positions of 20 right-hand nodes in world coordinates, in meters. |
| `hand_anchor_m` | 2 × 3 | Left and right wrist-point positions in world coordinates, in meters. |
| `wrist_depth_rgb_m` | 2 | Left and right wrist-point optical-axis Z depth in the head RGB camera frame, in meters. |
| `wrist_marker_uv_px` | 2 × 2 | Left and right wrist tracking points in the original head RGB image, in u, v pixel coordinates. |

The head RGB image is distorted, while depth uses rectified stereo-left coordinates. To overlay depth on RGB, use the [depth calibration](meta/depth.json), [camera extrinsics](meta/devices/extrinsics.json) and [RGB intrinsics and distortion](meta/devices/cameras/head_RGB.json). Equal image dimensions do not establish pixel correspondence.

## Validity and example

Frame 0 contains missing values in the head IMU orientation, world-space head pose and both world-space hand trajectories. Frames 1–1227 are finite for those fields. Use a validity mask for each signal. For velocities, require valid positions at both ends of each interval. The sample provides human motion for retargeting experiments. Robot joint commands and task execution on a robot require a separate mapping and validation.

After downloading the Git LFS files, run this example **from the episode directory** with Python, NumPy and h5py. Frame 614 (about 20.47 s) falls in the annotated approach to the third apple. The example reads a single depth frame and measures left thumb-to-index fingertip separation in the episode’s world frame.

```python
import h5py
import numpy as np

i = 614
with h5py.File("head_depth/head_depth.h5") as depth_file:
    depth_mm = depth_file["depth"][i]
    print("Valid depth pixels (%)", 100 * np.mean(depth_mm > 0))

with h5py.File("world_pose/world_motion.h5") as motion_file:
    hand = motion_file["left_world_m"][i]
    tips = hand[[3, 7]]
    if np.isfinite(tips).all():
        print("Thumb-to-index distance (m)", np.linalg.norm(tips[0] - tips[1]))
```

See the [episode metadata](meta/metadata.json) for its UUID and delivery settings, the [repository README](../../../README.md) for Git LFS download instructions, and the [episode license](LICENSE) for reuse terms.
