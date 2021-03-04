# Delaunay Triangluation

## <font style = "color:rgb(50,120,229)">What is Delaunay Triangulation?</font>

&nbsp;

<center><a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-dealunyTriangle-1.png"/></a></center>

**<center>Figure 1. Left : The image with input points shown as red dots. Right: The white lines display Delaunay triangulation calculated using the points. </center>**

Given a set of points in a plane, a triangulation refers to the subdivision of the plane into triangles, with the points as vertices. In Figure 1, we see a set of points on the left image shown using red dots and the triangulation shown using white lines. 

A set of points can have many possible triangulations, but Delaunay triangulation stands out because it has some nice properties. 

In a Delaunay triangulation, triangles are chosen such that **no point is inside the circumcircle of any triangle**. Figure 2. shows Delaunay triangulation of 4 points A, B, C and D.  The difference between the top and the bottom images is that in the top image, point C is further away from line BD as compared to the bottom image. This movement of point C changes the Delaunay triangulation. 

In the top image, for the triangulation to be a valid Delaunay triangulation, point C should be outside the circumcircle of triangle ABD, and point A should be outside the circumcircle of triangle BCD.

<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-angles.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-angles.png"/></a></center>
<center>**Figure 2 : Delaunay triangulation favors small angles**</center>

&nbsp;
An interesting property of Delaunay triangulation is that it does not favor **"skinny"** triangles ( i.e. triangles with one large angle ).

Figure 2 shows how the triangulation changes to pick "fat" triangles when the points are moved. In the top image, the points B and D have their x-coordinates at x = 1.5, and in the bottom image they are moved to the right to x = 1.75. In the top image angles ABC and ABD are large, and Delaunay triangulation creates an edge between B and D splitting the two large angles into smaller angles ABD, ADB, CDB, and CBD. On the other hand in the bottom image, the angle BCD is too large, and Delaunay triangulation creates an edge AC to divide the large angle.


## Algorithm to find Delaunay Triangulation

There are many algorithms to find the Delaunay triangulation of a set of points. The most obvious ( but not the most efficient ) one is to start with any triangulation, and check if the circumcircle of any triangle contains another point. If it does, flip the edges ( as shown in Figure 2. ) and continue until there are no triangles whose circumcircle contains a point.

Any discussion on Delaunay triangulation has to include Voronoi diagrams because the Voronoi diagram of a set of points is mathematical dual to its Delaunay triangulation.

[algorithm explained with pseudocode](https://towardsdatascience.com/delaunay-triangulation-228a86d1ddad)

```
subroutine triangulate
input : vertex list
output : `triangle list`
   initialize the triangle list
   determine the `supertriangle`
   add supertriangle vertices to the end of the vertex list
   add the supertriangle to the triangle list
   for each sample point in the vertex list
      initialize the `edge buffer`
      for each triangle currently in the triangle list
         calculate the triangle circumcircle center and radius
         if the point lies in the triangle circumcircle then
            add the three triangle edges to the edge buffer
            remove the triangle from the triangle list
         endif
         (모든 triangle에 대해 지금 보는 point 외접원에 들어있는지 체크.)
         (만약 들어있다면, edge 추가하고, 기존 triangle 삭제)
      endfor
      delete all doubly specified edges from the edge buffer
         this leaves the edges of the enclosing polygon only
         (가장 중요한 작업)
      add to the triangle list all triangles formed between the point 
         and the edges of the enclosing polygon
   endfor
   remove any triangles from the triangle list that use the supertriangle vertices
   remove the supertriangle vertices from the vertex list
end
 Suppose an edge e in E , if e meets the following conditions, it is called a Delaunay side:
```



## Voronoi Diagram

## <font style = "color:rgb(50,120,229)">What is a Voronoi Diagram ?</font>

&nbsp;

<center> <a href="https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-voronoiDiagram.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2017/12/opcv4face-w4-m3-voronoiDiagram.png"/></a></center>


**<center>Figure 3: Left : The image with input points shown as red dots. Right: The Voronoi diagram calculated using the input points.</center>** 


Given a set of points in a plane, a Voronoi diagram partitions the space such that the boundary lines are **equidistant** from neighboring points. Figure 3. shows an example of a Voronoi diagram. You will notice that every boundary line passes through the center of two points. 

> <font style="font-family:Poiret one" size="+2"> If you connect the points in neighboring Voronoi regions, you get a Delaunay triangulation!</font>

Given a set of points, you can calculate the Delaunay Triangulation or Voronoi Diagram using the class **Subdiv2D**. Here are the steps. A complete working example is shown in the next section.


## cv2.Subdiv2D()

```cpp
vector<Point2f> points;
// This is how you can add one point.
points.push_back(Point2f(x,y));

Mat img = imread("image.jpg");
Size size = img.size();
Rect rect(0, 0, size.width, size.height);

Subdiv2D subdiv(rect);
```

## Delaunay

```cpp
// Draw delaunay triangles
static void draw_delaunay( Mat& img, Subdiv2D& subdiv, Scalar delaunay_color )
{

    vector<Vec6f> triangleList; // 결과 저장
    subdiv.getTriangleList(triangleList);
    vector<Point> pt(3);
    Size size = img.size();
    Rect rect(0,0, size.width, size.height);

    for( size_t i = 0; i < triangleList.size(); i++ )
    {
        Vec6f t = triangleList[i];
        pt[0] = Point(cvRound(t[0]), cvRound(t[1]));
        pt[1] = Point(cvRound(t[2]), cvRound(t[3]));
        pt[2] = Point(cvRound(t[4]), cvRound(t[5]));
        
        // Draw rectangles completely inside the image.
        if ( rect.contains(pt[0]) && rect.contains(pt[1]) && rect.contains(pt[2]))
        {
            line(img, pt[0], pt[1], delaunay_color, 1, CV_AA, 0);
            line(img, pt[1], pt[2], delaunay_color, 1, CV_AA, 0);
            line(img, pt[2], pt[0], delaunay_color, 1, CV_AA, 0);
        }
    }
}

for( vector<Point2f>::iterator it = points.begin(); it != points.end(); it++)
    {
        subdiv.insert(*it);
        // Show animation
        if (animate)
        {
            Mat img_copy = img_orig.clone();
            // Draw delaunay triangles
            draw_delaunay( img_copy, subdiv, delaunay_color );
            imshow(win_delaunay, img_copy);
            waitKey(100);
        }
        
    }
```

```cpp
//Draw voronoi diagram
static void draw_voronoi( Mat& img, Subdiv2D& subdiv )
{
    vector<vector<Point2f> > facets;
    vector<Point2f> centers;
    subdiv.getVoronoiFacetList(vector<int>(), facets, centers);
    /*
    Parameters

    [idx]	Vector of vertices IDs to consider. For all vertices you can pass empty vector.
    [facetList] 	Output vector of the Voronoi facets.
    [facetCenters]	Output vector of the Voronoi facets center points.
    */
    vector<Point> ifacet;
    vector<vector<Point> > ifacets(1);

    for( size_t i = 0; i < facets.size(); i++ )
    {
        ifacet.resize(facets[i].size());
        for( size_t j = 0; j < facets[i].size(); j++ )
            ifacet[j] = facets[i][j];

        Scalar color;
        color[0] = rand() & 255;
        color[1] = rand() & 255;
        color[2] = rand() & 255;
        fillConvexPoly(img, ifacet, color, 8, 0);

        ifacets[0] = ifacet;
        polylines(img, ifacets, true, Scalar(), 1, CV_AA, 0);
        circle(img, centers[i], 3, Scalar(), CV_FILLED, CV_AA, 0);
    }
}
```