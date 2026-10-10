# Object Detection Notes

- https://viso.ai/deep-learning/object-detection

## How does object detection work?

Object detectors are generally categorized into 2 types:

1. Single-stage object detectors.
2. Two-stage object detectors.

### One-Stage vs. Two-Stage Deep Learning Object Detection

In general, deep learning-based object detectors extract features from the input image or video frame. An object detector solves two subsequent tasks:

**Task #1**: Find an arbitrary number of objects (possibly even zero)

**Task #2**: Classify every single object and estimate its size with a bounding box.

-----

**Single-Stage Detectors or One-stage Detectors**

To simplify the process, you can separate those tasks into two stages. Other methods combine both tasks into one step **_(single-stage detectors) to achieve higher performance at the cost of accuracy_**.

One-stage detectors predict bounding boxes over the images without the region proposal step. This process consumes less time and can therefore be used in real-time applications.

One-stage object detectors prioritize inference speed and are super fast but not as good at recognizing irregularly shaped objects or a group of small objects.

The most popular one-stage detectors include the YOLO, SSD, and RetinaNet. The latest real-time detectors are YOLOv7 (2022), YOLOR (2021), and YOLOv4-Scaled (2020). View the benchmark comparisons below.
The main advantages of object detection with single-stage algorithms include a generally faster detection speed and greater structural simplicity and efficiency compared to multi-stage detectors.

-----

**Two-stage Detectors**

In two-stage object detectors, the approximate object regions are proposed using deep features before these features are used for the image classification and bounding box regression for the object candidate.

The two-stage architecture involves (1) object region proposal with conventional Computer Vision methods or deep networks, followed by (2) object classification based on features extracted from the proposed region with bounding-box regression.

**_Two-stage methods achieve the highest detection accuracy but are typically slower_**. Because of the many inference steps per image, the performance (frames per second) is not as good as one-stage detectors.

Various two-stage detectors include region convolutional neural network (RCNN), with evolutions Faster R-CNN or Mask R-CNN. The latest evolution is the granulated RCNN (G-RCNN).

Two-stage object detectors first find a region of interest and use this cropped region for classification. However, early multi-stage detectors (like the original R-CNN) were not end-to-end trainable because cropping is a non-differentiable operation (Faster R-CNN later made the whole network trainable end-to-end by sharing features and using RoI pooling).


## Object detection model architecture

### I. R-CNN Model Family

_Region-based Convolutional Neural Networks_

1. **R-CNN** - This utilizes a selective search method to locate **RoIs** in the input images and uses a **DCN** (Deep Convolutional Neural Network)-based region wise classifier to classify the **RoIs** (Region of Interests) independently.
2. **SPPNet** and **Fast R-CNN** - This is an improved version of **R-CNN** that deals with the extraction of the **RoIs** from the feature maps. This was found to be much faster than the conventional **R-CNN** architecture.
3. **Faster R-CNN** - This is an improved version of **Fast R-CNN** that was trained end to end by introducing **RPN** (region proposal network). 
    - An **RPN** is a network utilized in generating **RoIs** by regressing the anchor boxes. Hence, the anchor boxes are then used in the object detection task.
4. **Mask R-CNN** adds a mask prediction branch on the **Faster R-CNN**, which can detect objects and predict their masks at the same time.
5. **R-FCN**  replaces the fully connected layers with the position-sensitive score maps for better detecting objects.
6. **Cascade R-CNN** addresses the problem of overfitting at training and quality mismatch at inference by training a sequence of detectors with increasing **IoU** (Intersection over Union) thresholds.

### II. YOLO Model Family

1. **YOLO** (v1) does not use anchor boxes at all: it divides the input image into an S × S grid and each grid cell directly regresses a few bounding boxes and predicts class probabilities. This was built using the Darknet framework.
2. **YOLOv2** improves the performance by introducing anchor boxes (chosen with k-means clustering) and a new bounding box regression method.
3. **YOLOv3** is an enhanced version of the v2 variant with a deeper feature detector network and minor representational changes. **YOLOv3** has relatively speedy inference times with it taking roughly 30ms per inference.
4. **YOLOv4** (**YOLOv3** upgrade) works by breaking the object detection task into two pieces, regression to identify object positioning via bounding boxes and classification to determine the object's class. **YOLO V4** and its successors are technically the product of a different set of researchers than versions 1-3.
5. **YOLOv5** (by Ultralytics, implemented in PyTorch instead of Darknet) is an improved/re-engineered version of the **YOLOv4** ideas (it also uses the mosaic augmentation technique, which was introduced in **YOLOv4**), focused on ease of use and deployment.

### III. SSD Family and Keypoint-Based Detectors (CornerNet / CenterNet)

1. **SSD (Single Shot Multibox Detector)** places anchor boxes densely over an input image and uses features from different convolutional layers to regress and classify the anchor boxes.
2. **DSSD (Deconvolutional Single Shot Detector )** introduces a deconvolution module into **SSD** to combine low level and high-level features. While **R-SSD** uses pooling and deconvolution operations in different feature layers to combine low-level and high-level features.
3. **RON (Reverse Connection With Objectness Prior Networks for Object Detection)** proposes a reverse connection and an objectness prior to extracting multiscale features effectively.
4. **RefineDet (Single-Shot Refinement Neural Network for Object Detection)** refines the locations and sizes of the anchor boxes for two times, which inherits the merits of both one-stage and two-stage approaches.
5. **CornerNet** is another keypoint-based approach, which directly detects an object using a pair of corners. Although CornerNet achieves high performance, it still has more room to improve.
6. **CenterNet** - there are actually two different 2019 papers with this name: **"CenterNet: Keypoint Triplets"** explores the visual patterns within each bounding box and detects an object using a triplet (two corners + center), rather than a pair, of keypoints; while **"Objects as Points"** treats objects as single points by predicting the x and y coordinate of the object’s center and its size (width and height). Both reported better speed/accuracy trade-offs than many SSD and R-CNN variants at the time.

### IV. Others

 - RetinaNet
 - Deformable convolutional networks
 - DETR (Detection Transformer)

=========================================

## The Best Image Detection Algorithm Today

The field of object detection is not as new as it may seem. In fact, object detection has evolved over the past 20 years. The progress of object detection is usually separated into two separate historical periods (before and after the introduction of Deep Learning):

### Object Detector Before 2014 – Traditional Object Detection period

1. Viola-Jones Detector (2001), the pioneering work that started the development of traditional object detection methods
2. HOG Detector (2006), a popular feature descriptor for object detection in computer vision and image processing
3. DPM (2008) with the first introduction of bounding box regression

### Object Detector After 2014 – Deep Learning Detection period

_The most important two-stage object detection algorithms_

1. RCNN and SPPNet (2014)
2. Fast RCNN and Faster RCNN (2015)
3. Mask R-CNN (2017)
4. Pyramid Networks/FPN (2017)
5. G-RCNN (2021)

_The most important one-stage object detection algorithms_

1. YOLO (2016)
2. SSD (2016)
3. RetinaNet (2017)
4. YOLOv3 (2018)
5. YOLOv4 (2020)
6. YOLOR (2021)
7. YOLOv7 (2022)
8. YOLOv8 (2023)
9. YOLOv9 (2024)

The creators of the original YOLO algorithms did not release YOLOv8 (it was released by Ultralytics). It’s important to note that it was published under an AGPL-3.0 License, a strong copyleft license: commercial use is allowed, but if you distribute it, or let users interact with it over a network, you must release your own source code under the same license (Ultralytics sells a separate commercial license to avoid this).
 
-----

## The Best Real-Time Object Detection Algorithm (Accuracy) ('20 - '22)

On the MS COCO dataset and based on the Average Precision (AP), the best real-time object detection algorithm is YOLOv7, followed by Vision Transformer (ViT) such as Swin and DualSwin, PP-YOLOE, YOLOR, YOLOv4, and EfficientDet.
 
## The Fastest Real-Time Object Detection Algorithm (Inference Time) ('20 - '22)

Also, on the MS COCO dataset, an important benchmark metric is inference time (ms/Frame, lower is better) or Frames per Second (FPS, higher is better).  The rapid advances in computer vision technology are very visible when looking at inference time comparisons.

Based on current inference times (lower is better), YOLOv7 achieves 3.5ms per frame, compared to YOLOv4 12ms, or the popular YOLOv3 29ms. Note how the introduction of YOLO (one-stage detector) led to dramatically faster inference times compared to any previously established methods, such as the two-stage method Mask R-CNN (333ms).

On a technical level, it is pretty complex to compare different architectures and model versions in a meaningful way. Edge AI is becoming an integral part of scalable AI solutions, and newer models come with lighter-weight edge-optimized versions (see YOLOv7-lite or TensorFlow Lite).

In comparison of the latest YOLO versions – YOLOv8 vs. YOLOv7 and YOLOv6 – the latest release (YOLOv8) shows the best performance in real-time benchmarks published by the creator.

## Data Sets
 
- COCO dataset (Common Objects in Context, released in 2014) - an image dataset whose object detection task uses 80 object categories (out of 91 originally defined) (cars, persons, sport balls, bicycles, dogs, cats, horses etc.).
    
    The dataset was gathered to solve common object detection problems. Nowadays some consider it to be becoming outdated, since its images (collected from Flickr) are relatively small and low resolution compared to today’s images.
    
- Newer datasets like OpenImages are taking its spot as the de-facto pre-training dataset.
 
## Concepts That Are New To My Ears

- RoI (Region of Interest) extraction process

- Zero-Shot and Few-Shot Learning
 
## Object detection vs image segmentation
 
- "**Image Classification**" assigns a single label (class) to the whole image, without saying where the object is.
 
- "**Image segmentation**" is the process of defining which pixels of an object class are found in an image.

- "**Semantic image segmentation**" will mark all pixels belonging to that tag (class), but won’t separate the individual objects (two touching cars are just one "car" region).

- "**Object Detection**" instead will not segment the object, but will clearly define the location of each individual object instance with a box.

- "**Instance Segmentation**" - Combining semantic segmentation with object detection, which first detects the object instances, and then segments each within the detected boxes (known in this case as regions of interest).
