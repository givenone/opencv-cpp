## <font style = "color:rgb(50,120,229)">Why is Face-Swap difficult ?</font>

The human brain treats human faces as a special category and has specialized machinery to process faces. We are very good at analyzing faces and can easily detect a fake face. It is easy to computationally replace a face in one image with a different face if you want to do it for giggles, but extremely difficult to do if you want to do it completely automatically at a quality that will fool people consistently. After all, we are trying to fool one of the most advanced cognitive machinery in the human brain.

Checkout how Snapchat’s faceswap filter went wrong on some occasions and created some buzz [here](https://www.buzzfeed.com/genamourbarrett/snapchats-that-prove-face-swap-is-the-most-terrifying-upd?utm_term=.iyg3dDkpq#.uu5BEo7DO) and [here](http://www.boredpanda.com/funny-snapchat-face-swaps/). 

# <font style = "color:rgb(50,120,229)">FaceSwap : Step by Step</font>

**cv의 seamless clone 이용**

## <font style = "color:rgb(50,120,229)">1. Face Alignment</font>

To replace one face with another, we first need to place one face approximately on top of the other so that it covers the face below. An example is shown in Figure 3.

<center> <a href="https://www.learnopencv.com/wp-content/uploads/2018/01/opv4face-w5-m2-faceAlignment.png"><img src = "https://www.learnopencv.com/wp-content/uploads/2018/01/opv4face-w5-m2-faceAlignment.png"/></a></center>

Figure 3. Face Alignment. Left : Detected facial landmarks and convex hull. Middle : Delaunay triangulation of points on convex hull. Right : Face alignment by affine warping triangles.

### <font style = "color:rgb(50,120,229)">Facial Landmark Detection</font>

The geometry of the two faces are very different and so we need to warp the source face so that it covers the target face, but we also want to make sure we do not warp it beyond recognition. To achieve this we first detect facial landmarks on both images using Dlib. However, unlike in Face Morphing, we do not and should not use all the points for face alignment. We simply need the points on the outer boundary of the face as shown in the image. We also take 12 points around the mouth for better alignment.

## <font style = "color:rgb(50,120,229)">2. Find Convex Hull </font>

In Computer Vision and Math jargon, the boundary of a collection of points or shape is called a "hull". A boundary that does not have any concavities is called a “Convex Hull”. In Figure 3. the image on the left shows facial landmarks detected using dlib in red and the convex hull of the points is shown in blue. The convex hull of a set of points can be calculated using OpenCV’s `[convexHull](https://docs.opencv.org/4.1.0/d3/dc0/group__imgproc__shape.html#ga014b28e56cb8854c0de4a211cb2be656)` function.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>

```cpp

void cv::convexHull	(	InputArray 	points,
OutputArray 	hull,
bool 	clockwise = false,
bool 	returnPoints = true 
)	
```

Where,

- **`points`** - Input 2D point set.
- **`hull`** - Output convex hull. It is either an integer vector of indices or vector of points. In the first case, the hull elements are 0-based indices of the convex hull points in the original array (since the set of convex hull points is a subset of the original point set). In the second case, hull elements are the convex hull points themselves.
- **`clockwise`** - Orientation flag. If it is true, the output convex hull is oriented clockwise. Otherwise, it is oriented counter-clockwise. The assumed coordinate system has its X axis pointing to the right, and its Y axis pointing upwards.
- **`returnPoints`** - Operation flag. In case of a matrix, when the flag is true, the function returns convex hull points. Otherwise, it returns indices of the convex hull points.

## <font style = "color:rgb(50,120,229)">3. Delaunay Triangulation</font> 

The next step in alignment is to do a Delaunay triangulation of the points on the convex hull. The triangulation is shown in the middle image in Figure 3. This allows us to divide the face into smaller parts. We had covered delaunay triangulation in detail in the previous week.

-> 이를 사용해서 mask 이미지를 만들 수 있다!

## <font style = "color:rgb(50,120,229)">4. Affine warp triangles</font> 

The final steps of face alignment is to consider corresponding triangles between the source face and the target face, and affine warp the source face triangle onto the target face. We had covered warping of triangles in the previous week. However, as you can see in the right image of Figure 3, aligning the face and slapping one face on top of the other hardly looks natural. The seams are visible because of lighting and skin tone differences between the two images. The next step shows how to seamlessly combine the two images.

여기까지 Alignment Step.
-----------------------------


## <font style = "color:rgb(50,120,229)">5. Blending of images</font>

The difference in lighting and skin tone between the source and destination images makes the warped image look unnatural. For rectifying this, we take two different routes for images and videos. This is because the method used for images is pretty slow and cannot be applied to videos as it will bring down the FPS drastically. The two methods are discussed below.

### <font style = "color:rgb(50,120,229)">Seamless Cloning</font>

Good technical ideas are like good magic. Good magicians use a combination of physics, psychology and good old sleight of hand to achieve the incredible. Image warping alone looks pretty bad. Combine it with Seamless Cloning and the results are magical! 

It is a feature in OpenCV 3 that allows you to seamlessly clone parts of the source image ( identified by a mask ) onto a destination image.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>

```cpp

void cv::seamlessClone	(	InputArray 	src,
InputArray 	dst,
InputArray 	mask,
Point 	p,
OutputArray 	blend,
int 	flags 
)	
```

Where, 

- **`src`** - Input 8-bit 3-channel image.
- **`dst`** - Input 8-bit 3-channel image.
- **`mask`** - Input 8-bit 1 or 3-channel image.
- **`p`** - Point in dst image where object is placed.
- **`blend`** - Output image with the same size and type as dst.
- **`flags`** - Cloning method that could be cv2.NORMAL_CLONE, cv2.MIXED_CLONE or cv2.MONOCHROME_TRANSFER


========================================================

# Beardify Filter

-> 랜드마크 기준 Alignment (convex hull)
-> Delaunay Triangle 사용한 Masking
-> warping과 alpha blending.

===========================================

# Aging Filter

Wrinkle Image와 Input Image의 Blending으로 이루어짐.

- 둘다 warping.
- 눈, 코, 입 등은 빼도록 mask 이미지를 만든다. (Convex HULL 구한 후 cv의 `fillConvexPoly` 함수 사용)
- CV의 seamlessClone의 `MIXED_CLONE` 옵션을 사용!
- cloning 후에는 soft edge를 위해 blurred mask를 사용하여 old 이미지와 alpha blending.
- **desaturate the final image**
  - Finally, we desaturate the final image. It has two effects. First, aging leads to a paler skin and desaturation simulates that effect. Second, the entire photo looks discolored and old. The overall effect is that we make the person look old and from an old time.


![aging](aging.png)
---결과 사진 ---

### <font style = "color:rgb(50,120,229)">How to Generate a Mask from Points</font>

Given a set of points on a polygon, we know we can create a mask using fillPoly or fillConvexPoly in OpenCV. 

Facial landmarks do not form a polygon. To find a face mask, we need to find a Convex Hull of all points. 

### <font style = "color:rgb(50,120,229)">What is a Convex hull?</font>
<center><a href="https://www.learnopencv.com/wp-content/uploads/2018/01/opcv4face-w5-m5-convexHull.png"><img src="https://www.learnopencv.com/wp-content/uploads/2018/01/opcv4face-w5-m5-convexHull.png" width=400/></a></center>

&nbsp;
The image on the left shows a few red points. The blue line is the Convex Hull. The convex hull of has two important properties

1. It is a **convex polygon.** This means that all internal angles are less than 180 degrees. The outer boundary has no concavities. 

2. It is a **hull**. The word hull is used in many contexts and almost everywhere it refers to the outer covering of something. In this case, the hull refers to the outer boundary of the points. All the points are either inside or on the hull. 

In OpenCV, you can use the function `convexHull` to find the convexHull.

=============================================

# Moving Least Square (MLS)

[논문](https://people.engr.tamu.edu/schaefer/research/mls.pdf) 참고.

$n$ control points $p_1, p_2, \ldots p_n$ and they are moved to new locations given by $q_1, q_2, \ldots q_n$. The warping function $f$.

Control Point 기준으로 Warping이 결정된다. Deformation 하고 싶은 control point는 $qi$를 조정. 움직이고 싶지 않으면 $qi$ == $pi$ 가 되도록 조정.

## <font style = "color:rgb(50,120,229)">How do you choose control points in MLS?</font>

In MLS, the warp is calculated so that when it is applied to the input control points, we get the output control points. There are two kinds of regions where you should add control points. 

1. Region you want to deform. Let’s say in a fun application called "Pinocchio", you want to elongate the nose. You should put a control point on the tip of the nose. 

2. Region you do not want to deform : In the same application, you may want to make sure the rest of the face is not deformed. So you put a few control points on the face. You may also want to make sure the boundary of the image is not distorted. So you put a few control points there as well. 

각 point에 대해 Normal Equation 사용하여 계산할 수 있지만, Pre-Computation을 통해 속도 향상할 수 있다. (아래 참고)

```Notice that, given a point v, everything in Aj can be precomputed yielding a simple, weighted sum. Table 1 provides timing results for the examples in this paper, which shows that these deformations may be performed over 500 times per second in our examples
```