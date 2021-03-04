## <font style = "color:rgb(50,120,229)">Step 1 : Facial Feature Detection</font>

For each facial image we calculate 68 facial landmarks using **Dlib**. Details about obtaining the landmark points were  already covered in the module on **Facial Landmark Detection**. The figure below shows the 68 landmark points.
<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-facialFeature.jpg"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-facialFeature.jpg"/></a></center>
<center>Figure 3 : Facial feature detection example.</center>

## <font style = "color:rgb(50,120,229)">Step 2 : Coordinate Transformation</font>

The input facial images can be of very different sizes. So we need a way to normalize the faces and bring them to the same reference frame.  To achieve this we warp the faces to a 600×600 image such that the left corner of the left eye is at pixel location ( 180, 200 ) and the right corner of the right eye is at pixel location ( 420, 200 ). Let us call this coordinate system the **output coordinate system** and the coordinates of the original images the **input coordinate systems**.

How were the above points chosen ? We wanted to make sure the points were on a horizontal line, and the face was centered at about a third of the height from the top of the image.  So we chose the corners of the eyes to be at ( 0.3 x width, height / 3 ) and ( 0.7 x width , height / 3 ).

We also know the location of the corners of the eyes in the original images; they are landmarks 36 and 45 respectively. We can therefore calculate a **similarity transform** ( rotation , translation and scale ) that transforms the points from the **input coordinate systems** to the **output coordinate system**.The table below shows the original 3000 X 2300 image on the left and the transformed 600 X 600 image on the right.
<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-similaritytransform.jpg"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-similaritytransform.jpg"/></a></center>
    <center>Figure 4 : Similarity transform used to convert the input image of size 3000×2300 to output image coordinates of size 600×600.</center>

&nbsp;
To find the similarity transform that will transform the points from the **input image coordinates** to the **output image coordinates**. To do this, we can use **estimateAffinePartial2D** function.

```cpp
cv::Mat cv::estimateAffinePartial2D	(	InputArray 	from,
InputArray 	to,
OutputArray 	inliers = noArray(),
int 	method = RANSAC,
double 	ransacReprojThreshold = 3,
size_t 	maxIters = 2000,
double 	confidence = 0.99,
size_t 	refineIters = 10 
)	
```
 
Where,

- **`from`** - First input 2D point set.
- **`to`** - Second input 2D point set.
- **`inliers`** - Output vector indicating which points are inliers.
- **`method`** - Robust method used to compute transformation. The following methods are possible:
  - **`cv::RANSAC`** - RANSAC-based robust method
  - **`cv::LMEDS`** - Least-Median robust method RANSAC is the default method.
- **`ransacReprojThreshold`** - Maximum reprojection error in the RANSAC algorithm to consider a point as an inlier. Applies only to RANSAC.
- **`maxIters`** - The maximum number of robust method iterations.
- **`confidence`** - Confidence level, between 0 and 1, for the estimated transformation. Anything between 0.95 and 0.99 is usually good enough. Values too close to 1 can slow down the estimation significantly. Values lower than 0.8-0.9 can result in an incorrectly estimated transformation.
- **`refineIters`** - Maximum number of iterations of refining algorithm (Levenberg-Marquardt). Passing 0 will disable refining, so the output matrix will be output of robust method.

There is one little problem though. OpenCV requires you to supply at least 3 point pairs. We can simply hallucinate a third point such that it forms an equilateral triangle with the two known points and then use **estimateAffinePartial2D** as if we had three points pairs.

Once a similarity transform is calculated, it can be used to transform the input image and the landmarks to the output coordinates. The image is transformed using **warpAffine** and the points are transformed using the **transform** function.


왼쪽 눈, 오른쪽 눈 landmark (2개) + dummy point (1개) 기준 간단한 Similarity transformation을 찾은 후 (`**estimateAffinePartial2D**`),
모든 landmark와 input image를 output coordinate로 transform(`**warpAffine*`).

## <font style = "color:rgb(50,120,229)">Step 3 : Face Alignment</font>

In the previous step, we were able to transform all the images and the landmarks to the output image coordinates. All the images are now of the same size, and the two corners of the eyes are aligned. It may be tempting to obtain the average image by averaging pixel values of these aligned images. However if you did this, you will end up with an image shown just below. Sure, the eyes are aligned, but other facial features are misaligned.
<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-naiveFaceAveraging.jpg"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-naiveFaceAveraging.jpg"/></a></center>
<center>Figure 5 : Result of naive face averaging</center>

&nbsp;
Next, we use a trick to align all the facial features. If we knew which point in one input image corresponded to which point in another input image we could easily align the two images perfectly. However, we do not have that information. We only know the locations of 68 corresponding points in the input images. We will use these 68 points to divide the images into triangular regions. and align these regions before averaging pixel values.

더 정교한 alignment를 위해서 Delaunay Triangluation 결과를 `Mean Face Points`로 Transform 한다. 

```
Delaunay triangulation allows us to break the image into triangles.  The result of Delaunay triangulation is a list of triangles represented by the indices of points in the 76 points ( 68 face points + 8 boundary points ) array. In the triangulation example shown below in the box, we see than landmarks 62, 68 and 60 form a triangle, and 32, 50, and 49 form another triangle and so on.
```

### <font style = "color:rgb(50,120,229)">Warp Triangles</font>

In the previous step we calculated the average location of facial landmarks and used these locations to calculate a Delaunay triangulation to divide the image into triangles. In Figure 7 below, the left image shows Delaunay triangles on the transformed input image and the middle image shows the triangulation on the average landmarks. Note that triangle 1 in the left image corresponds to triangle 1 in the middle image. The three vertices of triangle 1 in the left image and the corresponding vertices in the middle image can be used to calculate an affine transform. This affine transform can be used to transform all pixels inside triangle 1 in the left image to triangle 1 in the middle image. This procedure when repeated for every triangle in the left image, results in the right image. The right image is simply the left image warped to the average face.
<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-warpingBasedDelauny.jpg"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m4-warpingBasedDelauny.jpg"/></a></center>
<center>Figure 7 : Image Warping based on Delaunay Triangulation.</center>


~~`boundary에서 연속성 문제는 없을까?`~~

## <font style = "color:rgb(50,120,229)">Step 4 : Face Averaging</font>

The previous step, when applied to all input images, gives us images that are correctly warped to the average image landmark coordinates. To calculate the average image, we can simply add the pixel intensities of all warped images and divide by the number of images. Figure 2 shows the result of this averaging. It looks much better than the average we obtained in Figure 5.
