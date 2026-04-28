# 📐 ENGR422 Assignment 2 — Full-Mark Implementation Plan
# Camera Calibration and Depth Estimation (20 Marks)

---

## Rubric Overview — Know What You're Chasing

Before diving in, here's the grading breakdown so every step we take is targeted:

| # | Criterion | Weight | Target |
|---|-----------|--------|--------|
| 1 | Stereo Setup & Experiment Design | **3 marks** (15%) | Excellent |
| 2 | Calibration Quality & Explanation | **5 marks** (25%) | Excellent |
| 3 | Manual Measurement & Formula-Based Depth Calculation | **5 marks** (25%) | Excellent |
| 4 | Results, Comparison & Error Analysis | **4 marks** (20%) | Excellent |
| 5 | Report Quality & Reproducibility | **3 marks** (15%) | Excellent |

> [!IMPORTANT]
> **Phase 1 requires YOUR physical work** — I cannot capture images or measure distances. Once you provide the images and measurements, I will build the entire Jupyter Notebook and draft the report for you.

---

## ✅ Pre-Flight Decisions

Answer these before starting:

- [ ] **Camera choice:** Two identical phones (recommended) OR one phone moved between two fixed positions?
- [ ] **Calibration target:** Can you print a checkerboard on A4 paper? (Yes → print one; No → display on a flat tablet/monitor)
- [ ] **Scene objects:** Pick 3-5 rigid objects you can place at different distances (books, boxes, bottles, mugs)
- [ ] **Measuring tool:** Do you have a tape measure or ruler (at least 1m range)?

---

## Phase 1 — Physical Setup & Data Collection 🏠
**Rubric targets: Criterion 1 (Setup) + Criterion 3 (Manual Measurement)**
**Estimated time: 1.5 – 2 hours**

### Step 1.1: Prepare the Calibration Target

1. **Print a checkerboard pattern** — use a standard 9×6 inner corners grid (10×7 squares)
   - Measure the **exact square size** with a ruler (e.g., 24.7 mm) — write this down precisely
   - Glue or tape it flat onto a **rigid surface** (cardboard, clipboard, hardcover book)
2. If you cannot print, display one on a flat tablet — but note this in your report as a limitation

> [!TIP]
> **Full-mark tip:** Use a printed pattern, not a screen. Screens introduce glare and curved surfaces. The rubric says *"printed or hand-drawn checkerboard"* — a printed one on flat cardboard is ideal.

### Step 1.2: Capture Calibration Images

1. Set both cameras to the **same resolution** (e.g., 1920×1080 or 4032×3024). Record the exact resolution.
2. **If using two phones (stereo calibration):**
   - Mount both phones side by side (use books, tape, a ruler, or a DIY rig to stabilise them)
   - For each calibration shot, ensure the checkerboard is **fully visible in BOTH cameras simultaneously**
   - Capture **15–20 stereo pairs** with the checkerboard at:
     - Different distances (near, mid, far)
     - Different angles (tilted left, right, up, down)
     - Different positions in the frame (center, corners, edges)
3. **If using one phone moved between two positions (monocular + simulated stereo):**
   - Capture **15–20 images** of the checkerboard from a single position with varied angles
   - This calibrates intrinsic parameters only — document this clearly in your report

> [!IMPORTANT]
> **Minimum calibration images: 15.** The rubric says *"A single image is not acceptable"* and *"enough calibration images from different positions and angles."* More images = lower reprojection error = higher marks.

### Step 1.3: Set Up the Stereo Scene

1. Choose a well-lit area with **no strong reflections** (avoid glass, mirrors, shiny surfaces nearby)
2. Place **3–5 rigid objects** at clearly different distances:
   - **Near object:** ~30–40 cm from camera baseline
   - **Mid object:** ~60–80 cm from camera baseline
   - **Far object:** ~100–150 cm from camera baseline
3. Objects should have **good texture** (patterns, text, labels) — avoid plain white/solid surfaces
4. Mark the exact camera positions with tape on the table/surface

> [!TIP]
> **Full-mark tip:** Include at least one object with poor texture (e.g., a plain white mug) as a deliberate "failure case" — this gives you rich discussion material for Criterion 4 about where stereo matching fails.

### Step 1.4: Capture the Stereo Pair

1. Position the left camera and capture the **Left Image**
2. Without moving any objects, position the right camera and capture the **Right Image**
3. **If using two phones:** trigger both simultaneously (use a timer or have someone help)
4. Ensure both images have the **exact same scene** — nothing moved between shots

### Step 1.5: Measure the Baseline

1. Measure the distance between the **centres of the two camera lenses** using a ruler/tape
2. Record this in **millimetres** (e.g., 62.5 mm)
3. Take a photo showing the measurement being made

### Step 1.6: Measure Ground-Truth Distances

1. For each of your 3–5 target objects, measure the **perpendicular distance** from the camera baseline to the front surface of the object
2. Use a tape measure — measure in **millimetres** for precision
3. Record measurements in a table like this:

| Object | Distance from baseline (mm) |
|--------|----------------------------|
| Book   | 385                        |
| Bottle | 620                        |
| Box    | 1050                       |

### Step 1.7: Document the Setup (Critical for Criterion 1!)

1. **Take a "bird's-eye" or "third-person" photo** of your entire setup showing:
   - Camera positions
   - Object positions
   - Approximate measurement paths
2. **Draw a labelled diagram** (by hand or digitally) showing:
   - Camera 1 (Left), Camera 2 (Right)
   - Baseline distance ($b$)
   - Each object with its measured distance ($Z_1$, $Z_2$, $Z_3$)
   - Coordinate system orientation

> [!CAUTION]
> The rubric explicitly requires: *"a labelled photo or diagram of your setup showing where the measurements were taken from and to."* Missing this = dropping marks on Criteria 1 and 3.

### Phase 1 Deliverables Checklist

- [ ] 15–20 calibration images (or stereo pairs)
- [ ] 1 Left image + 1 Right image of the stereo scene
- [ ] Baseline measurement in mm
- [ ] Ground-truth distances for 3–5 objects in mm
- [ ] Third-person photo of setup
- [ ] Labelled diagram showing geometry
- [ ] Camera model name and image resolution noted
- [ ] Checkerboard square size in mm noted

---

## Phase 2 — Camera Calibration (Code) 🔧
**Rubric target: Criterion 2 (Calibration Quality & Explanation) — 25% of marks**
**I will build this as a Jupyter Notebook section**

### Step 2.1: Checkerboard Corner Detection
- Load all calibration images
- Convert to grayscale
- Use `cv2.findChessboardCorners()` with the correct pattern size
- Refine corners with `cv2.cornerSubPix()` for sub-pixel accuracy
- Visualise detected corners overlaid on at least 3–4 sample images

### Step 2.2: Single Camera Calibration
- Run `cv2.calibrateCamera()` to obtain:
  - **Camera Matrix (K):** containing $f_x$, $f_y$, $c_x$, $c_y$
  - **Distortion Coefficients:** $k_1$, $k_2$, $p_1$, $p_2$, $k_3$
- Calculate and display the **Reprojection Error** (target: < 0.5 px for Excellent)

### Step 2.3: Stereo Calibration (if using two cameras)
- Run `cv2.stereoCalibrate()` to obtain:
  - **Rotation Matrix (R):** relating left camera to right camera orientation
  - **Translation Vector (T):** relating left camera to right camera position
- The magnitude of T should roughly match your measured baseline

### Step 2.4: Parameter Interpretation (Critical for Full Marks!)

For each parameter, include a markdown cell explaining its **physical meaning:**

| Parameter | What It Means |
|-----------|---------------|
| $f_x$, $f_y$ | Focal length in pixel units (how many pixels per unit length at the image plane). Related to actual focal length by sensor pixel size. |
| $c_x$, $c_y$ | Principal point — where the optical axis intersects the image sensor, ideally near image centre. |
| $k_1$, $k_2$, $k_3$ | Radial distortion — models barrel/pincushion distortion. Larger values mean more lens distortion. |
| $p_1$, $p_2$ | Tangential distortion — due to lens not being perfectly parallel to the sensor. |
| $R$ | Rotation between left and right cameras — ideally close to identity if cameras are parallel. |
| $T$ | Translation between cameras — its magnitude should match the physical baseline. |
| Reprojection Error | Average pixel distance between projected 3D points and detected corners. Lower = better calibration. |

> [!WARNING]
> **The rubric explicitly penalises:** *"only pasting matrices without interpretation."* Every matrix MUST have a paragraph explaining what it means. This is where most students lose marks.

### Phase 2 Deliverables Checklist

- [ ] Corner detection visualised on sample images
- [ ] Camera matrix printed and explained
- [ ] Distortion coefficients printed and explained
- [ ] Reprojection error calculated and evaluated
- [ ] Stereo rotation/translation matrices (if stereo calibration) printed and explained
- [ ] Physical meaning of each parameter clearly written in markdown

---

## Phase 3 — Depth Estimation & Correspondence 📏
**Rubric targets: Criterion 3 (Manual Depth Calculation) + Criterion 4 (Results)**
**50% of total marks — this is the core of the assignment**

### Step 3.1: Image Rectification
- Use `cv2.stereoRectify()` and `cv2.initUndistortRectifyMap()` to rectify images
- Apply `cv2.remap()` to both left and right images
- Visualise rectified images side-by-side with **horizontal epipolar lines** drawn to prove alignment
- Briefly discuss: Are the lines truly horizontal? Is rectification successful?

### Step 3.2: Correspondence Selection (3–5 Points)
- For each target object, identify a **specific feature point** (e.g., corner of a book, cap of a bottle)
- Record its pixel coordinates in both the left and right rectified images:
  - $(x_L, y_L)$ in the left image
  - $(x_R, y_R)$ in the right image
- Visualise these points annotated on both images
- Justify why you chose each point (clear, unambiguous, on a textured region)

### Step 3.3: Manual Depth Calculation (Show Every Step!)

For each point, compute depth step-by-step in a notebook cell:

```
1. Disparity:  d = x_L - x_R  (in pixels)
2. Focal length: f = f_x from calibration (in pixels)
3. Baseline: b = measured baseline (in mm)
4. Depth: Z = (f × b) / d  (in mm)
5. Convert to cm or m as needed
```

> [!IMPORTANT]
> **Show the actual numbers**, not just code. Use markdown cells like:
> 
> *"For the book (Point 1): $d = 450 - 412 = 38$ px, $f = 1250.3$ px, $b = 62.5$ mm → $Z = \frac{1250.3 \times 62.5}{38} = 2057.4$ mm ≈ 205.7 cm"*
> 
> This is what "worked examples" means in the rubric.

### Step 3.4: Comparison Table

Build the required table:

| Target | Real Distance (mm) | $(x_L, y_L)$ | $(x_R, y_R)$ | Disparity (px) | Calculated Depth (mm) | Abs Error (mm) | % Error |
|--------|--------------------:|:-------------:|:-------------:|----------------:|----------------------:|----------------:|--------:|
| Book   | 385                 | (450, 300)    | (412, 300)    | 38              | 2057                  | ...             | ...     |
| Bottle | 620                 | (700, 250)    | (675, 250)    | 25              | 3126                  | ...             | ...     |
| Box    | 1050                | (500, 400)    | (487, 400)    | 13              | 6012                  | ...             | ...     |

### Step 3.5: Dense Disparity Map Visualisation
- Use `cv2.StereoSGBM_create()` to compute a dense disparity map
- Visualise using a colour map (`cv2.applyColorMap`)
- Annotate where the disparity looks reliable vs. noisy
- Discuss effects of texture, lighting, reflections, and occlusion

### Phase 3 Deliverables Checklist

- [ ] Rectified images with epipolar lines
- [ ] Correspondence points annotated on both images
- [ ] Step-by-step manual depth calculation for each point
- [ ] Comparison table with all required columns
- [ ] Dense disparity map visualised and annotated
- [ ] Discussion of disparity map quality

---

## Phase 4 — Analysis & Discussion 📝
**Rubric target: Criterion 4 (Error Analysis) — 20% of marks**

### Step 4.1: Answer ALL Required Discussion Questions

The rubric **explicitly lists 5 questions** that must be answered. Address each one thoroughly:

#### Q1: How did camera calibration affect your ability to estimate depth accurately?
- Discuss reprojection error and its relationship to depth accuracy
- Explain how incorrect focal length or principal point would propagate to depth errors
- Reference your actual calibration numbers

#### Q2: Which source of error was most important in your setup?
- Rank your error sources: calibration quality, baseline measurement precision, correspondence accuracy, camera alignment
- Use your actual error percentages to justify which was dominant
- Example: "Our 5% error on the near object is primarily due to a ±2px correspondence uncertainty, which at d=38px represents a 5.3% disparity error"

#### Q3: How would a larger baseline change the results?
- Larger baseline → larger disparity → better depth resolution for far objects
- But larger baseline → more occlusion → harder correspondence for near objects
- Reference the formula $Z = fb/d$ to show the mathematical relationship

#### Q4: Why is depth estimation harder for low-texture, reflective, or distant objects?
- Low texture → stereo matching algorithms cannot find unique correspondences
- Reflective surfaces → specular highlights differ between views
- Distant objects → very small disparity → small errors cause large depth errors (inverse relationship)

#### Q5: If you had to improve this home setup for a robot, what would you change first and why?
- Suggest: hardware-synchronised cameras, larger/calibrated baseline, structured light for textureless regions, IMU for alignment
- Discuss computational requirements and real-time constraints

> [!TIP]
> **Full-mark tip:** Don't just list answers — tie every point back to **your actual data and results**. The rubric says *"strong reasoning about failure sources"* — use specific numbers from your comparison table to support each argument.

### Phase 4 Deliverables Checklist

- [ ] All 5 discussion questions answered with references to your data
- [ ] Error sources ranked with quantitative justification
- [ ] Depth formula referenced in baseline discussion
- [ ] Specific examples from your disparity map cited

---

## Phase 5 — Report & Final Submission 📄
**Rubric target: Criterion 5 (Report Quality & Reproducibility) — 15% of marks**

### Step 5.1: Technical Report Structure (4–6 pages)

| Section | Content | Approx. Length |
|---------|---------|----------------|
| 1. Introduction & Objective | What is stereo vision, what this assignment does | 0.5 page |
| 2. Stereo Setup Description | Camera details, baseline, scene description, setup photo & diagram | 0.75 page |
| 3. Calibration Method & Parameters | How calibration was done, all parameters with interpretations | 1 page |
| 4. Manual Measurements & Geometry | Ground truth distances, labelled diagram, measurement method | 0.5 page |
| 5. Depth Calculation Method | Stereo depth equation, worked examples showing all steps | 0.75 page |
| 6. Results & Comparison Table | Full comparison table, disparity map figure | 0.75 page |
| 7. Discussion & Error Analysis | Answer all 5 required questions with data references | 1 page |
| 8. Conclusion | Summary of findings, limitations, future improvements | 0.25 page |
| 9. References | OpenCV docs, Hartley & Zisserman, Szeliski, lecture notes | 0.25 page |

### Step 5.2: Jupyter Notebook Standards
- [ ] All cells run sequentially with **Restart and Run All** — no errors
- [ ] Every code cell has a **markdown header cell** above it explaining what it does
- [ ] All figures have **titles, axis labels, and colorbars** where applicable
- [ ] Image paths are **relative** (not absolute) so the notebook is portable
- [ ] The notebook reads images from a `./images/` or `./data/` folder

### Step 5.3: File Organisation

```
Camera Calibration and Depth Estimation/
├── stereo_depth_estimation.ipynb      # Main Jupyter Notebook
├── report.pdf                         # 4-6 page technical report (or .docx)
├── images/
│   ├── calibration/                   # 15-20 checkerboard images
│   │   ├── calib_01.jpg
│   │   ├── calib_02.jpg
│   │   └── ...
│   ├── stereo/
│   │   ├── left.jpg                   # Left stereo image
│   │   └── right.jpg                  # Right stereo image
│   └── setup/
│       ├── setup_photo.jpg            # Third-person photo
│       └── setup_diagram.png          # Labelled diagram
└── README.md                          # Brief description
```

### Step 5.4: Final Submission Checklist

- [ ] Report is 4–6 pages, single-spaced, well-formatted
- [ ] Notebook runs top-to-bottom without errors
- [ ] All calibration and stereo images are included
- [ ] Setup photo and labelled diagram are included
- [ ] All matrices/parameters are interpreted (not just pasted)
- [ ] Comparison table has all 7 required columns
- [ ] All 5 discussion questions are answered
- [ ] References section is present

---

## 🔒 Rubric Verification Matrix

Use this final checklist to verify every "Excellent" criterion is met:

### Criterion 1: Stereo Setup & Experiment Design (15%)
| Requirement | Status |
|-------------|--------|
| Setup is clearly designed and documented | ⬜ |
| Baseline is documented and justified | ⬜ |
| Scene selection is explained | ⬜ |
| Capture process is described | ⬜ |
| Camera details (model, resolution) stated | ⬜ |
| How cameras were kept parallel is explained | ⬜ |
| Labelled setup photo/diagram included | ⬜ |

### Criterion 2: Calibration Quality & Explanation (25%)
| Requirement | Status |
|-------------|--------|
| 15+ calibration images used | ⬜ |
| Intrinsic parameters reported | ⬜ |
| Distortion coefficients reported | ⬜ |
| Extrinsic parameters reported (if stereo) | ⬜ |
| Reprojection error calculated | ⬜ |
| **Each parameter's physical meaning explained** | ⬜ |
| Pattern size and image resolution stated | ⬜ |

### Criterion 3: Manual Measurement & Formula-Based Depth (25%)
| Requirement | Status |
|-------------|--------|
| Real distances measured carefully (mm precision) | ⬜ |
| Baseline measured | ⬜ |
| 3+ target points/objects measured | ⬜ |
| Stereo depth equation used explicitly | ⬜ |
| Focal length value and unit stated | ⬜ |
| Disparity computation shown step-by-step | ⬜ |
| **Clear worked examples with actual numbers** | ⬜ |

### Criterion 4: Results, Comparison & Error Analysis (20%)
| Requirement | Status |
|-------------|--------|
| Comparison table with all 7 columns | ⬜ |
| Errors quantified (absolute + percentage) | ⬜ |
| Rectified images shown | ⬜ |
| Disparity map visualised | ⬜ |
| Disparity quality discussed (reliable vs. noisy regions) | ⬜ |
| All 5 discussion questions answered | ⬜ |
| Errors analysed with **strong reasoning** | ⬜ |

### Criterion 5: Report Quality & Reproducibility (15%)
| Requirement | Status |
|-------------|--------|
| Report is 4–6 pages, structured, professional | ⬜ |
| Notebook is organised with markdown headers | ⬜ |
| Notebook runs with Restart & Run All | ⬜ |
| Figures and tables support the discussion | ⬜ |
| File organisation is clean and portable | ⬜ |

---

## 📅 Suggested Timeline

| Day | Tasks |
|-----|-------|
| **Day 1** | Phase 1: Print checkerboard, set up cameras, capture calibration images, capture stereo pair, measure everything |
| **Day 2** | Phase 2: Run calibration code, verify reprojection error, interpret parameters |
| **Day 3** | Phase 3: Rectification, correspondence selection, manual depth calculations, disparity map |
| **Day 4** | Phase 4: Write error analysis, answer all 5 discussion questions |
| **Day 5** | Phase 5: Write/polish report, organise files, final Restart & Run All test |

---

## 🚀 Next Steps

**Your immediate action items:**
1. Answer the Pre-Flight Decisions above
2. Complete Phase 1 (physical setup, image capture, measurements)
3. Upload all images and measurements to this workspace folder
4. Tell me when ready — I will build the entire Jupyter Notebook and draft the report

> [!NOTE]
> I previously created a similar plan in [an earlier conversation](file:///C:/Users/moham/.gemini/antigravity/brain/a048f2ab-8b40-4ad4-bd97-d9227841b229/implementation_plan.md). This version is significantly more detailed, with explicit rubric mapping, verification matrices, and mark-maximising tips.
