A normal USB webcam does not work with the AGX Thor when using the normal Camera app. It has something to do with the gstreamer package not working with the CPU architecture of the AGX Thor and Jetson devices. However the webcams do work with the OpenCV package. The Depth Cameras work with their resepctive SDKs. If you just want to test a webcam's functionality, just install guvcview.

```bash
sudo apt update
sudo apt install guvcview
```
