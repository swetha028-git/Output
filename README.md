# Output
Isolation_Forest
"""isolation _forest.ipynb

Automatically generated by Colaboratory.

Original file is located at
    https://colab.research.google.com/drive/1W8MVh8cTIAEHR80o1G06QM2NkVYmLSxk
"""

import numpy as np
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from sklearn.ensemble import IsolationForest
import tensorflow as tf
import matplotlib.pyplot as plt

IMAGE_SIZE = 256
BATCH_SIZE = 32
CHANNELS = 3
EPOCHS = 50

train_data_dir = r'/content/drive/MyDrive/dataset'

dataset = tf.keras.preprocessing.image_dataset_from_directory(
    train_data_dir,
    seed=123,
    shuffle=True,
    image_size=(IMAGE_SIZE, IMAGE_SIZE),
    batch_size=BATCH_SIZE
)

class_names = dataset.class_names

def get_dataset_partitions_tf(ds, train_split=0.8, val_split=0.1, test_split=0.1, shuffle=True, shuffle_size=10000):
    assert (train_split + test_split + val_split) == 1

    ds_size = len(ds)

    if shuffle:
        ds = ds.shuffle(shuffle_size, seed=12)

    train_size = int(train_split * ds_size)
    val_size = int(val_split * ds_size)

    train_ds = ds.take(train_size)
    val_ds = ds.skip(train_size).take(val_size)
    test_ds = ds.skip(train_size).skip(val_size)

    return train_ds, val_ds, test_ds
train_ds, val_ds, test_ds = get_dataset_partitions_tf(dataset)

X_train = []
for image_batch, _ in train_ds:
    for image in image_batch:
        X_train.append(image.numpy().flatten())
X_train = np.array(X_train)

X_train = X_train / 255.0

isolation_forest = IsolationForest()
isolation_forest.fit(X_train)

def display_predictions(images_batch, labels_batch, predictions):
    plt.figure(figsize=(15, 15))
    for i in range(9):
        ax = plt.subplot(3, 3, i + 1)
        plt.imshow(images_batch[i].numpy().astype("uint8"))

        predicted_class, confidence = predictions[i]
        actual_class = class_names[labels_batch[i]]

        plt.title(f"Actual: {actual_class},\n Predicted: {predicted_class}.\n Confidence: {confidence}%" if predicted_class != 'Anomaly' else "Actual: {actual_class},\n Predicted: {predicted_class}.")

        plt.axis("off")

anomaly_scores = isolation_forest.decision_function(X_train)
predictions = []
for i, score in enumerate(anomaly_scores):
    if score < -0.5:  # Adjust threshold as needed
        predicted_class = 'Anomaly'
    else:
        predicted_class = class_names[np.argmax(score)]
    confidence = round(100 * np.max(score), 2) if predicted_class != 'Anomaly' else None
    predictions.append((predicted_class, confidence))

for images, labels in test_ds.take(1):
    display_predictions(images, labels, predictions)
    plt.show()
