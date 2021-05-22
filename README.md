# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

My pipeline consisted of 6 steps. I first created a cell in the notebook for each stage to allow me to tune the parameters in each stage to suitable values.

First, I converted the images to grayscale, then I applied gaussian smoothing with a kernal size of 5.

Next, I used Canny Edge Detection to find and edges in the image. I settled on low and high thresholds of 50 and 150 respectively as I felt these values detected what I required without making the image too noisy.

Once we had the edges, I defeined a trapezium shaped Region of Interest and applied this mask to the result of the Canny Edge Detection to filter out any edges outside of the area I expected the lane lines to be in.

The next step is to apply the Hough Transforms to detect lines within the masked image. 
This stage went through a number of interatons and improvements. The first iteration simply applied the hough transform and drew a number of small lines on the image. The right lane line is solid and was successfully drawn as a continuous line, however the left lane was segmented and was drawn as such by the hough transform.
My next iteration attempted to join this left lane into one continuous line also. I first tried this by extending the Maximum Gap between pixels to a larger number. This appeard to work well on the image.
However, once I applied the pipeline to the video, I saw a number of extra lines drawn on the video which I didn't expect and I soon realised the large gap size was connection points which should not have been. 
My final iteration was inspired by the hint in the draw_lines function to look into the slope of each line. 
I decided to look at each line in turn and split them based on the slope. I know both lanes sloped in towards the center of the image so if a line has a positive slope, it must be part of the left lane, and if the slope was negative it must be part of the right lane. 
Based on this distinction, I created four arrays. Two per lane 'side' for each x and y coordinates of all the points in that 'line'. 
Once I had a list of coordinates for all the points I'd defined as the left lane. I used Numpy Polyfit with a degree of 1 to find the polynomial coefficients for a line of best fit through these points. 
I knew I wanted the y-coordinate of each line to start at the bottom of the image, and end around half way down the image. So I had the y coordinates. I then rearranged the line equation 'y = mx + c' to solve for x. This gave 'x = (y - c)/m'. Plugging in each y coordinate the the coeffiences from the polyfit, gave me the x coordinates for the start and end of the line. I then repeated this process for the right lane. 

I ran this new method on the video and saw much improved results. However I did still notice occasional lines connecting accross the image between the two lane lines. 
To counter this, I decided to apply a filter on what points I added into the arrays for the polyfit function.
I knew the lane lines all roughly had the same absolute value for gradient. The hough transform would include any 'lines' detected by the edges. This would include markings or colour changes on the road. To remove these from consideration, I set values to filter out lines which were too 'vertical' or 'horizontal'. If the difference in x values were less than 0.001 I'd consider that line 'vertical' and ignore it.
If the absolute value of the gradient is less than 0.5, I'd class that line as 'horizontal' and ignore it. 
Adding this filter to the lines used for the fit improved the results I saw in the video.

The final step was to use the weighted_img function to combine the output from the Hough Lines method with the original image and I was able to see my 'detected' lines on top of the original.


### 2. Shortcomings

There are a number of shortcomings with this method which I have observed. 
* Region of Interest - The region of interest is defined using hard coded values based on the size of the image provided. This means if you use an image with different dimensions, the region would not make sense and cause bad results.
* Slope Filters - The filters I apply to remove 'horizontal' and 'vertical' lines are not very robust. It is very plausible to have other edges/lines in the image which satisfy these conditions but are not part of the lane line we are trying to detect. 
* Polyfit - The polyfit function I use to fit the line I draw on the image assumes the lane line is a linear (straight) line. If the lane line is curved, i.e. on a corner, the pipeline will still attempt to fit a straight line through all the points and will result in an error in positioning. 


### 3. Improvements

One potential improvement would be to make the Region of Interest a dynamic calculation. This might improve adaptability to other images/videos. However it is hard to know where to set this value as different cameras can have different orientations and FOV so the lane lines can't always be guarenteed to be in the same region proportional to the image.

Another improvement could be to use a higher degree polyfit for the lines so we aren't limited to just linear lines. This would help fit lines to curved lane lines better, however it would present some further challenges in categorising which of the hough lines belong to the left or right lane as the slope is variable. 
