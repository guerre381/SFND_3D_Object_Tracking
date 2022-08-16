# Report

# TTC Camera
the TTC comming form the lidar is unstable because of noisy points. Some points are jumping up and down and so is the bottom line.
The displacement of the noisy points is sometimes higher than the actual displacement of the preceeding car thherfore TTC can be helpless and aberrant. 
TTC can be negative indeed. It assumes that that the car is already in collision although it is not. The bottom line jumps so much that the descelerating speed is estimated high thus the
collision is predicted for the next cycle already which seems unlikely.

# TTC Lidar

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