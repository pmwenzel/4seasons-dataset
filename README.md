# 4Seasons Dataset

## Dataset Structure

The 4Seasons dataset contains recordings from a stereo-inertial camera system coupled with a high-end RTK-GNSS receiver for global positioning. For more details regarding the sensor setup, please refer to the [paper](https://arxiv.org/abs/2009.06364).

The dataset consists of 18 (**more to be released soon**) individual sequences. For each sequence, the recorded data is stored in the following structure:

```
.
├── KeyFrameData
├── distorted_images
│   ├── cam0
│   └── cam1
├── undistorted_images
│   ├── cam0
│   └── cam1
├── GNSSPoses.txt
├── Transformations.txt
├── imu.txt
├── result.txt
├── septentrio.nmea
└── times.txt
```

Here,
- `KeyFrameData`: contains the KeyFrameFiles.
- `distorted_images`: contains both the distorted images from the left and right camera, respectively.
- `undistorted_images`: contains both the undistorted images from the left and right camera, respectively.
- `GNSSPoses.txt`: is a list of `7DOF` globally optimized poses (include scale from VIO to GNSS frame) for all keyframes (after GNSS fusion and loop closure detection). Each line is specified as `frame_id`, translation (`t_x`, `t_y`, `t_z`), rotation as quaternion (`q_x`, `q_y`, `q_z`, `w`), `scale`, `fusion_quality` (not relevant), and `v3` (not relevant).
- `Transformations.txt`: defines transformations between different coordinate frames.
- `imu.txt`: contains raw IMU measurements. Each line is specified as `frame_id`, (angular velocity (`w_x`, `w_y`, `w_z`), and linear acceleration (`a_x`, `a_y`, `a_z`)).
- `result.txt:` contains the `6DOF` visual interial odometry poses for every frame (not optimized). Each line is specified as `timestamp` (in seconds), translation (`t_x`, `t_y`, `t_z`), rotation as quaternion (`q_x`, `q_y`, `q_z`, `w`).
- `septentrio.nmea:` contains the raw GNSS measurements in the NMEA format.
- `times.txt` is a list of times in unix timestamps (in seconds), and exposure times (in milliseconds) for each frame (`frame_id`, `timestamp`, `exposure`).

The KeyFrameData folder contains a `.txt` file for each keyframe. Each Keyframe
file contains the following data: timestamp, camera intrinsics, camToWorld
transformation, exposure time and point cloud data. Each point is saved with
the following properties:

* `u,v`: image coordinates of depth point in pixels
* `idepth_scaled`: inverse depth value in `1/m`
  * metric 3D camera coordinates are obtained as follows: `x=(u-cx)/(fx*idepth_scaled)`; `y=(v-cy)/(fy*idepth_scaled)`; `z=1/idepth_scaled`
* `idepth_hessian`: hessian component of the corresponding inverse depth
  * estimated inverse depth variance is obtained as follows: `var=1/idepth_hessian`
* `maxRelBaseline`: maximum relative stereo baseline
* `numGoodRes`: not relevant
* `status`: not relevant
* `color information`: color values of an 8 pixel patch around the depth pixel (see https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7898369 for definition)

The `Transformations.txt` looks as follows:

```
# transform_S_AS: translation vector, rotation quaternion
0.000000,0.000000,0.000000,0.000590,-0.005845,0.005162,0.999969

# TS_cam_imu: translation vector, rotation quaternion
0.175412,0.003689,-0.058106,-0.007202,0.708623,-0.705546,-0.002350

# transform_w_gpsw: translation vector, rotation quaternion
0.321948,-0.029350,0.156487,-0.000720,-0.000459,0.125142,0.992138

# transform_gps_imu: translation vector, rotation quaternion
0.000000,0.000000,0.000000,0.000000,0.000000,0.000000,1.000000

# transform_e_gpsw: translation vector, rotation quaternion
4172814.292643,857504.007380,4731704.606937,0.225469,0.276513,0.724007,0.590354

# GNSS scale
0.969397
```

where

* `transform_S_AS`: is from SLAM internal scale to metric scale.
* `TS_cam_imu` is from IMU to the camera.
* `transform_w_gpsw`: is from local GPS world (ENU) to visual world.
* `transform_gps_imu`: is from IMU to GPS.
* `transform_e_gpsw`: is from local GPS world (ENU) to global Earth frame (ECEF).
* `GNSS scale`: not relevant
* Translation vector values correspond to `x, y, z` components of translation.
* Rotation quaternion values correspond to `q_x, q_y, q_z, w` components of quaternion.

To transform from the local SLAM world to ECEF, we need to apply the following transformations (in python):

```
transform_e_gpsw @ np.linalg.inv(transform_w_gpsw) @ transform_S_AS @ scale_mat
```
where `scale_mat` is a 4x4 diagonal matrix with the first 3 diagonal elements multiplied with the `scale` from the `GNSSPoses.txt` file for each pose. In case of `6DOF` poses (`results.txt`, `KeyFrameData`) `scale_mat` is a 4x4 identity matrix. The transformations have to be converted to transformation matrices. This can be done for example using [pyquaternion](https://github.com/KieranWynn/pyquaternion).


The calibration folder has the following structure:

```
.
├── calib_0.txt
├── calib_1.txt
├── calib_stereo.txt
├── camchain.yaml
├── undistorted_calib_0.txt
├── undistorted_calib_1.txt
└── undistorted_calib_stereo.txt
```

Here,
- `calib_0.txt:` contains the intrinsic parameters of the left camera. Camera model, fx, fy, cx, cy, and distortion coefficients.
- `calib_1.txt:` contains the intrinsic parameters of the right camera. Camera model, fx, fy, cx, cy, and distortion coefficients.
- `calib_stereo.txt:` contains the 4x4 matrix denoting the rigid transformation from the right to the left camera.
- `camchain.yaml:` contains the intrinsics and extrinsics for both cameras together in .yaml format.
- `undistorted_calib_0.txt:` contains the intrinsic parameters of the left camera. Camera model, fx, fy, cx, cy, and distortion coefficients.
- `undistorted_calib_1.txt:` contains the intrinsic parameters of the right camera. Camera model, fx, fy, cx, cy, and distortion coefficients.
- `undistorted_calib_stereo.txt:` contains the 4x4 matrix denoting the rigid transformation from the right to the left camera.
