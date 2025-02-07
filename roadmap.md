---
layout: default
title: "Project Roadmap"
---

# Project Roadmap

The project is divided into the following major steps:

## Step 1: Hardware & Environment Setup
- **Hardware Preparation:**
  - Connect and power the OAK‑D with the Jetson Nano.
  - Calibrate the stereo cameras using DepthAI’s calibration tools.
  - Install the latest DepthAI SDK, ROS2, and required Python dependencies.
- **Documentation:**
  - Record calibration files, intrinsics/extrinsics.
  - Verify system resources (CPU, GPU, RAM, power supply).

## Step 2: Model Preparation and Conversion
- **Train/Fine-Tune YOLOv4‑tiny:**
  - Gather and annotate a dataset (cones, people, cars, etc.).
  - Fine-tune the model on the dataset.
- **Convert the Model:**
  - Use DepthAI conversion tools to create a blob file.
  - Validate the blob with a standalone DepthAI demo.
- **Documentation:**
  - Log training parameters and conversion details.

## Step 3: Implement the Obstacle Detection Node with Stereo Depth
- **Pipeline Setup:**
  - Configure ColorCamera, Neural Network, and StereoDepth nodes.
  - Create output streams (color, NN detections, and depth).
- **Data Processing:**
  - Retrieve synchronized color frames, detections, and depth maps.
  - Calculate bounding box centers and compute depth measurements.
  - Publish obstacle detection data on `/obstacle_detections`.

## Step 4: Implement the Sensor Fusion Node
- **Subscriptions & Fusion:**
  - Subscribe to lane detection and obstacle detection topics.
  - Fuse lane error and avoidance offsets with proper scaling.
  - Publish the fused error on `/fused_error`.

## Step 5: Update the Path Planner / Controller Node
- **Integration:**
  - Update the PID controller to use the fused error.
  - Test and tune for smooth steering commands.

## Step 6: Build a Real-Time Parameter Tuning GUI
- **Using rqt Tools or a Custom PyQt GUI:**
  - Adjust critical parameters (confidence thresholds, gains) in real time.

## Step 7: Comprehensive Testing and Calibration
- **Unit and Integration Testing:**
  - Verify each node and test the complete pipeline in a simulator.
  - Document calibration and testing results.

## Step 8: Documentation and Presentation
- **Final Touches:**
  - Document system architecture, calibration, and testing.
  - Prepare demo videos and presentation materials.
