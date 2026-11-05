
# README: Unsupervised Sequence Anomaly Detection with Physics-Guided LSTM Autoencoder

This notebook establishes an **unsupervised sequence reconstruction** pipeline to detect cyber anomalies using network flow data. The primary objective is to build a robust statistical baseline of normal behavior and flag deviations as anomalies.

## 📁 Dataset Setup Requirement
Because the Friday dataset (`friday.csv`) is extremely large, please ensure it is uploaded directly to your environment's root directory: 
* Target Path: `/content/friday.csv` 


## Technical Process & Workflow

1. **Data Acquisition & Safe Ingestion**
   * Load a subset of network traffic records chronologically (`friday.csv`).
   * Standardize column metadata and clean infinite/NaN records to preserve runtime memory safety.

2. **Feature Engineering & Scaled Behavioral Fingerprint**
   * Isolate key behavioral markers (e.g., *Flow Duration*, *Total Fwd/Bwd Packets*, *Flow Bytes/s*, *Flow Packets/s*).
   * Apply a robust logarithmic transformation to reduce multi-scale skewness and normalize distributions.
   * Scale numerical ranges relative to strictly benign network behavior using `RobustScaler`.

3. **Temporal Grouping & Padding (Sliding Window)**
   * Group records chronologically by network conversation keys (*Src IP*, *Dst IP*, *Src Port*, *Dst Port*, *Protocol*).
   * Partition conversations into temporal sequences of 5 time-steps.
   * Automatically pad short-lived sequences with zeros to avoid window fragmentation.

4. **Advanced Physics-Guided Chaotic Feature Space**
   * Calculate statistical properties of sequence dynamics to guide the LSTM:
     * **Shannon Entropy** to represent signal uncertainty and chaos.
     * **Sequence Variance** to track sudden physical behavioral shifts.
     * **Rolling Standard Deviation** to capture sequence volatility.
   * Concatenate these statistical signals with the normalized behavioral features to build an expanded physics-guided space.

5. **Deep Sequence-to-Sequence Autoencoder Model**
   * Build an LSTM Autoencoder with `Masking` layers to ignore zero-padding inputs.
   * Train with **Sample Weighting** to ensure zero-padded time-steps are completely ignored during reconstruction loss backpropagation.
   * Train the architecture strictly on normal/benign network patterns so the model learns a high-fidelity representation of legitimate profiles.

6. **Strict Leak-Free Split & Evaluation**
   * Partition benign sequences strictly into **60% Train, 20% Validation, and 20% Test** splits.
   * Sample anomalies exclusively to pair with the 20% Test partition to construct a balanced, zero-leak evaluation dataset.
   * Predict reconstruction error (Mean Absolute Error) on benign sequences to calculate baseline statistics (mean and standard deviation).
   * Calibrate a sensitive dynamic threshold to capture complex, multi-stage attacks.
