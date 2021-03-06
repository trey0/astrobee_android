# Android Science Camera Image 

This is a guest science android application that takes full-resolution
pictures with the science camera and publishes them on the

  /hw/cam_sci/compressed

topic via ROS. The image dimensions, etc, are published on 

  /hw/cam_sci_info

This app has a minimal GUI consisting of an image preview window. The
user should start and stop this application remotely via the guest
science manager and not by using its GUI.

## Building the app

It is expected that you will have read $ANDROID_PATH/build_essential_apks.md 
and $ANDROID_PATH/running_gs_app.md for background information.

Run on your development machine:

  cd $ANDROID_PATH/gs_examples/sci_cam_image
  ANDROID_HOME=$HOME/Android/Sdk ./gradlew assembleDebug

Copy the obtained APK to the LLP processor of the robot, for example, as:

  rsync -avzP app/build/outputs/apk/app-debug.apk bsharp-llp:sci_cam_image.apk

## Setting up some tools on LLP

Copy 

  $ANDROID_PATH/scripts/gs_manager.sh 

and

  freeflyer/tools/gds_helper/src/gds_simulator.py

to the home directory on LLP. Note that the last tool is not in the
android freeflyer repository, rather in the main freeflyer repository.


## Setting up ROS communication

This app assumes sets the ROS master URI as 

  http://llp:11311 

If that is not the case, the code must be edited to set the correct value
and rebuilt. You may need to also set the environmental variable
ROS_MASTER_URI to the same value in any shell that is used to do ROS
communication.

## Installing the sci_cam_image APK

Connect to LLP. Run:

  adb uninstall gov.nasa.arc.irg.astrobee.sci_cam_image 
  adb install -g sci_cam_image.apk

## Running this APK using the Guest Science Manager.

Connect to LLP in several terminals. In one, start the ROS nodes on the bot:

  roslaunch astrobee astrobee.launch mlp:=mlp llp:=llp

In a second session on LLP start the Guest Science Manager as:

  ./gs_manager.sh start

followed by starting the command-line GDS tool:

  python ./gds_simulator.py

and follow the prompts. This APK figures as SciCamImage in the
list. Starting it (press on 'b') will turn on publishing the science
camera images. Yet, no pictures will be taken. 

To take a picture, press on option 'd' to send a custom science
command, and then choose '1' to take a single picture. If desired to
turn on continuous picture taking, which will take and publish
pictures as fast as the camera will allow it (typically one per
second), send another custom science command, and this time choose the
value '2'. To turn off continuous picture taking follow the same
approach, but choose instead '3'. If continuous picture taking is on,
and the user chooses to take one picture only, continuous picture
taking will be stopped, and then just one picture will be taken.

To quit the guest science manager (after stopping SciCamImage and
exiting the simulator), do:

  ./gs_manager.sh stop

To see logging info as this app is running one can do (in a separate
terminal on LLP):

  adb logcat | grep -E -i "science|sci_cam"

## Running this APK in debug mode. 

To investigate any problems with this APK it can be run without the
Guest Science Manager, with logging on. For that, in one terminal on
LLP launch 'roscore', then in a second one run:

  adb logcat -b all -c   # wipe any existing logs
  adb logcat -s sci_cam  # print log messages for sci_cam_image

and in a third one run:

  adb shell am start -n gov.nasa.arc.irg.astrobee.sci_cam_image/gov.nasa.arc.irg.astrobee.sci_cam_image.SciCamImage

After this, one of the following self-explanatory commands can be sent
to the sci cam:

 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.TAKE_SINGLE_PICTURE
 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.TURN_ON_CONTINUOUS_PICTURE_TAKING
 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.TURN_OFF_CONTINUOUS_PICTURE_TAKING
 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.TURN_ON_LOGGING
 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.TURN_OFF_LOGGING
 adb shell am broadcast -a gov.nasa.arc.irg.astrobee.sci_cam_image.STOP

To see if any images are being published one can use rviz to display 
the image topic (see above) or just echo the camera info:

  rostopic echo /hw/cam_sci_info

In cases when the sci cam refuses to quit, the app should be
uninstalled (see above), which will force it to stop.
