# Development container

ROS 2 Jazzy and Gazebo Harmonic (gz-sim 8) on Ubuntu 24.04. The image runs natively on
Apple Silicon (arm64) and on x86_64 hosts.

## Usage

```bash
cd docker
docker compose build        # build the image
docker compose up -d        # start the container in the background
docker exec -it offroad_team_follower_dev bash   # open a shell (repeat for more terminals)
docker compose down         # stop and remove the container
```

The repository is mounted at `/root/ws/src/offroad_team_follower`. Run `colcon build` from
`/root/ws`; the build output stays inside the container.

## Rendering

There is no GPU passthrough on macOS, so Mesa renders on the CPU (llvmpipe). Camera sensors
work without a display by running the server with EGL:

```bash
gz sim -s -r --headless-rendering <world.sdf>
```

This uses the default `ogre2` engine. The older `ogre` engine needs an X display and only
runs under `xvfb-run`.

## Check that sensors render

```bash
gz sim -s -r --headless-rendering camera_sensor.sdf &
gz topic -e -n 1 -t /camera | grep -E '^(width|height)'   # expect 320 and 240
```
