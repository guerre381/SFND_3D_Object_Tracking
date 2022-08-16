
# SFND 3D Object Tracking

Welcome to the final project of the camera course. By completing all the lessons, you now have a solid understanding of keypoint detectors, descriptors, and methods to match them between successive images. Also, you know how to detect objects in an image using the YOLO deep-learning framework. And finally, you know how to associate regions in a camera image with Lidar points in 3D space. Let's take a look at our program schematic to see what we already have accomplished and what's still missing.

<img src="images/course_code_structure.png" width="779" height="414" />

In this final project, you will implement the missing parts in the schematic. To do this, you will complete four major tasks: 

1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 

2. Second, you will compute the TTC based on Lidar measurements. 

3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches.

4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 

## Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
  * Install Git LFS before cloning this Repo.
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory in the top level project directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./3D_object_tracking`.

##  Report
**FP.1: Match 3D Objects**
We want to go from key points matching to bounding box matching.
For each key point matches in previous and current frame, we count key points contained in bounding boxes on both sides. Here, we use a map where bounding box id pairs are keys and the number of shared key points as values.
After its creation, we iterate through the map´s keys: 
We visit each bounding box in current frame given a corresponding bounding box in the previous frame, and keep the ids of bounding boxes having the most key points in common. We end up with a unique association of bounding box ids pairs that are most likely encapsulating the same object in both frames.

**FP.2: Compute Lidar-based TTC**
We use LIDAR points reflected by the preceding vehicle in 2 continuous timestamps. 
In both point clouds, we observe the longitudinal distance to the closest point along the x-axis. That is the distance to the preceding vehicle. 
The change in distance and the framerate are used to calculate the instantaneous speed of the preceding vehicle. Once the speed is injected in the continuous velocity model, we get the TTC.  

**FP.3: Associate Keypoint Correspondences with Bounding Boxes**
Given a bounding box in the current frame, we filter key points fitting in, from the current and the previous frame. Now we have all pair of key points belonging to the same objects in both frames.

**Fp.4: Compute Camera-based TTC**
 After all key points matches related to the same object have been identified in the process of bounding box matching, we can observe key points related to that object only in both frames.
We calculate the distance between each key points:
 - Within the previous bounding box.
 - Within the current bounding box.
 
The distance ratio between corresponding segments are stored in a vector. The ratio indicates how much the preceding car has changed in scale.  We could take the average but the ratios can be noisy due to falsely matching points hence we use the median.
Using the Thales theorem combined with the pinhole representation of the camera, we can derive the TTC.

**FP.5: Performance evaluation 1 - TTC Camera**

the TTC comming form the lidar is unstable because of noisy points. Some points are jumping up and down and so is the bottom line.
The displacement of the noisy points is sometimes higher than the actual displacement of the preceeding car thherfore TTC can be helpless and aberrant. 
TTC can be negative indeed. It assumes that that the car is already in collision although it is not. The bottom line jumps so much that the descelerating speed is estimated high thus the
collision is predicted for the next cycle already which seems unlikely.

**FP.6: Performance Evaluaction 2 - TTC Lidar**

- In the folloowing table we compare the performace for pairwise combination of detectors and descriptors.
  There are empty cells because the combination is simply not possible or processing time was too long.

- For the analysis, we keep track of the following 3 metrics:
  - m is the average of TTC camera across all images.
  - s is the standard eviation of TTC camera.
  - min is the minimum TTC camera across all images
  - max is the maximum TTC camera across all images


| detector/descriptor | SIFT                                       | BRIEF                                      | FREAK                                      | AKAZE                                      | ORB                                        |
|---------------------|--------------------------------------------|--------------------------------------------|--------------------------------------------|--------------------------------------------|--------------------------------------------|
| SIFT                | m=11.111, std=2.504, min=8.691, max=18.794 | m=11.111, std=2.659, min=8.395, max=18.932 | m=11.500, std=2.895, min=8.724, max=20.517 |                                            |                                            |
| HARRIS              |                                            |                                            |                                            |                                            |                                            |
| FAST                | m=11.389, std=1.375, min=9.253, max=14.238 | m=11.277, std=1.206, min=9.620, max=13.532 | m=11.444, std=1.087, min=9.973, max=13.348 |                                            | m=11.222, std=1.218, min=9.672, max=13.572 |
| BRISK               | m=13.444, std=3.525, min=9.559, max=25.506 | m=13.444, std=3.653, min=9.325, max=24.256 | m=13.722, std=4.118, min=9.099, max=23.559 |                                            | m=13.444, std=2.927, min=10.24, max=19.143 |
| ORB                 | m=10^8  , std=infs , min=-infs, max=524.52 | m=1-0^8 , std=infs , min=-infs, max=44.497 | m=-10^8 , std=infs , min=-infs, max=60.267 |                                            | m=1106.8, std=4591 , min=8.287, max=19504  |
| AKAZE               | m=11.778, std=2.309, min=8.963, max=16.221 | m=11.944, std=2.239, min=9.097, max=15.766 | m=11.666, std=2.225, min=8.806, max=15.459 | m=11.777, std=2.208, min=8.974, max=16.060 | m=11.777, std=2.224, min=8.975, max=15.623 |

- The best detector is FAST. Whatever descriptor is used afterwards, the estimated TTC stays in rnage 9 to 14 which seems correct when compared with the rest of the estimations including lidar outputs. 
- When used with FAST, the best descriptor could be FREAK because the standard deviation is small. It shows how precise and consistant the measurement are. 
- However, standard deviation is not the best metric. TTC changes over time since the car couls be descelerating as we approcah a traffic light.
- FAST and ORB together are also good. They both showed the best performances in mid-term assignment on feature tracking.