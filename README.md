
# **Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms and gradients to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

## Steps Followed
I have followed the following steps in detecting the lane.
> 1. Camera Calibration
> 2. Distortion Correction
> 3. Applying Color Thershold
> 4. Applying Gradient Thershold
> 5. Combining Color & Gradient Thershold
> 6. Perspective Transform 
> 7. Finding Lane On Warped Image
> 8. Updating Input Image

### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
The code for this step is contained in the [IPython Notebook](AdvanceLaneFindingCode.ipynb)
> `GetCordinatesForCalibration` : To get the object point and image point (cell block:2)

> `calibrateCamera` : To get the camera calibration matrix and distortion co-efficients (cell block:3)

> `un_distort` : To un distort images (cell block:5)
 
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text](.\Images\CameraCalibration.JPG)

## Pipeline (single images)
The following input frame is taken for explaining my code
![inputimage](.\Images\InputFrame.jpg)

### 1. Provide an example of a distortion-corrected image.

I have used the camera matrix and distortion co-efficient obtained from function `calibrateCamera`, to undistort the input image. The below one is the distortion corrected image
![inputimage](.\Images\Distortion_Correction.JPG)
> **Note** : As I am using `cv2 library` to read images and `matplotlib.pyplot` to display, The color of the input & undistorted images are modified in the above image 

### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

> ### Choose Color Channel
>> I have chose the RGB & HSV color spaces for performing color threshold. From the below output images, It is observed that `R Channel` from RGB and `S Channel` from HSV are good.
>> + R Channel : Good in detecting white lines
>> + S Channel : Helps in removing the effect of light variation
>> ![ChannelSelection](.\Images\ChannelSelection.JPG)

> ### Apply Color Threshold
>> Here, I am applying color thershold to get the required features from the selected channel. `color_thresh` method ( written in cell 9) helps to do color thresholding. 
>> ![ColorThreshold](.\Images\ColorThresholding.JPG)
>> After applying color thershold, the lane lines are detected more clearly than the normal version. At the same time, it introduced some noises in the output image also.

> ### Combine Color Channel
>> The combination of R Channel and S channel will remove the noise as well as it will help to detect the lane lines in different condition. `Combine_RS` method ( written in cell 11) will combine both R & S Channel with pre-defined boundary. 
>> ![CombineColorChannel](.\Images\CombineChannel.JPG)

----

> ### Gradient Threshold
>> Output of color thresholding is good to detect lane lines. But to further improve the quality, we are performing gradient thershold. Here `Sobel` operator is used for finding gradient of the image. 

>> We can perform gradient thershold by ***THREE*** ways. They are `Sobel X`, `Sobel Y` and `Sobel XY`.

>> We can apply both magnitude as well as direction thershold on each of the above Sobel operator. one common method `dir_threshold` (written in cell 13) will be used to perform any of the above combinations.
>> ![GradientThershold](.\Images\GradientThreshold.JPG)
>> From the above output, Magnitude gradient worked well in detecting lane lines

----

> ### Combine Color & Gradient Threshold
>> We are going to combine the R&S channel output with Magnitude gradient thershold. 
>> They were combined using `OR gate`, as we don't want to lose any input informations.
>> ![Combine](.\Images\combine.JPG)

----
Finally, The whole process is simplified into one method `cvtToBinary`. This will take `UnDistorted Image` as input and produces `Binary Image` as output. This method is implemented in cell 17.
>> ![Combine](.\Images\cvtBinary.JPG)

### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `ImageWrapper()`, which appears in the 19 code cell of the IPython notebook.  The `ImageWrapper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points by manually verifying it

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 770, 461      | 1250, 29      | 
| 1250, 657     | 1250, 657     |
| 193,667       | 193,667       |
| 567,461       | 193,29        |

![PerspectiveTransform](.\Images\PerspectiveTransform.JPG)

### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I am using `sliding lines` methodology to find the lane lines in warped image. This is similar to `Sliding Window`. Instead of taking non-zero points in each window, this will take non-zero points in selected line. The aim of this method to improve speed of the lane detection by performing less computation.

***Algorithm :***
1. Calculate the histogram of the warped image
2. Divide Warped image into 50 equally seperated lines ( Sliding lines )
3. Get the peak points for both lane lines
4. With in the specified offset (width of the line), record all non-zero co-ordinates
5. Calculate average value of Non-Zero points at that line which act as a base for next line
6. Repeat Step 4 -Step 5  for all lines
7. Calculate the second order polynomial for the recorded co-ordinates. This will represent the lane line
The above algorithm is implementes in `getCoOrdinates` method written in 22nd cell

>Detected Points
>![DetectingPoints](.\Images\DetectedPoints.JPG)

From the above picture, we can see large portion of unwanted noises are omitted by our algorithm. At the same time, there can be chances that it may skip some valid co-ordinates (or) some times there may be no points detected in the lane. These problems are handled by remembering the past frame details. The detailed implementation will be present in the method `CoOrdinateCorrection` written in 24th cell.

Conditions used in correcting co-ordinates:
* If more than 50% of sliding lines are detected with Non-Zero values, then no correction required
* If more then 30% of sliding lines are detected with Non-Zero values, then 10% of previous line data added
* If more then 20% of sliding lines are detected with Non-Zero values, then 30% of previous line data added
* If the detected points are less then 20% ( but not equal to zero), then all previous data added with current data
* If no line detected, then use previous data

Method `DrawLines` written in cell no 25 will return the modified co-ordinate values. This is the final points which is used for plotting lines. `FindLanes` (cell no:27) used to fill the color between the detected lines.

>![DrawLines](.\Images\FindLane.JPG)


### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center
Using the final x,y cordinates value, i will calculate the radious of curvature by the following formula:
```python
curverad = ((1 + (2*co_efficient[0]*y_eval*ym_per_pix + co_efficient[1])**2)**1.5) / np.absolute(2*co_efficient[0])
```

The position of the vehicle respect to road is calculated by comparing the center of the input frame with center of two lane lines.
```python
    LaneCenter = (CumulativeRight[0][719]+CumulativeLeft[0][719])//2
    ImageCenter = 640
    if ImageCenter > LaneCenter:
        Orient="Left"
    else:
        Orient="Right"
    Deviation=abs(ImageCenter-LaneCenter)*xm_per_pix
```
They above calculations are done in `UpdateComments` method (cell no:)

***Note :*** I am using 30 meters per 720 pixel in `Y-direction` and 3.7 meters per 700 pixel in `X-direction`

### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The whole pipeline is compressed into one final method `Update_Frame` implemented in cell no #.
>![ResultFrame](.\Images\ResultFrame.JPG)

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](test_output_videos/project_video.mp4)

<video width="920" height="340" controls src="test_videos_output/project_video.mp4" />

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

### Problems Faced :

#### 1. Different light condition
> When vehicle crosses the shadow / gray road. It has failed to detect the lane lines.

> ***Solution***:  By tunning the threshold values of color channels & magnitude and direction of the sobel operator, we increased the accuracy

#### 2. When sliding line failed to detect lane lines
> Some times, sliding line doesn't detect any points in the lane. That is due to small offset value given.
>* If I increase the offset size, it result line more affected by the noise. 
>* If I use the previous frame data, it may not work in the sharpe turns

> ***Solution***: 
> 1. If the detected points(in particular line) is greater than given thresh hold, then it will be added to the list. This helped me to reduce the noise
> 2. Instead of completely depending on the previous frame data, mixture of current and previous frame used.

#### 3. Finding the offset value
> To find the offset value, best suits for all frames is kind of difficult. Change in offset will largely affect my performance of the code

> ***Solution***:
> 1. Converted the failing part of video into frames.After evaluating nature of the data, Tried to reduce the error without changing the offset ( by making use of previous frame data / changing thershold value / by applying mask)
> 2. Executig video with different offset value to find the best fit data.
