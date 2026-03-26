# TSFDO

<!-- **TSFDO** (Two-Stage Filter-Based Distributed Odometry) is a fully distributed odometry framework for multi-robot systems (MRSs), built upon the invariant extended Kalman filter. It tightly integrates LiDAR, IMU, and UWB measurements, enabling each robot in the system to estimate the global state rather than only a local subset of states through inter-robot communication and distributed information fusion.

## Framework overview

<p align="center">
  <img src="figures/figure1.png" alt="figure1" width="800"/>
</p>

<p align="center">
  <em>figure1.  Framework of the proposed distributed odometry</em>
</p>

To evaluate the performance and advantages of this distributed odometry framework, we conduct a series of experiments using both a public dataset, S3E and data collected by ourselves. 

📄 Related paper is available on arxiv: [TSFDO](https://arxiv.org/abs/xxxx).

-->

## 1. Custom data
### 1.1 Sensor Configuration

<p align="center">
  <img src="figures/figure2.png" alt="figure1" width="500"/>
</p>

<p align="center">
  <em>figure1.  UAVs used for recording the dataset</em>

<p align="center">
  <img src="figures/figure3.png" alt="figure2" width="500"/>
</p>

<p align="center">
  <em>figure2.  Details about the UAV and the ground vehicle</em>

Four self-made UAVs and a ground mobile robot is utilized for data collection.

| Device | Type | Details |
|:--------:|:------:|:------:|
| LiDAR(Sequence-UAV_1,2,3,4)  | Livox MID 360 |  Range:70m FOV:360°*59° Freq:10Hz  |
| LiDAR(Sequence-GR_1,2,3)  | Velodyne VLP-16 | Range:100m FOV:360°*40° Freq:10Hz |
| LiDAR(Sequence-Hete_1,2,3)  | Simulated MID360 |  Freq:10Hz  |
| IMU(Sequence-UAV_1,2,3,4)  | Livox MID 360 built-in ICM40609 |   Freq:200Hz   |
| IMU (Sequence-GR_1,2,3) | WHEELTEC N100 9-axis |    Freq:100Hz  |
| IMU(Sequence-Hete_1,2,3)  |  GAZEBO imu_plugin   |   Freq:50Hz |
| UWB |  Nooploop LinkTrack(Tag-Tag TWR) P-BS2 |  Range:200m Accuracy:±10cm Freq:50Hz  |
| RTK  | Beitian BG-620 GNSS Receiver (RTK mode)|  Accuracy:±1.5cm Freq:1Hz  |

### 1.2 Sensor Synchronization
#### 1.2.1 Time synchronization
**Time synchronization across all sensors**： Time synchronization is not considered in this work due to the heterogeneous nature of the dataset and compatibility limitations among the evaluated algorithms. Specifically, the proposed system follows a tightly-coupled LiDAR–IMU odometry framework, where high-rate IMU measurements are used for continuous state propagation and LiDAR motion compensation, thus reducing the dependency on strict hardware-level synchronization. Besides, Livox Mid360 provides time-synchronized imu data.

Furthermore, the UWB sensor operates at a relatively high frequency, providing dense range measurements over time. Although these measurements are not strictly synchronized with the LiDAR and IMU due to independent sensor clocks, the high update rate enables effective temporal alignment through interpolation and state propagation within the estimation framework.

**Time synchronization across agents**: In outdoor settings with access to Global Navigation Satellite System (GNSS) signals, we use GNSS time as the global reference to synchronize the timing across agents after the collection.
#### 1.2.2 Sensor calibration
The LiDAR–IMU extrinsic parameters are treated as known constants, which are detailed in the YAML file of custom data used in our experiments. The lever-arm compensation of the UWB sensor is omitted, since ablation studies indicate that its influence on the overall estimation accuracy is negligible.
### 1.3 Ground Truth
For outdoor environments with good GNSS signal reception, a dual-antenna RTK device(Beitian BG-620 GNSS Receiver) is used to achieve highly accurate localization data with centimeter-level precision.

For simulated datasets, we obtain the ground truth from gazebo topic /gazebo/model_states.
### 1.4 Dataset Analysis

<p align="center">
  <img src="figures/figure4.png" alt="figure3" width="600"/>
</p>

<p align="center">
  <em>figure3.  Trajectories of the ground robot when collecting sequences: Sequence-GR_1 (blue), Sequence-GR_2 (yellow), and Sequence-GR_3 (red)</em>
</p>

Four UAVs form a multi-robot system for data collection.

For the ground robot sequence, data are collected by a single robot. Specifically, the data obtained from three different trajectories, containing IMU and LiDAR measurements, are merged into one multi-robot sequence, which is treated as data from a system of three robots as shown in the above figure. Using the ground truth obtained via GNSS, the relative distances among the robots are computed and used as UWB measurements. In this way, a multi-robot dataset containing IMU, LiDAR, and UWB information is constructed. 

The heterogeneous sequences generated in the Gazebo simulation are collected using a team of four robots: two UAVs (robots 1 and 2) and two ground robots (robots 3 and 4). Sequence-Here 3 contains the longest trajectories among all sequences used in our experiments.

**Analysis of our datasets**
| Sequence | Time[s] | Ground Truth | Length[m] | Size | Sensors|
|:--------:|:------:|:------:|:------:|:------:|:------:|
| Sequence-UAV_1  | 100 | RTK | uav0:109  uav1:109  uav4:109  uav5:96 | 1.1GB |IMU LiDAR UWB RTK|
| Sequence-UAV_2  | 107 | RTK | uav0:103  uav1:107  uav4:94   uav5:75 | 1.3GB |IMU LiDAR UWB RTK|
| Sequence-UAV_3  | 179 | RTK | uav0:109  uav1:106  uav4:115  uav5:177| 2.0GB |IMU LiDAR UWB RTK|
| Sequence-UAV_4  | 126 | RTK | uav0:111  uav1:106  uav4:99   uav5:44 | 1.3GB |IMU LiDAR UWB RTK|
| Sequence-GR_1   | 406 | RTK | Alpha:248 Bob:229 Carol:178 | 5.1GB |IMU LiDAR RTK|
| Sequence-GR_2   | 436 | RTK | Alpha:226 Bob:199 Carol:240 | 5.3GB |IMU LiDAR RTK|
| Sequence-GR_3   | 692 | RTK | Alpha:420 Bob:424 Carol:443 | 9.7GB |IMU LiDAR RTK|
| Sequence-Hete_1 | 72  | /gazebo/model_states | iris_0:304 iris_1:252 iris_2:189 iris_3:230   | 522MB |simulated IMU LiDAR UWB|
| Sequence-Hete_2 | 79  | /gazebo/model_states | iris_0:394 iris_1:361 iris_2:188 iris_3:229   | 582MB |simulated IMU LiDAR UWB|
| Sequence-Hete_3 | 205 | /gazebo/model_states | iris_0:1616 iris_1:1174 iris_2:492 iris_3:500 | 1.5GB |simulated IMU LiDAR UWB|

### 1.5 Dataset Format

Our research utilizes the ROS bag format for sensor data storage, a standard in robotics known for efficient data management and playback.Data from the same operational sequence are merged into one ROS bag file.

**Data Organization**: Our dataset is meticulously organized with a clear directory structure. Calibration parameters are detailed in YAML file. Each data sequence is complete with a .bag file for primary sensor measurements. Additionally, to aid in thorough performance assessments, we’ve included auxiliary files like <agent_id>.tum, which contain ground truth data.

**Ground Truth Format**: The ground truth data, which is essential for evaluating the accuracy of Collaborative SLAM algorithms, is provided as TXT files. These files contain timestamped poses in UTM coordinates and orientation quaternions, formatted as follows: [timestamp, tx, ty, tz, qx, qy, qz, qw].

**Information of ROS topics included in our datasets.** 
| Sensor | Topic | Type |
|:--------:|:------:|:------:|
| LiDAR(Sequence-UAV_1,2,3,4)  | /agent_id/livox/lidar |  livox_ros_driver2/CustomMsg  |
| IMU(Sequence-UAV_1,2,3,4)    | /agent_id/livox/imu   |  sensor_msgs/Imu  |
| UWB(Sequence-UAV_1,2,3,4)    | /agent_id/nlink_linktrack_nodeframe2 |  nlink_parser/LinktrackNodeframe2  |
| RTK(Sequence-UAV_1,2,3,4)    | /agent_id/global_position/raw/fix |  sensor_msgs/NavSatFix  |
| LiDAR(Sequence-GR_1,2,3)     | /agent_id/velodyne_points | sensor_msgs/PointCloud2 |
| IMU(Sequence-GR_1,2,3)       | /agent_id/imu/data |  sensor_msgs/Imu  |
| RTK(Sequence-GR_1,2,3)       | /agent_id/fix |  sensor_msgs/NavSatFix  |
| simulated UWB(Sequence-GR_1,2,3)       | /agent_id/nlink_linktrack_nodeframe2 | std_msgs/Float32MultiArray  |
| simulated UWB(Sequence-Hete_1,2,3)      | /agent_id/nlink_linktrack_nodeframe2 | std_msgs/Float64MultiArray |
| LiDAR(Sequence-Hete_1,2,3)    | /agent_id/scan|  livox_ros_driver/CustomMsg |
| IMU(Sequence-Hete_1,2,3)      | /agent_id/mavros/imu/data  /agent_id/imu/data | sensor_msgs/Imu |

Agent_id stands for (uav0,uav1,uav4,uav5) or (Alpha,Bob,Carol) or (iris_0,iris_1,iris_2,iris_3).
The detailed breakdown of the individual data fields contained within the Ultra-Wideband (UWB) dataset please refer to [S3E](https://pengyu-team.github.io/S3E).

This project provides partial experimental data (in ROS bag format) for obtaining the experimental results in the paper.  

- [Rosbag](https://huggingface.co/datasets/Test20261024/TSFDO_Dataset/tree/main)

<!-- For the ground robot sequences, please refer to the alternative link below. 
- [Alternative link](https://pan.baidu.com/s/1LiQXvI2Qkc5VXlA5uhnezg?pwd=e3j6) Extraction code: `e3j6` -->

### 1.6 Global Extrinsic Transformations

ScanContext, a lightweight spatial feature descriptor for 3D LiDAR, is used to describe and match features. 

Then, mobile robots exchange scan context features and perform scan matching. Once a potential inter-robot loop closure candidate is detected, incremental pairwise consistent measurement set maximization (PCM) is performed to remove outliers. A two-stage optimization is performed by each robot, first establishing a global-to-local coordinate transformation. 

Finally, coordinate transformations are exchanged among robots to build the observation model.

<!-- > ⚠️ Note: Please cite this paper when using the dataset and comply with the open-source license.

---

## 2. Experimental results on the custom dataset

### 2.1 Effectiveness experiments

<p align="center">
  <img src="figures/figure5.png" alt="figure5" width="500"/>
</p>

<p align="center">
  <em>figure5.  State estimates and ground truth of Robot 4 on sequence Sequence-UAV_1 of our custom dataset</em>
</p>

This figure shows the comparison among ego-state estimates (blue solid line) , self-state estimates (red solid line), and groud truth on Sequence-UAV_1. It is demonstrated that the UWB-based update effectively improves the accuracy of self-state estimates, even in cases where the self-state estimates are already relatively accurate.

<p align="center">
  <img src="figures/figure6.png" alt="figure6" width="500"/>
</p>

<p align="center">
  <em>figure6. Trajectory estimates of all four drones calculated by Robot 1</em>
  
</p>

<table>
  <tr>
    <td>
      <img src="figures/figure7.png" alt="7" width="450"/>
      <p align="center"><em>figure7. State estimates of Robot 1 computed by Robot 1 on sequence Sequence-UAV_4 of our custom dataset</em></p>
    </td>
    <td>
      <img src="figures/figure8.png" alt="8" width="450"/>
      <p align="center"><em>figure8. State estimates of Robot 2 computed by Robot 1 on sequence Sequence-UAV_4 of our custom dataset</em></p>
    </td>
  </tr>
</table>

<table>
  <tr>
    <td>
      <img src="figures/figure9.png" alt="9" width="450"/>
      <p align="center"><em>figure9. State estimates of Robot 3 computed by Robot 1 on sequence Sequence-UAV_4 of our custom dataset</em></p>
    </td>
    <td>
      <img src="figures/figure10.png" alt="10" width="450"/>
      <p align="center"><em>figure10. State estimates of Robot 4 computed by Robot 1 on sequence Sequence-UAV_4 of our custom dataset</em></p>
    </td>
  </tr>
</table>

These figures shows the state estimates of four UAVs calculated by the first UAV on Sequence-UAV_4, and the comparison between these estimates and their corresponding groud truths. It can be clearly observed that the robot is able to estimate the states of all other individuals under the connected communication topology given in the following figure, even during intervals when two robots cannot directly exchange information.

<p align="center">
  <img src="figures/figure11.png" alt="figure11" width="500"/>
</p>

<p align="center">
  <em>figure11.  Communication topology among the four UAVs</em>
  
</p>

The dashed line represents the intermitted communication.

### 2.2 Ablasion experiments

<p align="center">
  <img src="figures/figure12.png" alt="figure12" width="500"/>
</p>

<p align="center">
  <em>figure12.  Evaluation of mutual state estimation across different joint state estimation processes</em>
  
</p>

We consider three cases for the joint state estimation stage: **1)** the update step is omitted; **2)** the consensus step is omitted; and **3)** all three steps are performed. This table reports the accuracy of mutual state estimation for robot pairs with discontinuous communication in the three cases. 

A comparison between cases 1) and 3) shows that the UWB-based update provides further correction of the mutual-state estimates, but the improvement is limited. Meanwhile, the difference in estimation accuracy between cases 2) and 3) highlights the critical role of the consensus step in ensuring the precision of the mutual state estimation. Based on these facts, the absence of the consensus step will significantly undermine the performance of TSFDO.

<p align="center">
  <img src="figures/figure13.png" alt="figure13" width="600"/>
</p>

<p align="center">
  <em>figure13.  State estimates of Robot 3 computed by Robot 1 under different communication intervals on sequence Sequence-UAV_3 without the consensus step.
 </em>
</p>

This figure shows the state estimates of robot 3 computed by robot 1 under different communication intervals on Sequence-UAV_3, when there is no consensus step in the joint state estimation stage. It is demonstrated that the mutual state estimation accuracy can still be maintained at a relatively higher communication frequency. As the communication interval getting longer, the mutual state estimation performance degrades significantly. 

---

## 📌 Citation

If this repository and dataset are helpful for your research, please cite our paper:
```bibtex
@inproceedings{your_paper,
  title={Paper Title},
  author={Your Name and Others},
  booktitle={arXiv},
  year={2025}
}
```
---

## 🙏 Acknowledgements

This project would not have been possible without the following resources and support:

- Thanks to the [ROS](https://www.ros.org/) community and other open-source projects such as [S3E](https://pengyu-team.github.io/S3E), [CoLRIO](https://github.com/PengYu-team/Co-LRIO), [DiSCo-SLAM](https://github.com/RobustFieldAutonomyLab/DiSCo-SLAM), [Swarm-LIO2](https://github.com/hku-mars/Swarm-LIO2), [S-FAST-LIO](https://github.com/zlwang7/S-FAST_LIO) for providing tools and frameworks.  
- Thanks to all team members for their collaboration during the experiments.   -->
