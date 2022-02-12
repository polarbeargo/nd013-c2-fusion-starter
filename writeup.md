# Writeup: Track 3D-Objects Over Time
[image1]: ./img/ID_S1_EX1.png
[image2]: ./img/S1_EX1Settings.png
[image3]: ./img/code1.png
[image4]: ./img/S1_EX1s.png
[image5]: ./img/ID_S1_EX2setting.png
[image6]: ./img/S1_EX1s2.png
[image7]: ./img/S1_EX1s3.png
[image8]: ./img/ID_s4Settings.png
[image9]: ./img/ID_S3Setting.png
[image10]: ./img/ID_S2setting.png
[image11]: ./img/show_pcl.png
[image12]: ./img/ID_S1_EX2.png
[image13]: ./img/Open3D3.png
[image14]: ./img/Open3D1.png
[image15]: ./img/Open3D4.png
[image16]: ./img/bevFromPCL.png
[image17]: ./img/ID_S2_EX1.png
[image18]: ./img/bev_from_pcl.png
[image19]: ./img/height.png
[image20]: ./img/intensity.png
[image21]: ./img/intensityBEV.png
[image22]: ./img/heightBEV.png
[image23]: ./img/ID_S3Settings.png
[image24]: ./img/labelsVSDetected.png
[image25]: ./img/labelsVSDetected2.png
[image26]: ./img/pcl.png
[image27]: ./img/pcl1.png
[image28]: ./img/pcl2.png
[image29]: ./img/pcl3.png
[image30]: ./img/pcl4.png
[image31]: ./img/pcl5.png
[image32]: ./img/pcl6.png
[image33]: ./img/pcl7.png
[image34]: ./img/pcl8.png
[image35]: ./img/pcl9.png
[image36]: ./img/pcl10.png
[image37]: ./img/pcl11.png
[image38]: ./img/ID_S4_EX1.png
[image39]: ./img/ID_S4_EX2.png
[image40]: ./img/ID_S4_ex3code.png
[image41]: ./img/modelBasedDetection.png
[image42]: ./img/groundtruthLabelsAsObjects.png
[image43]: ./img/s1.gif
[image44]: ./img/s2.gif
[image45]: ./img/s3.gif
[image46]: ./img/s1.png
[image47]: ./img/s2.png
[image48]: ./img/S4.png

# Project: Sensor Fusion and Object Detection

| Visualization | 
| ------------- | 
| ![][image43]  |  
| ![][image44]  | 
| ![][image45]  |  

| Plot | 
| ------------- | 
| ![][image46]  |  
|![][image47]   | 
|![][image48]   | 

Please use this starter template to answer the following questions:  
 
### 1. Write a short recap of the four tracking steps and what you implemented there (filter, track management, association, camera fusion). Which results did you achieve? Which part of the project was most difficult for you to complete, and why?  
 - Filter:  
     - Implemented the predict() function for an EKF. Implement the F() and Q() functions to calculate a system matrix for constant velocity process model in 3D and the corresponding process noise covariance depending on the current timestep dt.  
     - Implement the update() function as well as the gamma() and S() functions for residual and residual covariance.  
     - At the end of the update step, save the resulting x and P by calling the functions set_x() and set_P() that are already implemented in `student/trackmanagement.py`.   
 - Track management:   
    - In the Track class, replace the fixed track initialization values by initialization of track.x and track.P based on the input meas, which is an unassigned lidar measurement object of type Measurement.   
    - Transform the unassigned measurement from sensor to vehicle coordinates with the sens_to_veh transformation matrix implemented in the Sensor class.      
    - Initialize the track state with 'initialized' and the score with 1./params.window, where window is the window size parameter, as learned in the track management lesson.   
    - In the Trackmanagement class, implement the manage_tracks() function to complete the following tasks:  
      - Decrease the track score for unassigned tracks.  
      - Delete tracks if the score is too low or P is too big (check params.py for parameters that might be helpful). Note that you can delete tracks by calling the given function delete_track(), which will remove a track from track_list.  
    - In the Trackmanagement class, implement the handle_updated_track() function to complete the following tasks:  
       - Increase the track score for the input track.  
       - Set the track state to 'tentative' or 'confirmed' depending on the track score.  
       - Use numpy.matrix() for all matrices as learned in the exercises.  
 - Association:  
    - In the Association class, implement the associate() function to complete the following tasks:
       - Replace association_matrix with the actual association matrix based on Mahalanobis distances for all tracks in the input track_list and all measurements in the input meas_list. Use the MHD()function to implement the Mahalanobis distance between a track and a measurement. 
       - Use the gating() function to check if a measurement lies inside a track's gate. If not, the function shall return False and the entry in association_matrix shall be set to infinity.
       - Update the list of unassigned measurements unassigned_meas and unassigned tracks unassigned_tracks to include the indices of all measurements and tracks that did not get associated.
    - In the Association class, implement the get_closest_track_and_meas() function to complete the following tasks:  
       - Find the minimum entry in association_matrix, delete corresponding row and column from the matrix.
       - Remove corresponding track and measurement from unassigned_tracks and unassigned_meas. 
       - Return this association pair between track and measurement. If no more association was found, i.e. the minimum matrix entry is infinity, return numpy.nan for the track and measurement.  
 - Camera fusion:  
    - In the Sensor class, implement the function in_fov() that checks if the input state vector x of an object can be seen by this sensor. The function should return True if x lies in the sensor's field of view, otherwise False. Don't forget to transform from vehicle to sensor coordinates first. The sensor's field of view is given in the attribute fov.  
    - In the Sensor class, implement the function get_hx() with the nonlinear camera measurement function h as follows:  
      - transform position estimate from vehicle to camera coordinates,  
      - project from camera to image coordinates,  
      - make sure to not divide by zero, raise an error if needed,  
      - return h(x).  
    - In the Sensor class, simply remove the restriction to lidar in the function generate_measurement() in order to include camera as well.  
    - In the Measurement class, initialize camera measurement objects including z, R, and the sensor object sensor.   
 
Overall steps I found step 2 trackmanagement is very challenging compare the result to other steps. The RMSE in step 2 is quite high in this scenario (around 0.78), also the green boxes don't fit the car in the image very well. At the step 3 association the console output shows that a single measurement has been used several times, there is an error in the association matrix or in the deletion of used rows and columns will need to printing the association matrix for futher debugging.

### 2. Do you see any benefits in camera-lidar fusion compared to lidar-only tracking (in theory and in your concrete results)?  
-  The visualization suppose to show that there are no confirmed “ghost tracks” that do not exist in reality. There may be initialized or tentative “ghost tracks” as long as they are deleted after several frames.  If we still saw some initialized or tentative ghost tracks here, we can deplausibilize them through sensor fusion with camera. With The console output shows lidar updates followed by camera updates. The visualization shows that the tracking performs well, again no confirmed ghost tracks or track losses should occur. The RMSE plot should show at least three confirmed tracks. Two of the tracks should be tracked from beginning to end of the sequence (0s - 200s) without track loss. The mean RMSE for these two tracks should be below 0.25 (See plot above).

### 3. Which challenges will a sensor fusion system face in real-life scenarios? Did you see any of these challenges in the project?  
At step 2 trackmanagementthe RMSE is quite high in this scenario (around 0.78), also the green boxes don't fit the car in the image very well. It is part of the real-world challenges that our assumptions about the data are not always met. We can, however, compensate this offset through sensor fusion once we include other sensors.  There is an error in the association matrix or in the deletion of used rows and columns will need to printing the association matrix for futher debugging.  

### 4. Can you think of ways to improve your tracking results in the future?  
- Fine-tune parameterization and see how low an RMSE can be achieve! Apply the standard deviation values for lidar that found in the mid-term project. The parameters in `misc/params.py` should enable a first running tracking, but there is still a lot of room for improvement through parameter tuning.  
- Implement a more advanced data association such as Global Nearest Neighbor (GNN) or Joint Probabilistic Data Association (JPDA).  
- Feed camera detections from Project 1 to the tracking.   
- Adapt the Kalman filter to also estimate the object's width, length, and height, instead of simply using the unfiltered lidar detections as we did.  
- Use a non-linear motion model such as a bicycle model, which is more appropriate for vehicle movement than our linear motion model, since a vehicle can only move forward or backward, not in any direction.  

