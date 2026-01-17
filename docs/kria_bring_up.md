# KR260 Bring-Up: ROS 2 + Logitech Camera

This doc captures the bring-up steps for a KR260 running Ubuntu 24.04, with a Logitech USB camera.

## Prereqs

- Ubuntu 24.04 on the KR260
- Network access to `packages.ros.org`
- A UVC-compatible Logitech camera connected via USB

## Install ROS 2 Jazzy and tools

```bash
sudo apt update
sudo apt install -y curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  | sudo tee /etc/apt/keyrings/ros-archive-keyring.gpg >/dev/null

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
  | sudo tee /etc/apt/sources.list.d/ros2.list >/dev/null

sudo apt update
sudo apt install -y \
  ros-jazzy-ros-base \
  ros-jazzy-v4l2-camera \
  ros-jazzy-image-tools \
  ros-jazzy-image-transport \
  ros-jazzy-rqt-image-view \
  v4l-utils

sudo usermod -aG video $USER

# optional: auto-source ROS 2 for new shells
if ! grep -q "source /opt/ros/jazzy/setup.bash" ~/.bashrc; then
  echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
fi
```

Re-login or run `newgrp video` to apply the `video` group change.

## Verify camera detection

```bash
ls -l /dev/video*
v4l2-ctl --list-devices
lsusb
```

You should see the Logitech camera and `/dev/video0` (and sometimes `/dev/video1`).

## Run the camera node

```bash
source /opt/ros/jazzy/setup.bash
ros2 run v4l2_camera v4l2_camera_node --ros-args -p video_device:=/dev/video0
```

In another terminal, verify the stream:

```bash
source /opt/ros/jazzy/setup.bash
ros2 topic list
ros2 topic info /image_raw
ros2 topic echo /image_raw --once
```

## View images

If a display is attached to the kit:

```bash
source /opt/ros/jazzy/setup.bash
ros2 run rqt_image_view rqt_image_view
```

Or:

```bash
source /opt/ros/jazzy/setup.bash
ros2 run image_tools showimage --ros-args -r image:=/image_raw
```

## Troubleshooting

- If `/image_raw` exists but no data publishes, make sure no other process is
  holding `/dev/video0`:

```bash
fuser -v /dev/video0
```

- If the node logs mention `Failed mapping device memory`, ensure no other
  process is using the camera and try again.

- A warning about missing calibration file is expected unless you have created
  one under `~/.ros/camera_info/`.

