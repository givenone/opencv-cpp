# Distortion

Meshgrid를 이용하여 Inverse-Mapping 해주면 됨.

`cv::remap` 함수 이용!
```cpp
#include <opencv2/imgproc.hpp>

void cv::remap	(	InputArray 	src,
                    OutputArray 	dst,
                    InputArray 	map1,
                    InputArray 	map2,
                    int 	interpolation,
                    int 	borderMode = BORDER_CONSTANT,
                    const Scalar & 	borderValue = Scalar() 
                    )		
```

Applies a generic geometrical transformation to an image.

The function remap transforms the source image using the specified map:

`dst(x,y)=src(mapx(x,y),mapy(x,y))`

where values of pixels with non-integer coordinates are computed using one of available interpolation methods. mapx and mapy can be encoded as separate floating-point maps in map1 and map2 respectively, or interleaved floating-point maps of (x,y) in map1, or fixed-point maps created by using convertMaps. The reason you might want to convert from floating to fixed-point representations of a map is that they can yield much faster (2x) remapping operations. In the converted case, map1 contains pairs (cvFloor(x), cvFloor(y)) and map2 contains indices in a table of interpolation coefficients.

This function cannot operate in-place.

Parameters

-  `src`	Source image.
- `dst`	Destination image. It has the same size as map1 and the same type as src .
- `map1`	The first map of either (x,y) points or just **x** values having the type CV_16SC2 , CV_32FC1, or CV_32FC2. See convertMaps for details on converting a floating point representation to fixed-point for speed.
- `map2`	The second map of **y** values having the type CV_16UC1, CV_32FC1, or none (empty map if map1 is (x,y) points), respectively.
- `interpolation`	Interpolation method (see InterpolationFlags). The methods INTER_AREA and INTER_LINEAR_EXACT are not supported by this function.
- `borderMode`	Pixel extrapolation method (see BorderTypes). When borderMode=BORDER_TRANSPARENT, it means that the pixels in the destination image that corresponds to the "outliers" in the source image are not modified by the function.
- `borderValue`	Value used in case of a constant border. By default, it is 0.

```cpp
Mat barrel(Mat &src, float k)
{
  int w = src.cols;
  int h = src.rows;

  // Meshgrid of destiation image
  Mat Xd = cv::Mat::zeros(src.size(), CV_32F);
  Mat Yd = cv::Mat::zeros(src.size(), CV_32F);

  float Xu, Yu;
  for (int y = 0; y < h; y++)
  {
    for (int x = 0; x < w; x++)
    {
      // Normalize x and y
      Xu = ( (float) x / w )- 0.5;
      Yu = ( (float) y / h )- 0.5;
    
      // Radial distance from center
      float r = sqrt(Xu * Xu + Yu * Yu);
      
      // Implementing the following equation
      // dr = k * r * cos(pi*r)
      float dr = k * r * cos(M_PI * r);
      
      // Outside the maximum radius dr is set to 0
      if (r > 0.5) dr = 0;
    
      // Remember we need to provide inverse mapping to remap
      // Hence the negative sign before dr
      float rn = r - dr;

      // Applying the distortion on the grid
      // Back to un-normalized coordinates  
      Xd.at<float>(y,x) =  w * (rn * Xu / r + 0.5);
      Yd.at<float>(y,x) = h * (rn * Yu / r + 0.5);
        
    }
  }

  // Interpolation of points
  Mat dst;
  cv::remap( src, dst, Xd, Yd, INTER_CUBIC, BORDER_CONSTANT, Scalar(0,0, 0) );
  return dst;


// 눈 부분만 추출.
Rect roiEyeRight ( (landmark_pts[43].x-radius)
                , (landmark_pts[43].y-radius)
                , ( landmark_pts[46].x - landmark_pts[43].x + 2*radius )
                , ( landmark_pts[47].y - landmark_pts[43].y + 2*radius ) );
Rect roiEyeLeft ( (landmark_pts[37].x-radius)
               , (landmark_pts[37].y-radius)
               , ( landmark_pts[40].x - landmark_pts[37].x + 2*radius )
               , ( landmark_pts[41].y - landmark_pts[37].y + 2*radius ) );
}
```