# <font style = "color:rgb(50,120,229)">Facial Landmark Detection in Dlib</font>

Landmark detection is a two step process.

## <font style="color:rgb(50,120,229)">Face Detection </font>
In the first step, you need to detect the face in an image. For best results we should use the same face detector used in training the landmark detector.  The output of a face detector is a rectangle (x, y, w, h) that contains the face. 

We have seen that OpenCV’s face detector is based on HAAR cascades. Dlib’s face detector is based on Histogram of Oriented Gradients features and Support Vector Machines (SVM). 


## <font style="color:rgb(50,120,229)">Landmark detection</font>
The landmark detector finds the landmark points inside the face rectangle. 

```cpp
#include <dlib/opencv.h>
#include <dlib/image_processing.h>
using namespace dlib;

// Get the face detector
frontal_face_detector faceDetector = get_frontal_face_detector();

// The landmark detector is implemented in the shape_predictor class
// it can be used to detect other landmark points ( e.g. the four corners of a credit card ) when trained to do so.
shape_predictor landmarkDetector;

// Load the facial landmark detector model file. The information in the file makes the generic **`shape_predictor`** - a facial landmark detector. Loading the file is accomplished using the deserialize function.
string PREDICTOR_PATH = DATA_PATH + "models/shape_predictor_68_face_landmarks.dat";

// Load the landmark model
deserialize(PREDICTOR_PATH) >> landmarkDetector;

// In the code below, we read in an image using OpenCV’s imread function. We then convert the image to Dlib’s image format so as to use Dlib’s routines. The conversion from OpenCV to Dlib image format is very fast because no data is copied. Only the header is changed.

// Read Image
string imageFilename(DATA_PATH + "images/family.jpg");
cv::Mat img = cv::imread(imageFilename);

// Convert OpenCV image format to Dlib's image format
cv_image<bgr_pixel> dlibIm(img);

// Detect faces in the image
std::vector<dlib::rectangle> faceRects = faceDetector(dlibIm);
cout << "Number of faces detected: " << faceRects.size() << endl;
```

Dlib’s landmark detector needs two inputs

- Input image.
- Face rectangle

The output landmarks are stored in an instance of the class full_object_detection.

A few important methods of full_object_detection are listed below

- num_parts() : Number of landmark points.
- part(i) : The ith landmark point
- part(i).x() and part(i).y() can be used to access the x and y coordinates of the ith landmark point.

```cpp

// Vector to store landmarks of all detected faces
std::vector<full_object_detection> landmarksAll;

// Loop over all detected face rectangles
for (int i = 0; i < faceRects.size(); i++)
{

    // For every face rectangle, run landmarkDetector
    full_object_detection landmarks = landmarkDetector(dlibIm, faceRects[i]);
    
    // Print number of landmarks
    if (i == 0) cout << "Number of landmarks : " << landmarks.num_parts() << endl;
    
    // Store landmarks for current face
    landmarksAll.push_back(landmarks);
    
    // Next, we render the outline of the face using detected landmarks.
    // Draw landmarks on face
    renderFace(img, landmarks);
    
    // The code below saves the landmarks to results/family_0.txt … results/family_4.txt
        // Write landmarks to disk
    std::stringstream landmarksFilename;
    landmarksFilename << landmarksBasename <<  "_"  << i << ".txt";
    cout << "Saving landmarks to " << landmarksFilename.str() << endl;
    writeLandmarksToFile(landmarks, landmarksFilename.str());    
}
```

## Facial Alignment

### <font style = "color:rgb(50,120,229)">5-point landmark detector in Dlib</font>
In the new 5-point model, the landmark points consists of 2 points at the corners of the eye; for each eye and one point on the nose-tip. It is shown in the image given below.

<center> <img src = "https://www.learnopencv.com/wp-content/uploads/2018/09/fld-5-point-model.jpg" width=500/></center>

```cpp
dlib::frontal_face_detector faceDetector = dlib::get_frontal_face_detector();
// The landmark detector is implemented in the shape_predictor class
dlib::shape_predictor landmarkDetector;
string PREDICTOR_PATH = DATA_PATH + "models/shape_predictor_5_face_landmarks.dat";
dlib::deserialize(PREDICTOR_PATH) >> landmarkDetector;
//Read image
Mat img = imread(DATA_PATH+"images/face2.png");
// Detect landmarks
vector<Point2f> points = getLandmarks(faceDetector, landmarkDetector, img);


// Convert image to floating point in the range 0 to 1
img.convertTo(img, CV_32FC3, 1/255.0);
// Dimensions of output image
Size size(600,600);
// Variables for storing normalized image
Mat imNorm;
// Normalize image to output coordinates.
normalizeImagesAndLandmarks(size, img, imNorm, points, points);
// imNorm -> align 된 이미지 저장됨!!
```

[link 예시 참고](https://github.com/kkrunal77/Face-Processing-General-Tracking-and-Stabilization)

`similarity Transform`과 `Warp Affine` 사용

```cpp
void normalizeImagesAndLandmarks(Size outSize, Mat &imgIn, Mat &imgOut, vector<Point2f>& pointsIn, vector<Point2f>& pointsOut)
{
  int h = outSize.height;
  int w = outSize.width;


  vector<Point2f> eyecornerSrc;
  if (pointsIn.size() == 68)
  {
    // Get the locations of the left corner of left eye
    eyecornerSrc.push_back(pointsIn[36]);
    // Get the locations of the right corner of right eye
    eyecornerSrc.push_back(pointsIn[45]);
  }
  else if(pointsIn.size() == 5)
  {
    // Get the locations of the left corner of left eye
    eyecornerSrc.push_back(pointsIn[2]);
    // Get the locations of the right corner of right eye
    eyecornerSrc.push_back(pointsIn[0]);
  }

  vector<Point2f> eyecornerDst;
  // Location of the left corner of left eye in normalized image.
  eyecornerDst.push_back(Point2f( 0.3*w, h/3));
  // Location of the right corner of right eye in normalized image.
  eyecornerDst.push_back(Point2f( 0.7*w, h/3));

  // Calculate similarity transform
  Mat tform;
  similarityTransform(eyecornerSrc, eyecornerDst, tform);

  // Apply similarity transform to input image
  imgOut = Mat::zeros(h, w, CV_32FC3);
  warpAffine(imgIn, imgOut, tform, imgOut.size());

  // Apply similarity transform to landmarks
  transform( pointsIn, pointsOut, tform);

}
```

## <font style = "color:rgb(50,120,229)">Results</font>

| <center><img src="https://www.learnopencv.com/wp-content/uploads/2018/09/cv4f-mod3.2-face2.png"/></center> | <center><img src="https://www.learnopencv.com/wp-content/uploads/2018/09/cv4f-mod-3.2-AlignedFace.jpg"/></center> |
| -------- | -------- | 
| <center><img src="https://www.learnopencv.com/wp-content/uploads/2018/09/cv4f-mod3.2-face1.png"/></center> | <center><img src="https://www.learnopencv.com/wp-content/uploads/2018/09/cv4f-mod-3.2-AlignedFace1.jpg"/></center> |
|<center>Original Image</center> | <center>Aligned Face</center> |

# <font style = "color:rgb(50,120,229)">References and Further Reading</font>

1. https://melgor.github.io/blcv.github.io/static/2017/12/28/demystifying-face-recognition-iii-face-preprocessing/
2. https://github.com/luoyetx/face-alignment-at-3000fps

### Speed Up Face Detection

As you have seen in the previous section, landmark detection is a two step process. First, the faces are detected in an image, and then landmark detector is run inside each face bounding box.

The landmark detector runs in 1 millisecond. The face detector, depending on the size of the image, can take anywhere between 15 milliseconds to 60 milliseconds or even more. Face detection is the biggest bottleneck that needs to be addressed.

The following steps will help speed up face detection with small ( probably negligible ) loss in accuracy.

### Resize Frame

Facial Landmark Detector algorithms requires the user to provide a bounding box containing a face. The algorithm takes as input the image along with this box and returns the landmarks. The speed of face detection depends on the the resolution of the image because with smaller resolution images, you look for a smaller range of face sizes. The downside is that you will miss out smaller faces, but in many applications we have one person looking at the camera / webcam.

An easy way to speed up face detection is to resize the frame. My webcam records video at 720p ( i.e. 1280×720 ) resolution and I resize the image to a fixed height and vary the width accordingly. The bounding box obtained after face detection should be resized back by dividing the coordinates by the scale used for resizing the original frame. This allows us to do landmark detection at full resolution.

### Skip frame

Typically webcams record video at 30 fps. In a typical application you are sitting right in front of the webcam and not moving much. So there is no need to detect the face in every frame. We can simply do facial landmark detection based on facial bounding box obtained a few frames earlier. If you do face detection every 3 frames, you can have just sped up landmark detection by almost three times.

Is it possible to do better than using the previous location of the frame ? Yes, we can use object tracking methods to track the location of the face in frames where detection is not done, but in a webcam / selfie application it is an overkill.