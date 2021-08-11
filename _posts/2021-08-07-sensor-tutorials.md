---
layout: post
title: "VI-sensor Project: Accompanying Tutorials"
categories: tutorial
tags: vio slam sensor
---

> Here's the tutorials I put together for my VI-sensor project. I wrote them for myself several years ago so they were pretty minimal. I tried to flesh them out a bit more so maybe they can still be useful to others.

<!--more-->

- TOC
{:toc}

## Summary
While making the [VI-sensor]({{ site.baseurl }}{% post_url 2021-07-21-sensor %}), I ran through lots of different tutorials and tried lots of things, and I wrote down some of the steps so I wouldn't get lost later. They actually saved me a couple times when I had to start over one time. Several years later, even though some things have been obsoleted, maybe these refurbished tutorials will be helpful to someone starting on similar hardware or software.

As I experimented a lot, I didn't end up using all the tutorials for the final VI-sensor. The ones that were not necessary are marked with an asterisk. The tutorials are in a rough order. Some previous tutorials may need to be completed before starting a later one.

### Prerequisites
- **Tech**: A computer generally speaking, specific requirements mentioned in sub tutorials below.
- **Skills**: terminal (command prompt), light C++/Python/general programming, ROS, basic knowledge of hardware and software (what is an IMU for, what is Arduino).

Though I had some exposure to all the above, I strengthened a lot of skills, especially ROS, as I went along. I would say the core prereqs are some terminal and programming familiarity.

### Directory Tree

There are tons of files and workspaces all over the place. I didn't organize things that well, but for reference here's the hierarchy I had, with the home directory (~/) as root. Bolded files are important. It might help to check back here if I'm referring to some errant launch or config file.
- Arduino/Razor_AHRS/
- catkin_ws/src/
    - pointgrey_camera_driver/pointgrey_camera_driver/
        - launch/**camera.launch**
        - src/
    - razor_imu_9dof/
        - config/**my_razor.yaml**
        - launch/**razor-pub.launch**
        - nodes/
- kalibr_ws/
- maplab_ws/src/maplab/applications/rovioli/
    - scripts/**my_run_rovioli**
    - share/
        - **my_imu-sigmas-rovio.yaml**
        - **my_imu-sparkfun.yaml**
        - **my_ncamera.yaml**

---

## IMU Guide
I used the [9dof Razor IMU](https://learn.sparkfun.com/tutorials/9dof-razor-imu-m0-hookup-guide/all) from Sparkfun, which has been discontinued :( Specific version is M0 14001. Mostly follow the guide at the [ROS wiki](http://wiki.ros.org/razor_imu_9dof#Software_Installation), but with modifications which I'll list below:
- Before everything:
    - `$ sudo adduser <username> dialout`
    - logout, log back in
- On step 3, no breakout is required for this version of the board. Just a USB cable.
- For step 4, I used Used Arduino 1.8.6 (latest available at the time)
- I think I did step 4.1.1 instead of 4.1.2. I had to build from source.
- On step 4.2, follow above tutorial until opening Razor_AHRS in Arduino. Go to the [Sparkfun tutorial](sfe.io/t567) and follow from "Installing the 9DoF Razor Arduino Core" until the end of "Select the Board and Serial Port." Also, using the link at the beginning of the following section, download Sparkfun MPU-9250 DMP Library and install it (add it as Arduino library). Finally, upload code to the board. (Note: do NOT `include` the library in the code. Adding the library into Arduino IDE is enough.). Return to ROS wiki tutorial (skip the rest of 4.2).
- Edit the USER SETUP AREA in Razor_AHRS.ino, in the ~/Arduino directory! (NOT ~/catkin_ws)
- On step 4.3: Create config file (in ~/catkin_ws), rename the port in my_razor.yaml to whatever the port is called in Arduino IDE (eg /dev/ttyACM0)
- Step 6.2 will run a visualization demo. Prior to running the given command, run `$ source ~/catkin_ws/devel/setup.bash`
- On step 7.1.2, don't adjust the gyro (all ~0)
- I only did hard iron correction in 7.1.3, using the Arduino serial monitor (then in my_razor.yaml, setting calibration_magn_use_extended parameter to false). Add changes to Razor_AHRS. These values must also be placed in the my_razor.yaml file in the catkin workspace and sourced. The razor-launch-and-display.launch (and possibly other launch files in the launch directory) all take my_razor.yaml as a parameter, overriding the values we wrote in Arduino and flashed onto the IMU board (yes, dumb).

That should be the end of the ros wiki tutorial. The polling rate is probably still low, so go back in Razor_AHRS.ino, and change `OUTPUT__DATA_INTERVAL` to 10 (to increase the rate to 100 Hz), and change the `read_sensors` function as follows:

```cpp
static uint32_t lastDisplayMs = 0;
void read_sensors() {
#if HW__VERSION_CODE == 14001
    uint32_t currMs = millis();
    if (currMs > lastDisplayMs) {
        lastDisplayMs = currMs;
        loop_imu();
    }
    //...rest of function omitted
}
```

Reflash, and now if you launch razor-pub.launch, then in another terminal window run `$ rostopic hz imu`, you will see average rate: 100 Hz!

Extra notes:
- Edit+flash the Razor_AHRS.ino file from the Arduino directory, NOT the Razor_AHRS.ino file in the catkin workspace! (Note from 2021: I'm not sure why this is necessary...) However, the my_razor.yaml is configured in the catkin workspace and sourced.
- The "Using the MPU-9250 DMP Arduino Library" section of the [tutorial](sfe.io/t567) is useful for understanding some of the Arduino code, particularly Sensors.ino

Here's a sketch of the important sections of the razor_9dof code:
```
main()
    read_sensors()
        Sensors.loop_imu() //get data
        hardware trigger //every 33ms
    else if (output_mode == OUTPUT_MODE_ANGLES_AG_SENSORS)
        Sensors.output_both_angles_and_sensors_text() //Logging to serial
```

---

## Generic USB Camera Guide\*
The first camera I tried was a GoPro camera. I thought it would at least be a step up from a webcam, but I was wrong. My VI-sensor was just the IMU taped on top of the GoPro...it did not work for several reasons:
- not global shutter
- resolution was too large making the algorithm run too slowly
- IMU would wobble around making calibration useless

But I'll leave this tutorial here anyways.

### Requirements
The GoPro requires extra hocus pocus like an external capture card. It's possible to just use a webcam.
- GoPro 2018
    - double ended type-A USB 3.0 cable
    - HDMI to micro HDMI cable
    - HDMI to USB converter/capture card (high end / low latency) (USB 3.0 allows for 1080p 60fps I think?)
- Or just a webcam

### Steps
Turn on the GoPro. Set to 1080p and 30fps. Plug in the hdmi cable <-> capture card <-> usb cable <-> laptop usb port. The gopro screen should go black.

Run `$ sudo apt-get install ros-kinetic-usb-cam`

Create folder(s) called usb_camera/launch in your catkin workspace (just to have a place to put the usb_cam-related launch files). Create a launch file with the usbcam and optionally an image display to view the live video on the computer. The value for the video_device param can be found with the following terminal commands:

```
$ sudo apt-get install v4l-utils
$ v4l2-ctl --list-devices
```

Note: v4l-utils may already be installed

Note: The GoPro may automatically shut off (another reason why a GoPro was a bad idea) so turn it back on as necessary. 

Now run `$ roslaunch </path/to/usb_launchfile>`. Boom done.

Notes:
- USB C cable must be disconnected from GoPro
- If image is glitched somehow, more often than not it's one of the wired connections. Usually just jiggle the cables a bit.

Run `$ rostopic hz <cam topic>` in a separate terminal and notice that the fps is low. This is likely due to autoexposure. I tried several settings:
1. Autoexposure on (default)
2. Turn off autoexposure on GoPro
3. Turn off autoexposure on GoPro and in launch file

In terms of fps: 3 > 2 > 1. To achieve (2) or (3), set add new autoexposure param and set to false in launch file. Also use image_raw not image_compressed.

---

## FLIR Camera Guide (flycap sdk & ros driver)
I used a Firefly MV monochrome global shutter USB2.0 camera from FLIR (Point Grey). I'm pretty sure it's been discontinued and it's probably for the best. I think the closest thing FLIR offers now is the [FireFly S](https://www.flir.com/products/firefly-s/).

This guide uses FlyCapture2 sdk for FLIR Firewire and USB2 cams! For USB3, GigE, and 10GigE cams, use Spinnaker sdk and the [ros driver](https://github.com/ros-drivers/flir_camera_driver) instead. I will refer to this URL as FDP (flir downloads page): www.ptgrey.com/support/downloads/

I used Firefly MV monochrome, amd64, Ubuntu 16 (FLIR does not offer flycap sdk for Ubuntu 14 and below)

### Background Info
Go to the [point grey camera ros drivers](https://github.com/ros-drivers/pointgrey_camera_driver)
Navigate to pointgrey_camera_driver/cmake/download_flycap

Find something like below:

```python
'x86_64': {
    'current': (
        'https://www.ptgrey.com/support/downloads/10767/', (
            'flycapture2-2.11.3.121-amd64/libflycapture-2.11.3.121_amd64.deb',
            'flycapture2-2.11.3.121-amd64/libflycapture-2.11.3.121_amd64-dev.deb'),
        'usr/lib/libflycapture.so.2.11.3.121'),
```

As you can see, the flycap sdk version used is flycapture2-2.11.3.121. However, in this very version, FFMV/FMVU cameras cannot start!! (according to release notes; available on FDP) This bug is fixed in 2.11.3-164 and above.

{% include callout.html content="Looking at the repo in 2021, it's possible they fixed this with commit e27f3a0." icon="neutral" %}

### Install

Go to FDP. Download latest sdk: 2.13.3-31 for me. Follow instructions in README.txt to install.
```
$ cd ~/catkin_ws/src
$ git clone https://github.com/ros-drivers/pointgrey_camera_driver
```
Open pointgrey_camera_driver/cmake/download_flycap and find the same block as above. Edit like so:

```python
'x86_64': {
    'current': (
        'https://www.ptgrey.com/support/downloads/AAAAA/', (
            'flycapture2-BBBBB-amd64/libflycapture-BBBBB_amd64.deb',
            'flycapture2-BBBBB-amd64/libflycapture-BBBBB_amd64-dev.deb'),
        'usr/lib/libflycapture.so.BBBBB'),
```

Where AAAAA is the link to the file on the FDP (go to FDP hover over sdk link, right click, copy link address), and BBBBB is version number. So mine looks like this:

```python
'x86_64': {
    'current': (
        'https://www.ptgrey.com/support/downloads/11176/', (
            'flycapture2-2.13.3.31-amd64/libflycapture-2.13.3.31_amd64.deb',
            'flycapture2-2.13.3.31-amd64/libflycapture-2.13.3.31_amd64-dev.deb'),
        'usr/lib/libflycapture.so.2.13.3.31'),
```

Run:
```
$ cd ~/catkin_ws
$ catkin_make
```

Due to the version bs, you cannot just `$ sudo apt-get install ros-kinetic-ptgrey-camera-drivers`; you must build from (modified) source.

If you mess up, run the uninstall script located in the sdk folder downloaded from FDP, and remove the ros package (depending whether you built from source or downloaded binary: either remove cloned repo and `catkin clean`, OR `$ sudo apt-get remove ros-kinetic-blabla` and maybe deal with orphaned packages).

### Running

Tada! Now plug in the camera and type:
`$ flycap`

Select your camera and press ok to view a video feed! Press the gear icon to adjust imaging settings. Changing settings in ROS instead of Flycap can be done with dynamic reconfigure; see tutorial for that further down. Some settings in dynamic reconfigure, however, seem to be bugged (afaik)! See the comment at the end of the dynamic reconfigure tutorial.

BIG NOTE: If you are already publishing on camera/image_raw, then you will not see a video feed if you open flycap as well. However, changing the settings from flycap will still take effect.

Note: To view with img_view, use `$ rosrun image_view image_view image:=camera/image_raw`. I recall this not working before but now it does and I'm accepting it.

Note: I cannot get this to work with kalibr validator.

---

## Camera Calibration Given Photo Collection\*
Before finding kalibr I think I used this method to calibrate my camera.

Mostly follow [this guide](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_calib3d/py_calibration/py_calibration.html#calibration)

{% include callout.html content="That link is dead. This should be the same guide: https://docs.opencv.org/4.5.2/dc/dbb/tutorial_py_calibration.html" icon="neutral" %}

Print a 9x6 checkerboard pattern on any size paper (Letter/A4/etc works). Paste/tape paper flatly onto a flat surface. (emphasis on flat)

Here's the code to get calibration params.

```python
import numpy as np
import cv2
import glob
# import pdb; pdb.set_trace()

#Protip: python reprojection_error.py > error.txt
#Explanation: https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_calib3d/py_calibration/py_calibration.html#calibration

PATH = "/Users/user/Documents/Prog/temp/"

#For displaying large images
cv2.namedWindow("output", cv2.WINDOW_NORMAL)

# termination criteria
criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 30, 0.001)

# prepare object points, like (0,0,0), (1,0,0), (2,0,0) ....,(6,5,0)
objp = np.zeros((6*9,3), np.float32)
objp[:,:2] = np.mgrid[0:9,0:6].T.reshape(-1,2)

# Arrays to store object points and image points from all the images.
objpoints = [] # 3d point in real world space
imgpoints = [] # 2d points in image plane.

images = glob.glob('*.JPG')

print("Reading %d images from %s" % (len(images), PATH))
for fname in images:
    img = cv2.imread(PATH + fname)
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)

    # Find the chess board corners
    ret, corners = cv2.findChessboardCorners(gray, (9,6), None)

    # If found, add object points, image points (after refining them)
    if ret == True:
        objpoints.append(objp)

        corners2 = cv2.cornerSubPix(gray,corners,(11,11),(-1,-1),criteria)
        imgpoints.append(corners2)

        # Draw and display the corners
        img = cv2.drawChessboardCorners(img, (9,6), corners2,ret)

        imS = cv2.resize(img, (720, 480))
        cv2.imshow("img", imS)
        cv2.waitKey(500)

cv2.destroyAllWindows()

ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1],None,None)

print("Projection (Intrinsic)")
print(mtx)
print("Distortion (Extrinsic)")
print(dist)

img = cv2.imread(PATH + "GOPR0003.JPG")
h,  w = img.shape[:2]
newcameramtx, roi = cv2.getOptimalNewCameraMatrix(mtx,dist,(w,h),1,(w,h))

# undistort
dst = cv2.undistort(img, mtx, dist, None, newcameramtx)

# crop the image
x,y,w,h = roi
dst = dst[y:y+h, x:x+w]
cv2.imwrite(PATH + 'calibresult.png', dst)

tot_error = 0
for i in range(len(objpoints)):
    imgpoints2, _ = cv2.projectPoints(objpoints[i], rvecs[i], tvecs[i], mtx, dist)
    error = cv2.norm(imgpoints[i],imgpoints2, cv2.NORM_L2)/len(imgpoints2)
    tot_error += error

print("mean reprojection error: ", tot_error/len(objpoints))
```

Changes from the code in the tutorial:
- Code assumes (9,6) pattern (from link above).
- Prints relevant matrices and error
- Fixes reprojection error calculation
- Better image display for very large images (exceeding screen size)

It should output a calibresult.jpg, which is one undistorted image (the image name must be specified in code).

Add calibration results to my_config.yaml.

---

## Wires

In order to set up the triggering, we need to stably mount the camera and IMU and connect them. I 3D-printed a holder which I included in the middle of the [project summary]({{ site.baseurl }}{% post_url 2021-07-21-sensor %}). The design is weird because I had other plans for it and also because, well, that's the best I could come up with. The correct wiring can be found on appropriate data sheets.

---

## Triggering Camera From IMU

The most important tutorial! Fun fact I couldn't find this one so I looked back at the old code (which I had luckily) and whipped up a 2021 version just for this post. There may be errors or omissions though.

I basically worked off another tutorial which I'll call [grau](https://grauonline.de/wordpress/?page_id=1951). There are two important differences. First, I swapped the bluefox2 library with the pointgrey library, just because I used a different camera. My IMU was different as well, incorporating Arduino onboard, so I didn't need mpu6050_serial_to_imu. Those two differences require me to explain the steps separately from grau, unlike the IMU tutorial, but I still recommend his tutorial for background information.

Here are the main steps:
- Before everything, use Flycap to set parameters on camera
- Output trigger data from IMU to serial
- Publish IMU and trigger data over ROS
- Receive trigger messages with pointgrey camera driver
- Send images over ROS

ROS is the bridge between the IMU, camera, and the PC. Trigger data is only used to sync the IMU and camera. At the end, the PC should see synced IMU and camera messages over ROS, which will be fed into the SLAM algorithm.

### Flycap

Follow the Flycap tutorial above. In the Flycap GUI, there should be settings for enabling trigger and disabling autoexposure.

### IMU Serial

Assuming the camera and imu are wired together, first revisit Arduino/Razor_AHRS/Razor_AHRS.ino from the IMU tutorial above, and include camera triggering every 33ms. Note that the camera has a minimum triggering frequency. See the data sheet for details.

```cpp
static unsigned long lastDisplayMs = 0;
static unsigned long triggerMs = 0;
static unsigned long triggerCounter = 0;
void read_sensors() {
#if HW__VERSION_CODE == 14001
  unsigned long currMs = millis();
  if (currMs > lastDisplayMs) {
    lastDisplayMs = currMs;
    loop_imu();
    //raise 33 for lower fps
    if (currMs - triggerMs > 33) {
      digitalWrite(10, LOW);
      digitalWrite(10, HIGH);
      triggerCounter++;
      triggerMs = currMs;
    }
  }
#else
  Read_Gyro(); // Read gyroscope
  Read_Accel(); // Read accelerometer
  Read_Magn(); // Read magnetometer
#endif // HW__VERSION_CODE
}
```

Add the following to Arduino/Razor_AHRS/Output.ino *at the end* of `output_both_angles_and_sensors_text()`, in order to log necessary trigger data.

```cpp
LOG_PORT.print(triggerCounter); LOG_PORT.print(",");
LOG_PORT.print(triggerMs); LOG_PORT.println();
```

Remember if you change the Arduino code you have to reflash the IMU.

### IMU ROS

In razor_imu_9dof/nodes/imu_node.py, publish [TimeReference](http://docs.ros.org/en/melodic/api/sensor_msgs/html/msg/TimeReference.html) messages. This replaces the mpu6050_serial_to_imu from grau.

```cpp
/// after init_node()
pub_trigger = rospy.Publisher('imu/trigger_time', TimeReference, queue_size=50)

imuMsg = Imu()
triggerMsg = TimeReference()

...

/// in the while loop
triggerCounter = float(words[9])
if (triggerCounter > lastTriggerCounter):
    triggerMsg.header.seq = triggerCounter
    triggerMsg.header.stamp = rospy.Time(int(words[10])/1000, (int(words[10])%1000)*1000*1000)
    triggerMsg.time_ref = rospy.Time(0,0)
    pub_trigger.publish(triggerMsg)

    lastTriggerCounter = triggerCounter
```

The serial output is read line by line with `line = ser.readline()`, and split by comma into `words`. `words[9]`, is `triggerCounter` and `words[10]`, is `triggerMs`.

### Camera ROS (Receive Trigger)

Add a ring buffer to pointgrey_camera_driver/pointgrey_camera_driver/src/nodelet.cpp. The idea is if the write head is running laps around the read head then it's out of sync.

```cpp
void callback(const sensor_msgs::TimeReference::ConstPtr &time_ref) {
  //if ( (time_ref->header.seq & 63) == 0){
  //  ROS_WARN("recv triggertime seq %10u", time_ref->header.seq);
  //}
  //  ros::Duration(0.001).sleep();
  TriggerPacket_t pkt;
  pkt.triggerTime = time_ref->header.stamp;
  pkt.triggerCounter = time_ref->header.seq;
  fifoWrite(pkt);
}

void fifoWrite(TriggerPacket_t pkt){
  fifo[fifoWritePos]=pkt;
  fifoWritePos = (fifoWritePos + 1) % FIFO_SIZE;
  if (fifoWritePos == fifoReadPos){
    ROS_WARN("FIFO overflow!");
  }
}

bool fifoRead(TriggerPacket_t &pkt){
  if (fifoReadPos == fifoWritePos) return false;
  pkt = fifo[fifoReadPos];
  fifoReadPos = (fifoReadPos + 1) % FIFO_SIZE;
  return true;
}

bool fifoLook(TriggerPacket_t &pkt){
  if (fifoReadPos == fifoWritePos) return false;
  pkt = fifo[fifoReadPos];
  return true;
}
```

### Camera ROS (Publish Image)

Move down to the try block of the `STARTED` case and create a new case to handle the camera triggering mode. Now the camera will only take and publish frames when it receives the trigger signal. Code again heavily from grau. 

```cpp
/// this case already existed -- no triggering
if (triggerMode != 0) {
  // Publish the full message
  pub_->publish(wfov_image);

  // Publish the message using standard image transport
  if(it_pub_.getNumSubscribers() > 0)
  {
    sensor_msgs::ImagePtr image(new sensor_msgs::Image(wfov_image->image));
    it_pub_.publish(image, ci_);
  }
}

/// new case for triggering
else {
  // wait for new trigger packet to receive
  TriggerPacket_t pkt;
  while (!fifoLook(pkt)) {
    // ros::Duration(0.001).sleep();
  }

  // a new video frame was captured - check if we need to skip it if one trigger packet was lost
  if (pkt.triggerCounter == nextTriggerCounter) {
    fifoRead(pkt);
    // uint shutter = pg_.getShutter();
    const ros::Duration expose_duration = ros::Duration(SHUTTER * 1e-6 / 2);
    wfov_image->header.stamp = pkt.triggerTime + expose_duration;
    ci_->header.stamp = wfov_image->image.header.stamp;
    wfov_image->info = *ci_;

    // Publish the full message
    pub_->publish(wfov_image);

    // Publish the message using standard image transport
    if(it_pub_.getNumSubscribers() > 0)
    {
      sensor_msgs::ImagePtr image(new sensor_msgs::Image(wfov_image->image));
      it_pub_.publish(image, ci_);
    }

  } else {
    //ros::Duration(0.001).sleep();
    outOfSyncCounter++;      
    if ((outOfSyncCounter % 100) == 0){
      //ROS_WARN("trigger not in sync (seq expected %10u, got %10u)!", nextTriggerCounter, pkt.triggerCounter);  
      NODELET_WARN("trigger not in sync (%d)!", outOfSyncCounter);   
    }
  }
  nextTriggerCounter++;
  // ros::spin();
}
```

### Wrap Up

To run, launch the camera and IMU in separate terminals. There may have been a need to start one before the other but I forgot.

---

## Camera Calibration with Kalibr
Kalibr can calibrate cameras, as well as imu-camera interaction. I can't remember for sure if it requires imu-camera synchronization...

Build from source! The other option has limited capabilities.

Some commands (e.g. kalibr_camera_validator) fail (infuriatingly) on Ubuntu 16. They must be run on Ubuntu 14. This is an obstacle for validating with FLIR firefly camera.

Follow the youtube tutorial on the [wiki](https://github.com/ethz-asl/kalibr/wiki). You have to calibrate imu parameters outside of Kalibr before doing imu-camera calibration.

### Record rosbag
- Plug in imu and camera
- Source the kalibr workspace
- Run razor_9dof_imu/launch/razor-pub.launch and pointgrey_camera_driver/launch/camera.launch in separate windows
- In another window, run `$ rosbag record -O output.bag camera/image_raw imu`.
- Follow instructions in video to correctly move cam+imu for good calibration. Run for ~60 seconds.

To review the rosbag, close the imu and image streams, and run the 2 lines in different terminals:
```
$ roscore
$ rosbag play output.bag
```

The video can then be viewed with img_view.

### Calibrate Cameras

First, calibrate camera(s):
`$ kalibr_calibrate_cameras --models pinhole-radtan --topics camera/image_raw --bag output.bag --target april_6x6_80cm.yaml`

For more info on the `--models` flag see [supported models](https://github.com/ethz-asl/kalibr/wiki/supported-models). The format is the camera model abbreviated name and distortion model abbreviated name separated by a dash (e.g. pinhole-radtan). Topic must NOT have preceding slash for whatever reason (e.g. imu not /imu). Target is recommended to be some kind of april grid, since it's more robust.

### Calibrate Camera+IMU

Then get camera+imu calibration:
`$ kalibr_calibrate_imu_camera --cam camchain-output.yaml --imu imu.yaml --bag output.bag --target april_6x6_80cm.yaml`

Cam flag is output of kalibr_calibrate_cameras. Imu must be calibrated externally and imu.yaml must be filled in appropriately.

### Using In Maplab

To convert kalibr yaml to maplab yaml (ncamera_calibration file), use the following command: 
```
$ kalibr_maplab_config --to-ncamera \
--label cam_name \
--cam camchain-imucam-output.yaml
```

Label flag is arbitrary. Cam flag is the result of cam-imu calibration (output of kalibr_calibrate_imu_camera command).

---

## ROVIOLI + maplab

### Background
I'm using the maplab framework. ROVIO exists as a separate repository if desired. See [maplab github](https://github.com/ethz-asl/maplab), [maplab wiki](https://github.com/ethz-asl/maplab/wiki), and [install instructions](https://github.com/ethz-asl/maplab/wiki/Installation-Ubuntu).

Maplab is a VI mapping framework, which supports operations such as map merging, visual-inertial batch optimization, and loop closure. ROVIOLI, which is based on an estimator called ROVIO, is a VI mapping front end for maplab, with added maplab modules for map building and localization. maplab can just be used with ROVIOLI as a ready-to-go (and quite good) VI mapping/localization system, but other estimators can be integrated as well.

### Build Map

You need three calibration files (see [wiki](https://github.com/ethz-asl/maplab/wiki/Sensor-Calibration-Format)). Put calibration files into maplab/applications/rovioli/share.

Be sure to convert kalibr output to maplab cam format (see [wiki](https://github.com/ethz-asl/maplab/wiki/Initial-sensor-calibration-with-Kalibr), or the last note in the previous section).

Follow [this tutorial](https://github.com/ethz-asl/maplab/wiki/Running-ROVIOLI-in-VIO-mode) to build a map from a rosbag or rostopic. Instead of a launch file we use a bash script to run everything. Some examples are in [maplab/applications/rovioli/scripts](https://github.com/ethz-asl/maplab/tree/master/applications/rovioli/scripts).

### Refine Calibration

You can optionally refine the calibration, only after obtaining a decent map (see [wiki](https://github.com/ethz-asl/maplab/wiki/sensor-calibration-refinement)).

### Visualization

To visualize with rviz, download [this rviz config](https://github.com/ethz-asl/maplab/blob/pre_release_public/july-2018/applications/rovioli/share/rviz-rovioli.rviz). Then run `$ rosrun rviz rviz -d rviz-rovioli.rviz`. Of course, you will need to set all the appropriate visualization flags in the rovioli launch script. Examples of these can be found in the same link above, for building a map from rosbag/rostopic. An overview of viz rostopics can be found [here](https://github.com/ethz-asl/maplab/wiki/Map-visualization).

In standalone ROVIO, some core parameters could be set at compile time with flags, but in ROVIOLI, these ROVIO parameters unfortunately require editing the code directly.

### Relocalization

Something to note about the map built by ROVIOLI. In ROVIO, the mapping part of SLAM is achieved with photometric features (visualized as green rectangles), whose count must be carefully managed. While these features are robust for local state estimation, they are not efficient for building maps suitable for loop closure, state estimation, etc., so in ROVIOLI, a separate module is run in parallel to grab landmarks better suited for these tasks. These are finalized at shutdown, meaning mapping is not done in real time, and online localization from this map is impossible.

The map will be saved to the location specified with `--save_map_folder`.

This map should be optimized for use in relocalization. There are two ways to do this:
- launch script flag: `--optimize_map_to_localization_map`
- [maplab commands](https://github.com/ethz-asl/maplab/wiki/Preparing-a-single-session-map)

A smaller "summary map" can be generated from this, using this command: `$ generate_summary_map_and_save_to_disk --summary_map_save_path path/to/save/localization/summary/map`

The next time you run ROVIOLI with relocalization, use the `--vio_localization_map_folder` flag.

Basic map manipulation can be done in the [maplab console](https://github.com/ethz-asl/maplab/wiki/Console-map-management).

---

## Point cloud and mesh visualization\*

no rviz, thank you

pcd viz:
```
$ sudo apt install pcl-tools
$ pcl_viewer <pointcloud.pcd>
```

mesh viz:
```
$ sudo apt-get install meshlab
$ meshlab
```

open obj file in meshlab gui

---

## Some Fixes For Dynamic Reconfigure Service
Assuming everything is setup in the code, run the node, then run `$ rosrun rqt_reconfigure rqt_reconfigure`

Specific to the pointgrey_camera_driver, the default params are set in pointgrey_camera_driver/cfg/PointGrey.cfg

Some parameters in the config seem to be bugged (afaik). So far I have found this true for frame rate and brightness. So go in PointGreyCamera.cpp, into `setNewConfiguration()`, and comment out the relevant calls to `setProperty()`.

To update the config (and of course, if you make changes to the driver files, e.g. PointGreyCamera.cpp), you must recompile them, i.e. rerun `catkin_make`.

---

## Pose and Image to Unity

I'm sorry to disappoint, but I can't seem to find the tutorial or the code for this part. All I have are the last couple of entries in my dev notes.

> --/--/-- Decided ros sharp was overkill because all of the URDF stuff. Opted for ROSBridgeLib, which is just client implementation for Unity of all (most) ros messsages. It sucks and no working tutorial provided, and no active development, and simple data values are behind accessors, and everything has to be static?!?!?! Switch back to ros sharp, but have "Duplicate Assembly" errors, even after restarting in new project. Discovered I was dumb and copied the wrong folder. Unity client successfully connects to rosbridgeserver, but images are not updated in Unity.
> 
> --/--/-- Discovered first 12 pixels in image data were bogus; turns out it is embedded info from camera. So no problem with ros side. On Unity side, texture and pose weren't being updated because turns out can't do stuff with textures outside render thread (ros sharp creates new thread). So moved rendering to IEnumerator and used UnityMainThreadDispatcher. Also, using LoadImage() on compressed image rostopic is muuuuch slower than LoadRawTextureData() on regular image topic. Result is image stream shows! Some opacity issue. Deep profile helps a ton. Position works! Rotation works! Turns out need to create new Position and Rotation and assign, rather than editing the values directly (?). At least for pose, could maybe get away with less frequent position updates + smoothing/lerping. 