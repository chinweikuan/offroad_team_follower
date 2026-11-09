# Development image: ROS 2 Jazzy + Gazebo Harmonic, rendered on the CPU.
#
# Build and run with compose.yaml (see docker/README.md).

# Official multi-arch ROS 2 Jazzy image (Ubuntu 24.04). It runs natively on Apple
# Silicon (arm64); osrf/ros:*-desktop-full images are amd64-only and would need emulation.
FROM ros:jazzy-perception

ENV DEBIAN_FRONTEND=noninteractive

# ros-jazzy-ros-gz pulls in Gazebo Harmonic through the ROS gz-*-vendor packages, so the
# separate packages.osrfoundation.org repository is not needed.
# mesa-utils and xvfb provide software OpenGL and a virtual display for camera sensors.
RUN apt-get update && apt-get install -y --no-install-recommends \
        git \
        tmux \
        vim \
        mesa-utils \
        xvfb \
        ros-jazzy-ros-gz \
        ros-jazzy-navigation2 \
        ros-jazzy-nav2-bringup \
        ros-jazzy-grid-map \
        ros-jazzy-xacro \
        ros-jazzy-foxglove-bridge \
    && rm -rf /var/lib/apt/lists/*

# Fail the build early if the Gazebo CLI is missing. The vendor packages install gz under
# /opt/ros/jazzy/opt, so it is only on PATH after sourcing the ROS setup script.
RUN bash -c "source /opt/ros/jazzy/setup.bash && gz sim --version"

WORKDIR /root/ws

RUN echo "source /opt/ros/jazzy/setup.bash" >> /root/.bashrc && \
    echo "if [ -f /root/ws/install/setup.bash ]; then source /root/ws/install/setup.bash; fi" >> /root/.bashrc

CMD ["/bin/bash"]
