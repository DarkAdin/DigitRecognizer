{
 "cells": [
  {
   "cell_type": "markdown",
   "id": "f03c0c02-02c9-4703-b4c6-52875d369215",
   "metadata": {},
   "source": [
    "# Digit Recognizer\n",
    "\n",
    "Simple implementation of a computer vision algorithm for learning and predicting labels on the MNIST dataset for the Kaggle competition of Digit Recognizer: https://www.kaggle.com/competitions/digit-recognizer\n",
    "\n",
    "* Reading of the data\n",
    "* Normalizing it\n",
    "* Converting into matrix form\n",
    "* Augment the data\n",
    "    * By rotating the images\n",
    "    * By zooming in/out of the images\n",
    "* Learning through a Convolutional Neural Network\n",
    "* Prediction on a test partition\n",
    "* Evaluation of the learning process\n",
    "* Performing the predictions on the unlabeled data\n",
    "* Saving the predictions on disk\n",
    "* Saving the model itself on disk (re-train it, etc.)\n",
    "\n",
    "It returns and saves predictions on disk directly submittable into the competition."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "id": "b5b64dfc-c2aa-47de-9b26-67701ca42816",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "2024-08-28 20:27:04.437592: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.\n",
      "To enable the following instructions: SSE3 SSE4.1 SSE4.2 AVX AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.\n"
     ]
    }
   ],
   "source": [
    "import tensorflow as tf\n",
    "from tensorflow import keras\n",
    "from tensorflow.keras import datasets, layers, models\n",
    "import matplotlib.pyplot as plt\n",
    "import pandas as pd\n",
    "import numpy as np"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "id": "b05338e4-1196-4d99-9da7-2936e447ebd2",
   "metadata": {},
   "outputs": [],
   "source": [
    "# White background in graphs\n",
    "plt.style.use('dark_background')\n",
    "\n",
    "# Dimensions of every pyplot figure. Change to suit your needs\n",
    "plt.rcParams['figure.figsize'] = [10, 5]"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "7250c5a5-2222-4fe0-af27-67b932872fec",
   "metadata": {},
   "source": [
    "# Read data"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "8f8d1b91-9c26-4d02-b3af-6404a936f315",
   "metadata": {},
   "source": [
    "## If you have matrix-based data"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "id": "e798409c-26a8-4a02-bd3f-ae111c9b18a5",
   "metadata": {},
   "outputs": [],
   "source": [
    "#(train_images, train_labels), (test_images, test_labels) = datasets.cifar10.load_data()\n",
    "#train_images, test_images = train_images/255.0, test_images/255.0"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "id": "a717a48d-1008-4522-b78a-15957de30fb2",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_images = pd.read_csv('train.csv')\n",
    "test_images = pd.read_csv('test.csv')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "id": "1e6b13e9-928c-488a-b2a7-180f6656400f",
   "metadata": {},
   "outputs": [],
   "source": [
    "template = pd.read_csv('sample_submission.csv')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "id": "5a2fda08-a087-4712-8d2c-4931b16e69e0",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_labels = train_images['label']\n",
    "train_images.drop(['label'], inplace = True, axis = 1)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "id": "c200cb1e-9c02-4345-8604-a5ab6fa5f0e2",
   "metadata": {},
   "outputs": [],
   "source": [
    "# Normalize\n",
    "train_images, test_images = train_images/255.0, test_images/255.0"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "f2f30244-b584-48c3-b470-8265a2dbbb36",
   "metadata": {},
   "source": [
    "## If you have images"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "a22eaaf7-542b-4b8d-927c-6d2de4407b22",
   "metadata": {},
   "outputs": [],
   "source": [
    "from keras import backend as K\n",
    "from tensorflow.keras.preprocessing.image import ImageDataGenerator\n",
    "\n",
    "#anchura, altura = 120, 120\n",
    "\n",
    "# Las imágenes DEBEN estar colocadas en carpetas divididas por clases\n",
    "# y esas carpetas deben llamarse como las clases\n",
    "\n",
    "entrenamiento = '/home/adan/Pistachos/Pistachio_Image_Dataset/Pistachio_Image_Dataset/Train/'\n",
    "validacion = '/home/adan/Pistachos/Pistachio_Image_Dataset/Pistachio_Image_Dataset/Test/'\n",
    "\n",
    "if K.image_data_format() == 'channels_first':\n",
    "    input_shape = (3, anchura, altura)\n",
    "else:\n",
    "    input_shape = (anchura, altura, 3)\n",
    "\n",
    "train_datagen = ImageDataGenerator(\n",
    "    rescale= 1.0/255.0,\n",
    "    #shear_range=0.2,\n",
    "    #zoom_range=0.2,\n",
    "    #horizontal_flip=True\n",
    ")\n",
    "\n",
    "test_datagen = ImageDataGenerator(\n",
    "    rescale= 1.0/255.0\n",
    ")\n",
    "\n",
    "train_generator = train_datagen.flow_from_directory(\n",
    "    entrenamiento,\n",
    "    target_size=(anchura, altura),\n",
    "    #batch_size=32,\n",
    "    #classes = ['siirt','kirmizi'],\n",
    "    #class_mode='binary'\n",
    ")\n",
    "\n",
    "validation_generator = test_datagen.flow_from_directory(\n",
    "    validacion,\n",
    "    target_size=(anchura, altura),\n",
    "    #batch_size=32,\n",
    "    #classes = ['siirt','kirmizi'],\n",
    "    #class_mode='binary'\n",
    ")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "9d80390c-92e4-427c-814c-170d777c1f2c",
   "metadata": {},
   "outputs": [],
   "source": [
    "len(validation_generator.classes)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "02cbc76d-02f4-48f7-812f-f556632ca6bc",
   "metadata": {},
   "outputs": [],
   "source": [
    "#class_names = ['airplane', 'automobile', 'bird', 'cat', 'deer','dog', 'frog', 'horse', 'ship', 'truck']\n",
    "#plt.figure(figsize=(7,7))\n",
    "#for i in range(25):\n",
    "    #plt.subplot(5,5,i+1)\n",
    "    #plt.xticks([])\n",
    "    #plt.yticks([])\n",
    "    #plt.grid(False)\n",
    "    #plt.imshow(train_images[i])\n",
    "    #plt.imshow(train_datagen[i])\n",
    "#    # The CIFAR labels happen to be arrays, \n",
    "#    # which is why you need the extra index\n",
    "#    plt.xlabel(class_names[train_labels[i][0]])\n",
    "#plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "id": "58e8c2e4-866d-460b-a531-1485e2851c26",
   "metadata": {},
   "outputs": [],
   "source": [
    "Sequential = keras.models.Sequential\n",
    "Dense = keras.layers.Dense\n",
    "Conv2D = keras.layers.Conv2D\n",
    "MaxPooling2D = keras.layers.MaxPooling2D\n",
    "Flatten = keras.layers.Flatten\n",
    "load_model = keras.models.load_model\n",
    "AvgPooling2D = keras.layers.AveragePooling2D\n",
    "Softmax = keras.layers.Softmax\n",
    "Input = keras.layers.Input"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "87980ed5-8efb-40b7-851b-4173f6caafc5",
   "metadata": {},
   "source": [
    "### Convert into matrix-based"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "id": "c7d4f988-3d56-4b0c-971d-7c3c1beb0a32",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_images, test_images = np.array(train_images), np.array(test_images)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "id": "b3226cad-2b1c-41c7-9ffc-baeed9ff6c34",
   "metadata": {},
   "outputs": [],
   "source": [
    "channels = 1\n",
    "train_nuevos, test_nuevos = [], []\n",
    "for i in range(len(train_images)):\n",
    "    train_nuevos.append(train_images[i].reshape(28,28,channels))\n",
    "for i in range(len(test_images)):\n",
    "    test_nuevos.append(test_images[i].reshape(28,28,channels))\n",
    "train_nuevos, test_nuevos = np.array(train_nuevos), np.array(test_nuevos)\n",
    "train_images, test_images = None, None"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "882771a7-7abb-4183-9c94-f405e387ebe8",
   "metadata": {},
   "source": [
    "# Data augmentation"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "99466b1b-68fd-4568-8360-da4a2a4dd650",
   "metadata": {},
   "source": [
    "## Flipping\n",
    "\n",
    "Flipping a strangely-written $5$ could lead to a very convincing $2$, for example. Since flipping could potentially lead to increasing the noise in the data, we leave this out, but this is something that can also be done."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "id": "9b7ad35d-085d-4992-a2f7-aba667062231",
   "metadata": {},
   "outputs": [],
   "source": [
    "#flipper = keras.layers.RandomFlip()\n",
    "#train_flipped = flipper(train_nuevos)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "8ed94059-77c2-4f31-b864-5edab63b23cf",
   "metadata": {},
   "source": [
    "## Rotating"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "id": "1c955430-2666-42c1-8501-39109f9e7e07",
   "metadata": {},
   "outputs": [],
   "source": [
    "rotator = keras.layers.RandomRotation(\n",
    "    factor = (-0.1, 0.1),\n",
    "    fill_mode = 'nearest'\n",
    ")\n",
    "train_rotated = rotator(train_nuevos)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "f8101855-9fee-4a15-9b90-e4f3023afc98",
   "metadata": {},
   "source": [
    "## Random zoom"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "id": "32bf27c4-4e5d-49d6-a52d-335496d485d0",
   "metadata": {},
   "outputs": [],
   "source": [
    "zoomer = tf.keras.layers.RandomZoom(\n",
    "    -0.35, 0.35,\n",
    "    fill_mode = 'nearest'\n",
    ")\n",
    "train_zoomed = zoomer(train_nuevos)"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "434dc724-78c5-4a1c-8b41-54c6ea3055fd",
   "metadata": {},
   "source": [
    "## Check the transformations done"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "id": "b46b293b-db16-4cb2-8d79-9fe92fb9eb61",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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",
      "text/plain": [
       "<Figure size 1000x500 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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",
      "text/plain": [
       "<Figure size 1000x500 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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",
      "text/plain": [
       "<Figure size 1000x500 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "chosen = 4777\n",
    "plt.imshow(train_nuevos[chosen])\n",
    "plt.show()\n",
    "plt.imshow(train_rotated[chosen])\n",
    "plt.show()\n",
    "plt.imshow(train_zoomed[chosen])\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "ea0583c2-f295-48a1-ba88-d08d367deef3",
   "metadata": {},
   "source": [
    "## Merge the augmentations"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 32,
   "id": "4ecc95ec-fcb2-4407-a453-d649e3009437",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_nuevos = np.concatenate((train_nuevos, train_rotated, train_zoomed), axis = 0)\n",
    "#train_nuevos = np.concatenate((train_nuevos, train_rotated), axis = 0)\n",
    "train_rotated = None\n",
    "train_zoomed = None\n",
    "#train_flipped = None"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "98698be0-6af1-46d4-b515-b634e3d3f588",
   "metadata": {},
   "source": [
    "## Clone the labels\n",
    "\n",
    "To compensate for the new data."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 33,
   "id": "9e6ab5a2-c17a-4152-ae1c-15e5919c7f91",
   "metadata": {},
   "outputs": [],
   "source": [
    "train_labels_augmented = pd.Series(3*list(train_labels))"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "d1ad2c34-a987-4af3-ac69-84b8c9805b59",
   "metadata": {},
   "source": [
    "# Split"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "id": "a27984a8-cee6-417c-b986-84909e165bbf",
   "metadata": {},
   "outputs": [],
   "source": [
    "from sklearn.model_selection import train_test_split as TTS\n",
    "x_tr, x_te, y_tr, y_te = TTS(train_nuevos, train_labels_augmented, test_size = 0.25, random_state = None)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 35,
   "id": "3b623945-d3e9-4e3f-bbb4-b1280e22a2f1",
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "/home/adan/.local/lib/python3.12/site-packages/keras/src/layers/convolutional/base_conv.py:107: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.\n",
      "  super().__init__(activity_regularizer=activity_regularizer, **kwargs)\n"
     ]
    },
    {
     "data": {
      "text/html": [
       "<pre style=\"white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace\"><span style=\"font-weight: bold\">Model: \"sequential\"</span>\n",
       "</pre>\n"
      ],
      "text/plain": [
       "\u001b[1mModel: \"sequential\"\u001b[0m\n"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "text/html": [
       "<pre style=\"white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace\">┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓\n",
       "┃<span style=\"font-weight: bold\"> Layer (type)                    </span>┃<span style=\"font-weight: bold\"> Output Shape           </span>┃<span style=\"font-weight: bold\">       Param # </span>┃\n",
       "┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩\n",
       "│ conv2d (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Conv2D</span>)                 │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">26</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">26</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">256</span>)    │         <span style=\"color: #00af00; text-decoration-color: #00af00\">2,560</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ max_pooling2d (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">MaxPooling2D</span>)    │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">13</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">13</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">256</span>)    │             <span style=\"color: #00af00; text-decoration-color: #00af00\">0</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ conv2d_1 (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Conv2D</span>)               │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">11</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">11</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">256</span>)    │       <span style=\"color: #00af00; text-decoration-color: #00af00\">590,080</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ max_pooling2d_1 (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">MaxPooling2D</span>)  │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">5</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">5</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">256</span>)      │             <span style=\"color: #00af00; text-decoration-color: #00af00\">0</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ conv2d_2 (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Conv2D</span>)               │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">3</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">3</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">256</span>)      │       <span style=\"color: #00af00; text-decoration-color: #00af00\">590,080</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ flatten (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Flatten</span>)               │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">2304</span>)           │             <span style=\"color: #00af00; text-decoration-color: #00af00\">0</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Dense</span>)                   │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">128</span>)            │       <span style=\"color: #00af00; text-decoration-color: #00af00\">295,040</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense_1 (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Dense</span>)                 │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">96</span>)             │        <span style=\"color: #00af00; text-decoration-color: #00af00\">12,384</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense_2 (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Dense</span>)                 │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">10</span>)             │           <span style=\"color: #00af00; text-decoration-color: #00af00\">970</span> │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ softmax (<span style=\"color: #0087ff; text-decoration-color: #0087ff\">Softmax</span>)               │ (<span style=\"color: #00d7ff; text-decoration-color: #00d7ff\">None</span>, <span style=\"color: #00af00; text-decoration-color: #00af00\">10</span>)             │             <span style=\"color: #00af00; text-decoration-color: #00af00\">0</span> │\n",
       "└─────────────────────────────────┴────────────────────────┴───────────────┘\n",
       "</pre>\n"
      ],
      "text/plain": [
       "┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓\n",
       "┃\u001b[1m \u001b[0m\u001b[1mLayer (type)                   \u001b[0m\u001b[1m \u001b[0m┃\u001b[1m \u001b[0m\u001b[1mOutput Shape          \u001b[0m\u001b[1m \u001b[0m┃\u001b[1m \u001b[0m\u001b[1m      Param #\u001b[0m\u001b[1m \u001b[0m┃\n",
       "┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩\n",
       "│ conv2d (\u001b[38;5;33mConv2D\u001b[0m)                 │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m26\u001b[0m, \u001b[38;5;34m26\u001b[0m, \u001b[38;5;34m256\u001b[0m)    │         \u001b[38;5;34m2,560\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ max_pooling2d (\u001b[38;5;33mMaxPooling2D\u001b[0m)    │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m13\u001b[0m, \u001b[38;5;34m13\u001b[0m, \u001b[38;5;34m256\u001b[0m)    │             \u001b[38;5;34m0\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ conv2d_1 (\u001b[38;5;33mConv2D\u001b[0m)               │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m11\u001b[0m, \u001b[38;5;34m11\u001b[0m, \u001b[38;5;34m256\u001b[0m)    │       \u001b[38;5;34m590,080\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ max_pooling2d_1 (\u001b[38;5;33mMaxPooling2D\u001b[0m)  │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m5\u001b[0m, \u001b[38;5;34m5\u001b[0m, \u001b[38;5;34m256\u001b[0m)      │             \u001b[38;5;34m0\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ conv2d_2 (\u001b[38;5;33mConv2D\u001b[0m)               │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m3\u001b[0m, \u001b[38;5;34m3\u001b[0m, \u001b[38;5;34m256\u001b[0m)      │       \u001b[38;5;34m590,080\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ flatten (\u001b[38;5;33mFlatten\u001b[0m)               │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m2304\u001b[0m)           │             \u001b[38;5;34m0\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense (\u001b[38;5;33mDense\u001b[0m)                   │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m128\u001b[0m)            │       \u001b[38;5;34m295,040\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense_1 (\u001b[38;5;33mDense\u001b[0m)                 │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m96\u001b[0m)             │        \u001b[38;5;34m12,384\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ dense_2 (\u001b[38;5;33mDense\u001b[0m)                 │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m10\u001b[0m)             │           \u001b[38;5;34m970\u001b[0m │\n",
       "├─────────────────────────────────┼────────────────────────┼───────────────┤\n",
       "│ softmax (\u001b[38;5;33mSoftmax\u001b[0m)               │ (\u001b[38;5;45mNone\u001b[0m, \u001b[38;5;34m10\u001b[0m)             │             \u001b[38;5;34m0\u001b[0m │\n",
       "└─────────────────────────────────┴────────────────────────┴───────────────┘\n"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "text/html": [
       "<pre style=\"white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace\"><span style=\"font-weight: bold\"> Total params: </span><span style=\"color: #00af00; text-decoration-color: #00af00\">1,491,114</span> (5.69 MB)\n",
       "</pre>\n"
      ],
      "text/plain": [
       "\u001b[1m Total params: \u001b[0m\u001b[38;5;34m1,491,114\u001b[0m (5.69 MB)\n"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "text/html": [
       "<pre style=\"white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace\"><span style=\"font-weight: bold\"> Trainable params: </span><span style=\"color: #00af00; text-decoration-color: #00af00\">1,491,114</span> (5.69 MB)\n",
       "</pre>\n"
      ],
      "text/plain": [
       "\u001b[1m Trainable params: \u001b[0m\u001b[38;5;34m1,491,114\u001b[0m (5.69 MB)\n"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "text/html": [
       "<pre style=\"white-space:pre;overflow-x:auto;line-height:normal;font-family:Menlo,'DejaVu Sans Mono',consolas,'Courier New',monospace\"><span style=\"font-weight: bold\"> Non-trainable params: </span><span style=\"color: #00af00; text-decoration-color: #00af00\">0</span> (0.00 B)\n",
       "</pre>\n"
      ],
      "text/plain": [
       "\u001b[1m Non-trainable params: \u001b[0m\u001b[38;5;34m0\u001b[0m (0.00 B)\n"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "model = Sequential([\n",
    "    Conv2D(256, (3,3), activation = 'gelu', input_shape = (28, 28, 1)),\n",
    "    MaxPooling2D(2,2),\n",
    "    Conv2D(256, (3,3), activation = 'gelu'),\n",
    "    MaxPooling2D(2,2),\n",
    "    Conv2D(256, (3,3), activation = 'gelu'),\n",
    "    Flatten(),\n",
    "    Dense(128, activation = 'gelu'),\n",
    "    Dense(96, activation = 'gelu'),\n",
    "    Dense(10, activation = 'linear'),\n",
    "    Softmax()\n",
    "])\n",
    "model.summary()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "id": "edd9aae9-6660-4b0c-bd1b-f5f98bd90f52",
   "metadata": {},
   "outputs": [],
   "source": [
    "model.compile(\n",
    "    optimizer='adam',\n",
    "    #optimizer='adamw',\n",
    "    #loss=tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True), # False si la última capa es Softmax\n",
    "    #loss=tf.keras.losses.SparseCategoricalCrossentropy(),\n",
    "    loss = 'sparse_categorical_crossentropy',\n",
    "    #optimizer = 'rmsprop',\n",
    "    #loss = 'binary_crossentropy', ## PARA CUANDO SOLO HAYA 2 CLASES!!!!\n",
    "    metrics=['accuracy']\n",
    ")"
   ]
  },
  {
   "cell_type": "markdown",
   "id": "18934a9d-40a3-42f8-a53c-1b5e7bd7b9e5",
   "metadata": {},
   "source": [
    "El método ```fit``` acepta ```generators``` aparte de las meras matrices de datos y las etiquetas. Esto permite entrenar un modelo sin tener que cargar todos los datos en memoria porque los ```generators``` devuelven solo una parte de los datos por paso.\n",
    "\n",
    "Los ```generators``` contienen no solo los datos sino también las etiquetas. De no utilizarlos, hay que especificar también las etiquetas."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 37,
   "id": "fa7c8c6c-e8dd-4f22-8eab-6d22edcc0ed5",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Epoch 1/3\n",
      "\u001b[1m2954/2954\u001b[0m \u001b[32m━━━━━━━━━━━━━━━━━━━━\u001b[0m\u001b[37m\u001b[0m \u001b[1m1637s\u001b[0m 553ms/step - accuracy: 0.9175 - loss: 0.2574 - val_accuracy: 0.9817 - val_loss: 0.0614\n",
      "Epoch 2/3\n",
      "\u001b[1m2954/2954\u001b[0m \u001b[32m━━━━━━━━━━━━━━━━━━━━\u001b[0m\u001b[37m\u001b[0m \u001b[1m1649s\u001b[0m 558ms/step - accuracy: 0.9834 - loss: 0.0559 - val_accuracy: 0.9824 - val_loss: 0.0653\n",
      "Epoch 3/3\n",
      "\u001b[1m2954/2954\u001b[0m \u001b[32m━━━━━━━━━━━━━━━━━━━━\u001b[0m\u001b[37m\u001b[0m \u001b[1m1625s\u001b[0m 550ms/step - accuracy: 0.9868 - loss: 0.0430 - val_accuracy: 0.9864 - val_loss: 0.0500\n"
     ]
    }
   ],
   "source": [
    "#nb_train_samples = len(train_generator.classes)\n",
    "#nb_validation_samples = len(validation_generator.classes)\n",
    "\n",
    "history = model.fit(\n",
    "    x_tr, y_tr,\n",
    "    validation_data = (x_te, y_te),\n",
    "    callbacks = [\n",
    "        keras.callbacks.EarlyStopping(monitor = 'val_loss', patience = 2) # Stop learning if val_loss doesn't change in 2 epochs\n",
    "    ],\n",
    "    #train_generator,\n",
    "    #validation_data = validation_generator,\n",
    "    #steps_per_epoch = 1000,\n",
    "    #validation_steps = 400,\n",
    "    epochs = 3\n",
    ")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 38,
   "id": "3e9367e6-8053-4870-aa77-09cbcdc6e4ff",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAAA2AAAAHACAYAAADEEQtjAAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjguNCwgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy8fJSN1AAAACXBIWXMAAA9hAAAPYQGoP6dpAABfaklEQVR4nO3deXhV1dn38W8gBAWDQxFBFMSCRmydEKooWqvUV2vFEVBKKSIq1KFqq0K12mLBoYXWqdWq4FSsEyooikWsylRQnwoiIjKoAdHIkEAGEljvH+ckJOQEckKSneH7ua77krXOPvvc2U96Hn7svddOAQKSJEmSpBrXJOoGJEmSJKmxMIBJkiRJUi0xgEmSJElSLTGASZIkSVItMYBJkiRJUi0xgEmSJElSLTGASZIkSVItMYBJkiRJUi1JjbqB+mz//fcnJycn6jYkSZIkRSw9PZ1Vq1btdDsDWBXtv//+ZGZmRt2GJEmSpDqiffv2Ow1hBrAqKj7z1b59e8+CSZIkSY1Yeno6mZmZlcoFBrBdlJOTYwCTJEmSVCkuwiFJkiRJtcQAJkmSJEm1xAAmSZIkSbXEe8BqUEpKCnvttRfp6emkpKRE3Y7qoBACOTk5rF+/nhBC1O1IkiSphhnAasi+++7L0KFDycjIiLoV1QOLFy/mH//4B998803UrUiSJKkGpQD+s3sVpKenk52dTatWrcqtgpiamsoDDzzAxo0beeaZZ/j666/ZsmVLRJ2qLmvatClt2rShb9++7LHHHgwfPpyioqKo25IkSVISdpQNtucZsBrQrl07dtttN/70pz+xZMmSqNtRHbds2TLWrl3LzTffTNu2bfnyyy+jbkmSJEk1xEU4akCTJrHDWlBQEHEnqi+Kf1eaNm0acSeSJEmqSQYwSZIkSaolBjBJkiRJqiUGMEmSJEmqJQYw1Wmpqa4TI0mSpIbDAKYyTj/9dN555x3WrVtHVlYWkydP5uCDDy55vX379kycOJFvv/2WjRs3Mm/ePHr06FHy+k9/+lPmzZtHXl4e33zzDc8//3zJayEE+vTpU+bz1q1bx6BBgwDo2LEjIQQuvPBCZsyYQV5eHj/72c/YZ599+Oc//8kXX3zBpk2b+PDDD+nfv3+Z/aSkpHDDDTfw6aefkp+fz8qVKxk5ciQA06dP59577y2z/T777EN+fj6nnHJK9Rw4SZIk1aIUoAPQG2gRcS/JMYCpjJYtWzJ27Fi6d+/OqaeeytatW5k0aRIpKSm0bNmS//znP+y///6cffbZHHnkkdx1110lqz6eeeaZvPDCC7zyyiscffTRnHrqqcyfPz/pHu68807uueceDjvsMF5//XV222033nvvPc466yy+973v8dBDD/HEE0+UCX5jxozhxhtvZNSoUXTt2pWLL76YNWvWAPDwww9z8cUXk5aWVrL9gAEDWLVqFTNmzNjFIyZJkqSaswdwDHAR8HvgaeADYCOwEpgGfD+y7qoqWMlXenp6CCGE9PT0cq917NgxPP7446Fjx47bvTYvwBe1XPN26eds3bp1CCGEww8/PAwdOjRs2LAh7L333gm3nTlzZnjiiScq3FcIIfTp06fM3Lp168KgQYNKjlsIIVx99dU77WvKlCnh7rvvDkDYY489Ql5eXhgyZEjCbdPS0kJWVla48MILS+bef//98Lvf/S7y36Od/85YlmVZlmU19EoJ0DHA6QGuDnB/gH8H+DJAqET9PPKfYUfZYPvyBpta1RY4IOomdujggw9m1KhRHHfccbRu3brk7FaHDh046qij+OCDD1i3bl3C9x511FH84x//2OUetj9r1qRJE2666Sb69etH+/btad68Oc2bN2fTpk0AHHbYYey2225Mnz494f42b97Mk08+ySWXXMKzzz7LkUceyZFHHsk555yzy71KkiSpstKBQ0tVRvy/XYDdk9hPIfAZ8AmwGPioetusYQawWvVVnf/MyZMn88UXXzB06FBWrVpFkyZN+Oijj0hLSyMvL2+H793Z61u3biUlJaXMXLNmzcptVxysil1//fVce+21/OpXv2LBggVs2rSJv/zlLyWXFO7scyF2GeL//d//0b59ey655BKmT5/O559/vtP3SZIkKRlNgI6UD1mHAvsnua9viAWsT0rVYmA5UFRN/dY+A1it6h51Azu0zz770LVrVy6//HLeffddAE444YSS1z/88EMuvfRS9t5774RnwT788ENOPfVUJkyYkHD/33zzDe3atSsZd+7cmZYtW+60r169evHSSy/x1FNPAbEFN7p06cLHH38MwKeffkpubi6nnnoqjzzySMJ9LFy4kPnz5zN06FAuvvhirrrqqp1+riRJkirSivIBq/hs1m5J7GczsJSyAav4z4mvuqrvDGAqUbzy4WWXXcbq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",
      "text/plain": [
       "<Figure size 1000x500 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    },
    {
     "data": {
      "image/png": "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",
      "text/plain": [
       "<Figure size 1000x500 with 1 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "plt.plot(history.history['accuracy'], label='accuracy', linewidth = 2, color = 'blue')\n",
    "plt.plot(history.history['val_accuracy'], label = 'val_accuracy', linewidth = 2, color = 'green')\n",
    "plt.xlabel('Epoch')\n",
    "plt.ylabel('Accuracy')\n",
    "plt.legend(loc='best')\n",
    "plt.show()\n",
    "\n",
    "plt.plot(history.history['loss'], label='loss', linewidth=2, color = 'red')\n",
    "plt.plot(history.history['val_loss'], label='val_loss', linewidth=2, color = 'orange')\n",
    "plt.xlabel('Epoch')\n",
    "plt.ylabel('Loss')\n",
    "plt.legend(loc = 'best')\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "id": "88884070-f3e5-4134-9771-cb4cf0146cf1",
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "\u001b[1m875/875\u001b[0m \u001b[32m━━━━━━━━━━━━━━━━━━━━\u001b[0m\u001b[37m\u001b[0m \u001b[1m121s\u001b[0m 139ms/step\n",
      "985/985 - 132s - 134ms/step - accuracy: 0.9864 - loss: 0.0500\n"
     ]
    }
   ],
   "source": [
    "#test_loss, test_acc = model.evaluate(validation_generator, verbose=2)\n",
    "predicciones = model.predict(test_nuevos)\n",
    "test_loss, test_acc = model.evaluate(x_te, y_te, verbose = 2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 40,
   "id": "14b5d99a-86dc-4973-925f-e945046fe0bd",
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0.986380934715271"
      ]
     },
     "execution_count": 40,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "test_acc"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 52,
   "id": "ded3b061-8b51-41f9-8695-e13febd7ab45",
   "metadata": {},
   "outputs": [],
   "source": [
    "model.save('Augmented_Train.keras')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "id": "7e14dbe2-3fb9-4ce5-a0dc-3f6865f7a148",
   "metadata": {},
   "outputs": [],
   "source": [
    "lista = list(set(y_te))\n",
    "clases_predecidas = []\n",
    "for i in range(len(predicciones)):\n",
    "    clases_predecidas.append(lista[list(predicciones[i]).index(max(list(predicciones[i])))])"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "id": "d055e72f-6ada-44f7-a2c3-04cd24bd52e8",
   "metadata": {},
   "outputs": [],
   "source": [
    "template['Label'] = clases_predecidas"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 45,
   "id": "c2b0dc3b-4023-4d88-becd-d935e034226f",
   "metadata": {},
   "outputs": [],
   "source": [
    "pd.DataFrame(template).to_csv('Results_submission.csv', index = False)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "id": "05919dc8-a895-4059-a87b-3eacff36137c",
   "metadata": {
    "scrolled": true
   },
   "outputs": [],
   "source": [
    "#for i in range(len(test_labels)):\n",
    "#    print (class_names[test_labels[i][0]], class_names[list(predicciones[i]).index(max(predicciones[i]))])"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.12.5"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 5
}