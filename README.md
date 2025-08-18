# mtu-xsens-imu-Sensor

## Project Objective
Predict surrounding temperature and magnetic field values that are influenced by device behavior (movement, frequency, noise, etc.)

## Implementation Steps:
### Data Collection
This project uses data collected from the Xsens MTi-3-8A7G6 Inertial Measurement Unit (IMU) — a compact, high-performance MEMS-based sensor module. The MTi-3 includes:

3D Accelerometer

3D Magnetometer

Temperature Sensor

Onboard Sensor Fusion (Roll, Pitch, Yaw)

**Device & Software Configuration:**

- Device ID: 0388A23E

- Product Code: MTi-3-8A7G6

- Firmware Version: 1.20.1

- Hardware Version: 5.0.0

- Filter Profile: general (50.10)

- Coordinate System: ENU (East-North-Up)

- Software Used:

  - MT Manager: v2025.0.0

  - Xsens Device API (XDA): v2025.0.0 (Build 241121001, Rev 1732180523)

**Disabled Preprocessing & Calibration:**

To ensure raw, unaltered sensor data, the following features were disabled:

- Orientation Smoother

- Position/Velocity Smoother

- Continuous Zero Rotation Update

- Active Heading Stabilization (AHS)

- In-run Compass Calibration (ICC)

This configuration captures raw sensor outputs for accurate modeling of environmental effects.

**Sampling & Conditions:**

Data was collected at three sampling frequencies:

- 25 Hz

- 50 Hz

- 100 Hz

For each frequency, two environmental conditions were tested:

- Clean Environment — Minimal movement, stable surroundings

- Noisy Environment — Introduced mechanical movement or environmental interference

This resulted in six datasets for robust comparative analysis.

**Recorded Features:**
1. SampleTimeFine: High-resolution timestamp for each sample
2. PacketCounter: Sequential packet ID (for detecting dropped/missing data)
3. Acc_X, Acc_Y, Acc_Z: Standard-resolution acceleration (m/s²)
4. AccHR_X, AccHR_Y, AccHR_Z: High-resolution acceleration (g)
5. MAG_X, MAG_Y, MAG_Z: Magnetic field readings (µT)
6. Roll, Pitch, Yaw: Orientation angles (degrees)
7. Temperature: Sensor temperature (°C)

### Tools and Libraries used:
The analysis and modeling were implemented in Python, widely adopted for scientific computing due to its open-source ecosystem and extensive library support.
- Pandas and NumPy were used for structured data handling and high-performance numerical computations.
- Matplotlib and Seaborn facilitated time-series visualization.
- SciPy provided signal processing and statistical tools, including linregress for regression diagnostics.
- Scikit-learn supported machine learning workflows, including model training, scaling, and evaluation metrics.
- Joblib was used for efficient model serialization to enable reuse in deployment stages.

### Noise Characterization
To understand the characteristics of sensor noise, Power Spectral Density (PSD) Analysis:

**1. Power Spectral density**

Power Spectral Density (PSD) is a function that describes how the power of a signal is distributed over frequency. It essentially shows how much power a signal has at different frequencies, making it useful for analyzing signals that extend over time, especially random or periodic signals.

**Welch Method**

Welch's method is used to estimate the Power Spectral Density (PSD) of a signal, i.e., how the power (or variance) of a signal is distributed over frequency.

Welch’s method improves PSD estimation by:

- Segmenting the signal into overlapping windows.

- Applying a window function (e.g., Hann) to reduce spectral leakage.

- Computing the FFT for each segment.

- Averaging the squared magnitudes to reduce variance.

This approach yields a smoother, more reliable PSD compared to a single FFT.

**Welch’s Method (as implemented in code):**

<img width="938" height="382" alt="image" src="https://github.com/user-attachments/assets/22e6954d-4b32-4093-9e4c-ce50387e38f6" />
<img width="693" height="242" alt="image" src="https://github.com/user-attachments/assets/c61cf347-be50-45c2-9353-2ce45207bc5c" />

**3. Noise classification function**

<img width="296" height="191" alt="image" src="https://github.com/user-attachments/assets/5e165fd7-2418-4d77-8566-3a71d5adcccd" />

- Takes the slope of the PSD in log–log scale.

- Uses thresholds to decide the noise type:

  - Flat slope ≈ 0, White noise.

  - Slope ≈ -1, Pink noise.

  - Slope ≈ -2, Brownian noise.
    
<img width="487" height="160" alt="image" src="https://github.com/user-attachments/assets/8601c11c-6058-4aab-9a48-303fc10f5a7f" />

- Selects frequency bins within [fmin, fmax].

- Converts both frequency and PSD into log10 scale.

- Fits a straight line:

  - log10(𝑃𝑆𝐷) = 𝑠𝑙𝑜𝑝𝑒.log10(𝑓) + 𝑖𝑛𝑡𝑒𝑟𝑐𝑒pt

- Returns:

  - slope: spectral slope

  - intercept: y-intercept of the fitted line

  - r²: goodness of fit

<img width="493" height="109" alt="image" src="https://github.com/user-attachments/assets/aaed2fb5-8c0c-4ee2-894a-b244ab184032" />

- Defines three frequency regions for slope fitting:

  - Low (very slow drift)

  - Mid (often pink noise)

  - High (white noise region up to Nyquist = fs/2)

<img width="437" height="203" alt="image" src="https://github.com/user-attachments/assets/ed1153a9-fc2a-4c03-93ce-0347477e6b19" />

  - Loops through each dataset and each axis (X, Y, Z).

  - Skips axis if it’s missing in the DataFrame.
    
  - Extracts signal from DataFrame.
    
  - Removes DC offset (mean).
    
  - Chooses segment length (nperseg) for Welch’s method:
    
  - Default = 1024 or a quarter of data length.
    
  - If too short, use half of the data length.

  - Calls manual Welch implementation.

  - Returns:

    - f: frequency bins
     
    - Pxx: estimated power spectral density

<img width="707" height="258" alt="image" src="https://github.com/user-attachments/assets/0879ccb2-0522-4bd0-88d9-b9ab897d6c1c" />

- For each band:

  - Fit slope (fit_slope).

  - Classify noise type (classify_noise_type).

  - Plot fitted line (loglog) on top of PSD.
   
**Output:**

<img width="871" height="903" alt="image" src="https://github.com/user-attachments/assets/ddae4b24-1073-4994-b871-86963926b321" />

<img width="911" height="911" alt="image" src="https://github.com/user-attachments/assets/0df5c81f-1014-4f2d-bb5a-80948a53fb64" />

<img width="845" height="906" alt="image" src="https://github.com/user-attachments/assets/bebac199-dc7d-4536-bff0-e4728955e6d5" />

<img width="846" height="907" alt="image" src="https://github.com/user-attachments/assets/4e32f85a-7d79-4c67-8144-67a31b80362a" />

<img width="881" height="470" alt="image" src="https://github.com/user-attachments/assets/e08c9d7c-25a4-4ee3-a726-72a8499322ad" />

**Key Observations:**

**1. 25 Hz Sampling**

- AccHR_X & AccHR_Y

  - Slopes = -1.0 to -1.3 across bands, pink noise (1/f) dominates.

  - This means system variability mostly follows correlated (not random) fluctuations.

- AccHR_Z

  - Low slope = -0.7, closer to pink noise.

  - Mid slope = -0.18, almost white noise (flat region).

  - High slope = -0.68, back to pink noise.

  - So Z-axis mixes both pink + white noise.
    
**2. 50 Hz Sampling**

- AccHR_X

  - Low slope = -1.58, Brownian noise (1/f²) at low freq (drift).

  - Mid slope = -0.93, pink noise.

  - High slope = -0.48, tending to white noise.

   - Interpretation: Long-term drift + short-term randomness.

- AccHR_Y

  - Slopes = -1.3 to -1.4, pink noise dominates (strong correlation).

  - High freq slope = -0.95, still pink noise.

- AccHR_Z

  - Slopes = -0.99 to -0.51, all pink noise with weak correlation at higher band.
      
**3. 100 Hz Sampling**

- AccHR_X

  - Low slope = -0.94, pink noise.

  - Mid slope = -2.32, Brownian noise (1/f²) strong drift at mid freq.

  - High slope = -0.53, pink noise.

  - Clear sign of random walk component.
    
- AccHR_Y

  - Low slope = -1.09, pink noise.

  - Mid slope = -2.20, Brownian noise.

  - High slope = -0.42, closer to white noise.

  - Again: drift in mid band, random noise at high freq.

- AccHR_Z

  - Low slope = 0.17, white noise (flat).

  - Mid slope = -1.97, Brownian noise (random drift).

  - High slope = 0.04, white noise again.

  - Very mixed behavior, but drift dominates mid frequencies.

### Combine clean and noisy data with labels and frequency information

<img width="501" height="582" alt="image" src="https://github.com/user-attachments/assets/b81a167f-6422-44d2-bb39-8b36d2286c10" />

**Output:**

<img width="1062" height="1074" alt="image" src="https://github.com/user-attachments/assets/f0806061-4678-42f8-a2b6-99b73295ce0c" />

### Define the features (X) and the target variables (y) for the model

<img width="903" height="98" alt="image" src="https://github.com/user-attachments/assets/e9bec38b-1c4b-41de-b2a0-ffb7dab3d686" />

### Split the combined dataset into training and testing sets

<img width="822" height="199" alt="image" src="https://github.com/user-attachments/assets/f6c5067a-5114-45ec-b383-412e80eb70ee" />

- train_test_split splits into training (80%) and test (20%) sets.

- random_state=42 ensures reproducibility (same split each time).

### Import RandomForestRegressor model

Random Forest is an ensemble learning algorithm that builds multiple decision trees on random subsets of data and averages or votes their results. This makes it accurate, reduces overfitting, and works well for both classification and regression tasks.

<img width="446" height="140" alt="image" src="https://github.com/user-attachments/assets/a388e919-071f-45a6-9ca9-ad549bbd1efe" />

- n_estimators=100: Builds 100 decision trees in the forest.

- random_state=42: Ensures reproducibility of results.

- fit(X_train, y_train): Trains the model on input features (X_train) and target values (y_train).

### Calculate evaluation metrics(RandomForestRegressor model)
<img width="293" height="363" alt="image" src="https://github.com/user-attachments/assets/c03bb9d6-40b1-477e-9c8a-35d6e52d0529" />

**1. MSE (Mean Squared Error):**
   Average squared difference between predicted and actual values. Lower = better.

**2. RMSE (Root Mean Squared Error):**
   Square root of MSE. Easier to interpret because it has the same unit as the target variable.

**3. MAE (Mean Absolute Error):**
   Average absolute difference between predicted and actual values. Lower = better.

**4. R² (Coefficient of Determination):**
   Measures how well the model explains variance in the data.
   - 1.0 = perfect prediction
   - 0 = no better than mean prediction

**Temperature:** 
- MSE = 0.0001, RMSE = 0.0077, MAE = 0.0002, R² = 1.0000
- Excellent fit (almost perfect).

**Mag_X:**
- MSE = 0.0002, RMSE = 0.0130, MAE = 0.0002, R² = 0.9891
- Very high accuracy.

**Mag_Y:**
- MSE = 0.0005, RMSE = 0.0215, MAE = 0.0002, R² = 0.9312
- Still very good but slightly less accurate than Mag_X.

**Mag_Z:**
- MSE = 0.0002, RMSE = 0.0153, MAE = 0.0002, R² = 0.9809
- Excellent performance.

### visualize the model's performance(RandomForestRegressor model)
<img width="1706" height="423" alt="image" src="https://github.com/user-attachments/assets/29d00f8c-e6fd-40e8-b04a-60a4288c25b5" />

**1. Temperature:**
- Points lie almost exactly on the line.
- Indicates near-perfect predictions (R² = 1.0).
  
**2. Mag_X:**
- Predictions follow the line closely but with some spread.
- A few outliers are visible, but overall performance is still strong (R² = 0.9891).

**3. Mag_Y:**
- Strong alignment with the diagonal but more scattered than Mag_X.
- Some deviations (outliers) exist, which explains why R² is slightly lower (= 0.9312).

**4. Mag_Z:**
- Predictions are mostly close to the line, but noticeable scatter and outliers appear.
- R² is lower compared to Temperature and Mag_X (= 0.9809).

### actual vs. predicted values(By RandomForestRegressor model)
<img width="921" height="343" alt="image" src="https://github.com/user-attachments/assets/8e6f8a0f-7587-40da-8af2-f457e317fbc5" />

###  Import K-Nearest Neighbors (KNN) Regressor

<img width="377" height="173" alt="image" src="https://github.com/user-attachments/assets/c002d0d4-418c-466f-b6b8-04fd33621b87" />

- It standardizes features by removing the mean and scaling to unit variance.
<img width="632" height="250" alt="image" src="https://github.com/user-attachments/assets/b7a181ba-4267-4a4b-9ac4-b8f48d819618" />

<img width="504" height="190" alt="image" src="https://github.com/user-attachments/assets/69a9ab03-80ed-42a5-af53-c66031e84ba8" />

K-Nearest Neighbors (KNN) is a non-parametric, distance-based algorithm that predicts outcomes by averaging the values of the k closest neighbors in the feature space. In the code, a KNN Regressor with 5 neighbors is initialized, trained on scaled training data, and used to make predictions on scaled test data. Scaling is essential since KNN relies on distance for neighbor selection.

### Calculate evaluation metrics(KNN model)
**1. Temperature Prediction**

MSE: 0.0017, Very low error.

RMSE: 0.0408, The predictions are off by ~0.04 units on average.

MAE: 0.0010, Practically negligible error.

R²: 0.9999, Almost perfect fit (model explains ~99.99% of variance).

Conclusion: KNN predicts temperature extremely well with near-perfect accuracy.

**2. Magnetometer X (Mag_X)**

MSE: 0.0020

RMSE: 0.0449

MAE: 0.0007

R²: 0.8699, Strong fit (87% variance explained).

Conclusion: Predictions for Mag_X are very accurate and reliable.

**3. Magnetometer Y (Mag_Y)**

MSE: 0.0022

RMSE: 0.0467

MAE: 0.0007

R²: 0.6768, Moderate fit (only 67% variance explained).

Conclusion: The model struggles more with Mag_Y, showing higher unexplained variance.

**4. Magnetometer Z (Mag_Z)**

MSE: 0.0029

RMSE: 0.0538

MAE: 0.0006

R²: 0.7638, Decent fit (76% variance explained), but weaker compared to Temp and Mag_X.

Conclusion: Predictions for Mag_Z are acceptable but less strong.

### visualize the model's performance(KNN Model)

<img width="1688" height="430" alt="image" src="https://github.com/user-attachments/assets/6aee306d-03e3-421e-9756-7d78b8a60407" />

**1. Temperature (KNN)**

   - Points lie almost perfectly along the diagonal line.
   - Very little scatter, predictions almost identical to actual values.

**2. Mag_X (KNN)**

  - Points generally follow the diagonal, but there is moderate scatter around the line.
  - Some deviations for negative and positive values.
  - Matches numerical result (R² ≈ 0.87) → good but not perfect.

**3. Mag_Y (KNN)**
  - Scatter is higher compared to Mag_X.
  - Many points deviate significantly from the diagonal, especially in mid-range values.
  - Consistent with R² ≈ 0.67 → weaker fit.

**4. Mag_Z (KNN)**
  - Shows the highest scatter of all three magnetometer axes.
  - Large deviations, especially for negative values.
  - Matches R² = 0.76, confirming only moderate reliability.

### actual vs. predicted values(KNN model)
<img width="1040" height="345" alt="image" src="https://github.com/user-attachments/assets/3aedc707-d7e3-40a9-b7a2-7c614ae721ea" />


### comparison of RandomForestRegressor vs. KNN Regressor

**1. Random Forest:**

  - Consistently high accuracy across all variables.
    
  - Handles noisy/complex data (like Magnetometer signals) much better.
    
  - Less prone to overfitting due to ensemble averaging.

**2. KNN:**

     - Performs almost perfectly for Temperature (smooth data).
    
     - Accuracy drops significantly for magnetometer axes (sensitive to large dataset, noise and local fluctuations).
     
     - Strongly dependent on proper scaling and the choice of k.
     
     
