# RTMP/RTSP/HLS Bridge for Wyze Cam

Docker container to enable RTMP, RTSP, and HLS streams for Wyze cams based on [noelhibbard's script](https://gist.github.com/noelhibbard/03703f551298c6460f2fd0bfdbc328bd#file-readme-md) with [kroo/wyzecam](https://github.com/kroo/wyzecam), [aler9/rtsp-simple-server](https://github.com/aler9/rtsp-simple-server), and [shauntarves/wyze-sdk](https://github.com/shauntarves/wyze-sdk).

Exposes a local RTMP, RTSP, and HLS stream for all your Wyze Cameras including v3. No Third-party or special firmware required.

Should work on most x64 systems as well as on some arm-based systems like the raspberry pi. 
[See here](#armraspberry-pi-support) for instructions to run on arm.

## Changes in v0.3.2

- 🆕 Multi-Factor Authentication Support. [See here](#Multi-Factor-Authentication) 

## Usage

- git clone this repo or download the latest [release](https://github.com/mrlt8/docker-wyze-bridge/releases)
- Copy and rename `docker-compose.sample.yml` to `docker-compose.yml` 
- Edit `docker-compose.yml` with your wyze credentials
- run `docker-composer up`

(May need to pass a verification code if you have Two-Step Verification. [See here](#Multi-Factor-Authentication).)

Once you're happy with your config you can use `docker-compose up -d` to run it in detached mode.


## URIs

`camera-nickname` is the name of the camera set in the Wyze app and are converted to lower case with hyphens in place of spaces. 

e.g. 'Front Door' would be `/front-door`


- RTMP:  
```
rtmp://localhost:1935/camera-nickname
```
- RTSP:  
```
rtsp://localhost:8554/camera-nickname
```
- HLS:  
```
http://localhost:8888/camera-nickname/stream.m3u8
```
- HLS can also be viewed in the browser using:
```
http://localhost:8888/camera-nickname
```


## Filtering

The default option will automatically create a stream for all the cameras on your account, but you can use the following environment options in your `docker-compose.yml` to filter the cameras.

All options are cAsE-InSensiTive, and take single or multiple comma separated values.


#### Examples:

- Whitelist by Camera Name (set in the wyze app):
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_NAMES=Front Door, Driveway, porch
```
- Whitelist by Camera MAC Address:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MACS=00:aA:22:33:44:55, Aa22334455bB
```
- Whitelist by Camera Model:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MODEL=WYZEC1-JZ
```
- Whitelist by Camera Model Name:
```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_MODEL=V2, v3, Pan
```
- Blacklisting:

You can reverse any of these whitelists into blacklists by adding *block, blacklist, exclude, ignore, or reverse* to `FILTER_MODE`. 

```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FILTER_NAMES=Bedroom
    - FILTER_MODE=BLOCK
```

## Multi-Factor Authentication

Two-factor authentication ("Two-Step Verification" in the wyze app) is supported and will automatically be detected, however additional steps are required to enter your verification code.

- echo the verification code directly to `/wyze-bridge/tokens/mfa_token`
```bash
docker exec -it wyze-bridge sh -c 'echo "123456" > /wyze-bridge/tokens/mfa_token'
```
- Mount `/wyze-bridge/tokens/` locally and add your verification code to a new file `mfa_token`: 
```YAML
volumes: 
    - ./tokens:/wyze-bridge/tokens/
```


## ARM/Raspberry Pi Support

The default configuration will use the x64 tutk library, however, you can edit your `docker-compose.yml` to use the 32-bit arm library by setting `dockerfile` as `Dockerfile.arm`:

```YAML
    wyzecam-bridge:
        container_name: wyze-bridge
        restart: always
        build: 
            context: ./app
            dockerfile: Dockerfile.arm
        environment:
            - WYZE_EMAIL=
            - WYZE_PASSWORD=
```

## Bitrate and Resolution

Bitrate and resolution of the stream from the wyze camera can be adjusted with `- QUALITY=HD120`.
- Resolution can be set to `SD` (640x360 cams/480x640 doorbell) or `HD` (1920x1080 cam/1296x1728 doorbell). Default - HD.
- Bitrate can be set from 60 to 240 kb/s. Default - 120.
- Bitrate and resolution changes will apply to ALL cameras with the current version.

```yaml
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - QUALITY=SD60
```

## Custom FFmpeg Commands

You can pass a custom [command](https://ffmpeg.org/ffmpeg.html) to FFmpeg by using `FFMPEG_CMD` in your docker-compose.yml:

```YAML
environment:
    - WYZE_EMAIL=
    - WYZE_PASSWORD=
    - FFMPEG_CMD=-f h264 -i - -vcodec copy -f flv rtmp://rtsp-server:1935/
```
Additional info:
- The `ffmpeg` command is implied and is optional.
- The camera name will automatically be appended to the command, so you need to end with the rtmp/rtsp url.


## rtsp-simple-server
[rtsp-simple-server](https://github.com/aler9/rtsp-simple-server) options can be configured by editing `/app/rtsp-simple-server.yml`.

In particular, increasing **readBufferCount** seems to help if you are getting dropped frames from your camera.


## Debugging options

`- DEBUG_FFMPEG=True` Enable additional logging from FFmpeg

`- FRESH_DATA=True` Remove local cache and pull new data from wyze servers.

