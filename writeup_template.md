## Project: Perception Pick & Place

---


# Required Steps for a Passing Submission:
1. Extract features and train an SVM model on new objects (see `pick_list_*.yaml` in `/pr2_robot/config/` for the list of models you'll be trying to identify). 
2. Write a ROS node and subscribe to `/pr2/world/points` topic. This topic contains noisy point cloud data that you must work with.
3. Use filtering and RANSAC plane fitting to isolate the objects of interest from the rest of the scene.
4. Apply Euclidean clustering to create separate clusters for individual items.
5. Perform object recognition on these objects and assign them labels (markers in RViz).
6. Calculate the centroid (average in x, y and z) of the set of points belonging to that each object.
7. Create ROS messages containing the details of each object (name, pick_pose, etc.) and write these messages out to `.yaml` files, one for each of the 3 scenarios (`test1-3.world` in `/pr2_robot/worlds/`).  [See the example `output.yaml` for details on what the output should look like.](https://github.com/udacity/RoboND-Perception-Project/blob/master/pr2_robot/config/output.yaml)  
8. Submit a link to your GitHub repo for the project or the Python code for your perception pipeline and your output `.yaml` files (3 `.yaml` files, one for each test world).  You must have correctly identified 100% of objects from `pick_list_1.yaml` for `test1.world`, 80% of items from `pick_list_2.yaml` for `test2.world` and 75% of items from `pick_list_3.yaml` in `test3.world`.
9. Congratulations!  Your Done!

# Extra Challenges: Complete the Pick & Place
7. To create a collision map, publish a point cloud to the `/pr2/3d_map/points` topic and make sure you change the `point_cloud_topic` to `/pr2/3d_map/points` in `sensors.yaml` in the `/pr2_robot/config/` directory. This topic is read by Moveit!, which uses this point cloud input to generate a collision map, allowing the robot to plan its trajectory.  Keep in mind that later when you go to pick up an object, you must first remove it from this point cloud so it is removed from the collision map!
8. Rotate the robot to generate collision map of table sides. This can be accomplished by publishing joint angle value(in radians) to `/pr2/world_joint_controller/command`
9. Rotate the robot back to its original state.
10. Create a ROS Client for the “pick_place_routine” rosservice.  In the required steps above, you already created the messages you need to use this service. Checkout the [PickPlace.srv](https://github.com/udacity/RoboND-Perception-Project/tree/master/pr2_robot/srv) file to find out what arguments you must pass to this service.
11. If everything was done correctly, when you pass the appropriate messages to the `pick_place_routine` service, the selected arm will perform pick and place operation and display trajectory in the RViz window
12. Place all the objects from your pick list in their respective dropoff box and you have completed the challenge!
13. Looking for a bigger challenge?  Load up the `challenge.world` scenario and see if you can get your perception pipeline working there!

## [Rubric](https://review.udacity.com/#!/rubrics/1067/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Exercise 1, 2 and 3 pipeline implemented
#### 1. Complete Exercise 1 steps. Pipeline for filtering and RANSAC plane fitting implemented.
![ransac](http://i64.tinypic.com/2vtv21h.jpg)

**We need to implement a perception pipeline to identify an object in a cluttered tabletop environment, for that reason we need to filter our Point Clouds in order to remove useless and excessive or adverasial data**

* **Voxel Grid Downsampling: downsample our data by taking the average of the points in the cloud confined by a voxel**

* **Pass through Filter: If you have prior information about the location of your target in the scene, you can "crop" the PC**

* **next we want to perform RANSAC plane fitting segmentation, to divide our PC into meaningful clusters (here we will remove the table**


* **RANSAC: Random Sample Concensus: identifies points in the PC based on a particular model (plane, cylinder etc.) and puts the associate data in "inliers" and the other on "outliers" but you need prior knowledge of the shape of the object you need to remove**

* **Outlier removal: due to various reasons our PC may contain outliers, that can lead to complications in the estimation of PC characteristics, so we use a statistical analysis in the neighborhood of each point, and remove certain points**

#### 2. Complete Exercise 2 steps: Pipeline including clustering for segmentation implemented.  
**Segment objects in our PC without having to assume a model shape,but based on particular features (color, position, texture..)**

* **DBSCAN is used instead of k-means when there are an unknown number of clusters in your data but you know some characteristics of the clusters like distance, also good when there are outliers present, and no need for termination definition.**

* **We will write a ros node that takes camera data as a PC, filters the PC, then segments individual objects using Euclidean clustering**

![objects](http://i66.tinypic.com/dnc95e.png)

#### 2. Complete Exercise 3 Steps.  Features extracted and SVM trained.  Object recognition implemented.

![demo-1](https://user-images.githubusercontent.com/20687560/28748231-46b5b912-7467-11e7-8778-3095172b7b19.png)


* **We need to extract the features that best describe the objects we are looking for despite position and orientation of these objects, here we will use as features two concanated normalized histograms of color and surface normals, then we generate these features by taking PC of the desired objects we want to identify in many orientations, and we train a Support Vector Machine Classifier to detect them, in multiple worlds and configurations**

![norm_table](http://i65.tinypic.com/2nkta2o.png)

### Pick and Place Setup

#### 1. For all three tabletop setups (`test*.world`), perform object recognition, then read in respective pick list (`pick_list_*.yaml`). Next construct the messages that would comprise a valid `PickPlace` request output them to `.yaml` format.

![world1](http://i65.tinypic.com/zx3cqq.png)
![world2](http://i67.tinypic.com/4rduoj.png)
![world3](http://i64.tinypic.com/dxntc3.png)

* **After we have identified the objects we would like to grab them and put them in the associated boxes left and right of the robot, for that we need to construct messages that can be passed for a valid 'PickPlace' request**

* **object name and group: recieved from the pick list yaml file**
* **arm name to be invoked recieved from the dropbox yaml file**
* **pick pose calculated from the centoroids of the individual objects**
* **postion to place the objects recieved from the dropbox yaml file based on group or hand (we also put some randomness on the final drop location to avoid our dropbox to become cluttered with objects)**
Spend some time at the end to discuss your code, what techniques you used, what worked and why, where the implementation might fail and how you might improve it if you were going to pursue this project further.  

* **We used filtering techniques like outlier removal, downsample, passthrough filters, segmentation to remove the table and euclidean segmentation for the objects, then we made features and data and trained an SVM classifier, and provided ros messages to services to execute grasp and move routines**

* **I havent include collidable object detection so the arm might karate chop the table or other objects while trying to grasp an object because the planning routine controlled by move it doesnt see them**
* **In the challenge world the robot needs to move around first in order to capture and detect objects in the whole scene, but when it moves it doesnt detect them**
* **Finally i could have broken down the code into 2 different nodes one for perception pipeline and another for sequential object detection**