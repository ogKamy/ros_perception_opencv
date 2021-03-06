# To Launch
See notes for various applications.

# Notes

One of the most useful perception skills is being able to recognize objects. This allows you to create robots that can, for instance, grasp objects or understand the world around them a little bit better.
There are two main elements:

- **Recognize flat surfaces**: This skill allows it to detect places where objects normally are, like tables and shelves.It's the first step to searching for objects.
- **Recognize objects**: Once you know where to look, you have to be able to recognise different object in the scene and localise where they are from your robot's location.

The first step in recognising objects is knowing where these objects are. For this, you are going to use a port of the **object_recognition_core** package to be able to detect flat surfaces and represent that detections in RViz.

The package uses a binary called detection with a configuration file as the argument. This file, called _detection.tabletop_fetch.ros.ork_, in the directory _conf_, is where all the input sensors and values for the table detection are set. It's a YAML file with a different extension, .ork.

Of all the parameters you have here, the only ones that are relevant most of the time are the following ones:

- rgb_frame_id: '/head_camera_rgb_optical_frame'
- rgb_image_topic: '/head_camera/rgb/image_raw'
- rgb_camera_info: '/head_camera/rgb/camera_info'
- depth_image_topic: '/head_camera/depth_registered/image_raw'
- depth_camera_info: '/head_camera/depth_registered/camera_info'

This sets the correct image topics as inputs so that the recognition can be made

And now, open the RVIZ and add all of the elements you want to see (like the Camera element or PointCloud2 elements). To visualize the Table detection, you will have to add **OrkTable** element. You select the topic where the table data is published, in this case, **/table_array**. You can then check certain options, like "bounding_box" to have a bounding box around the detection, or the "top" option to see what is being considered as the top of the surface.

More info:
http://wg-perception.github.io/ork_tutorials/
http://wg-perception.github.io/ork_tutorials/tutorial02/tutorial.html
http://wiki.ros.org/tabletop_object_detector

## 2D and 3D Object Finder

See _start_find_object_2d.launch_.

This is a basic approach to detecting objects, although you can already differentiate between objects and localize them with **object_recognition_core**. For this, you will use **find-object-2d** package.

The first step is to get closer to the table to see the objects as closely as possible, for better detection.

### 2D Object Finder

You only have to set the RGB camera image source and the system is ready to go. In this case, it's **/head_camera/rgb/image_raw**.

To teach the Fetch robot to recognize a coke can., select the Edit->AddObjectFromScene. You can also add previously taken images directly. But, bear in mind that there are some peculiarities. The images appear in this objectrecogniser mirrored, if you compare them with the images from the cameras directly. So be careful with that.

So in the Adding Object menu, you just have to follow the steps to select a region of the image that you consider to be the object. Once done, you should be detecting the object. This system compares the image with the saved ones and looks for matches. If it matches in enough points, it considers it the desired object.

Now you can add as many objects as you want, or even turn the object around and take images from different points of view. This will make it detect the object all the time, but keep in mind that without the proper filtering, the system will consider them to be different objects.

So, the last step is to save all of the objects added. There are 2 main ways:

- Saving the Objects as images: File-->Save_Objects. This will save all of the images taken in a folder
- Saving the Whole session: File-->Save_Session. This will save a binary with all of the images and settings. This is the most compact way of doing it, although you won't have access to the images of the objects. It depends on your needs

You can also save the object images directly by right clicking on them, but **BEWARE** that if you do so, these images can't be used for later detection because they aren't mirrored, therefore, the recognition won't work. You can see an example of it here. It's the same image, but one is the manually saved version that is not mirrored and isn't detected, while the other one is correctly saved.

So, once you have your session or your images stored, you need to be able to always start an object recognition session with all of that stored data. To do so, you have the following options:

### For saved sessions in my_object_recognition_pkg, in the directory saved_pictures2d:

See _saved_session_2d.launch_.

### For saved images in my_object_recognition_pkg, in the directory saved_pictures2d:

See _saved_objects_2d.launch_.

More Info:
http://wiki.ros.org/find_object_2d
http://introlab.github.io/find-object/

### Move and Spawn objects

You have to test your object_recognition system with the same object in different positions, or even in movement.
You also need to test it with various objects in the scene to be sure that it doesn't mistake a human head with a coke.

To make an object move in a scene, there are 2 steps:

- **Make the object movable in the Gazebo:** For that, you will use the move_model.launch from our spawn_robot_tools_pkg.This makes a topic called "name_object/cmd_vel" available on which you can then publish and move the model around.
- **Publish in the correct topic:** To move the object through the keyboard

See _make_cokecan_movable.launch_ and _move_coke_keyboard.launch_.

To move other objects in the scene, just change the name of the model.
To know the name of that the model has in the Gazebo, you can ask the gazebo service:
`rosservice call /gazebo/get_world_properties`

### Spawn new objects in the scene

This method needs you to have the sdf files of the models on your package and the whole model installed in the .gazebo/models path. These models are in the _models_ directory of this package.

Execute the _spawn_coke.launch_ launch file to spawn the model where you wish, probably in the table location.
For reference, the coke_can is spawned in the center of the table at XYZ = (-2.0,0.0,0.8).
Bear in mind that if there is already an existing model with the same name, you won't be able to spawn it.

So, in conclusion, to spawn an object, for example the coke_can, and make it movable at the same time, you will have to execute _spawn_ready_coke.launch_.

## 3D Object Detection

The only real difference with the 2D detection will be the sensors involved and the fact that the ObjectPoseStamped will be transformed into TFs.

You have to create another round of session photos in the 3D system because, otherwise, the detections won't work as well as they should. Especially for the TF transformations.

See: *start_find_object_3d_session.launch*

Launching this, you should then get the TF of the object detected published. If you have multiple images of the same object,you will get multiple frames of objects. It's up to you to filter them.

You can also see the object detected by executing the following command in another terminal while the prior launch is working:

`rosrun find_object_2d print_objects_detected`

The TFs appearing can be lowered by decreasing the time you consider a TF obsolete. Because most of these TFs are from previous detections that stay there for a while until they are old enough to be considered irrelevant.