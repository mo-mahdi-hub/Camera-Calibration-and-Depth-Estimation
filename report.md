# ENGR 422 — Assignment 2
# Camera Calibration and Depth Estimation

**Student:** Mohammed Mahdi  
**Course:** ENGR 422 — Computer Vision  
**Date:** May 2026

---

## 1. Introduction & Objective

Stereo vision is a fundamental technique in computer vision that recovers three-dimensional depth information from two-dimensional images. By capturing the same scene from two horizontally displaced viewpoints, the relative shift (disparity) of corresponding points between the two images can be measured. Because objects closer to the camera exhibit larger disparities while distant objects exhibit smaller ones, depth can be computed using the well-known stereo triangulation equation:

$$Z = \frac{f \times b}{d}$$

where $Z$ is the depth (mm), $f$ is the focal length (pixels), $b$ is the baseline distance between cameras (mm), and $d$ is the horizontal disparity (pixels).

The objective of this assignment is to implement a complete stereo vision pipeline: (1) calibrate a camera using a printed checkerboard pattern to extract intrinsic parameters, (2) capture a stereo image pair of a scene containing multiple objects at known distances, (3) manually calculate depth using the stereo equation and compare it against ground-truth measurements, and (4) generate a dense disparity map and critically analyse the sources of error in the system.

---

## 2. Stereo Setup & Experiment Design

### 2.1 Camera and Equipment

All images were captured using an **Apple iPhone** rear camera at full resolution of **5712 × 4284 pixels**. The calibration images were taken in portrait orientation and the stereo pair in landscape orientation. For processing efficiency and improved calibration accuracy, all images were resized to **50% resolution (2856 × 2142 pixels)** during computation.

A stereo rig was simulated using a **monocular sliding approach**: the phone was placed on a flat table edge, the left image was captured, and then the phone was slid exactly 80 mm to the right along a straight ruler to capture the right image. This method guarantees that both views share **identical intrinsic parameters** (same lens, same sensor, same settings), eliminating any inter-camera calibration differences. The phone was kept flush against the table edge throughout to maintain parallel optical axes.

### 2.2 Calibration Target

A **10 × 7 inner corner** checkerboard pattern (11 × 8 squares) was printed on A4 paper and taped flat onto a rigid clipboard to prevent warping. The physical square size was measured with a ruler and confirmed to be **25.0 mm**. Fifteen (15) calibration images were captured from a variety of distances, angles, and positions within the frame to ensure robust corner detection across the entire field of view.

### 2.3 Scene Description

Three distinct objects were placed at different known distances from the camera baseline on a textured floral fabric background:

| Object | Description | Distance from Camera (mm) |
|--------|-------------|---------------------------|
| **Red Box** | A bright red cardboard box with printed text "2026" | 350 mm (nearest) |
| **Pringles Can** | A cylindrical snack container with a colourful printed label | 650 mm (middle) |
| **Hedgehog Figurines** | Small dark-coloured decorative figurines | 950 mm (farthest) |

The objects were chosen to provide a range of textures and geometries: the Red Box has flat surfaces with text (good for matching), the Pringles Can has a curved reflective surface (challenging for matching), and the Hedgehog Figurines are small, dark, and low-texture (deliberately included as a failure case for discussion).

### 2.4 Baseline Measurement

The baseline distance $b$ between the two camera positions was measured using a ruler placed along the table edge. The measured value is **$b = 80$ mm**. This relatively small baseline was chosen because the objects are close to the camera (350–950 mm); a larger baseline would cause excessive occlusion for the nearest object.

### 2.5 Setup Documentation

A third-person photograph was captured showing the complete experimental setup: the camera positions, the three objects at their respective distances, the ruler used for baseline measurement, and the floral fabric background. This photograph is included in the project repository as `images/setup/setup_photo.jpg`.

**Setup Geometry Diagram:**

```
                          Wall (1450 mm)
           ─────────────────────────────────────
           |                                     |
           |     [Hedgehog]  (950 mm)            |
           |                                     |
           |          [Pringles]  (650 mm)       |
           |                                     |
           |   [Red Box]  (350 mm)               |
           |                                     |
    ───────┼──────────┼──────────────────────────
           Camera L   Camera R
           |← 80 mm →|
```

---

## 3. Calibration Method & Parameters

### 3.1 Corner Detection

The OpenCV function `cv2.findChessboardCorners()` was used to detect the 10 × 7 = 70 inner corners in each of the 15 calibration images. Corners were refined to sub-pixel accuracy using `cv2.cornerSubPix()` with an (11, 11) search window. All **15 out of 15** images passed corner detection successfully at the processing resolution of 2856 × 2142 pixels.

### 3.2 Camera Calibration Results

The function `cv2.calibrateCamera()` was used to compute the intrinsic parameters. The results are:

**Camera Matrix $K$:**

$$K = \begin{bmatrix} 2026.80 & 0 & 1411.89 \\ 0 & 2021.35 & 1036.59 \\ 0 & 0 & 1 \end{bmatrix}$$

**Distortion Coefficients:**

| Coefficient | Value | Type |
|-------------|-------|------|
| $k_1$ | +0.1258 | Radial (2nd order) |
| $k_2$ | +0.0776 | Radial (4th order) |
| $p_1$ | −0.0078 | Tangential |
| $p_2$ | −0.0018 | Tangential |
| $k_3$ | −1.5823 | Radial (6th order) |

**Reprojection Error (RMS): 1.6441 pixels**

### 3.3 Parameter Interpretation

Each parameter in the camera matrix has a clear physical meaning:

**Focal Length ($f_x = 2026.80$ px, $f_y = 2021.35$ px):** These values represent the focal length of the camera lens expressed in pixel units. They convert real-world angular measurements into pixel distances on the sensor. The fact that $f_x \approx f_y$ (within 0.3%) confirms that the camera sensor has **square pixels**, which is expected for modern smartphone cameras. The slight difference arises from minor manufacturing imperfections in the sensor.

**Principal Point ($c_x = 1411.89$ px, $c_y = 1036.59$ px):** The principal point is where the camera's optical axis intersects the image sensor. Ideally, this should be at the exact centre of the image, which for a 2856 × 2142 image would be (1428, 1071). Our measured values are offset by approximately (16.1, 34.4) pixels from the theoretical centre. This small offset is normal and is caused by slight misalignment between the lens assembly and the sensor during manufacturing.

**Radial Distortion ($k_1 = +0.1258$, $k_2 = +0.0776$, $k_3 = -1.5823$):** Radial distortion causes straight lines in the real world to appear curved in the image. The positive $k_1$ value indicates slight **pincushion distortion** (lines bow outward from the centre). The large negative $k_3$ acts as a compensating higher-order correction. These values are typical for wide-angle smartphone lenses.

**Tangential Distortion ($p_1 = -0.0078$, $p_2 = -0.0018$):** Tangential distortion occurs when the lens is not perfectly parallel to the image sensor. Both values are very close to zero, indicating that the iPhone lens is well-aligned with its sensor — this is expected for a precision-manufactured device.

**Reprojection Error (1.6441 pixels):** The RMS reprojection error measures how accurately the calibrated camera model can predict where the checkerboard corners will appear. A value of 1.64 pixels at 2856 × 2142 resolution represents an angular accuracy of approximately 0.057% of the image width, which is **good** for a consumer smartphone camera. For reference, values below 1.0 px are considered excellent, and values below 2.0 px are considered good.

---

## 4. Manual Measurements & Geometry

### 4.1 Measurement Method

All ground-truth distances were measured using a standard ruler from the camera baseline (the table edge where the phone was placed) to the front surface of each object. Measurements were taken **perpendicular to the baseline**, along the optical axis of the camera. All values are reported in millimetres.

### 4.2 Ground-Truth Distance Table

| Object | Distance from Camera Baseline |
|--------|-------------------------------|
| Red Box | 350 mm |
| Pringles Can | 650 mm |
| Hedgehog Figurines | 950 mm |
| Background Wall | 1450 mm |
| **Baseline** | **80 mm** |

Note: All measurements are approximate, taken with a standard ruler. The estimated measurement uncertainty is ±5 mm for the closer objects and ±10 mm for the wall distance.

---

## 5. Depth Calculation Method

### 5.1 The Stereo Depth Equation

Depth is calculated using the standard stereo triangulation formula:

$$Z = \frac{f_x \times b}{d}$$

Where:
- $Z$ = depth from camera to object (mm)
- $f_x$ = focal length along the horizontal axis = **2026.80 pixels**
- $b$ = baseline = **80 mm**
- $d = x_L - x_R$ = horizontal disparity between corresponding points (pixels)

### 5.2 Correspondence Selection

The SIFT (Scale-Invariant Feature Transform) algorithm was used to automatically detect and match distinctive feature points between the left and right images. SIFT was chosen because it is robust to scale changes and minor viewpoint differences. Matches were filtered using Lowe's ratio test (threshold = 0.75) and an epipolar constraint (maximum vertical difference of 15 pixels) to reject outliers.

For each of the three target objects, a specific matched feature point was selected:

### 5.3 Worked Examples

**Object 1: Red Box (Nearest)**

A feature on the printed text of the Red Box was matched:
- Left image pixel: $(x_L, y_L) = (713.4,\ 958.3)$
- Right image pixel: $(x_R, y_R) = (205.1,\ 965.9)$
- Disparity: $d = |713.4 - 205.1| = 508.3$ pixels
- Depth: $Z = \frac{2026.80 \times 80}{508.3} = \textbf{319.0 mm}$
- Ground truth: 350 mm
- Error: $|319.0 - 350.0| = 31.0$ mm (8.86%)

**Object 2: Pringles Can (Middle)**

A feature on the Pringles label was matched:
- Left image pixel: $(x_L, y_L) = (1301.8,\ 1145.1)$
- Right image pixel: $(x_R, y_R) = (1007.6,\ 1144.1)$
- Disparity: $d = |1301.8 - 1007.6| = 294.2$ pixels
- Depth: $Z = \frac{2026.80 \times 80}{294.2} = \textbf{551.1 mm}$
- Ground truth: 650 mm
- Error: $|551.1 - 650.0| = 98.9$ mm (15.21%)

**Object 3: Hedgehog Figurines (Farthest)**

A feature on the fabric near the base of the figurines was matched:
- Left image pixel: $(x_L, y_L) = (2352.4,\ 1247.0)$
- Right image pixel: $(x_R, y_R) = (2155.7,\ 1247.0)$
- Disparity: $d = |2352.4 - 2155.7| = 196.7$ pixels
- Depth: $Z = \frac{2026.80 \times 80}{196.7} = \textbf{824.3 mm}$
- Ground truth: 950 mm
- Error: $|824.3 - 950.0| = 125.7$ mm (13.23%)

---

## 6. Results & Comparison Table

### 6.1 Depth Estimation Comparison Table

| Object | Pixel L $(x_L, y_L)$ | Pixel R $(x_R, y_R)$ | Disparity (px) | Measured Depth (mm) | Calculated Depth (mm) | Abs Error (mm) | Error (%) |
|--------|----------------------|----------------------|-----------------|----------------------|------------------------|----------------|-----------|
| Red Box | (713.4, 958.3) | (205.1, 965.9) | 508.3 | 350 | 319.0 | 31.0 | 8.86% |
| Pringles Can | (1301.8, 1145.1) | (1007.6, 1144.1) | 294.2 | 650 | 551.1 | 98.9 | 15.21% |
| Hedgehog Figurines | (2352.4, 1247.0) | (2155.7, 1247.0) | 196.7 | 950 | 824.3 | 125.7 | 13.23% |

**Average Error: 12.43%**

### 6.2 Observations

All three calculated depths **underestimate** the true distance. This systematic bias suggests either (a) the measured baseline of 80 mm is slightly smaller than the true optical baseline, or (b) there is a consistent offset in the ground-truth measurement origin. Despite this, the relative ordering of depths is perfectly preserved (Red Box < Pringles < Hedgehog), demonstrating that the stereo system correctly captures the relative geometry of the scene.

### 6.3 Dense Disparity Map

A dense disparity map was generated using the **StereoSGBM** (Semi-Global Block Matching) algorithm with the following tuned parameters:

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `blockSize` | 7 | Small enough to preserve sharp edges on the objects |
| `numDisparities` | 224 | Large enough to capture the high disparity of the Red Box (>500 px at full scale) |
| `uniquenessRatio` | 40 | Very strict filtering to reject ambiguous matches and reduce speckle noise |
| `speckleWindowSize` | 150 | Removes small isolated noise patches |
| `speckleRange` | 32 | Controls maximum disparity variation within connected components |

**Disparity Map Observations:**
- The **Red Box** appears in warm colours (high disparity), confirming it is the nearest object. However, only the right edge of the box is clearly mapped; the flat, uniformly coloured front face lacks distinctive texture, causing the algorithm to reject matches due to the high uniqueness ratio.
- The **Pringles Can** shows moderate disparity values (green/yellow), correctly placing it at an intermediate depth. The curved surface and printed label provide enough texture for partial matching.
- The **Hedgehog Figurines** appear in cooler colours (low disparity), correctly identified as the farthest objects.
- The **background fabric** shows extensive noise and holes. The repeating floral pattern confuses the block-matching algorithm, as similar patterns appear at multiple horizontal positions, leading to ambiguous correspondences.

---

## 7. Discussion & Error Analysis

### Q1: How did camera calibration affect your ability to estimate depth accurately?

Camera calibration is the foundation of the entire depth estimation pipeline. Without accurate intrinsic parameters, the stereo depth equation $Z = fb/d$ would produce meaningless results. In our experiment, the calibration achieved a reprojection error of **1.64 pixels**, which is good but not perfect. To understand the impact, consider that if our focal length $f_x = 2026.80$ px were off by just 1%, the depth estimates would also shift by approximately 1%. Similarly, the distortion coefficients correct for barrel/pincushion lens effects; without undistortion, the pixel coordinates of our correspondence points would be displaced, introducing systematic errors in disparity and therefore depth.

Our calibration used **15 images** from varied angles and positions, which provides a well-conditioned system of equations. The fact that all 15 images passed corner detection indicates good image quality and a well-printed checkerboard. However, the reprojection error of 1.64 px (rather than the ideal <1.0 px) suggests that some calibration images may have had slight motion blur or that the checkerboard was not perfectly flat in all images.

### Q2: Which source of error was most important in your setup?

After analysing our results, we rank the error sources as follows:

1. **Baseline measurement uncertainty (most significant):** Our baseline of 80 mm was measured with a standard ruler. An error of just ±2 mm (2.5%) would shift all calculated depths by 2.5%. Since all three objects show a **systematic underestimation** of depth, this strongly suggests the true optical baseline may be slightly larger than 80 mm (perhaps 85–90 mm). If the true baseline were 90 mm instead of 80 mm, the Red Box depth would be $\frac{2026.80 \times 90}{508.3} = 358.9$ mm, which is remarkably close to the ground truth of 350 mm.

2. **Correspondence accuracy:** SIFT matching has a typical accuracy of ±1–2 pixels. For the Red Box with a large disparity of 508.3 px, a 2-pixel error represents only 0.4% error. However, for the Hedgehog Figurines with a disparity of 196.7 px, the same 2-pixel error represents 1.0% error. This explains why the percentage error tends to be higher for more distant objects.

3. **Ground-truth measurement uncertainty:** The physical distances were measured with a ruler to the nearest ±5 mm. At 350 mm, this represents ±1.4%; at 950 mm, ±0.5%. This is a smaller contributor than the baseline uncertainty.

### Q3: How would a larger baseline change the results?

Increasing the baseline $b$ directly affects the stereo depth equation $Z = fb/d$. A larger baseline produces a **larger disparity** for every object in the scene, because $d = fb/Z$:

- **Advantage — Better depth resolution:** With a larger baseline, the disparity values become larger, making them easier to measure accurately. A 1-pixel error on a disparity of 500 px (0.2%) is far less impactful than a 1-pixel error on a disparity of 50 px (2%). This would significantly improve accuracy for **distant objects** like the Hedgehog Figurines, which currently have the smallest disparity.

- **Disadvantage — Increased occlusion:** A larger baseline means the two cameras see the scene from more different angles. This causes **occlusion**: parts of the scene visible to one camera are hidden behind foreground objects in the other camera's view. In our setup, the Red Box at 350 mm would occlude a larger portion of the scene behind it, making it impossible to find correspondences in those regions.

- **Disadvantage — Stereo blind spot:** The left edge of the left image and the right edge of the right image have no overlap in a stereo pair. A larger baseline increases these non-overlapping regions, reducing the usable field of view.

For our setup with objects at 350–950 mm, an ideal baseline would be approximately 100–120 mm, providing a good balance between disparity magnitude and occlusion.

### Q4: Why is depth estimation harder for low-texture, reflective, or distant objects?

**Low-texture objects (e.g., plain walls, flat surfaces):** Both SIFT feature detection and SGBM block matching rely on finding unique, distinctive patterns in the image. A plain white wall looks identical at every horizontal position, so there is no unique "signature" for the algorithm to lock onto. This is why our disparity map shows large black holes on the table surface and the background wall — the algorithm simply cannot determine the correct match and rejects the pixel entirely. This is visible in our disparity map where the flat front face of the Red Box is mostly unmapped despite being the closest object.

**Reflective/specular objects (e.g., glossy surfaces, metal):** When light reflects off a shiny surface, the bright "specular highlight" appears at different positions depending on the viewing angle. Since the left and right cameras have different viewing angles, the highlight shifts position between the two images in a way that does not correspond to the actual surface geometry. The stereo algorithm interprets this highlight shift as a false disparity, producing incorrect depth values. Our Pringles Can, which has a semi-reflective cylindrical surface, shows some matching difficulties in the disparity map for this reason.

**Distant objects:** As $Z$ increases, $d = fb/Z$ decreases. For very distant objects, the disparity becomes only a few pixels. At this scale, a single pixel of matching error causes enormous depth errors due to the inverse relationship: $\Delta Z \approx Z^2 / (fb) \times \Delta d$. For our Hedgehog Figurines at 950 mm, the disparity is 196.7 px. If the match were off by just 5 pixels, the depth would change by approximately $\frac{824.3^2}{2026.80 \times 80} \times 5 = 20.9$ mm. For an object at 5000 mm with a disparity of only ~32 px, the same 5-pixel error would cause a depth error of approximately 780 mm.

### Q5: If you had to improve this home setup for a robot, what would you change first and why?

For a robotics application, the most impactful improvements would be:

1. **Hardware-synchronised stereo camera (Priority 1):** Replace the single sliding phone with a dedicated stereo camera module (e.g., Intel RealSense D435 or Stereolabs ZED 2). This eliminates the largest source of error in our setup: the assumption that nothing moves between captures. A hardware rig also provides a **factory-calibrated, fixed baseline** with sub-millimetre precision, eliminating our ruler-based measurement uncertainty.

2. **Active stereo / structured light (Priority 2):** Project an infrared dot pattern onto the scene. This provides artificial "texture" to flat walls, tables, and other featureless surfaces, solving the textureless matching problem discussed in Q4. The Intel RealSense uses this technique and can produce dense, accurate depth maps even on plain white surfaces.

3. **Global shutter camera (Priority 3):** Smartphone cameras use **rolling shutters**, which capture each row of pixels at a slightly different time. For a moving robot, this causes image distortion (skew, wobble). A global shutter captures all pixels simultaneously, producing clean images even during motion.

4. **IMU (Inertial Measurement Unit) integration:** Adding an accelerometer and gyroscope allows the system to estimate the camera's exact pose (position and orientation) in real time. Combined with visual odometry, this enables accurate depth estimation even while the robot is in motion, compensating for vibrations and rotations that would otherwise degrade stereo matching quality.

---

## 8. Conclusion

This assignment successfully demonstrated the complete stereo vision pipeline using a consumer smartphone camera. Through camera calibration with 15 checkerboard images, we extracted accurate intrinsic parameters (focal length $f_x = 2026.80$ px, reprojection error = 1.64 px). Using these parameters with a measured baseline of 80 mm, we estimated the depth of three objects at distances ranging from 350 mm to 950 mm.

The system achieved an **average depth estimation error of 12.43%**, with the nearest object (Red Box) showing the best accuracy at 8.86% error and the middle object (Pringles Can) showing the highest error at 15.21%. All three depth estimates showed a systematic underestimation, which we attribute primarily to uncertainty in the baseline measurement.

The dense disparity map generated with StereoSGBM correctly identified the relative depth ordering of all objects but exhibited noise and gaps on textureless surfaces, confirming the well-known limitations of passive stereo vision. These limitations can be addressed through active stereo techniques and hardware-synchronised camera systems, as discussed in Section 7.

---

## 9. References

1. Bradski, G. & Kaehler, A. (2008). *Learning OpenCV: Computer Vision with the OpenCV Library*. O'Reilly Media.
2. Hartley, R. & Zisserman, A. (2004). *Multiple View Geometry in Computer Vision* (2nd ed.). Cambridge University Press.
3. Szeliski, R. (2022). *Computer Vision: Algorithms and Applications* (2nd ed.). Springer.
4. OpenCV Documentation — Camera Calibration and 3D Reconstruction. https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html
5. OpenCV Documentation — StereoSGBM Class Reference. https://docs.opencv.org/4.x/d2/d85/classcv_1_1StereoSGBM.html
6. Lowe, D. G. (2004). Distinctive Image Features from Scale-Invariant Keypoints. *International Journal of Computer Vision*, 60(2), 91–110.
