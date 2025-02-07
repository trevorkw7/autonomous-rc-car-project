---
layout: default
title: "Completed Work"
---

# Completed Work

Below are the key achievements and milestones that have been completed thus far, serving the foundation for the project:

## Robot Design & Hardware Integration
- **Custom Electronics Mounting:**  
  Designed and fabricated a robust mounting plate integrating the Jetson Nano, OAK-D camera, and supporting circuitry—with attention to thermal management and vibration dampening.
- **3D-Printed Enclosures:**  
  Engineered durable, lightweight protective enclosures for the camera and Single Board Computer (SBC) using high-strength PLA, ensuring easy access for debugging and modifications.

## Autonomous Navigation & Control
- **Deep Learning-Based Control:**  
  Developed and tested autonomous navigation algorithms using the DonkeyCar platform and DonkeySim simulator, achieving **3 successful autonomous laps**. Key contributions include:
  - **Behavioral Cloning:**  
    Trained a linear model to learn expert driving behavior from recorded simulation data, mapping raw visual input to precise control commands.
  - **CNN-Based Linear Model:**  
    Employed a convolutional neural network for efficient feature extraction with an optimized layer configuration for real-time inference on resource-constrained platforms.
  - **Model Optimization:**  
    Enhanced performance through data augmentation and hyperparameter tuning (via `myconfig.py`) to minimize discrepancies between predicted and expert driving actions.
  - **Cross-Platform Demonstration:**  
    Validated the model’s robustness by deploying it on both local and external simulation environments.
  [Watch Autonomous Laps Video →](videos.html#autonomous-laps)
- **Enhanced GPS Path Following Algorithm:**  
  Extended the base path–following code to improve GPS reliability by implementing sequential waypoint tracking and a wrap–around mechanism.  
  [Learn more →](enhanced-gps-path-following.html)

## System & Software Proficiency
- **Linux & Python Expertise:**  
  Completed comprehensive training in Linux and Python, developing essential command-line and scripting skills for embedded systems.
- **ROS2 Framework Mastery:**  
  Gained hands-on experience with ROS2, including node/topic communication, service and launch file management, and real-time robotics middleware.

## Embedded Systems & Localization
- **Jetson Nano Configuration:**  
  Set up a robust embedded Linux environment on the NVIDIA Jetson Nano, including remote access, multi-user configuration, and the installation of NVIDIA Jetpack SDK and GPU-accelerated OpenCV.
- **Advanced GNSS Localization:**  
  Integrated a high-precision GNSS receiver with RTK error correction to achieve centimeter-level 3D localization, crucial for precise outdoor navigation and mapping.

[Watch the Demo Videos →](videos.html)
