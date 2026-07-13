# CARLA Stereo Dataset Recorder

A Python-based dataset recording application for the [CARLA Simulator](https://carla.org/).

The application records synchronised sensor data from a simulated vehicle and exports it in a structure inspired by the KITTI dataset format. It is intended for visual SLAM, visual-inertial SLAM, localisation, odometry, and autonomous-vehicle perception experiments.

## Recorded Data

The recorder captures:

* Left stereo camera images
* Right stereo camera images
* Ground-truth vehicle poses
* IMU measurements
* Frame timestamps in `time.txt`
* Sensor data organised in KITTI-style directories

The left and right camera images are recorded at corresponding timestamps so that each image pair represents the same simulation frame.

## Dataset Structure

A generated sequence follows a structure similar to:

```text
dataset/
└── sequences/
    └── 00/
        ├── image_0/
        │   ├── 000000.png
        │   ├── 000001.png
        │   ├── 000002.png
        │   └── ...
        ├── image_1/
        │   ├── 000000.png
        │   ├── 000001.png
        │   ├── 000002.png
        │   └── ...
        ├── poses.txt
        ├── imu.txt
        └── time.txt
```

Where:

* `image_0/` contains the left-camera images.
* `image_1/` contains the right-camera images.
* `poses.txt` contains the ground-truth pose for each recorded frame.
* `imu.txt` contains the recorded IMU measurements.
* `time.txt` contains the timestamp associated with each stereo frame.

The exact output structure may vary depending on the configuration used in the recorder script.

## File Formats

### Stereo Images

Images are saved using zero-padded frame numbers:

```text
000000.png
000001.png
000002.png
```

The same filename in `image_0/` and `image_1/` represents a synchronised stereo pair.

For example:

```text
image_0/000025.png
image_1/000025.png
```

### Ground-Truth Poses

Ground-truth poses are stored in `poses.txt`.

Each line corresponds to one recorded stereo frame and contains the vehicle pose associated with that frame.

For KITTI-compatible pose output, each pose can be represented as a flattened `3 × 4` transformation matrix:

```text
r00 r01 r02 tx r10 r11 r12 ty r20 r21 r22 tz
```

The transformation matrix is:

```text
r00 r01 r02 tx
r10 r11 r12 ty
r20 r21 r22 tz
0   0   0   1
```

The pose convention and coordinate-frame conversion should be checked before directly comparing the output with KITTI trajectories, because CARLA and KITTI may use different coordinate conventions.

### IMU Data

IMU data is stored in `imu.txt`.

A typical row may contain:

```text
timestamp accel_x accel_y accel_z gyro_x gyro_y gyro_z
```

Where:

* `accel_x`, `accel_y`, and `accel_z` are linear acceleration measurements.
* `gyro_x`, `gyro_y`, and `gyro_z` are angular velocity measurements.
* `timestamp` is the simulation timestamp associated with the measurement.

Depending on the selected sensor rate, several IMU measurements may be recorded between consecutive stereo image frames.

### Timestamps

Frame timestamps are stored in `time.txt`.

Each line represents the timestamp of one stereo frame:

```text
0.000000
0.050000
0.100000
0.150000
```

The line number corresponds to the image frame number. For example, the first timestamp corresponds to `000000.png`.

## Requirements

The application requires:

* Python 3
* CARLA Simulator
* CARLA Python API
* NumPy
* Pillow or OpenCV, depending on the image-saving implementation

Install the required Python packages where applicable:

```bash
pip install numpy pillow opencv-python
```

The CARLA Python API must also be available in the Python environment.

Depending on the CARLA installation, this may require adding the CARLA Python package to `PYTHONPATH`:

```bash
export PYTHONPATH="${PYTHONPATH}:/path/to/CARLA/PythonAPI/carla/dist/carla-<version>-py3.egg"
```

Replace the path and version with those used by your CARLA installation.

## Running the Recorder

Start the CARLA server:

```bash
cd /path/to/CARLA
./CarlaUE4.sh
```

In another terminal, run the dataset recorder:

```bash
python3 dataset_recorder.py
```

Replace `dataset_recorder.py` with the actual filename of the recorder script.

Any configurable values, such as the following, should be set in the script or its configuration file before recording:

* CARLA host and port
* Town or map
* Output directory
* Sequence number
* Camera resolution
* Camera field of view
* Stereo baseline
* Camera frame rate
* IMU frequency
* Vehicle spawn point
* Recording duration
* Weather conditions
* Static or dynamic traffic settings

## Synchronisation

The recorder should use CARLA synchronous mode so that the following measurements remain aligned:

* Left image
* Right image
* Ground-truth pose
* Simulation timestamp
* IMU measurements

Synchronous simulation is particularly important for stereo and visual-inertial SLAM experiments because unsynchronised measurements can introduce artificial trajectory and estimation errors.

## Intended Applications

The generated data can be used for:

* Stereo visual odometry
* Stereo visual SLAM
* Visual-inertial SLAM
* Place recognition
* Loop-closure experiments
* Perceptual aliasing experiments
* Perceptual variation experiments
* Trajectory evaluation
* Autonomous-vehicle localisation research

The output may also be adapted for tools such as ORB-SLAM3 and EVO after confirming the required calibration, timestamp, pose, and coordinate-frame formats.

## Dataset Size

Generated datasets can become very large. Dataset folders, images, videos, and recordings should normally not be committed directly to GitHub.

A suitable `.gitignore` entry is:

```gitignore
# Generated datasets
dataset/
datasets/
output/
outputs/
recordings/

# Recorded sensor data
*.bag
*.db3

# Video output
*.mp4
*.avi
*.mkv

# Python cache
__pycache__/
*.pyc

# Virtual environments
venv/
.venv/
```

Only the recorder source code, configuration files, documentation, and small example outputs should normally be stored in the repository.

## Repository Contents

```text
carla-stereo-dataset-recorder/
├── dataset_recorder.py
├── README.md
├── .gitignore
└── config/
    └── recorder_config.yaml
```

The exact filenames may differ depending on the implementation.

## Limitations

* The output is KITTI-style but may not be immediately compatible with every KITTI-based application.
* Camera calibration must match the simulated sensor configuration.
* CARLA coordinate frames may require conversion before trajectory evaluation.
* Ground-truth and IMU formats should be verified against the requirements of the target SLAM algorithm.
* Large datasets should be stored using external storage rather than normal Git tracking.

## Licence

Add an appropriate licence before distributing or publishing the project.

For example, an MIT licence can be used when you own the code and are permitted to release it.

## Author

Mayowa Adebambo
