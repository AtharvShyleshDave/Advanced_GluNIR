# GluNIR

### Non-Invasive Glucose Monitoring Using NIR/PPG and TinyML

GluNIR is an experimental non-invasive glucose monitoring system that uses near-infrared (NIR) and photoplethysmography (PPG) optical signals, machine learning, and TinyML to estimate blood glucose levels.

The project combines optical signal acquisition, signal processing, machine-learning regression, 1D convolutional neural networks, and embedded deployment on an ESP32.

> **Project status:** Research and prototype. This system is not a medical device and must not be used for clinical diagnosis or treatment decisions.

---

## Overview

Traditional glucose monitoring generally requires a blood sample. GluNIR investigates whether information contained in optical signals measured from tissue can be used to estimate blood glucose concentration non-invasively.

The system follows the pipeline:

```text
NIR / PPG Sensor
       ↓
Signal Acquisition
       ↓
Signal Quality Check
       ↓
Filtering / Denoising
       ↓
Segmentation
       ↓
Normalization
       ↓
Feature Extraction / Waveform Representation
       ↓
Machine Learning
       ├── Random Forest Regressor
       ├── MLP Regressor
       └── 1D CNN Regressor
       ↓
Model Evaluation
       ↓
TinyML Conversion
       ↓
ESP32 Deployment
       ↓
Real-Time Glucose Estimation
```

---

## Why Regression?

Blood glucose is a continuous numerical measurement, typically represented in mg/dL.

Therefore, GluNIR formulates the problem as a regression task:

\[
\hat{y}=f_\theta(X)
\]

where:

- \(X\) = processed NIR/PPG signal
- \(y\) = reference blood glucose
- \(\hat{y}\) = predicted glucose concentration

The models therefore produce a continuous glucose estimate rather than a discrete class such as "low", "normal", or "high".

---

## Machine Learning Models

Three models were investigated to represent different nonlinear learning approaches.

### 1. Random Forest Regressor

A classical ensemble learning method using multiple decision trees.

**Purpose:**
- Establish a nonlinear machine-learning baseline
- Work with engineered signal features
- Provide a robust comparison against neural networks

---

### 2. MLP Regressor

A Multi-Layer Perceptron uses fully connected neural-network layers to learn nonlinear relationships between extracted signal features and glucose.

**Purpose:**
- Evaluate neural learning using engineered features
- Compare feature-based neural learning with tree-based regression

---

### 3. 1D CNN Regressor

A one-dimensional convolutional neural network operates directly on the temporal structure of the processed PPG/NIR waveform.

**Purpose:**
- Learn local waveform characteristics automatically
- Capture temporal and morphological patterns
- Provide a model suitable for TinyML deployment

A 1D CNN was selected for further embedded deployment based on the evaluated pipeline and its suitability for one-dimensional physiological signals.

---

## Why a 1D CNN?

PPG/NIR measurements are fundamentally time-series signals:

\[
x(t)=[x_1,x_2,\ldots,x_N]
\]

A 1D convolution slides a kernel along the temporal dimension to learn local waveform patterns.

A 2D CNN is primarily designed for spatial structures such as images, while a 3D CNN is generally used for volumetric or spatiotemporal data.

Therefore, a 1D CNN provides a more appropriate architecture for the GluNIR waveform representation while keeping computational requirements more suitable for embedded deployment.

---

## Signal Processing Pipeline

The acquired optical signal is processed before being supplied to the machine-learning models.

### Processing stages

1. Signal acquisition
2. Signal-quality checking
3. Filtering and denoising
4. Segmentation into analysis windows
5. Normalization
6. Feature extraction or waveform preparation
7. Model inference

Normalization parameters are derived from the training data to prevent information leakage into validation or test data.

---

## Feature Representation

For classical machine-learning and MLP approaches, signal windows can be represented using engineered features such as:

- Mean
- Standard deviation
- Variance
- RMS
- Minimum and maximum amplitude
- Peak-to-peak amplitude
- Pulse characteristics
- Rise and fall characteristics
- Pulse width
- Peak intervals
- Morphological characteristics
- Frequency-domain characteristics
- Spectral power

For the 1D CNN, the ordered temporal waveform can be supplied directly after preprocessing rather than relying entirely on manually engineered features.

---

## Model Training

The neural-network models are trained using an iterative optimization process:

```text
Input Signal
     ↓
Forward Propagation
     ↓
Predicted Glucose
     ↓
Loss Calculation
     ↓
Backpropagation
     ↓
Gradient Calculation
     ↓
Adam Optimizer
     ↓
Weight Update
     ↓
Repeat
```

The regression loss can be represented using Mean Squared Error:

\[
MSE=\frac{1}{N}\sum_{i=1}^{N}(y_i-\hat{y}_i)^2
\]

Adam is used as the optimizer for neural-network training.

**Important:** Adam, backpropagation, and training epochs are used during offline model training. They are not required during ESP32 inference.

---

## Evaluation

Model performance is evaluated using standard regression metrics.

### Mean Absolute Error

\[
MAE=\frac{1}{N}\sum_{i=1}^{N}|y_i-\hat{y}_i|
\]

MAE represents the average absolute prediction error in glucose units.

### Root Mean Squared Error

\[
RMSE=
\sqrt{
\frac{1}{N}
\sum_{i=1}^{N}
(y_i-\hat{y}_i)^2
}
\]

RMSE gives greater weight to larger prediction errors.

### Coefficient of Determination

\[
R^2=
1-
\frac{
\sum(y_i-\hat{y}_i)^2
}{
\sum(y_i-\bar{y})^2
}
\]

\(R^2\) measures how much of the variance in the target glucose measurements is explained by the model.

---

## Current Evaluation

An evaluated model configuration produced approximately:

| Metric | Result |
|---|---:|
| MAE | **19.2 mg/dL** |
| RMSE | **21 mg/dL** |
| \(R^2\) | **0.15** |

These results indicate that the current model has limited predictive strength, particularly in terms of explained variance. The relatively close MAE and RMSE also indicate that the error is not dominated by a small number of extremely large outliers.

Further improvement is required before the system could be considered suitable for practical or clinical use.

---

## Dataset and Validation

The project uses paired optical signal measurements and reference blood-glucose measurements for supervised learning.

A major consideration is **subject-independent evaluation**.

Samples originating from the same subject are highly correlated. Therefore, allowing samples from the same individual to appear across training and testing sets can lead to data leakage and overly optimistic performance.

The intended evaluation strategy separates subjects between training, validation, and test partitions wherever possible.

Synthetic or augmented samples, when used, should be generated only from the training data and should not replace an independent real-world test set.

---

## TinyML Deployment

After training and evaluation, the selected neural-network model is prepared for embedded inference.

```text
Trained 1D CNN
      ↓
Model Conversion
      ↓
TensorFlow Lite / Embedded Representation
      ↓
Optional Quantization
      ↓
TensorFlow Lite Micro
      ↓
ESP32
```

During deployment, the ESP32 performs inference:

```text
NIR / PPG Sensor
       ↓
ESP32
       ↓
Signal Preprocessing
       ↓
1D CNN
       ↓
Predicted Glucose
```

The training pipeline is **not** executed on the microcontroller.

---

## Hardware

The embedded prototype is based around an ESP32-class microcontroller and optical sensing hardware.

The hardware system is intended to provide:

- Optical signal acquisition
- On-device preprocessing
- TinyML inference
- Real-time glucose estimation
- Low-power embedded operation

Hardware documentation and firmware are maintained separately from the machine-learning training pipeline.

---


---

## Research Objectives

The primary objectives of GluNIR are:

- Investigate non-invasive optical glucose estimation.
- Develop a signal-processing pipeline for NIR/PPG measurements.
- Compare classical and neural nonlinear regression models.
- Investigate waveform-based 1D CNN learning.
- Evaluate model generalization using appropriate validation strategies.
- Convert the trained model into a TinyML-compatible format.
- Deploy glucose estimation inference on an ESP32-class microcontroller.

---

## Limitations

The current prototype has several limitations:

- Optical signals contain physiological and environmental information unrelated to glucose.
- Individual differences can significantly affect the signal.
- Motion, sensor placement, pressure, ambient light, temperature, and perfusion can introduce variation.
- Dataset size and subject diversity affect generalization.
- Model performance on laboratory datasets may not represent real-world performance.
- Current evaluation metrics are not sufficient to establish clinical accuracy or safety.

Consequently, GluNIR should be considered an experimental research prototype rather than a clinical glucose-monitoring device.

---

## Future Work

Potential future improvements include:

- Increasing the number and diversity of subjects.
- Collecting more real-world NIR/PPG measurements.
- Improving signal-quality detection.
- Optimizing filtering and segmentation.
- Investigating additional regression models such as XGBoost and SVR.
- Hyperparameter optimization.
- Quantization-aware TinyML optimization.
- Reducing model size and inference latency.
- Testing across different hardware and sensor configurations.
- Subject-independent external validation.
- More extensive error and agreement analysis.

---

## Technologies

- Python
- NumPy
- Pandas
- Scikit-learn
- TensorFlow / Keras
- TensorFlow Lite
- TensorFlow Lite Micro
- ESP32
- NIR / PPG optical sensing
- Signal processing
- TinyML

---

## Disclaimer

GluNIR is an academic/research prototype. It has not been validated as a medical device and must not be used for diagnosis, treatment, insulin dosing, or other clinical decisions.

---

## License

This project is intended for research and educational purposes. See `LICENSE` for the applicable licensing terms.
