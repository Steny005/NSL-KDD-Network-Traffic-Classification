# Network Traffic Classification using Machine Learning

## Overview

This project implements a machine-learning-based network traffic classification system that identifies network traffic as either **Benign (Normal)** or **Malicious (Attack)**. The project uses the **NSL-KDD dataset** and applies data preprocessing, feature encoding, scaling, model training, evaluation, and live prediction through a REST API.

The project compares a **Decision Tree** baseline with a **Random Forest** ensemble model. The trained Random Forest model is then exposed through a Flask REST API, allowing network traffic data to be submitted for classification. The project also demonstrates live packet capture using Scapy and sends extracted packet features to the prediction API.

## Dataset

The project uses the **NSL-KDD KDDTrain+ dataset**. The dataset contains network traffic records with numerical and categorical features describing network connections.

The dataset initially contains **125,973 records and 42 columns**. The `difficulty` column is removed, and the original attack labels are converted into a binary classification format:

* `0` → Normal traffic
* `1` → Attack traffic

The features include network connection information such as protocol type, service, connection flags, source and destination bytes, connection counts, error rates, and destination-host statistics.

## Technologies Used

* Python
* Pandas
* NumPy
* Scikit-learn
* Joblib
* Flask
* Pyngrok
* Requests
* Scapy
* Google Colab

## Project Workflow

### 1. Dataset Loading

The NSL-KDD dataset is downloaded and loaded into a Pandas DataFrame. The dataset columns are explicitly assigned, the `difficulty` feature is removed, and the target label is converted into a binary value.

### 2. Data Preprocessing

The `label` column is separated from the input features. The categorical features — `protocol_type`, `service`, and `flag` — are converted into numerical representations using one-hot encoding.

The dataset is divided into training and testing sets using an **80:20 stratified split**. Numerical features are standardized using `StandardScaler`, with the scaler fitted only on the training data to avoid data leakage. The preprocessing artifacts are saved using Joblib.

After preprocessing, the training set contains **100,778 samples**, while the test set contains **25,195 samples**, with **119 features** after encoding.

### 3. Model Training

Two classification models are trained:

* **Decision Tree Classifier** with a maximum depth of 10
* **Random Forest Classifier** with 100 decision trees

The Random Forest is used as the final ensemble model and is serialized using Joblib for later inference.

### 4. Model Evaluation

The models are evaluated using classification reports, a confusion matrix, and ROC-AUC.

The Random Forest achieved a reported **ROC-AUC score of 1.0000** on the test set. Its confusion matrix contains 13,462 correctly classified normal samples, 7 false positives, 19 false negatives, and 11,707 correctly classified attack samples.

The Decision Tree and Random Forest classification reports in the notebook both report approximately **1.00 accuracy, precision, recall, and F1-score** on the provided test split.

## REST API

The trained model is deployed using **Flask**. The saved Random Forest model, scaler, and feature-column list are loaded when the API starts.

The API provides a POST endpoint:

```text
/predict
```

The endpoint accepts network traffic information in JSON format, aligns the incoming data with the trained feature structure, applies the saved scaling transformation, and passes the processed data to the Random Forest model.

The API returns one of two classifications:

```text
MALICIOUS (ATTACK DETECTED)
```

or

```text
BENIGN (NORMAL TRAFFIC)
```

The notebook demonstrates the API running on port `5002` and exposing it through an ngrok tunnel.

## Example API Request

A sample network packet can be submitted using Python's `requests` library:

```python
sample_packet = {
    "duration": 0,
    "src_bytes": 181,
    "dst_bytes": 5450,
    "count": 8,
    "srv_count": 8,
    "serror_rate": 0.0,
    "srv_serror_rate": 0.0,
    "same_srv_rate": 1.0,
    "protocol_type_tcp": 1,
    "service_http": 1,
    "flag_SF": 1
}

response = requests.post(
    "YOUR_API_URL/predict",
    json=sample_packet
)

print(response.json())
```

The sample request in the notebook successfully returned:

```json
{
    "prediction": "BENIGN (NORMAL TRAFFIC)",
    "status": "success"
}
```

## Live Network Packet Detection

The project also includes a live packet-capture demonstration using **Scapy**. A network packet is captured, selected features are extracted dynamically, and the resulting information is sent directly to the Flask prediction endpoint.

The demonstration captures one packet and prints the model's classification result. In the recorded execution, the captured packet was classified as:

```text
BENIGN (NORMAL TRAFFIC)
```

## Saved Model Artifacts

The notebook generates the following files:

```text
scaler.joblib
feature_columns.joblib
best_ids_model.joblib
```

`scaler.joblib` contains the fitted numerical feature scaler, `feature_columns.joblib` stores the feature structure produced during preprocessing, and `best_ids_model.joblib` contains the trained Random Forest model.

## How to Run

Open the notebook in **Google Colab** or another Python/Jupyter environment.

Run the cells in order to:

1. Download and load the NSL-KDD dataset.
2. Preprocess and encode the network traffic features.
3. Split the data into training and testing sets.
4. Train the Decision Tree and Random Forest models.
5. Evaluate model performance.
6. Save the trained model and preprocessing artifacts.
7. Start the Flask prediction API.
8. Send sample network traffic to the `/predict` endpoint.
9. Optionally capture a live network packet using Scapy.

The notebook installs the required `pyngrok`, `flask`, and `scapy` packages during execution.

## Project Structure

```text
NETWORK_CLASSIFICATION/
│
├── NETWORK_CLASSIFICATION.ipynb
├── network_traffic_real.csv
├── scaler.joblib
├── feature_columns.joblib
└── best_ids_model.joblib
```

## Important Note

The live API and packet-capture portions are demonstrated in a Google Colab environment using Flask and an ngrok tunnel. The ngrok URL generated during a notebook session is temporary and should not be treated as a permanent API endpoint. For real-world deployment, the API should be hosted using an appropriate production server and deployment environment.

## Conclusion

This project demonstrates an end-to-end machine learning workflow for network traffic classification, beginning with the NSL-KDD dataset and continuing through preprocessing, model training, evaluation, model serialization, REST API deployment, and live packet classification. The combination of Random Forest classification with a Flask inference API provides a practical demonstration of how a trained network classification model can be integrated into a live prediction workflow.
