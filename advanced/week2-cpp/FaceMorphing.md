# <font style = "color:rgb(50,120,229)">Face Morphing</font>

Morphing is a technique which is widely used in motion pictures and animations to change one image or shape into another seamlessly. The idea behind Image Morphing is closely related to Face Averaging. 

<center> <img src ="https://www.learnopencv.com/wp-content/uploads/2019/10/hillary-bill.gif"/></a></center>



## <font style = "color:rgb(50,120,229)">Step 1 : Find Point Correspondences using Facial Landmark Detection</font>

As you might have guessed, we start by find facial landmarks using Dlib’s landmark detector. We have chosen images of former US Secretary of State Hillary Clinton and former US President Bill Clinton because the images are of very different sizes.

<table>
    <tr>
        <th><center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-facialLandmarkDetection1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-facialLandmarkDetection1.png" width=600/></a></center></th>
        <th><center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-facialLandmarkDetection2.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-facialLandmarkDetection2.png" width=500/></a></center></th>
    </tr>
    <tr>
    <td colspan="2"><center>Step 1: Facial Landmark Detection</center></td>
    </tr> 
</table>

## <font style = "color:rgb(50,120,229)">Step 3 : Delaunay Triangulation</font>

In addition to the 68 points, we add the 8 points on the boundary of the two images. This helps blend the background as well. Next we perform a Delaunay Triangulation on these 76 points. There are two sets of points corresponding to the two images. The two sets may result in a slightly different Delaunay triangulation. 

Which set of points should you use? Pause and guess the answer before you read on. 

The natural choice is to perform triangulation on the **average of the two set of points** as shown in the image below. The triangulation is calculated only once and stored as list of indices.
<table>
    <tr>
        <th><center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunypoints1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunypoints1.png"/></a></center></th>
        <th><center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunyTriangle.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunyTriangle.png"/></a></center></th>
        <th><center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunyPoints2.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-delaunyPoints2.png"/></a></center></th>
    </tr>
    <tr>
    <td colspan="3"><center>The points in the output coordinate system are shown in the left and right image. Delaunay triangulation is calculated on the average of the left and right shapes.</center></td>
    </tr> 
</table>

## <font style = "color:rgb(50,120,229)">Step 4 : Warping images and alpha blending</font>


**Create a morph using the following steps.**

* **Find location of feature points in morphed image**: In the morphed image M, we can find the locations of all 76 points (xm , ym) using equation (1).

* **Calculate affine transforms**: So we have a set of 76 points in image 1, another set of 76 points in image 2 and a third set of 76 points in the morphed image. We also know the triangulation defined over these points. Pick a triangle in image 1 and the corresponding triangle in the morphed image and calculate the affine transform that maps the three corners of the triangle in image 1 to the three corners of the corresponding triangle in the morphed image. In OpenCV, this can be done using **getAffineTransform**. Calculate an affine transform for every pair of triangles. Finally, repeat the process of image 2 and the morphed image.

* **Warp triangles**: For each triangle in image 1, use the affine transform calculated in the previous step to transform all pixels inside the triangle to the morphed image. Repeat this for all triangles in image 1 to obtain a warped version of image 1. Similarly, obtain a warped version for image 2. See the table below. 

* **Alpha blend warped images**: In the previous step we obtained warped version of image 1 and image 2. These two images can be alpha blended using equation (2), and this is your final morphed image. In the code I have provided warping triangles and alpha blending them is combined in a single step. See table below.


|<center>The left image shows the warped image1. The right image shows the warped image2. The middle image shows the final morphed image where the left and right warped images are blended based on alpha.</center>|
|------|
|<center><a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.25.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.25.png"/></a></center> | 
| <center>alpha = 0.25
Note that even though the right image is extremely distorted, it does not affect the final morph as much because its contribution is only equal to alpha (i.e. 25%).</center>     | 
|<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.5.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.5.png"/></a></center>|
|<center>alpha = 0.5</center>|
|<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.75-1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m5-warpedalpha.75-1.png"/></a></center>|
|<center>alpha = 0.75. Note that even though the left image is extremely distorted, it does not affect the final morph as much because its contribution is only equal to ( 1 - alpha )  (i.e. 25%).</center>


```cpp

// Returns 8 points on the boundary of a rectangle
void getEightBoundaryPoints(Size size, vector<Point2f>& boundaryPts)
{
  int h = size.height, w = size.width;
  boundaryPts.push_back(Point2f(0,0));
  boundaryPts.push_back(Point2f(w/2, 0));
  boundaryPts.push_back(Point2f(w-1,0));
  boundaryPts.push_back(Point2f(w-1, h/2));
  boundaryPts.push_back(Point2f(w-1, h-1));
  boundaryPts.push_back(Point2f(w/2, h-1));
  boundaryPts.push_back(Point2f(0, h-1));
  boundaryPts.push_back(Point2f(0, h/2));
}


// Calculate average points. Will be used for Delaunay triangulation.
vector<Point2f> pointsAvg;
for(int i = 0; i < points1.size(); i++)
{
  pointsAvg.push_back((points1[i] + points2[i])/2);
}
  
// 8 Boundary points for Delaunay Triangulation
vector <Point2f> boundaryPts;
getEightBoundaryPoints(size, boundaryPts);
  
for(int i = 0; i < boundaryPts.size(); i++)
{
  pointsAvg.push_back(boundaryPts[i]);
  points1.push_back(boundaryPts[i]);
  points2.push_back(boundaryPts[i]);
}
  
// Calculate Delaunay triangulation.
vector< vector<int> > delaunayTri;
calculateDelaunayTriangles(Rect(0,0,size.width,size.height), pointsAvg, delaunayTri); // 한 번만 구해놓으면 된다 (index list이므로 공통.)

Mat imgOut1, imgOut2;

// Triangle 기반 Warping.
warpImage(imgNorm1, imgOut1, points1, points, delaunayTri);
warpImage(imgNorm2, imgOut2, points2, points, delaunayTri);

// Blend warped images based on morphing parameter alpha
Mat imgMorph = ( 1 - alpha ) * imgOut1 + alpha * imgOut2;

```