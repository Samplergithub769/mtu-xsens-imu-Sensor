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

<img width="700" height="222" alt="image" src="https://github.com/user-attachments/assets/90f20b07-fb77-40dc-a2a3-cd77920358ad" />

Where:

<img width="981" height="273" alt="image" src="https://github.com/user-attachments/assets/f3ff71d0-9561-4f73-bf68-ac1d55aad4f7" />

**Output:**



