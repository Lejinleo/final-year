# 🚶 3D Gait Authentication System

> **AI-based biometric authentication using human gait patterns extracted from walking videos**

A deep-learning-based **gait authentication system** that identifies and authenticates individuals based on their unique walking patterns. The system processes a walking video, extracts human body keypoints using **MediaPipe Pose**, converts them into a structured skeletal representation, generates a gait embedding using a **deep neural network**, and compares the embedding with registered users using **Euclidean distance**.

The project provides a web-based interface where users can register their gait and subsequently authenticate themselves using a new walking video.

---

## 📌 Overview

Traditional biometric authentication systems commonly rely on fingerprints, facial recognition, iris recognition, or passwords. This project explores **gait biometrics**, where a person's identity is determined from their characteristic walking pattern.

Unlike face or fingerprint authentication, gait recognition can potentially work from a distance without requiring the person to directly interact with a sensor.

The system follows this pipeline:

```text
Walking Video
      ↓
Human Pose Detection
      ↓
MediaPipe Pose Landmarks
      ↓
18-Joint Skeletal Representation
      ↓
Frame Sampling & Normalization
      ↓
Deep Learning Model
      ↓
Gait Embedding
      ↓
Euclidean Distance
      ↓
Threshold-Based Authentication
      ↓
Authenticated / Access Denied
```

---

## ✨ Key Features

* 🎥 **Video-based gait authentication**
* 🦴 Human skeleton extraction using **MediaPipe Pose**
* 📐 Conversion of pose landmarks into an **18-joint representation**
* 🔄 Fixed-length gait sequence generation
* 🧠 Deep-learning-based gait embedding generation
* 🔐 Threshold-based biometric authentication
* 👤 New-user gait registration
* 📊 Comparison against multiple registered users
* 🌐 Web-based authentication interface
* ⚡ Flask REST API backend
* 🧹 Automatic temporary video cleanup
* 💾 Persistent storage of registered gait embeddings

---

## 🧠 How It Works

### 1. User Registration

During registration, the user provides:

* A unique user ID/name
* A walking video

The backend processes the video and generates a gait embedding.

The embedding is then stored in the local gallery database associated with the registered user.

```text
User
 ↓
Walking Video
 ↓
Pose Extraction
 ↓
Skeleton Sequence
 ↓
Normalization
 ↓
Gait Model
 ↓
Embedding
 ↓
Gallery Database
```

---

### 2. Authentication

During authentication, the user uploads a new walking video.

The system:

1. Extracts the person's skeleton from the video.
2. Resamples the detected skeleton sequence.
3. Normalizes the skeletal coordinates.
4. Generates a gait embedding.
5. Compares the embedding with stored user embeddings.
6. Finds the closest matching registered user.
7. Calculates the Euclidean distance.
8. Compares the distance against the security threshold.

```text
New Walking Video
       ↓
Pose Extraction
       ↓
Skeleton Sequence
       ↓
Normalization
       ↓
Gait Embedding
       ↓
Compare with Gallery
       ↓
Euclidean Distance
       ↓
Security Threshold
       ↓
 ┌───────────────┐
 │ Match         │ → Access Granted
 │ No Match      │ → Access Denied
 └───────────────┘
```

---

## 🔬 Pose Processing

The system uses **MediaPipe Pose** for human pose estimation.

MediaPipe detects **33 body landmarks** from each video frame. The project maps the required landmarks into an **18-joint skeletal representation** inspired by OpenPose-style body representation.

The extracted coordinates include important body regions such as:

* Nose
* Neck
* Shoulders
* Elbows
* Wrists
* Hips
* Knees
* Ankles
* Selected facial landmarks

The resulting representation contains **36 coordinate values** corresponding to 18 joints × 2 coordinates.

The implementation uses a fixed reference resolution of:

```text
Width  = 1280
Height = 980
```

This allows the extracted coordinates to remain compatible with the trained preprocessing pipeline.

---

## 🧮 Sequence Processing

After skeleton extraction, the system verifies that enough valid skeleton frames have been detected.

Videos producing fewer than **10 valid skeleton frames** are rejected.

Valid sequences are then resampled to exactly:

```text
24 frames
```

Each frame contains:

```text
36 features
```

Therefore, the model receives a sequence of:

```text
24 × 36
```

The features are normalized using the trained scaler before being passed to the neural network.

---

## 🧠 Deep Learning

The project uses a deep-learning gait representation model based on a **Siamese/Bi-GRU approach** developed during the training phase.

The purpose of the model is to learn a compact representation, or **embedding**, of the person's gait pattern.

The trained model is loaded during application startup and used to convert each processed gait sequence into an embedding vector.

### Model Pipeline

```text
Skeleton Sequence
       ↓
Normalization
       ↓
Bi-GRU / Siamese Representation
       ↓
Gait Embedding
```

The model is stored as:

```text
tower_model_phase2.keras
```

---

## 🔐 Authentication Algorithm

For every registered user, one or more gait embeddings can be stored.

For an authentication attempt, the system calculates the Euclidean distance between the newly generated embedding and the stored embeddings.

```text
Distance = || Live Embedding - Stored Embedding ||
```

The system calculates the average distance for each registered user and selects the user with the smallest distance.

The current security threshold is:

```text
0.115
```

If:

```text
Distance < 0.115
```

the user is considered authenticated.

Otherwise:

```text
Access Denied
```

The threshold is configured in `config.py`.

---

## 🏗️ System Architecture

```text
                    ┌───────────────────────┐
                    │       Web Client      │
                    │   HTML / CSS / JS     │
                    └───────────┬───────────┘
                                │
                                │ HTTP POST
                                ▼
                    ┌───────────────────────┐
                    │     Flask Backend     │
                    │      REST API         │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │     GaitSystem       │
                    │   AI Processing       │
                    └───────────┬───────────┘
                                │
                ┌───────────────┼────────────────┐
                ▼               ▼                ▼
        ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
        │   OpenCV    │ │  MediaPipe  │ │   Scaler    │
        │Video Reader │ │    Pose     │ │  Transform  │
        └─────────────┘ └─────────────┘ └─────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   Deep Learning      │
                    │      Model           │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │    Gait Embedding    │
                    └───────────┬───────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │ Gallery Embeddings   │
                    │      Database        │
                    └───────────────────────┘
```

---

## 🛠️ Technology Stack

### Programming

* **Python**
* **JavaScript**
* **HTML5**
* **CSS3**

### Artificial Intelligence & Computer Vision

* **TensorFlow**
* **Keras**
* **MediaPipe Pose**
* **OpenCV**
* **NumPy**
* **Scikit-learn**
* **Joblib**

### Deep Learning

* Siamese Network
* Bidirectional GRU (Bi-GRU)
* Gait Embeddings
* Contrastive Learning
* Euclidean Distance

### Backend

* **Flask**
* **Flask-CORS**
* REST API

### Development

* Google Colab
* VS Code
* Git
* GitHub

The repository currently pins Flask 3.0.0, Flask-CORS 4.0.0, TensorFlow 2.15.0, MediaPipe 0.10.14, OpenCV 4.9.0.80, and NumPy <2.0 among its backend dependencies.

---

## 📁 Project Structure

```text
final-year/
│
├── backend/
│   ├── app.py
│   ├── processor.py
│   ├── config.py
│   ├── requirements.txt
│   │
│   └── model_assets/
│       ├── tower_model_phase2.keras
│       ├── scaler.joblib
│       └── gallery_embeddings.pkl
│
├── index.html
├── auth.html
├── login.html
├── ABUT.html
├── HIW.html
│
├── app.js
├── login.js
├── login.css
└── style.css
```

The current GitHub repository contains the frontend files alongside the `backend` directory.

---

## ⚙️ Backend API

The Flask backend exposes two main endpoints.

### Register User

```http
POST /register
```

Parameters:

```text
user_id
gait_video
```

Example response:

```json
{
    "user_id": "user1",
    "message": "User registered successfully"
}
```

The uploaded video is temporarily stored, processed, and then removed after processing.

---

### Authenticate User

```http
POST /authenticate
```

Parameter:

```text
gait_video
```

Successful authentication returns information such as:

```json
{
    "authenticated": true,
    "user_id": "user1",
    "score": 0.08
}
```

A gait mismatch returns:

```json
{
    "authenticated": false,
    "details": "Gait mismatch",
    "score": 0.15
}
```

---

## 🚀 Installation

### 1. Clone the repository

```bash
git clone https://github.com/Lejinleo/final-year.git
cd final-year
```

### 2. Create a virtual environment

```bash
python -m venv venv
```

Activate it on Windows:

```bash
venv\Scripts\activate
```

### 3. Install dependencies

```bash
cd backend
pip install -r requirements.txt
```

### 4. Verify model assets

Make sure the following files are available inside:

```text
backend/model_assets/
```

```text
tower_model_phase2.keras
scaler.joblib
gallery_embeddings.pkl
```

The backend configuration explicitly expects these three assets.

### 5. Start the Flask server

```bash
python app.py
```

The server runs on:

```text
http://127.0.0.1:5000
```

### 6. Open the frontend

Open:

```text
index.html
```

in your browser and use the registration/authentication interface.

---

## 🎯 Usage

### Register

1. Open the application.
2. Select **Register**.
3. Enter a user name/ID.
4. Upload a walking video.
5. Submit the video.
6. The system extracts the gait features.
7. The generated embedding is stored in the gallery.

### Authenticate

1. Select **Sign In / Authenticate**.
2. Upload a new walking video.
3. The system extracts the gait embedding.
4. The embedding is compared against registered users.
5. The closest match is selected.
6. The distance is compared with the security threshold.
7. Access is either granted or denied.

---

## 📊 Model Assets

| File                       | Purpose                                |
| -------------------------- | -------------------------------------- |
| `tower_model_phase2.keras` | Trained gait embedding model           |
| `scaler.joblib`            | Feature normalization/scaling          |
| `gallery_embeddings.pkl`   | Stored registered-user gait embeddings |

These files are loaded automatically by `GaitSystem` when the Flask application starts.

---

## 🔒 Security Considerations

This project is an academic prototype demonstrating gait-based biometric authentication.

The current implementation uses:

* Gait embeddings rather than raw video for the stored gallery
* Distance-based matching
* Configurable authentication threshold
* Temporary video files that are deleted after processing
* CORS-enabled API communication

For production deployment, additional security measures would be required, including:

* HTTPS
* Sec
