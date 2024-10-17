<p align="center">
  <img align="center" alt="logo" src="docs/static/img/frigate.png">
</p>

# Frigate - NVR With Realtime Object Detection for IP Cameras

A complete and local NVR designed for [Home Assistant](https://www.home-assistant.io) with AI object detection. Uses OpenCV and Tensorflow to perform realtime object detection locally for IP cameras.

Use of a [Google Coral Accelerator](https://coral.ai/products/) is optional, but highly recommended. The Coral will outperform even the best CPUs and can process 100+ FPS with very little overhead.

- Tight integration with Home Assistant via a [custom component](https://github.com/blakeblackshear/frigate-hass-integration)
- Designed to minimize resource use and maximize performance by only looking for objects when and where it is necessary
- Leverages multiprocessing heavily with an emphasis on realtime over processing every frame
- Uses a very low overhead motion detection to determine where to run object detection
- Object detection with TensorFlow runs in separate processes for maximum FPS
- Communicates over MQTT for easy integration into other systems
- Records video with retention settings based on detected objects
- 24/7 recording
- Re-streaming via RTSP to reduce the number of connections to your camera
- WebRTC & MSE support for low-latency live view

## Documentation

View the documentation at https://docs.frigate.video

## Donations

If you would like to make a donation to support development, please use [Github Sponsors](https://github.com/sponsors/blakeblackshear).

## Screenshots

### Live dashboard

<div>
<img width="800" alt="Live dashboard" src="https://github.com/blakeblackshear/frigate/assets/569905/5e713cb9-9db5-41dc-947a-6937c3bc376e">
</div>

### Streamlined review workflow

<div>
<img width="800" alt="Streamlined review workflow" src="https://github.com/blakeblackshear/frigate/assets/569905/6fed96e8-3b18-40e5-9ddc-31e6f3c9f2ff">
</div>

### Multi-camera scrubbing

<div>
<img width="800" alt="Multi-camera scrubbing" src="https://github.com/blakeblackshear/frigate/assets/569905/d6788a15-0eeb-4427-a8d4-80b93cae3d74">
</div>

### Built-in mask and zone editor

<div>
<img width="800" alt="Multi-camera scrubbing" src="https://github.com/blakeblackshear/frigate/assets/569905/d7885fc3-bfe6-452f-b7d0-d957cb3e31f5">
</div>

## Prerequisite

1. Nodejs
2. Docker
3. Ubuntu

### For Local Development

1.  Delete existing files inside config folder if you want to generate new admin password

2.  Create config.yml with this contents (you can change this)

        mqtt:
          enabled: false
        cameras:
          test:
            ffmpeg:
              inputs:
                - path: /media/frigate/car-stopping.mp4
                  input_args: -re -stream_loop -1 -fflags +genpts
                  roles:
                    - detect
        version: 0.15-0
        auth:
          enabled: true

3.  Create a debug folder in root directory and add a sample mp4 file(s)

4.  Build docker frigate image

        docker buildx build --target=frigate --file docker/main/Dockerfile . \
            --tag frigate:latest \
            --load

5.  Run api server

        docker run --rm --publish=5000:5000 --volume=${PWD}/config:/config --volume=${PWD}/debug:/media/frigate/ frigate:latest

6.  The username and password in the docker logs (save it!)

    ![alt text](image.png)

7.  Go to web folder

    a. If you are running for the first time install dependency

        npm install

    b. Run the admin dashboard (developer version)

        npm run dev

### For Production build

1.  Delete existing files inside config folder if you want to generate new admin password

2.  Create config.yml with this contents (you can change this)

        mqtt:
          enabled: false
        cameras:
          test:
            ffmpeg:
              inputs:
                - path: /media/frigate/car-stopping.mp4
                  input_args: -re -stream_loop -1 -fflags +genpts
                  roles:
                    - detect
        version: 0.15-0
        auth:
          enabled: true

3.  Build docker frigate image

        docker buildx build --target=frigate --file docker/main/Dockerfile . \
            --tag frigate:latest \
            --load

4.  Create frigate container (you can edit this)

        docker run -d \
          --name frigate \
          --restart=unless-stopped \
          --mount type=tmpfs,target=/tmp/cache,tmpfs-size=1000000000 \
          --device /dev/bus/usb:/dev/bus/usb \
          --device /dev/dri/renderD128 \
          --shm-size=64m \
          -v /path/to/your/storage:/media/frigate \
          -v /path/to/your/config:/config \
          -v /etc/localtime:/etc/localtime:ro \
          -e FRIGATE_RTSP_PASSWORD='password' \
          -p 8971:8971 \
          -p 8554:8554 \
          -p 8555:8555/tcp \
          -p 8555:8555/udp \
          frigate

5.  The username and password in the docker logs (save it!)

    ![alt text](image.png)

6.  The admin dashboard is in port 8971

    ![alt text](image-1.png)
