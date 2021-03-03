# OpenCV - cpp

1. `core module` : The Core Functionality - basic building blocks of the library  
2. `imgproc module` : Image Processing - image processing functions
3. `highgui, imgcodecs, videoio modules` : Application utils- application utils (GUI, image/video input/output)
4. `calib3d module` : Camera calibration and 3D reconstruction - extract 3D world information from 2D images
5. `feature2d module` : 2D Features framework - feature detectors, descriptors and matching framework
6. `dnn module` : Deep Neural Networks - infer neural networks using built-in dnn module
7. `gapi module` : Graph API - graph-based approach to computer vision algorithms building

----------------------------------------------------

```cpp
// Include libraries
#include <opencv2/opencv.hpp>
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
using namespace std;
using namespace cv;

Mat cv::imread	(	const String & 	filename,
int 	flags = IMREAD_COLOR 
)

testImage.type()
testImage.size()

cout << "image size = " << img.size() << endl;
cout << "image channels = " << img.channels();

Mat imageCopy = image.clone();
```

#### <font style = "color:rgb(200,0,0)">NOTE</font>
It should be kept in mind that In OpenCV, size is represented as a tuple of `widthxheight` or `#columnsX#rows`.

## <font style="color:rgb(50,120,229)">Accessing Pixels</font>
In the above testImage, we saw that the first pixel has a value of 1. Let us check it.

Note that, we have zero-based indexing and we can access the first element using the index (0,0). the first index will be the row number and second index is column number.

```cpp
cout << (int)testImage.at<uchar>(0,0);
```

## <font style="color:rgb(50,120,229)">Modifying pixel values</font>
Similarly for modifying the value of a pixel, we can simply assign the value to the pixel. 

Let's change the value of the first element and check if the image is updated.

```cpp
testImage.at<uchar>(0,0)=200;

cout << testImage;
```

## <font style="color:rgb(50,120,229)">Manipulating Group of Pixels</font>
So, now we know how to manipulate individual pixels. But what about a region or group of pixels? It can be done using range function. It is a template class in OpenCV which is accepted by cv::Mat constructor. 

```cpp
Mat test_roi = testImage(Range(0,2),Range(0,4));
cout << "Original Matrix\n" << testImage << endl << endl;

cout << "Selected Region\n" << test_roi;

testImage(Range(0,2),Range(0,4)).setTo(111);
```


## <font style="color:rgb(50,120,229)">2. OpenCV's imshow</font>
This function will be used when we are running the C++ code from command line.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>

```cpp
void cv::imshow	(	const String & 	winname,
InputArray 	mat 
)
```
**Parameters**
- **`winname`** - Name of the window.
- **`mat`** - Image to be displayed.
```


## <font style="color:rgb(50,120,229)">1. cv::namedWindow</font>

This function is used to create a display window with a specific name. This name is provided as the first argument of this function. The second argument is a flag which decides whether the window can be **resized** (**`WINDOW_NORMAL`**) or it should be **fixed** to match the image size (**`WINDOW_AUTOSIZE`** - **Default flag**).

### <font style = "color:rgb(8,133,37)">Function Syntax</font>
[**`Docs`**](https://docs.opencv.org/4.1.0/d7/dfc/group__highgui.html#ga5afdf8410934fd099df85c75b2e0888b)
```cpp
void cv::namedWindow	(	const String & 	winname,
int 	flags = WINDOW_AUTOSIZE 
)
```
**Parameters**
- **`winname`** - Name of the window in the window caption that may be used as a window identifier.
- **`flags`** - Flags of the window

**<font color=green>Can you think of any situation where you would prefer to have a resizable display window?</font>**

## <font style="color:rgb(50,120,229)">2. cv::waitKey</font>

This function is widely used in image as well as video processing. It is a **keyboard binding function**. Its only argument is time in **milliseconds**. The function waits for specified milliseconds for any keyboard event. If you press any key in that time, the program continues. If **0** is passed, it waits **indefinitely** for a key stroke. It can also be set to detect specific key strokes which can be quite useful in video processing applications, as we will see in later sections.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>
[**`Docs`**](https://docs.opencv.org/4.1.0/d7/dfc/group__highgui.html#ga5628525ad33f52eab17feebcfba38bd7)
```cpp
int cv::waitKey	(	int 	delay = 0	)	
```
**Parameters**
- **`delay`** - Delay in milliseconds. 0 is the special value that means "forever".


## <font style="color:rgb(50,120,229)">3. cv::destroyWindow</font>

This function is used to destroy or close a particular display window. The name of the window is provided as an argument to this function.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>
[**`Docs`**](https://docs.opencv.org/4.1.0/d7/dfc/group__highgui.html#ga851ccdd6961022d1d5b4c4f255dbab34)
```cpp
void cv::destroyWindow	(	const String & 	winname	)
```
**Parameters**
- **`winname`** - Name of the window to be destroyed

## <font style="color:rgb(50,120,229)">4. cv::destroyAllWindows</font>

This function is used to destroy all display windows. This function does not take any arguments.

### <font style = "color:rgb(8,133,37)">Function Syntax</font>
[**`Docs`**](https://docs.opencv.org/4.1.0/d7/dfc/group__highgui.html#ga6b7fc1c1a8960438156912027b38f481)
```cpp
void cv::destroyAllWindows	(		)
```

# <font style="color:rgb(50,120,229)">Write the Image to Disk</font>
In most cases, you would want to save the output of your application. We do this using the imwrite function.


### <font style = "color:rgb(8,133,37)">Function Syntax</font>
[**`Docs`**](https://docs.opencv.org/4.1.0/d4/da8/group__imgcodecs.html#gabbc7ef1aa2edfaa87772f1202d67e0ce)
```cpp
bool cv::imwrite	(	const String & 	filename,
InputArray 	img,
const std::vector< int > & 	params = std::vector< int >() 
)
```

**Parameters**

- **filename** - String providing the relative or absolute path where the image should be saved.
- **img** - Image matrix to be saved.
- **params** - Additional information, like specifying the JPEG compression quality etc. Check the full list [**here**](https://docs.opencv.org/4.1.0/d4/da8/group__imgcodecs.html#ga292d81be8d76901bff7988d18d2b42ac)


## <font style="color:rgb(50,120,229)">Image Channels</font>

We can access the channels using an opencv function `split()` and merge them into a color image using `merge()`.

`bgr`

 ```cpp
Mat imgChannels[3];
split(img, imgChannels);
```

## <font style="color:rgb(50,120,229)">Access Color pixel</font>

```cpp
cout << testImage.at<Vec3b>(0,0); // [1, 1, 1] for example
testImage.at<Vec3b>(0,0) = Vec3b(0,255,255);
testImage(Range(0,3),Range(0,3)).setTo(Scalar(255,0,0));
```

### ConvertTo

```cpp
//Converting from float to unsigned char
image.convertTo(image, CV_8UC3, 1.0/scalingFactor, shift);
contrastHigh.convertTo(contrastHigh, CV_64F);
// operation도 가능.
contrastHigh = contrastHigh * (1+contrastPercentage/100.0);
```

Brightness
```cpp
int brightnessOffset = 50;

// Add the offset for increasing brightness
Mat brightHigh = image;
Mat brightHighChannels[3];
split(brightHigh, brightHighChannels);

for (int i=0; i < 3; i++){
    add(brightHighChannels[i],brightnessOffset,brightHighChannels[i]);
}

merge(brightHighChannels,3,brightHigh);
```