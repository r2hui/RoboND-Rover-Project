## Project 1: Search and Sample Return
### Submission by Richard Hui

---


**The basic goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (using defined metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: https://github.com/r2hui/RoboND-Rover-Project/blob/master/misc/mask.png 
[image2]: https://github.com/r2hui/RoboND-Rover-Project/blob/master/misc/find_rocks.png 
[image3]: https://github.com/r2hui/RoboND-Rover-Project/blob/master/misc/screenshot2.png 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation, which is primarily based on Udacity's [Rover Project Walkthrough](https://www.youtube.com/watch?v=oJA6QHDPdQw).


---
### Writeup 

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it, check!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

Rover_Project_Test_Notebook.ipynb provided as part of project submission. 

Modified function *perspect_transform* to return a mask, which shows the field-of-view from the camera.

![alt text][image1]
Warped on left, mask on right

Added function *find_rocks* to scan images and threshold for detecting yellow rocks. The function is based on function *color_thresh*.

![alt text][image2]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

Code from notebook. References to Project Walkthrough, given as #Youtube, [timestamp].

    # TODO: 
    # 1) Define source and destination points for perspective transform
    # 2) Apply perspective transform
    #Youtube, 14:12
    warped, mask = perspect_transform(img, source, destination)
    
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    #Youtube, 14:12
    threshed = color_thresh(warped)
    obs_map = np.absolute(np.float32(threshed) -1) * mask
    xpix, ypix = rover_coords(threshed)
    
    # 4) Convert thresholded image pixel values to rover-centric coords
    #Youtube, 14:12
    world_size = data.worldmap.shape[0]
    scale = 2 * dst_size
    xpos = data.xpos[data.count]
    ypos = data.ypos[data.count]
    yaw = data.yaw[data.count]
    x_world, y_world = pix_to_world(xpix, ypix, xpos, ypos, yaw, world_size, scale)
    obsxpix, obsypix = rover_coords(obs_map)
 
    
    # 5) Convert rover-centric pixel values to world coords
    #Youtube, 15:54
    obs_x_world, obs_y_world = pix_to_world(obsxpix, obsypix, xpos, ypos, yaw, world_size, scale)
    
    # 6) Update worldmap (to be displayed on right side of screen)
        # Example: data.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
        #          data.worldmap[rock_y_world, rock_x_world, 1] += 1
        #          data.worldmap[navigable_y_world, navigable_x_world, 2] += 1
        #Youtube, 15:54
    data.worldmap[y_world, x_world, 2] += 255
    data.worldmap[obs_y_world, obs_x_world, 0] = 255
    nav_pix = data.worldmap[:,:,2] > 0
    data.worldmap[nav_pix, 0] = 0

        #Finding some rocks, Youtube 16:32
    rock_map = find_rocks(warped, levels=(110, 110, 50))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world, :] = 255


### Autonomous Navigation and Mapping
#### For the Udacity Sample Return Robot Challenge, the primary metrics of interest are time, percentage mapped, fidelity and number of rocks found. For a passing submission, the rover must: 


- map at least 40% of the environment, 
- obtain 60% fidelity (accuracy) against the ground truth, and
- locate at least one of the rock samples


#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

##### Code from perception.py script, based on function process_image as found above. References to Project Walkthrough, given as #Youtube, [timestamp].

	# def perception_step(Rover):
    # Perform perception steps to update Rover()
    # TODO: 
    # NOTE: camera image is coming to you in Rover.img
    # 1) Define source and destination points for perspective transform
    # 2) Apply perspective transform
    #Youtube, 31:57
    dst_size = 5
    bottom_offset = 6
    image = Rover.img
    source = np.float32([[14,140],[301,140],[200,96],[118,96]])
    destination = np.float32([[image.shape[1]/2 - dst_size, image.shape[0] - bottom_offset],
                              [image.shape[1]/2 + dst_size, image.shape[0] - bottom_offset],
                              [image.shape[1]/2 + dst_size, image.shape[0] - 2*dst_size - bottom_offset],
                              [image.shape[1]/2 - dst_size, image.shape[0] - 2*dst_size - bottom_offset],])
    warped, mask = perspect_transform(Rover.img, source, destination)
        
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    threshed = color_thresh(warped) 
    obs_map = np.absolute(np.float32(threshed) - 1) * mask
    
	# 4) Update Rover.vision_image (this will be displayed on left side of screen)
        # Example: Rover.vision_image[:,:,0] = obstacle color-thresholded binary image
        #          Rover.vision_image[:,:,1] = rock_sample color-thresholded binary image
        #          Rover.vision_image[:,:,2] = navigable terrain color-thresholded binary image
    Rover.vision_image[:,:,2] = threshed * 255
    Rover.vision_image[:,:,0] = obs_map * 255
                      
    # 5) Convert map image pixel values to rover-centric coords
    xpix, ypix = rover_coords(threshed)
    
    # 6) Convert rover-centric pixel values to world coordinates
    world_size = Rover.worldmap.shape[0]
    scale = 2*dst_size
    x_world, y_world = pix_to_world(xpix, ypix, Rover.pos[0], Rover.pos[1], Rover.yaw,
                                    world_size, scale)
    obsxpix, obsypix = rover_coords(obs_map)
    obs_x_world, obs_y_world = pix_to_world(obsxpix, obsypix, Rover.pos[0], Rover.pos[1], Rover.yaw,
                                    world_size, scale)
    # 7) Update Rover worldmap (to be displayed on right side of screen)
        # Example: Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
        #          Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
        #          Rover.worldmap[navigable_y_world, navigable_x_world, 2] += 1
Optimizing Map Fidelity: added restriction in map updates when pitch and roll exceed 2 degrees.

    # Setting roll and pitch angle thresholds for restricting map updates
    if Rover.roll < 2 and Rover.pitch < 2:
        Rover.worldmap[y_world, x_world, 2] += 10
        Rover.worldmap[obs_y_world, obs_x_world, 0] += 1       

    # 8) Convert rover-centric pixel positions to polar coordinates
    # Update Rover pixel distances and angles
        # Rover.nav_dists = rover_centric_pixel_distances
        # Rover.nav_angles = rover_centric_angles
    dist, angles = to_polar_coords(xpix, ypix)
    Rover.nav_angles = angles
    
    #Youtube, 35:28
    rock_map = find_rocks(warped, levels=(110, 110, 50))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, Rover.pos[0], Rover.pos[1], Rover.yaw, world_size, scale)
        rock_dist, rock_ang = to_polar_coords(rock_x, rock_y)
        rock_idx = np.argmin(rock_dist)
        rock_xcen = rock_x_world[rock_idx]
        rock_ycen = rock_y_world[rock_idx]
        Rover.worldmap[rock_ycen, rock_xcen, 1] = 255
        Rover.vision_image[:,:,1] = rock_map * 255
    else:
        Rover.vision_image[:,:,1] = 0
    return Rover 

##### Code from decision.py script.

	# def decision_step(Rover):

    # Implement conditionals to decide what to do given perception data
    # Here you're all set up with some basic functionality but you'll need to
    # improve on this decision tree to do a good job of navigating autonomously!

    # Example:
    # Check if we have vision data to make decisions with
Optimizing for Finding All Rocks: added bias to steering angle for hugging left wall. Using random magnitude of left steering to help avoiding running circles, especially in wide spaces.
    
	steer_bias = random.randint(9,15)
    
    if Rover.nav_angles is not None:
        
        # Check for Rover.mode status
        if Rover.mode == 'forward':
                        
            # Check the extent of navigable terrain
            if len(Rover.nav_angles) >= Rover.stop_forward:
Optimizing for general completion: added routine to reverse when encountering an obstacle (throttling ahead but no velocity).
               	
			#Unstuck routine
                if Rover.vel == 0 and Rover.throttle == Rover.throttle_set:
                    Rover.throttle = -1
                    Rover.steer = -15
                    time.sleep(4)
                    Rover.mode = 'stop'

                # If mode is forward, navigable terrain looks good 
                # and velocity is below max, then throttle 
                elif Rover.vel < Rover.max_vel:
                    # Set throttle value to throttle setting
                    Rover.throttle = Rover.throttle_set

                else: # Else coast
                    Rover.throttle = 0

                Rover.brake = 0
                # Set steering to average angle clipped to the range +/- 15
                Rover.steer = np.clip(np.mean(Rover.nav_angles * 180/np.pi) + steer_bias, -15, 15)
            # If there's a lack of navigable terrain pixels then go to 'stop' mode
            elif len(Rover.nav_angles) < Rover.stop_forward:
                    # Set mode to "stop" and hit the brakes!
                    Rover.throttle = 0
                    # Set brake to stored brake value
                    Rover.brake = Rover.brake_set
                    Rover.steer = 0
                    Rover.mode = 'stop'

        # If we're already in "stop" mode then make different decisions
        elif Rover.mode == 'stop':
            # If we're in stop mode but still moving keep braking
            if Rover.vel > 0.2:
                Rover.throttle = 0
                Rover.brake = Rover.brake_set
                Rover.steer = 0
            # If we're not moving (vel < 0.2) then do something else
            elif Rover.vel <= 0.2:
                # Now we're stopped and we have vision data to see if there's a path forward
                if len(Rover.nav_angles) < Rover.go_forward:
                    Rover.throttle = 0
                    # Release the brake to allow turning
                    Rover.brake = 0
                    # Turn range is +/- 15 degrees, when stopped the next line will induce 4-wheel turning
                    Rover.steer = -15 # Could be more clever here about which way to turn
Optimizing for general completion: added 0.4 second delay to steering while stopped to clear walls/obstacles.
				
				time.sleep(0.4)
                # If we're stopped but see sufficient navigable terrain in front then go!
                if len(Rover.nav_angles) >= Rover.go_forward:
                    # Set throttle back to stored value
                    Rover.throttle = Rover.throttle_set
                    # Release the brake
                    Rover.brake = 0
                    # Set steer to mean angle
                    Rover.steer = np.clip(np.mean(Rover.nav_angles * 180/np.pi), -15, 15)
                    Rover.mode = 'forward'
    # Just to make the rover do something 
    # even if no modifications have been made to the code
    else:
        Rover.throttle = Rover.throttle_set
        Rover.steer = 0
        Rover.brake = 0
        
    # If in a state where want to pickup a rock send pickup command
    #if Rover.near_sample and Rover.vel == 0 and not Rover.picking_up:
    #    Rover.send_pickup = True
    #Testing below 
Picking up Rocks: Modified send_pickup code to brake and stop before picking up rocks.
    
	if Rover.near_sample and not Rover.picking_up:
        # Set mode to "stop" and hit the brakes!
        Rover.throttle = 0
        # Set brake to stored brake value
        Rover.brake = Rover.brake_set
        Rover.steer = 0
        Rover.mode = 'stop'
        Rover.send_pickup = True
    
    
    return Rover

#### Other changes (abbreviated) to class RoverState() in drive_rover.py:
Optimizing for Time Tip: Modified throttling, braking and maximum velocity values to make rover travel faster and more efficiently.

	class RoverState():
    def __init__(self):
        self.throttle_set = 1.0 # Throttle setting when accelerating
        self.brake_set = 20 # Brake setting when braking
        self.max_vel = 3 # Maximum velocity (meters/second)

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results! **

Screen resolution: 1024x768;
Graphics quality: Fastest;
FPS:35

Approach undertaken: Observing the rocks were near walls, program the rover so the it travels close to walls.

Techniques used, did it work and why:

1. Restricting map updates when pitch and roll exceed a value. Worked well to keep map fidelity high as it tended to decrease when pitch / roll angles spiked up. This compensation works as the transformed images are altered when there is a pitch or roll on the rover.
2. Making the rover travel faster. Reduced time to cover 60% of map.
3. Wall crawling (on the left). Does help in navigating through entire course. Also helps in locating all the rocks (and be in better position to pick up rocks. 
4. Randomizing steering bias. No effect on overall metric though does help avoid being stuck in a certain steering angle and thus travel in a circle.
5. Unstuck routine. No effect on overall metric though does help avoid being stuck at an obstacle. 
6. Stopping to pick up rocks. No effect on overall metric though helps to be in position to pick up rocks.


Where the pipeline might fail: Although helpful, the unstuck routines are not foolproof, and the rover may still sometimes be trying to throttle through obstacles.

How to improve:

1. Programming to avoid previously visited, mapped areas
2. Programming to "close" boundaries on map
3. Programming rover to actively pick up a rock, upon its detection, instead of continuing to navigate based on nav_angles.
4. Programming rover to go back to starting position after picking up 6 rocks.
5. Improving unstuck routines.
6. Programming better detection of obstacles, especially rocks.


![alt text][image3]
In 225s, 93% mapped, 65% fidelity, 6 rocks located, 1 rock collected
