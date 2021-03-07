The principal components are the Eigen vectors of the covariance matrix.

# Eigen Face

[다크 프로그래머 블로그](https://darkpgmr.tistory.com/110)

이미지에서 픽셀 밝기값을 일렬로 연결하여 벡터로 만들면 이들 각각의 얼굴 이미지는 45x40 = 1,800 차원의 벡터로 생각할 수 있다 (즉, 각각의 이미지는 1,800 차원 공간에서 한 점(좌표)에 대응).
이제 이 20개의 1,800차원 점 데이터들을 가지고 PCA를 수행하면 데이터의 차원 수와 동일한 개수의 주성분 벡터들을 얻을 수 있다. 이렇게 얻어진 주성분 벡터들을 다시 이미지로 해석한 것이 eigenface이다 (얼굴 이미지를 가지고 얻은 벡터이기에 eigenface라 부른다).  위 그림에서 볼 수 있듯이 앞부분 eigenface들은 데이터들에 공통된 요소(얼굴의 전반적인 형태)를 나타내고 뒤로 갈수록 세부적인 차이 정보를 나타낸다. 그리고 더 뒤로 가면 거의 노이즈(noise)성 정보를 나타낸다.

### Covariance Matrix

https://www.visiondummy.com/2014/04/geometric-interpretation-covariance-matrix 참고.

Covariance Matrix의 Eigenvalue Decomposition은 Data Spread와 관련 있음.

## OpenCV PCA (Principal Component Analysis)

[doc](https://docs.opencv.org/3.4/d3/d8d/classcv_1_1PCA.html)

```cpp
// Example usage
PCA pca(data, Mat(), PCA::DATA_ORDER_ROW, 10); 
Mat mean = pca.mean; 
Mat eigenVectors = pca.eigenvectors;

// Example usage
mean, eigenVectors = cv2.PCACompute(data, mean=None, maxComponents=10)
```

# Fisher Face

### Linear Discriminant Analysis (LDA)

unlike PCA, LDA takes class (category) information in the data.

- Data belonging to the same class should cluster tightly in the new space
- Data belonging to different classes should far removed (i.e. separable) from each other.

# Local Binary Pattern Histogram

`cv2.createLBPHFaceRecognizer()` (2.4 Version)

Local Binary Pattern ( LBP ) of a pixel is calculated by comparing the intensity of neighborhood pixels with this pixel and representing the result as a binary number. LBPs are calculated on grayscale images. To calculate LBP for a pixel we take a patch in the image with this pixel at the center. If intensity of center pixel is greater than or equal to a neighborhood pixel we denote this comparison as 1 else 0. If we compare 8 neighbors around a pixel in a 3x3 patch, we get an 8-bit binary number denoting the texture around a pixel such as 11010001. For each neighborhood pixel there can be only 2 values (0 or 1). So for 8 neighbors there are  28 =256 combinations possible for LBP code around a pixel. So in decimal we will have these encodings from 0, 1, 2 … upto 255.

![img](https://learnopencv.com/wp-content/uploads/2018/01/opcv4face-w7-m5-thresholdImage.png)

## LBP Histogram

Now that we have LBP codes for all pixels in an image, how should we represent the whole image using these LBP codes? Short answer: Histograms.

We can calculate histogram of LBP codes by binning them into 256 bins (numbered 0, 1, ...255). The value at each bin represents the frequency of that LBP code. Hence we will get a 256 dimensional vector.

Is the LBP representation of the image unique? No. We have discarded the spatial information by computing the histogram. If we split the entire image into 4 equal sized patches and shuffle these patches. Although the new image is now visually very different from the original image, the global descriptor of this new image will be still be very close to original image.

One way to solve this problem is to divide image into many small patches, say 20x20 then calculate histogram of LBP codes for each such patch and concatenate the histograms one after another. So if our image is of size 100x100. We will have 25 patches and a 25*256 = 6400 dimensional vector. This way our global feature descriptor also has information about features of various local patches. This method also makes the final feature descriptor to be more robust against pose and illumination variations.

![img](https://learnopencv.com/wp-content/uploads/2018/01/opcv4face-w7-m5-FeatureHistogramFaceImage.jpg)