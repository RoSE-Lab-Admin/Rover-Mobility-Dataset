# Rover Mobility in a Lunar Analogue Dataset

**Authors:** Bailey C. Hopkins*, Ryan Hartzell, Ethan J. Paul, Cameron Hinkle, Charles Yuroff, Christopher Dreyer, Frances Zhu  
**Affiliation:** Robotic Space Exploration (RoSE) Lab, Colorado School of Mines  
**Contact:** bailey_hopkins@mines.edu  
**Dataset Version:** 1.0 (Initial release, ~3 TB)  
**License:**  
**Hosting:** 
**Citation:** Hopkins et al., *Rover Mobility in a Lunar Analogue Dataset*, in submission (2025). DOI to be added upon publication.  

---

## Overview

The **Rover Mobility in a Lunar Analogue Dataset** provides an open, high-fidelity record of 1,680 controlled rover trials performed in the **Mines Lunar Surface Simulator (MLSS)** at the Colorado School of Mines.  
The dataset couples synchronized onboard telemetry, motion-capture ground truth, and overhead LiDAR terrain scans. It is designed to support:

- Physics-based terramechanics modeling and validation  
- Data-driven mobility and slip estimation methods  
- Digital-twin simulation and perception-to-mobility coupling studies  

All trials were conducted using the **RoSEy CubeRover**, a 6U-class platform built to Astrobotic’s CubeRover standard.

---

## Dataset Organization

At the top level of the S3 bucket:

```
<insert-s3-bucket>/
│
├── Trial_Data/
│   ├── <Speed>_<Radius>_<Direction>_<Inclination>/
│   │   ├── Trial_1/
│   │   │   ├── Raw_Data/
│   │   │   │   ├── imu_data.mcap
│   │   │   │   ├── motor_data.mcap
│   │   │   │   ├── wheel_cameras/
│   │   │   │   ├── motion_capture.mcap
│   │   │   │   └── lidar_scan.mcap
│   │   │   ├── Processed_Data.csv
|   |   |   └── Processed_Videos/
│   │   │   │   ├── overhead.mp4
│   │   │   │   └── wheel_cams.mp4
│   │   ├── Trial_2/
│   │   └── ...
│   └── ...
│
└── Off_Nominal_Data/
    └── [Trials retained for completeness but not used in analysis]
```

Each **configuration folder** is named using the convention  
`Speed_Radius_Direction_Inclination`, for example:

```
5cms_120cm_CCW_0deg/
```

Every **trial subfolder** (`Trial_1`, `Trial_2`, …) includes:
- A `Raw_Data/` directory with individual ROS 2 MCAP logs for each sensor stream  
- A `Processed_Data.csv` file aggregating synchronized, downsampled signals in a common “trial frame”  
- A `Processed_Data/` directory with graphs, videos, and DEMs

Incomplete or corrupted runs are archived under **Bad_Data/** to maintain dataset integrity and traceability.

---

## Data Content and Schema

### 1. Processed Data (CSV)

Each `Processed_Data.csv` represents one trial’s unified time series expressed in the trial-aligned reference frame.  
Below is an example schema:

| Column | Units | Description |
|:-------|:------|:-------------|
| `timestamp` | s | Absolute ROS time |
| `x`, `y`, `z` | m | Rover COM position in trial frame |
| `roll`, `pitch`, `yaw` | rad | Euler orientation (XYZ convention) |
| `ax`, `ay`, `az` | m/s² | Linear acceleration (IMU) |
| `wx`, `wy`, `wz` | rad/s | Angular velocity (IMU) |
| `w1_angle`, `w1_rate` … `w4_rate` | rad, rad/s | Per-wheel encoder telemetry |
| `vx_body`, `vy_body` | m/s | Rover body-frame linear velocity |
| `terrain_slope` | deg | Mean local slope from LiDAR pre-scan |
| `terrain_height_change` | mm | Mean ΔZ from post- vs pre-scan DEM |
| `trial_id` | — | Trial identifier |

### 2. ROS 2 Topics (Raw Data)

| Topic | Message Type | Rate (Hz) | Approx File Size per Trial | Description |
|:------|:--------------|:----------|:---------------------------|:-------------|
| `/imu/data` | `sensor_msgs/Imu` | 100 | 2 MB | Quaternion, angular vel, linear acc |
| `/motor/encoder` | `custom_msgs/MotorData` | 10 | 80 KB | Wheel position & speed |
| `/wheel_camera/image_raw` | `sensor_msgs/Image` | 15 fps | 450 MB | RGB view of each wheel |
| `/optitrack/pose` | `geometry_msgs/PoseStamped` | 120 | 1 MB | Rover pose & orientation ground truth |
| `/lidar/points` | `sensor_msgs/PointCloud2` | static scans | 700 MB | Terrain mesh before & after trial |

All sensor data are recorded as **ROS 2 MCAP** files, viewable using:
```bash
ros2 bag info <file>.mcap
ros2 bag play <file>.mcap
```

---

## Experimental Parameters

| Variable | Range / Levels | Notes |
|:----------|:----------------|:------|
| Linear Velocity | 1 – 10 cm/s (1 cm/s increments) | Controlled skid-steer commands |
| Turning Radius | 0 – 120 cm (+∞ straight) | CW and CCW rotations tested |
| Inclination | −30° to +30° (5° steps) | Positive = uphill traverse |
| Repetitions | 6 – 10 per configuration | Ensures statistical robustness |

Total number of trials: 1 , 680.  
Each complete trial ≈ 1.8 GB, aggregate dataset ≈ 3 TB (Version 1.0).

---

## Accessing the Dataset on AWS

Once published, the dataset will be hosted in an **open-access Amazon S3 bucket**.

Example commands (to be updated with final bucket info):

```bash
# List available configurations
aws s3 ls <insert-s3-bucket>/Trial_Data/

# Copy one processed file
aws s3 cp <insert-s3-bucket>/Trial_Data/5cmps_120cm_CCW_0deg/Trial_1/Processed_Data.csv .

# Sync entire dataset (use with caution)
aws s3 sync <insert-s3-bucket>/ ./local_rosey_data/ --no-sign-request
```

For partial or programmatic access:

```python
import boto3, pandas as pd
s3 = boto3.client('s3', region_name='<insert-region>')
obj = s3.get_object(Bucket='<insert-bucket>', Key='Trial_Data/5cmps_120cm_CCW_0deg/Trial_1/Processed_Data.csv')
df = pd.read_csv(obj['Body'])
print(df.head())
```

---

## Related Resources

- **Dr. Frankie Zhu & the RoSE Lab:**  
  [https://franceszhu.space/](https://franceszhu.space/)

- **RoSEy CubeRover specifications:**  
  [https://github.com/RoSE-Lab-Admin/CubeRover](https://github.com/RoSE-Lab-Admin/CubeRover)

---

## Data Quality and Known Limitations

- **Incomplete Trials:** Some runs suffered from sensor dropouts, misalignment, or mechanical anomalies. These are preserved under `Bad_Data/` for transparency.  
- **Boundary Effects:** Tests at non-zero inclinations retain wooden side-frames that slightly alter regolith flow near edges.  
- **Optical Interference:** Small sinusoidal banding may appear in LiDAR ΔZ maps due to motion-capture frequency interference.  

Each `Processed_Data.csv` has been bias-corrected, synchronized to the least-sampled stream, and transformed into a path-aligned **trial frame** for consistency across sessions.

---

## Citation and Acknowledgment

When using this dataset, please cite the following paper:

> Hopkins, B. C., Hartzell, R., Paul, E. J., Hinkle, C., Yuroff, C., Dreyer, C., & Zhu, F. (2025). *Rover Mobility in a Lunar Analogue Dataset*. Journal of Terramechanics (submitted).  

Please also reference the AWS dataset entry once live:

> *Rover Mobility in a Lunar Analogue Dataset*. Registry of Open Data on AWS.  
> DOI and URL to be added upon publication.

---

## Versioning and Future Updates

- **Version 1.0 (2025):** Includes 1,680 trials (~3 TB) on CSM-LHT-T simulant.  
- **Planned Updates:** Future versions will add variable wheel designs, additional sensors, and new mobility trials.  
- **Change Tracking:** Updates will be tagged as `v1.x` on GitHub and documented in `CHANGELOG.md`.

---

## License

The dataset is released for public research and educational use.  

---

*This documentation is part of the RoSE Lab open data initiative to promote reproducible terramechanics research and rover mobility analysis.*
