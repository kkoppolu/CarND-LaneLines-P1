# Project Reflections

## The pipeline
The pipeline consists of the following steps. The output of each pipeline step is fed into the next pipeline step until the image/video is annotated with the detected lanes.:

1) [Read the image](#read-the-image)
2) [Convert to grayscale](#convert-to-grayscale)
3) [Image blur](#image-blur)
4) [Edge Detection](#edge-detection)
5) [Region Selection](#region-selection)
6) [Line detection](#line-detection)
7) [Lane fitting](#lane-fitting)
8) [Annotation](#annotation)

### Read the image
The image is read in through the `imread` function of the `matplotlib.image` package

###  Convert to grayscale
The image is converted to grayscale for better gradient variation detection. This is done through the provided helper function - `grayscale` which in turn uses the OpenCV libraries.

### Image blur
The image is blurred using Gaussian filter to smoothen the gradient variations and avoid sharp gradient variations. A kernel size of 3 is selected as per the general recommendation.

### Edge detection
The edges in the image are detected by using the Canny Edge detection helper method - `canny`. A ratio of 3 is maintained between lower threshold and higher threshold inputs to the function. The lower threshold is tweaked until satisfactory results are obtained.

### Region Selection
A polygon region is selected in the image based on the image size and lane positions. This is done by masking out all pixels outside of the polygon of interest. The provided helper function `region_of_interest` is used.

*Improvement*: The code uses hard-coded values for the selected region vertices. As a result, the solution will not work across a generic image set. This impacts the accuracy of subsequent pipleline steps. A better approach would be to dynamically select the polygon vertices based on the shape of the image.

### Line Detection
Line detection is performed through Hough transform. The provided helper function `hough_lines` is used which runs the Hough transform algorithm and draws the lines on an image with black background. A very fine grid resolution is chosen. The other parameters of the function are tuned based on the desired output on the given output set such that the video/images are annotated with lanes most of the time.
*Improvement*: The parameters could have been chosen based on an educated feedback loop instead of manually verifying the expected results. Maybe this is something that will come up in subsequent classes.

### Lane fitting
- The output from the Hough transform is used to fit two lines - one for the left lane and one for the right lane.
- Right lanes are identified based on the positive slope characteristic. Negative slope likewise represents left lanes. 
- The corresponding (x,y) co-ordinate points are collected for left lane and right lane respectively.
- Co-ordinate points of near horizontal lines (extremely low values of slope) are ignored to avoid noise in the output. The threshold value used for this purpose is 0.2 (approximately 11 degrees). This is based on the assumption that lanes cannot be this tight (something which can be revisited).
- The minimum value `y` from the selected co-ordinate points is computed which will be used as an extrapolation point. 
- numpy's `poly1d` is used to construct the following extrapolation relation:
```x = f(y)```
- From the extrapolation relation, the x values for the extrapolated lines are calculated for y values ranging from `y_min` to `y_max`.
- The resulting extrapolated co-ordinates are used for drawing colored lines representing detected lanes.

### Annotation
The given helper function `weighted_image` is used for combining the original image with the image containing the detected lanes.


## Notes:
- A helper function `read_image` is written which takes in a file path and returns the image read in.
- A new function `draw_lines` is written and the old function is renamed as `draw_lines_old`.
- Images files with lane annotations are in `test_images_output`
- Video files with lane annotations are in `test_videos_output`
- Output directories `test_images_output` and `test_videos_output` are created if they do not exist by the corresponding methods of interest.
- The solutions breaks down when executed on the challenge problem. This is due to the shortcomings already mentioned in the pipeline in addition to other image features closely resembling the lanes on the road. Bright lighting and shadows on the road for instance throw off the detection pipeline. Normalization of the image color space could be an option in making the detection better.