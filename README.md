## 🧠 PROJECT OVERVIEW

This project is an **American Sign Language recognition system** that allows users to sign using hand gestures in front of a webcam. The system detects the gestures, classifies the ASL alphabet, and converts it to text in a GUI. The text can be spoken aloud, saved, or translated further.

---

## 🧩 COMPONENT BREAKDOWN

### ✅ 1. `data_collection_binary.py` / `data_collection_final.py`

These scripts are responsible for:

* Capturing video input from the webcam.
* Using `mediapipe` to extract **21 hand landmarks**.
* Saving the hand keypoints or image frames with **labels (A–Z)**.
* Organizing them into folders (`train/`, `test/`, etc.).
* **Output:** Image dataset and keypoints, later used for training.

---

### ✅ 2. `cnn8grps_rad1_model.h5`

This is your **trained CNN model**, saved using Keras. It classifies hand signs into 8 main groups and further distinguishes them into 26 letters using rule-based heuristics.

The model was trained on the **hand-drawn landmark-based images**, not raw frames. Based on the training script logic and architecture, the data was likely structured as:

* 8 classes (groups of letters).
* Input size: `(400, 400, 3)`
* CNN model (used `Conv2D`, `MaxPooling`, `Dropout`, etc.)
* Final `Dense(8, softmax)` layer for group classification.

---

### ✅ 3. `prediction_wo_gui.py`

This is the **non-GUI testing script**:

* Loads the `.h5` model.
* Takes a static image input (e.g. `"6.jpg"`).
* Preprocesses the image.
* Predicts group using the model, then maps it back to a specific letter using geometric rules.

---

### ✅ 4. `application.py` (💡 GUI Interface)

This is the full GUI application:

* Built with `tkinter`.
* Real-time webcam capture.
* Uses `cvzone.HandTrackingModule` to detect and extract hand landmarks.
* Creates a hand-skeleton drawing on a blank image (`white.jpg`).
* Uses your trained model to classify the image into 1 of 8 groups.
* **Custom logic (based on landmark positions)** further distinguishes letters.
* GUI also includes:

  * Current letter.
  * Live sentence.
  * Suggestion buttons.
  * "Speak" and "Clear" functionality.
  * "Save History" and "Next" sentence.
  * Uses `pyttsx3` for text-to-speech.
  * Uses `enchant` dictionary for English word suggestions.

---

## 🔁 HOW IT WORKS (END TO END)

1. **Dataset Collection**: Hand gesture images are captured and grouped by label.
2. **Training**: Model is trained on grouped classes (8 groups). Later, rules are applied to derive exact ASL letters.
3. **Prediction**: Given a hand pose:

   * Frame → Preprocessing → White skeletonized image.
   * Model predicts one of the 8 letter groups.
   * Heuristics on landmark points (distances, positions) determine the final alphabet.
4. **GUI App**: Displays live video feed, prediction, suggestions, allows speaking or saving sentences.

---

## 💬 COMMON QUESTIONS YOU MIGHT BE ASKED

### 🔹 Q1: Why use 8 groups instead of 26 classes directly?

**Answer**: To improve accuracy and reduce model complexity. Similar-looking signs are grouped, and then geometric heuristics distinguish them (e.g. M, N, T, S are visually similar).

---

### 🔹 Q2: What are the heuristics used in classification?

**Answer**: Based on distances and angles between key hand landmarks like:

* Thumb and index finger.
* Wrist to tip distances.
* Horizontal/vertical position comparison of finger tips.
* This is done manually inside `predict()` in `application.py`.

---

### 🔹 Q3: Why is `white.jpg` used?

**Answer**: It's a blank canvas where hand joints are drawn using `cv2.line()` to create a **skeleton image**, reducing background noise and improving model consistency.

---

### 🔹 Q4: What happens if two similar signs are predicted?

**Answer**: The model returns a top-2 list. The heuristic-based `predict()` function checks `ch1`, `ch2`, and then resolves ambiguity using hand geometry.

---

### 🔹 Q5: How does the GUI handle word suggestions?

**Answer**: After adding each letter:

* The current word is extracted using `.rfind(" ")`.
* `enchant.Dict("en-US")` is used to check spelling and generate suggestions.
* Four buttons update with top word suggestions.

---

### 🔹 Q6: What if the prediction is wrong?

**Answer**: User can:

* Choose from the 4 suggestions.
* Use backspace gesture (specific gesture for deleting).
* Clear or retype.

---

### 🔹 Q7: Why is `speak_fun` using `say -v Alex`?

**Answer**: It uses macOS's built-in `say` TTS with Alex voice. If not on macOS, you could use `pyttsx3.speak()` or other platforms' TTS.

---

### 🔹 Q8: Can this model work on other sign languages?

**Answer**: No, it is specifically designed for **American Sign Language (ASL)**. Each language has unique signs and grammar.

---

## 🔬 Model Training Code Summary

Here’s a compact version of what the training script did:

```python
model = Sequential([
    Conv2D(32, (3,3), activation='relu', input_shape=(400,400,3)),
    MaxPooling2D(2,2),
    Dropout(0.25),
    
    Conv2D(64, (3,3), activation='relu'),
    MaxPooling2D(2,2),
    Dropout(0.25),
    
    Flatten(),
    Dense(128, activation='relu'),
    Dropout(0.5),
    Dense(8, activation='softmax')  # 8 groups
])

model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
model.fit(train_data, epochs=10, validation_data=val_data)
model.save('cnn8grps_rad1_model.h5')
```

---

## 🛠️ Improvements You Could Do

1. **Upgrade to NumPy 2-compatible libraries** (you had errors related to this).
2. Replace manual heuristics with a second-stage ML classifier.
3. Add "backspace" and "space" gesture recognitions more robustly.
4. Use `MediaPipe Hands` + `TFLite` model for faster mobile deployment.
5. Add sentence grammar correction using GPT API.

---

## ✅ Final Notes

This is a **well-engineered ASL-to-text system** with:

* Real-time gesture recognition.
* GUI application with voice and suggestion features.
* Modular dataset collection and training scripts.
* Highly flexible for extension into other gestures/commands.

---
