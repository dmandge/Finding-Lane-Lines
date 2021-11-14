# Finding Lane Lines  
Software pipeline to identify positions of the left and right lane lines on the road

Overview
---

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  The goal of this project is to write a software pipeline using image analysis techniques, that will identify position of the left and right lanes lines on the road, in the images and video stream.

![1-testimage](https://user-images.githubusercontent.com/59345845/141664295-3b3b46d4-72a7-4f44-bd47-0ae398a509b5.JPG)

My pipeline consisted of 7 steps as listed below
1. Grayscale –
* Grayscaling takes the original color image in three color channels (r,b,g) as input and returns an image with only one color channel. This is the first step towards building a code to detect boundaries of an object in an image or series of image. Rapid changes in the brightness (pixels) in a grayscale image is where we find strongest gradients (edges)
* Output image from function ‘grayscale’ is provided as input to function ‘gaussian_blur()’

![2-Grayscale](https://user-images.githubusercontent.com/59345845/141664296-cdf4da65-863c-4143-bfbb-c67c2813018e.JPG)

2. Gaussian Blur -
* Applying Gaussian blur suppresses noise and spurring gradients by averaging adjacent pixel values. We apply additional layer of adjustable Gaussian blurring before applying Canny edge detection to smooth the image out.
* Two inputs to the function ‘gaussain_blur()’ are grayscale image ‘gray’ and kernel size =3. Higher kernel size applies smoothing over a larger area.
* Output of image from function ‘gaussian_blur()’ is used to apply Canny edge detection

![3-GaussianBlur](https://user-images.githubusercontent.com/59345845/141664297-4925caad-44d5-4bf8-8e85-0018cfba4216.JPG)

3. Canny Edge Detection –
* Canny function from OpenCV detects strong gradients in the smoothened image ‘gaussian_img’ by highlighting pixels above specified ‘high_threshold’ value and rejecting pixels below ‘low_threshold’ values. Once strong edges are detected, pixels between high and low threshold interval are highlighted if they are connected to pixels above high threshold.
* The output image from Canny edge detection – ‘canny_img’ is a binary image with strong edges highlighted in white and rest of the area blacked out.
* I have set the low to high threshold ratio to 1:3 and in the middle region (60 to 160) of the 0 to 255 scale to detect sufficiently dark images.
* High threshold – 160
* Low threshold – 60

![4-CannyEdgeDetection](https://user-images.githubusercontent.com/59345845/141664298-b91b0599-4ef6-4d8b-a28d-ac8b819d4de3.JPG)

4. Region of interest –
* Region of interest is applied to narrow down the region so as to try and eliminate almost everything else on the image except lane lines
* The function ‘region_of_interest()’ takes image processed with Canny edge detection along with [x,y] coordinates of four vertices that would form polygon of region of interest.
* I have defined the four vertices as a fraction of height and width of the image focusing on the lower half where lanes generally lie in all the test images and videos.
* The output image contains only the image defined by polygon formed from four vertices and rest of the image is set to black

![5-RegionOfInterest](https://user-images.githubusercontent.com/59345845/141664299-ea5828db-66df-426c-af9a-94f185617cd9.JPG)

5. Hough lines –
* Hough lines function detects lines on the masked image using openCV function ‘HoughLinesP’, after applying Hough Transform based on following input parameters
  * Masked image containing only region of interest
  * Distance and angular resolution of grid in Hough space – kept minimum at 1 and pi/180 (1 degree) respectively for a finer grid
  * Threshold number of intersections in a grid cell needed for it to be defined as line (=35) set high enough such that it only detects true lane lines and filters the noise out
  * Minimum length of line allowed (=5)
  * Maximum gaps in pixels allowed to be part of same line(=2)
* Output of this function is array of lines with coordinates of their endpoints
* This is used in the function ‘draw_lines()’ to defined one single line for left and right lane lines

![6-HoughTransform](https://user-images.githubusercontent.com/59345845/141664300-b15738cd-fd98-4a10-80ad-2bcd4de78e1d.JPG)
![6-HoughTransform2](https://user-images.githubusercontent.com/59345845/141664301-7816422a-71fc-4695-baab-4cb9b9ab1ebb.JPG)

6. Draw lines –
* In order to detect the complete left and right lane lines, and draw one single line on the left and right lanes, I modified the draw_lines() function as follows
  * Iterate through each line endpoints (x1,y1,x2,y2) from ‘lines’ array and calculate their slope and center
    * Slope = (y1-y1)/(x2-x1)
    * Center = [(x2+x1)/2, (y2+y)/2]
  * If the slope is greater than a small positive number (> 0.1), the slope and corresponding center is saved in a separate array of left slope and left center. Same steps are followed for right lane if the slope is smaller than a small negative value (<-0.1)
  * Averaged all left/right slope and center values to get single slope and center for left and right lane
  * Two calculate endpoints of the left and right lane lines, I assumed two y coordinates equal to image height and 60% of image height. Then, I calculated corresponding x coordinates using slope and centers for left and right lanes
    * Formula used - (y-y')=M(x-x'), where [x’,y’] are coordinates of calculated average center
  * Converted all coordinate values into integers to provide as input to line function in openCV
  * And used cv2.line function two draw lines on the original image with color red and thickness of 5

7. Overlay images with Hough lines –
* As final step, I overlaid the left and right lane lines on top of the original image using weighted image function. The function takes processed black image after Hough Transform with only lane lines drawn on it and overlays it on top of the original image
* Transparency of the processed image is set to a higher value than original image

![final](https://user-images.githubusercontent.com/59345845/141664302-1ba70e1f-1e3d-4930-9095-daf34963df45.JPG)
