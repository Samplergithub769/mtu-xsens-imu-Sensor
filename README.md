# mtu-xsens-imu-Sensor

## Project Objective
Predict surrounding temperature and magnetic field values that are influenced by device behavior (movement, frequency, noise, etc.)

## Implementation Steps:
### Data Data Acquisition

The experimental dataset was obtained using the Xsens MTi-3-8A7G6 Inertial Measurement Unit (IMU), a compact MEMS-based sensing device. Data collection was conducted using MT Manager (v2025.0.0) and the Xsens Device API (XDA v2025.0.0), with the coordinate frame set to East-North-Up (ENU). To ensure acquisition of uncompensated sensor measurements, all internal smoothing and calibration features—including orientation smoother, position/velocity smoother, continuous zero rotation update, active heading stabilization, and in-run compass calibration — were disabled [15].
Measurements were recorded at three sampling rates (25 Hz, 50 Hz, and 100 Hz) under two distinct environmental scenarios:
•	Clean Environment – minimal external interference and vibration.
•	Noisy Environment – induced mechanical motion and magnetic disturbances.
The recorded parameters included high-resolution timestamps, packet counters, standard and high-resolution accelerometer readings, magnetic field components (X, Y, Z), orientation angles (roll, pitch, yaw), and ambient temperature. 

