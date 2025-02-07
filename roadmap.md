---
layout: default
title: "Project Roadmap"
---

# Project Roadmap

Our vision is to develop a robust, real-time obstacle avoidance system for an Autonomous RC Car—one that mirrors the challenges faced in advanced vision systems used by current self-driving leaders like Tesla and Waymo. By integrating classical control methods with cutting-edge computer vision and deep learning, we are building a platform that handles multi-object detection and segmentation under stringent real-time constraints.

Leveraging our strong foundation in hardware integration, autonomous navigation, and precise localization (see [Completed Work](completed-work.html)), our next phases will focus on elevating the perception system to deliver intelligent, adaptive obstacle avoidance.

---

## Completed Milestones

- [x] **Hardware & Environment Setup**  
  - Integrated custom electronics mounting, 3D-printed enclosures, and the OAK‑D stereo camera with the Jetson Nano.  
  - Calibrated stereo cameras using DepthAI’s tools to ensure accurate depth perception.

- [x] **Autonomous Navigation & Control**  
  - Developed deep learning-based control algorithms on the DonkeyCar platform and DonkeySim simulator.  
  - Achieved [**3 successful autonomous laps**](videos.html#autonomous-laps) in both simulation and real-world tests.
  - Deployed a CNN-based linear model for behavioral cloning, optimized with data augmentation and hyperparameter tuning.

- [x] **Enhanced GPS Path Following Algorithm**  
  - Implemented sequential waypoint tracking and a wrap–around mechanism for robust GPS-based path tracking.  
  - Details available on the [Enhanced GPS Path Following Algorithm](enhanced-gps-path-following.html) page.

- [x] **Embedded Systems & Localization**  
  - Configured the Jetson Nano for remote access and multi-user operation.  
  - Integrated advanced GNSS with RTK error correction for centimeter-level 3D localization.

---

## Upcoming Milestones

- [ ] **Deploy & Calibrate Classical Computer Vision for Center-Line Following**  
  - **Environment & Deployment:**  
    - Implement a Docker-based unified environment with dedicated ROS workspaces for ROS1 and ROS2.  
    - Set up dual architecture support (ARM for Jetson boards and x86 for simulations).
  - **Navigation & Lane-Following:**  
    - Deploy the core navigation package that coordinates sensor, actuator, and lane detection nodes.
    - Implement vision-based lane detection using OpenCV in the HSV color space and compute cross-track error relative to the centerline.
    - Calibrate the PID controller for precise steering and throttle control.
  - **Outcome:**  
    - Establish a reliable baseline for center-line following, crucial for later fusion with advanced obstacle detection.

- [ ] **Implement Obstacle Detection Node**  
  - **YOLOv4‑tiny Integration:**  
    - Fine-tune the YOLOv4‑tiny model on a custom obstacle dataset (e.g., cones, people, cars).  
    - Convert the model to a DepthAI blob and validate with standalone demos.
  - **Pipeline Configuration:**  
    - Set up the DepthAI pipeline with ColorCamera, Neural Network, and StereoDepth nodes.  
    - Integrate DepthAI API calls to process YOLO detections and compute bounding box centers with corresponding depth values.

- [ ] **Integrate Stereo Depth**  
  - **StereoDepth Node Setup:**  
    - Configure the StereoDepth node (optimal resolution, confidence thresholds, subpixel mode).  
    - Ensure time-synchronized depth maps with RGB images for accurate distance measurement.
  - **Data Synchronization:**  
    - Develop functions to retrieve synchronized sensor data for precise obstacle distance estimation.

- [ ] **Fine-Tune Obstacle Avoidance**  
  - **Algorithm Refinement:**  
    - Adjust YOLO detection thresholds and depth extraction functions to reduce false positives/negatives.  
    - Optimize obstacle distance computation for real-time decision-making.
  - **Parameter Optimization:**  
    - Experiment with settings for confidence thresholds, non-maximum suppression, and depth filtering.

- [ ] **Implement Sensor Fusion Node**  
  - **Fusion Algorithm:**  
    - Merge lane detection outputs with obstacle detection data (lateral offsets, distances, confidence levels).  
    - Calculate an avoidance offset using a scaled function (e.g., `avoidance_offset = k_obs * max(distance, 0.1) * lateral_offset`) and fuse it with lane error.
  - **Dynamic Tuning:**  
    - Expose critical parameters (such as `k_obs`) as dynamic ROS2 parameters for on-the-fly adjustments.
  - **Robustness:**  
    - Implement error handling and logging to manage low-confidence detections or conflicting obstacle signals.

- [ ] **Update Path Planner / Controller**  
  - **PID Controller Enhancement:**  
    - Modify the PID controller to integrate the fused error signal from the sensor fusion node.  
    - Tune PID gains to accommodate the new dynamic error signal, ensuring smooth and robust vehicle control.

- [ ] **Develop Real-Time Parameter Tuning GUI**  
  - **GUI Development:**  
    - Create a responsive GUI (using rqt or a custom PyQt interface) to adjust parameters like YOLO confidence, `k_obs`, and PID gains in real time.
  - **User Experience:**  
    - Ensure the GUI supports rapid calibration during field tests, facilitating swift adjustments and performance optimization.

- [ ] **Comprehensive Integration Testing**  
  - **Unit & Integration Testing:**  
    - Validate each component (obstacle detection, sensor fusion, controller) in simulation and real-world environments.
  - **System-Level Calibration:**  
    - Conduct calibration sessions with known obstacles, iterate on design, and refine the overall pipeline for a robust, reliable obstacle avoidance system.


