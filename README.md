
### Advanced Lane Finding Project 

The software pipeline of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

We describe the software processing flow developed in details below - 

### Step 1: Camera Calibration Stage

Real cameras use curved lenses to form an image, and light rays often bend a little too much or too little at the edges of these lenses. This creates an effect that distorts the edges of images, so that lines or objects appear more or less curved than they actually are. This is called radial distortion, which is the most common type of distortion.

Therefore the first step is to calibrate for these distortions using reference images.

The calibration step is as follows - 

1.   Read chessboad images and convert to gray scale
2.   Find the chessboard corners using cv2.findChessboardCorners() function - The number of corner points to be expected is known apriori by viewing the calibration images. Here it is nx=9, ny=6. 
3.   The mtx, dist matrices are obtained by projecting these found corner points onto the object 3D location grids using the cv2.calibrateCamera() function

### Step 2: Camera Distotion Correction

Using the distortion co-efficients and camera matrix obtained from the above camera calibration stage, we undistort the images using the cv2.undistort function.

### Step 3: Gradient and Color Binary Thresholding

I used a combination of color and gradient thresholds to generate a binary image to detect the lane boundaries.

#### Step 3.1: S-channel Color space thresholding:
Firstly I convert the image to the HSV colour space and grab the S-channel only - samples of which show that it enables us to see the lane lines regardless of the colours better. Thresholding the S-channel between 150 and 255 was found to provide good results.

#### Step 3.2: L-channel Color space thresholding:
Additionally, I have converted the image to LUV color space and have used the L-channel and found 225 to 255 threshold provides good detection of the lane boundaries even under shadows or sun brightness, where typically there are issues.

#### Step 3.3: Gradient along x-axis thresholding:
Additionally the Sobel function has been used to generate the gradient of the image in the X direction. By taking the gradient in the X direction we can better pick out vertical features in the image, i.e. lane lines. The gradient is then thresholded as well between 20 and 100.

#### Step 3.4: Gradient of magnitude and directional thresholding:
Additionally the Sobel function along the x-axis and y-axis of the image is computed to finally estimate the directional gradient and magnitude gradient of the image. The AND-combination of the directional and magnitude gradient was used.   

#### Step 3.5: Combining to generate binary image
Finally all the above gradients and color thresholds have been merged using the OR-function to create the final binary image

### Step 4: Perspective Transformation

The straight_lines2.jpg image was used to manually select source and destination points viz the lane trapezoid pixels to generate the perspective transformation to get a top-down or birds-eye-view of the road. This will enable us to find lane lines and their curvature subsequently in the pipeline. The source and destination points was kept constant.

### Step 5 & 6: Detect Lane Lines and fit polynomial

With our binary warped image above I then searched for the lane lines as follows:

1.   Find initial lane lines using a histogram along warped image
2.   Split the warped image into 9 equi-spaced windows along the y-axis
3.   Use margin of 100 pixels around the two histogram peaks found around left half and right half from the center
4.   The centroid of these pixel locations are used as the detected left lane and the detected right lane
5.   Fit a 2nd order polynomial to fit the left and right lanes


The fit_lane() function implements the sliding window technique to find the lane lines and fit a polynomial using 9 windows from the bottom to the top of the image. This function assumes no prior knowledge of the lane lines and picks them from scratch through the histogram peaks and sliding window approach.

### Step 7: Generate the search window

The search window helps in avoiding searching the entire binary warped image again for the lane boundaries, rather the earlier detected lanes are used with a boundary to search for next centroid peaks along the 9 sliding windows in this image. This would reduce the computational cost of blindly searching the warped binary image again. Since my pipeline is presently running offline, I have not used it in my SW pipeline, rather am searching for the lane boundaries from scratch through histogram every frame.

### Step 8: Overlay the detected lane boundaries back onto the original image

Now we can wrap the detected lanes on the original images using inverse perspective transform. The final_vizual() function has been used to unwrap the detected lanes and plot them onto the initial undistorted image frames.

### Step 9: Video Processing and estimation of radius of curvature and central offset

#### Step 9.1 Radius of curvature
The curvature of the lanes f(y) is calculated by using the formulae R(curve) Curvature
f(y) = Ay^2 + By + C
f'(y) = 2Ay + B
f''(y) = 2A
R(curve) = ((1+(2Ay+B)^2)^1.5)/(|2A|)

Finally the average curvature of the left lane and right lane is reported on the video/image. 

#### Step 9.2 Central Offset
The vehicle position is calculated as the difference between the image center and the lane center. 

#### Step 9.3 Smoothing of detected lanes
The estimated left and right lane fits over the last 10 frames are used to estimate/smooth the left and right lane boundaries. This helps in removing spurious jumps or deviations in estimation. The smoothening function is implemented in the process_image() function

#### Step 9.4 Sanity Checks
Two sanity checks are used - 
##### Step 9.4.1 If the right_lane_x - left_lane_x < 500 and left_lane and right lane bends in opposite direction
Then don't rely on present averaged lane fits but rather use the last good fits
##### Step 9.4.2 If radius of curvature computed by left and right lanes differ more than 1000 or the average is more than 2000 (specifically given the project_video problem)
Then again,don't rely on present averaged lane fits but rather use the last good fits 


### Discussion & Thoughts -

This has been an extrmely rewarding and challenging problem to work on. Although I am not 100% happy with the results I achieve with the challenge_video and the harder_challenge_video.

Adaptively chose the source and destination points from the images, use different binary threshold images during different scene/environment. 

Seperate set of conditions need to applied based on the environment, specially with respect to where the lanes are picked from. In the harder challenge video sometimes lanes are absent and sometimes too much light blurs the image completely. Some form of interpolation beyond what has been used in my project here needs to be deployed to solve the issue.

