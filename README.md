<div align="center">
  <p>
    <a align="center" href="" target="_blank">
      <img
        width="850"
        src="https://github.com/myselfbasil/Guide-to-Run-BodyPose3D-Deepstream-v7.0-Model/blob/034c29c2a8cc384b64f4dc02c92df09527bdce0e/assets/header_img.png"
      >
    </a>
  </p>
</div>

# Guide to Run BodyPose3D Deepstream v7.0 Model

Reference Docs:

https://github.com/NVIDIA-AI-IOT/deepstream_reference_apps/tree/master/deepstream-bodypose-3d

### Installation

Preferably clone the repo in /opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/ and define project home as:

```bash
export BODYPOSE3D_HOME=<parent-path>/deepstream-bodypose-3d.
```

Install NGC CLI from : https://org.ngc.nvidia.com/setup/installers/cli

 and download PeopleNet from: https://catalog.ngc.nvidia.com/orgs/nvidia/teams/tao/models/peoplenet

and BodyPose3DNet from : https://catalog.ngc.nvidia.com/orgs/nvidia/teams/tao/models/bodypose3dnet

```bash
$ mkdir -p $BODYPOSE3D_HOME/models
$ cd $BODYPOSE3D_HOME/models
# Download PeopleNet
$ ngc registry model download-version "nvidia/tao/peoplenet:deployable_quantized_v2.5"
# Download BodyPose3DNet
$ ngc registry model download-version "nvidia/tao/bodypose3dnet:deployable_accuracy_v1.0"
```

### Download BodyPose3DNet

By now the directory tree should look like this:

```bash
$ tree $BODYPOSE3D_HOME -d
$BODYPOSE3D_HOME
├── configs
├── models
│   ├── bodypose3dnet_vdeployable_accuracy_v1.0
│   └── peoplenet_vdeployable_quantized_v2.5
├── sources
│   ├── deepstream-sdk
│   └── nvdsinfer_custom_impl_BodyPose3DNet
└── streams
```

Download and extract Eigen 3.4.0 under the project foler.

```bash
$ cd $BODYPOSE3D_HOME
$ wget https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.tar.gz
$ tar xvzf eigen-3.4.0.tar.gz
$ ln eigen-3.4.0 eigen -s
```

For Deepstream SDK version older than 6.2, copy and build custom NvDsEventMsgMeta into Deepstream SDK installation path. Copy and build custom NvDsEventMsgMeta into Deepstream SDK installation path. The custom NvDsEventMsgMeta structure handles pose3d and pose25d meta data.

```bash
# Copy deepstream sources
cp $BODYPOSE3D_HOME/sources/deepstream-sdk/eventmsg_payload.cpp /opt/nvidia/deepstream/deepstream/sources/libs/nvmsgconv/deepstream_schema
# Build new nvmsgconv library for custom Product metadata
cd /opt/nvidia/deepstream/deepstream/sources/libs/nvmsgconv
make; make install
```

Note: that this step is not necessary for Deepstream SDK version 6.2 or newer.

### Build the applications

```bash
# Build custom nvinfer parser of BodyPose3DNet
cd $BODYPOSE3D_HOME/sources/nvdsinfer_custom_impl_BodyPose3DNet
make
# Build deepstream-pose-estimation-app
cd $BODYPOSE3D_HOME/sources
make
```

If the above steps are successful, deepstream-pose-estimation-app shall be built in the same directory. Under $BODYPOSE3D_HOME/sources/nvdsinfer_custom_impl_BodyPose3DNet, libnvdsinfer_custom_impl_BodyPose3DNet.so should be present as well.

Run the Application:

```bash
$ ./deepstream-pose-estimation-app -h
Usage:
  deepstream-pose-estimation-app [OPTION?] Deepstream BodyPose3DNet App

Help Options:
  -h, --help                        Show help options
  --help-all                        Show all help options
  --help-gst                        Show GStreamer Options

Application Options:
  -v, --version                     Print DeepStreamSDK version.
  --version-all                     Print DeepStreamSDK and dependencies version.
  --input                           [Required] Input video address in URI format by starting with "rtsp://" or "file://".
  --output                          Output video address. Either "rtsp://" or a file path is acceptable. If the value is "rtsp://", then the result video is published at "rtsp://localhost:8554/ds-test".
  --save-pose                       The file path to save both the pose25d and the recovered pose3d in JSON format.
  --conn-str                        Connection string for Gst-nvmsgbroker, e.g. <ip address>;<port>;<topic>.
  --publish-pose                    Specify the type of pose to publish. Acceptable value is either "pose3d" or "pose25d". If not specified, both "pose3d" and "pose25d" are published to the message broker.
  --tracker                         Specify the NvDCF tracker mode. The acceptable value is either "accuracy" or "perf". The default value is "accuracy".
  --fps                             Print FPS in the format of current_fps (averaged_fps).
  --width                           Input video width in pixels. The default value is 1280.
  --height                          Input video height in pixels. The default value is 720.
  --focal                           Camera focal length in millimeters. The default value is 800.79041.
```

Follow the commands:

```bash
$ cd /opt/nvidia/deepstream/deepstream-7.0/deepstream_reference_apps
  /deepstream-bodypose-3d/sources
$ ./deepstream-pose-estimation-app --input file:///opt/nvidia/deepstream
   /deepstream-7.0/deepstream_reference_apps/deepstream-bodypose-3d
   /streams/bodypose.mp4
```

Now you can verify that the model is running, change the input from a video file to an RTSP IP Cam

 There are a lot of applications, but the one I have used is :

[RTSP Camera - Apps on Google Play](https://play.google.com/store/apps/details?id=video.surveillance.rtsp.camera)

Install it and run it. With this app, You can even choose which camera of your phone(front/rear) to use.

Note: For this to work, what I did was to connect my phone to my wifi and eneble tethering to my pc or what you can do is like connect your laptop or PC to the same wifi as your mobile.

In the app you will get the rstp cam IP like: “rtsp://admin:admin@192.168.1.11:1935”

and follow this command:

```bash
$ ./deepstream-pose-estimation-app --input rtsp://admin:admin@192.168.1.11:1935
```

There you go! Now you can see the model running live on your IP cam feed.
---

**Made with 🫶🏻 by Basil**

You can view my docker hub documentation here: [hub.docker.com](https://hub.docker.com/repository/docker/basilshaji32/deepstream-bodypose3d/general)

Check out my medium guide here: [medium.com](https://medium.com/@basilshaji32/guide-to-run-bodypose3d-deepstream-v7-0-model-abafad03860e)

You can go through my notion website: [notion.com](https://basilshaji.notion.site/Bounding-Box-Image-Augmentation-Using-the-Albumentations-Library-for-the-KITTI-Dataset-60ccfe6b93db42c581519ae5044c30f5?pvs=4)
