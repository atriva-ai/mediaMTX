Sure! Here's a clear and concise `README.md` for your MediaMTX Docker setup to serve local RTSP streaming with a YAML config loading local video files:

```markdown
# MediaMTX Docker RTSP Streaming Setup

This repository contains a Docker Compose setup for **MediaMTX** to stream local video files over RTSP on your local network.

---

## Overview

- **MediaMTX** streams video files via RTSP.
- Uses a **local YAML config** to define streams.
- **FFmpeg** runs in a separate container to push video streams into MediaMTX.
- Ports mapped to host for easy access via RTSP clients.

---

## Prerequisites

- Docker and Docker Compose installed on your machine.
- Local video files placed in the `./videos/` directory.

---

## Files Structure

```

.
├── docker-compose.yml
├── mediamtx.yml
└── videos/
└── stream1.mp4

````

- `docker-compose.yml`: Defines MediaMTX and ffmpeg streamer services.
- `mediamtx.yml`: MediaMTX configuration file defining stream paths.
- `videos/`: Directory with local video files to stream.

---

## Configuration Details

### `mediamtx.yml`

```yaml
server:
  logfile: stdout

paths:
  stream1:
    runOnInitRestart: yes
    # runOnInit removed because streaming handled by ffmpeg container
````

* Defines `stream1` path to be served by MediaMTX.
* No `runOnInit` command here because streaming is done externally.

---

### `docker-compose.yml`

```yaml
version: "3.9"

services:
  mediamtx:
    image: bluenviron/mediamtx:latest
    container_name: mediamtx-rtsp
    restart: unless-stopped
    volumes:
      - ./mediamtx.yml:/mediamtx.yml
    ports:
      - 8554:8554       # RTSP streaming port
      - 8888:8888       # MediaMTX web interface
    command: /mediamtx.yml

  streamer:
    image: jrottenberg/ffmpeg:4.3-ubuntu
    depends_on:
      - mediamtx
    volumes:
      - ./videos:/videos
    command: >
      # Use following to simply passthrough input video to RTSP
      # -stream_loop -1 -re -i /videos/stream1.mp4
      # -c copy -f rtsp rtsp://mediamtx:8554/stream1
      # Use following to transcode to H.264 720p/15fps with IP frames only
      -stream_loop -1 -re -i /videos/stream1.mp4
      -vf scale=-2:720           # scale video height to 720p, width auto-adjusted to keep aspect ratio (multiple of 2)
      -r 15                      # set frame rate to 15 fps
      -c:v libx264               # encode video with H.264
      -x264opts bframes=0        # disable B-frames
      -preset veryfast           # encoding speed/quality tradeoff
      -f rtsp rtsp://mediamtx:8554/stream1
```

---

## How to Run

1. Place your video file(s) inside the `videos` folder, e.g., `stream1.mp4`.

2. Start the containers:

```bash
docker-compose up -d
```

3. MediaMTX will listen on port `8554` for RTSP streams and port `8888` for the web interface.

---

## Accessing the Stream

* RTSP stream URL (from your host machine or LAN):

```
rtsp://localhost:8554/stream1
```

* Open this URL in VLC or any RTSP-compatible player.

* Access MediaMTX web interface at:

```
http://localhost:8888
```

---

## Notes

* `ffmpeg` streamer container loops the video endlessly and pushes the stream to MediaMTX.
* MediaMTX serves the RTSP stream to any client on your local network.
* Adjust volumes and ports as needed for your setup.

---

## Troubleshooting

* If you do not see the stream in your player:

  * Check container logs with `docker logs -f mediamtx-rtsp` and `docker logs -f streamer`.
  * Confirm the video file exists in `./videos`.
  * Verify port mappings and firewall settings.

---

## License

MIT License

