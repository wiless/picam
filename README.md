# picam
Experimental project for building a desktop webcam using raspberry pi with a camera module


# Reference Tools

1. See how V4L2Loop is created (from droidcam) https://github.com/aramg/droidcam/tree/master/linux 
2. Wiki on v4l2loop devices are created and loaded https://github.com/umlaeute/v4l2loopback
3. Examples on writing to V4L2loop device using FFmpeg https://github.com/umlaeute/v4l2loopback/wiki/FFmpeg
 
## Using the Raspberry PI camera as a network webcam
Your boss wants you in a webex but you have no camera available, but a raspberrypi + camera module (CSI)? No problem. Build up a network webcam in a minute. With ffmpeg + v4l2loopback.

  Bring the raspberrypi camera on track (Camera server)
  To stream the camera data (raw h264 stream, 1280x720 and 1MBit/s) over network just use the raspivid command:

  raspivid -t 0 -hf -fps 30 -w 1280 -h 720 -b 1000000 -l -o tcp://0.0.0.0:5000
  So, a tcp server is now ready to deliver our data. If you do not want to limit the h264 data bandwidth, just omit the -b parameter.

  Bring up ffmpeg with v4l2loopback (Client which receives video feed)
  v4l2loopback config
  First, you need to get your /dev/video0 device up which is accepted by firefox:

  modprobe v4l2loopback devices=1 max_buffers=2 exclusive_caps=1 card_label="VirtualCam #0"
  Be sure to run this command as root. Otherwise no kernel module can be loaded.

  ffmpeg stream start
  Now comes the final step. We will receive the stream from network and writing it into the /dev/video0 device:

  ffmpeg -f h264 -i tcp://RASPICAM:5000 -f v4l2 -pix_fmt yuv420p /dev/video0
  Set RASPICAM to your hostname/ip of the raspberrypi with the camera.

  BONUS: Let your graphics card do the hard work (NVIDIA with VDPAU)
  To offload h264 graphics decoding to your graphics card via VDPAU you can do this:

  ffmpeg -f h264 -hwaccel vdpau -i tcp://goldeneye:5000 -f v4l2 -pix_fmt yuv420p /dev/video0
  Your graphics card will now do the hard work. Check in top/htop if you have less load on your ffmpeg task.

## Instead of Network based
- Use USB tethering on linux (https://raspberrypi.stackexchange.com/questions/55928/ssh-the-pi-from-computer-with-a-usb-cable-only) 
- https://gist.github.com/gbaman/975e2db164b3ca2b51ae11e45e8fd40a#gistcomment-3077298 
