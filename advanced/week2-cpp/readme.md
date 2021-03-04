# week 2

```cpp

// <Function Basis>
// Alpha blending using multiply and add functions
Mat& blend(Mat& alpha, Mat& foreground, Mat& background, Mat& outImage)
{
    Mat fore, back;
    multiply(alpha, foreground, fore);
    multiply(Scalar::all(1.0)-alpha, background, back);
    add(fore, back, outImage);
    
    return outImage;
}

// <Direct Access>
// Alpha Blending using direct pointer access
Mat& alphaBlendDirectAccess(Mat& alpha, Mat& foreground, Mat& background, Mat& outImage)
{

    int numberOfPixels = foreground.rows * foreground.cols * foreground.channels();

    float* fptr = reinterpret_cast<float*>(foreground.data);
    float* bptr = reinterpret_cast<float*>(background.data);
    float* aptr = reinterpret_cast<float*>(alpha.data);
    float* outImagePtr = reinterpret_cast<float*>(outImage.data);

    int i,j;
    for ( j = 0; j < numberOfPixels; ++j, outImagePtr++, fptr++, aptr++, bptr++)
    {
        *outImagePtr = (*fptr)*(*aptr) + (*bptr)*(1 - *aptr);
    }

    return outImage;
}

// 참고로, 
// .data에 접근
// reinterpret_cast는 임의의 포인터 타입끼리 변환을 허용하는 캐스트 연산자 입니다.
// 함수 사용이 더 빠르다.

```

## Warping

## <font style = "color:rgb(50,120,229)">Nonlinear Transforms</font>

Interesting image warps are almost always nonlinear in nature. For example, the cartoon faces displayed below would never be possible with a single linear transformation to the entire image. 

<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w2-m4-cartoonfaces.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w2-m4-cartoonfaces.png" width=550/></a></center>

&nbsp;
In a non-linear warp, a point $(x_i,y_i)$ in the input image, maps to a point $(x_o,y_o)$ by a non-linear mapping $M$ function.

$$
\begin{align*}x_o &= M_x( x_i, y_i) \\ y_o &= M_y(x_i, y_i)\end{align*}
$$

in 2D, the warping function can be approximated by many triangles. To accomplish this, the image can be divided into non-overlapping triangles. The warping function is calculated only at the vertices of the triangles to obtain their locations in the output image. Now, we have a collection of triangles in the input image and the corresponding triangles in the output image.

<center><a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png" width=550 /></a></center>


Recall, there is a unique affine transform that maps one triangle to any other triangle as long as the three points are not collinear ( i.e. they do not lie on a line ). So we can calculate one affine transform per input-output triangle pair and warp all pixels inside the triangle in the source image to the triangle in the destination image. This leads to huge efficiency gain without much loss in accuracy. 

In many graphics packages, warping a triangle is a standard operation that can be achieved using a single line of code.

Single Triangle은 Warp Affine을 사용하여 Transformation 가능.

```cpp
vector<Point2f> tri1Cropped, tri2Cropped;
  vector<Point> tri2CroppedInt;
  for(int i = 0; i < 3; i++)
  {
    tri1Cropped.push_back( Point2f( tri1[i].x - r1.x, tri1[i].y -  r1.y) );
    tri2Cropped.push_back( Point2f( tri2[i].x - r2.x, tri2[i].y - r2.y) );

    // fillConvexPoly needs a vector of Point and not Point2f
    tri2CroppedInt.push_back( Point((int)(tri2[i].x - r2.x), (int)(tri2[i].y - r2.y)) );

  }

  // Given a pair of triangles, find the affine transform.
  Mat warpMat = getAffineTransform( tri1Cropped, tri2Cropped );

  // Apply the Affine Transform just found to the src image
  Mat img2Cropped = Mat::zeros(r2.height, r2.width, img1Cropped.type());
  warpAffine( img1Cropped, img2Cropped, warpMat, img2Cropped.size(), INTER_LINEAR, BORDER_REFLECT_101);

```

## <font style = "color:rgb(50,120,229)"> Delaunay Triangulation</font>

&nbsp;

<center><a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png"/></a></center>

**<center>Figure 1. Left : The image with input points shown as red dots. Right: The white lines display Delaunay triangulation calculated using the points. </center>**

Given a set of points in a plane, a triangulation refers to the subdivision of the plane into triangles, with the points as vertices. In Figure 1, we see a set of points on the left image shown using red dots and the triangulation shown using white lines. 

A set of points can have many possible triangulations, but Delaunay triangulation stands out because it has some nice properties. 

In a Delaunay triangulation, triangles are chosen such that **no point is inside the circumcircle of any triangle**. Figure 2. shows Delaunay triangulation of 4 points A, B, C and D.  The difference between the top and the bottom images is that in the top image, point C is further away from line BD as compared to the bottom image. This movement of point C changes the Delaunay triangulation. 

In the top image, for the triangulation to be a valid Delaunay triangulation, point C should be outside the circumcircle of triangle ABD, and point A should be outside the circumcircle of triangle BCD.

<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-angles.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-angles.png"/></a></center>
<center>**Figure 2 : Delaunay triangulation favors small angles**</center>


```cpp
// In a vector of points, find the index of point closest to input point.
static int findIndex(vector<Point2f>& points, Point2f &point)
{
  int minIndex = 0;
  double minDistance = norm(points[0] - point);
  for(int i = 1; i < points.size(); i++)
  {
    double distance = norm(points[i] - point);
    if( distance < minDistance )
    {
      minIndex = i;
      minDistance = distance;
    }

  }
  return minIndex;
}

// Write delaunay triangles to file
static void writeDelaunay(Subdiv2D& subdiv, vector<Point2f>& points, const string &filename)
{

  // Open file for writing
  std::ofstream ofs;
  ofs.open(filename);

  // Obtain the list of triangles.
  // Each triangle is stored as vector of 6 coordinates
  // (x0, y0, x1, y1, x2, y2)
  vector<Vec6f> triangleList;
  subdiv.getTriangleList(triangleList);

  // Will convert triangle representation to three vertices
  vector<Point2f> vertices(3);

  // Loop over all triangles
  for( size_t i = 0; i < triangleList.size(); i++ )
  {
    // Obtain current triangle
    Vec6f t = triangleList[i];

    // Extract vertices of current triangle
    vertices[0] = Point2f(t[0], t[1]);
    vertices[1] = Point2f(t[2], t[3]);
    vertices[2] = Point2f(t[4], t[5]);

    // Find landmark indices of vertices in the points list
    int landmark1 = findIndex(points, vertices[0]);
    int landmark2 = findIndex(points, vertices[1]);
    int landmark3 = findIndex(points, vertices[2]);
    // save to file.

    ofs << landmark1 << " " << landmark2 << " " << landmark3 << endl;

  }
  ofs.close();
}
```

- opencv의 `subdiv2d` class 사용