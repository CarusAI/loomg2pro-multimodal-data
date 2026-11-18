# LoomG2pro Multimodal Manipulation Samples


Four human demonstrations of tabletop manipulation, one episode per object. Each recording combines five camera views with depth, hand motion, tactile responses and head orientation on a common 30 Hz timeline.

| Object | Episode | Frames |
|---|---|---:|
| Apple | [ep_0001](data/apple/ep_0001/) | 1,228 |
| Banana | [ep_0002](data/banana/ep_0002/) | 658 |
| Block | [ep_0002](data/block/ep_0002/) | 596 |
| Cup | [ep_0005](data/cup/ep_0005/) | 559 |



## Explore an episode

Start with `camera/head_RGB.mp4` for the first-person view. The same folder holds stereo grayscale and wrist videos. The English action descriptions in `annotations/auto_actions.json` give inclusive start and end frame indices. Use `meta/timeline/sync.parquet` to locate the corresponding samples.

Depth is stored in millimeters in `head_depth/`, with zero marking invalid pixels. The hand motion capture files in `mocap/` contain 20 nodes per hand. Head and hand trajectories in `world_pose/` use meters. Tactile values in `tactile/` are ADC readings. Apple has 1,132 channels per hand and the other three samples have 1,125. Consult each episode’s tactile layout before comparing channels.

Head and hand positions in `world_pose/` are expressed in meters. The head IMU file,`imu/head_imu.h5`, stores one orientation quaternion per delivered frame in
`aligned/orientation`, in `(x, y, z, w)` order. Each quaternion rotates vectors from the current IMU sensor frame into the sensor frame of the episode’s first
raw IMU sample. Frame 0 has no valid orientation in any of the four samples.


![Five camera views, depth and tactile traces from the apple demonstration](assets/apple-preview.jpg)

![Left and right hand motion capture skeletons with nodes numbered 0–19](assets/mocap_20_nodes_en.png)


## Download

Install Git LFS, then run:

```bash
git lfs install
git clone https://github.com/CarusAI/loomg2pro-multimodal-data.git
cd loomg2pro-multimodal-data
git lfs pull
```


## Assessment scope

The four episodes listed above contain 3,041 frames in total. They were selected from a 147-episode source delivery. An earlier quality review covered 120 episodes, including these four. The complete archive subsequently passed the 7-Zip extraction check after volume 012 was replaced. Volume 004 still differs from the supplier’s SHA-256 table in its final digit. The earlier sensor-quality review did not cover the 27 subsequently recovered cup episodes.

## Quality and reuse

For all four episodes, video frame counts matched the episode metadata. We decoded the first, middle and last frame of each video and sampled five depth frames per
episode to measure nonzero pixel coverage. These checks support frame-indexed playback. Acquisition-time offsets and metric depth accuracy remain unverified.

In each episode, `left_world_m` and `right_world_m` each contain one frame with non-finite coordinates. Mask invalid values separately for each hand. When
computing velocity, require valid positions at both ends of each frame interval. Action annotations use inclusive frame indices and begin at frame 1.

The samples contain human demonstrations. Robot action commands, a retargeting mapping and robot execution results are outside the scope of this release.

Each episode includes the source license (CC BY-NC-SA 4.0). See the [license accompanying the apple sample](data/apple/ep_0001/LICENSE).
