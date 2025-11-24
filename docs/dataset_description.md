# Rover Mobility in a Lunar Analogue Dataset

**Authors:** Bailey C. Hopkins*, Ryan Hartzell, Ethan J. Paul, Cameron Hinkle, Charles Yuroff, Christopher Dreyer, Frances Zhu

**Affiliation:** Robotic Space Exploration (RoSE) Lab, Colorado School of Mines

**Contact:** bailey_hopkins@mines.edu

**Dataset Version:** 1.0 (Initial release, ~3 TB)

**License:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)

**Hosting:** Amazon Web Services (AWS) Open Data Sponsorship Program

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
arn:aws:s3:::amzn-s3-rose-mobility/
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
│   │   │   └── Processed_Videos/
│   │   │       ├── overhead.mp4
│   │   │       └── wheel_cams.mp4
│   │   ├── Trial_2/
│   │   └── ...
│   └── ...
│
└── Off_Nominal_Data/
    └── [Trials retained for completeness but not used in analysis]
```

Each **configuration folder** is named using the convention
`Speed_Radius_Direction_Inclination` (using raw numeric values), for example:

```
5.0_120.0_Right_0.0/
```

Every **trial subfolder** (`Trial_1`, `Trial_2`, …) includes:
- A `Raw_Data/` directory with individual ROS 2 MCAP logs for each sensor stream
- A `Processed_Data.csv` file aggregating synchronized, downsampled signals in a common “trial frame”
- A `Processed_Videos/` directory with synchronized visualizations

Incomplete or corrupted runs are archived under **Off_Nominal_Data/** to maintain dataset integrity and traceability.

---

## Data Content and Schema

### 1. Processed Data (CSV)

Each `Processed_Data.csv` represents one trial’s unified time series expressed in the trial-aligned reference frame.
Below is the schema:

| Column | Units | Description |
|:-------|:------|:-------------|
| `time_s` | s | Relative time (seconds from start) |
| `pos_x`, `pos_y`, `pos_z` | m | Rover COM position (Ground Truth/MoCap) |
| `quat_x`, `quat_y`, `quat_z`, `quat_w` | - | Orientation quaternion (Ground Truth/MoCap) |
| `ang_vel_x`, `ang_vel_y`, `ang_vel_z` | rad/s | Angular Velocity (IMU) |
| `lin_acc_x`, `lin_acc_y`, `lin_acc_z` | m/s² | Linear Acceleration (IMU) |
| `enc[1-4]` | rad | Cumulative wheel encoder position (1=FL, 2=FR, 3=BL, 4=BR) |
| `enc[1-4]_rad` | rad | Cumulative wheel encoder position (duplicate stream) |
| `vel[1-4]_rad_s` | rad/s | Wheel angular velocity |
| `current[1-4]` | raw | Motor current draw (raw ADC units) |

### 2. ROS 2 Topics (Raw Data)

| Topic | Message Type | Rate (Hz) | Description |
|:------|:--------------|:----------|:-------------|
| `/imu/data` | `sensor_msgs/Imu` | 100 | Quaternion, angular vel, linear acc |
| `/motor/encoder` | `custom_msgs/MotorData` | 10 | Wheel position & speed |
| `/wheel_camera/image_raw` | `sensor_msgs/Image` | 15 fps | RGB view of each wheel |
| `/optitrack/pose` | `geometry_msgs/PoseStamped` | 120 | Rover pose & orientation ground truth |
| `/lidar/points` | `sensor_msgs/PointCloud2` | static | Terrain mesh before & after trial |

All sensor data are recorded as **ROS 2 MCAP** files, viewable using:
```bash
ros2 bag info <file>.mcap
ros2 bag play <file>.mcap
```

---

## Experimental Parameters

| Variable | Range / Levels | Notes |
|:----------|:----------------|:------|
| Linear Velocity | 1 – 10 cm/s | Controlled skid-steer commands |
| Turning Radius | 0 – 120 cm (+∞ straight) | CW and CCW rotations tested |
| Inclination | −30° to +30° | Positive = uphill traverse |
| Repetitions | 6 – 10 per configuration | Ensures statistical robustness |

Total number of trials: 1,680.
Each complete trial ≈ 1.8 GB, aggregate dataset ≈ 3 TB (Version 1.0).

---

## Accessing the Dataset on AWS

The dataset is hosted in an **open-access Amazon S3 bucket** provided by the AWS Open Data Sponsorship Program.

**Bucket Name:** `amzn-s3-rose-mobility`
**Region:** `us-east-2`

### CLI Example
```bash
# List available configurations
aws s3 ls s3://amzn-s3-rose-mobility/Trial_Data/ --no-sign-request

# Copy one processed file
aws s3 cp s3://amzn-s3-rose-mobility/Trial_Data/5.0_120.0_Right_0.0/Trial_1/Processed_Data.csv . --no-sign-request
```

### Python Example
```python
import boto3
import pandas as pd
from io import BytesIO

# Anonymous access (no credentials needed)
s3 = boto3.client('s3', region_name='us-east-2', config=boto3.session.Config(signature_version=boto3.UNSIGNED))

bucket = 'amzn-s3-rose-mobility'
key = 'Trial_Data/5.0_120.0_Right_0.0/Trial_1/Processed_Data.csv'

obj = s3.get_object(Bucket=bucket, Key=key)
df = pd.read_csv(obj['Body'])
print(df.head())
```

---

## Related Resources

- **RoSE Lab:** [https://franceszhu.space/](https://franceszhu.space/)
- **RoSEy CubeRover:** [https://github.com/RoSE-Lab-Admin/CubeRover](https://github.com/RoSE-Lab-Admin/CubeRover)

---

## Citation and Acknowledgment

When using this dataset, please cite:

> Hopkins, B. C., et al. (2025). *Rover Mobility in a Lunar Analogue Dataset*. [In Submission].

Please also acknowledge the AWS Open Data Sponsorship Program.

---

*This documentation is part of the RoSE Lab open data initiative to promote reproducible terramechanics research.*
