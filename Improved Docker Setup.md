# Better Container Setup for ECE 148

Hey everyone, I found a way better setup for our containers that fixes the USB issues with DepthAI cameras and makes development way smoother. The default setup works but this method lets you SSH directly into the container instead of SSHing to the Jetson and then using docker exec (which is super annoying tbh).

## What Problems This Fixes
- No more `[depthai] [warning] USB protocol not available` errors!!
- One-step SSH directly to container (instead of two steps)
- Better for remote VSCode development
- Still keeps all the ROS stuff working the same

## Quick Setup

Here's how to set it up:

1. **Stop any existing container** (if you already have one)
```bash
docker stop test_container
```

2. **Create a new container with proper USB access**
```bash
docker run \
  --name test_container_sshd \
  -it \
  -d \
  --privileged \
  -p 2222:22 \
  -v /dev/bus/usb:/dev/bus/usb \
  --device-cgroup-rule='c 189:* rmw' \
  --device /dev/video0 \
  djnighti/ucsd_robocar:devel
```

3. **Set up SSH server inside container**
```bash
docker exec -it test_container_sshd bash -c '
mkdir -p /run/sshd
echo "X11Forwarding yes" >> /etc/ssh/sshd_config
echo "X11UseLocalhost no" >> /etc/ssh/sshd_config
echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
apt-get update
apt-get install -y openssh-server xauth x11-apps
passwd root  # set a password you'll remember
service ssh start
'
```

## How to Use It

### Option 1: Direct SSH from your laptop
```bash
ssh -X -p 2222 root@jetson-ip-address
```
(Replace jetson-ip-address with your Jetson's actual IP)

### Option 2: Script for easy access from Jetson
Make a file called `ros.sh` with:
```bash
#!/bin/bash
# Start the SSH container if it's not already running
docker start test_container_sshd
# Make sure SSH is running
docker exec -it test_container_sshd service ssh start >/dev/null 2>&1
# Exec into the container
docker exec -it test_container_sshd bash
```

Then run it with:
```bash
bash ros.sh
```

## Why This Is Better

1. **One-Step Connection**: Just SSH directly to container from anywhere
2. **No USB Issues**: DepthAI cameras work properly without warnings
3. **Way Better for VSCode**: Use Remote-SSH extension to connect directly
4. **Same ROS Environment**: All the ROS stuff still works the same way
5. **Keeps Working After Reboot**: Just restart container, don't need to reconfigure

## Pro Tip
If you use VSCode, you can now set up the Remote-SSH extension to connect directly to the container on port 2222. It makes development WAY easier - you get proper syntax highlighting, IntelliSense, and can edit files directly.

Let me know if anyone has questions! This setup saved me tons of time.
