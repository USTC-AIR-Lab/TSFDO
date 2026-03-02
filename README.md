# TSFDO

**TSFDO** (Two-Stage Filter-Based Distributed Odometry) is a fully distributed odometry framework for multi-robot systems (MRSs), built upon the invariant extended Kalman filter. It tightly integrates LiDAR, IMU, and UWB measurements, enabling each robot in the system to estimate the global state rather than only a local subset of states through inter-robot communication and distributed information fusion.

## Framework overview

<p align="center">
  <img src="figures/figure1.png" alt="figure1" width="800"/>
</p>

<p align="center">
  <em>figure1.  Framework of the proposed distributed odometry</em>
</p>

To evaluate the performance and advantages of this distributed odometry framework, we conduct a series of experiments using both a public dataset, S3E and data collected by ourselves. 

<!--📄 Related paper is available on arxiv: [TSFDO](https://arxiv.org/abs/xxxx). -->

---

## 1. Custom dataset
### 1.1 Sensor Configuration

<p align="center">
  <img src="figures/figure2.png" alt="figure2" width="500"/>
</p>

<p align="center">
  <em>figure2.  UAVs used for recording the dataset</em>

<p align="center">
  <img src="figures/figure3.png" alt="figure3" width="500"/>
</p>

<p align="center">
  <em>figure3.  Details about the UAV and the ground vehicle</em>

Four self-made UAVs and a ground mobile robot is utilized for data collection.

| Device | Type | Details |
|:--------:|:------:|:------:|
| LiDAR(Sequence-UAV_1,2,3,4 & Hete1,2,3)  | Livox MID 360 |  Range:70m  FOV:360°*59°  Freq:10Hz  |
| LiDAR(Sequence-GR_1,2,3)  | Velodyne VLP-16 | Range:100m FOV:360°*40° Freq:10Hz |
| IMU(Sequence-UAV_1,2,3,4 & Hete1,2,3)  | Livox MID 360 built-in ICM40609       |      |
| IMU (Sequence-GR_1,2,3) |                      |      |
| UWB |        |      |
| RTK  |                   |    |

### 1.2 Sensor Synchronization

### 1.3 Ground Truth

### 1.4 Dataset Analysis

<p align="center">
  <img src="figures/figure4.png" alt="figure4" width="600"/>
</p>

<p align="center">
  <em>figure4.  Trajectories of the ground robot when collecting sequences: Sequence-GR_1 (blue), Sequence-GR_2 (yellow), and Sequence-GR_3 (red)</em>
</p>

Four UAVs form a multi-robot system for data collection. For the ground robot dataset, data are collected by a single robot. 

Specifically, the data obtained from three different trajectories, containing IMU and LiDAR measurements, are merged into one multi-robot sequence, which is treated as data from a system of three robots as shown in the above figure. Using the ground truth obtained via GNSS, the relative distances among the robots are computed and used as UWB measurements. In this way, a multi-robot dataset containing IMU, LiDAR, and UWB information is constructed. 

These sequences obtained by ground robot are employed to compare the performance of our method with existing cooperative odometry approaches in scenarios where robot trajectories within a multi-robot system do not intersect except for the start point. The choice of the ground robot is motivated by its LiDAR type, since the existing methods in comparison are compatible only with Velodyne LiDAR, whereas our UAVs are equipped with Livox Mid-360 LiDAR.

### 1.5 Dataset Format

This project provides partial experimental data (in ROS bag format) for obtaining the experimental results in the paper.  

- [Rosbag](https://huggingface.co/datasets/WenZhong1024/USTC-AIS-Lab_TSFDO/tree/main)  
<!-- [Alternative link (Baidu Netdisk)](https://pan.baidu.com/s/xxxx) Extraction code: `****`  -->

> ⚠️ Note: Please cite this paper when using the dataset and comply with the open-source license.

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

- Thanks to the [ROS](https://www.ros.org/) community and other open-source projects such as [S3E](https://pengyu-team.github.io/S3E), [CoLRIO](https://github.com/PengYu-team/Co-LRIO), [DiSCo-SLAM](https://github.com/RobustFieldAutonomyLab/DiSCo-SLAM) [S-FAST-LIO](https://github.com/zlwang7/S-FAST_LIO) for providing tools and frameworks.  
- Thanks to all team members for their collaboration during the experiments.   
