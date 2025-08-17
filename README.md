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
To understand the characteristics of sensor noise, two methods were used:

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

1. 25 Hz Sampling

- AccHR_X & AccHR_Y

  - Slopes = -1.0 to -1.3 across bands, pink noise (1/f) dominates.

  - This means system variability mostly follows correlated (not random) fluctuations.

- AccHR_Z

  - Low slope = -0.7, closer to pink noise.

  - Mid slope = -0.18, almost white noise (flat region).

  - High slope = -0.68, back to pink noise.

  - So Z-axis mixes both pink + white noise.
    
2. 50 Hz Sampling

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
      
3. 100 Hz Sampling

- AccHR_X

  - Low slope ≈ -0.94, pink noise.

  - Mid slope ≈ -2.32, Brownian noise (1/f²) strong drift at mid freq.

  - High slope ≈ -0.53, pink noise.

  - Clear sign of random walk component.
    
- AccHR_Y

  - Low slope ≈ -1.09 → pink noise.

  - Mid slope ≈ -2.20 → Brownian noise.

  - High slope ≈ -0.42 → closer to white noise.

  - Again: drift in mid band, random noise at high freq.

- AccHR_Z

  - Low slope ≈ 0.17 → white noise (flat).

  - Mid slope ≈ -1.97 → Brownian noise (random drift).

  - High slope ≈ 0.04 → white noise again.

  - Very mixed behavior, but drift dominates mid frequencies.
 

