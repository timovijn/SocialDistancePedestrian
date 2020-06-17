# Social Distance Monitor

![Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/Screenshot_2020-06-15_at_11.37.02.png](Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/Screenshot_2020-06-15_at_11.37.02.png)

---

---

# Introduction

**Problem to solve • Relevant example • Extensibility to other domains • Summary of result • Clear hypothesis and problem statement • Requirements**

With the rise of the COVID-19 pandemic, society calls for innovative solutions to solve the emerging problems and aid humanity in dealing with the new life standards and regulations. One of these regulations is that of social distancing, where civilians are asked to not get any closer to each other than a by government predefined distance. As the goal is to reduce the total amount of COVID-19 contaminations, the need for an application that can show where and how often the regulations are violated is high. This blog post comprises of a tool, the Social Distance Monitor (SDM), that evaluates recordings of surveillance cameras and then presents the total amount of violations and the locations where the majority of these violations occur. This way, authorities can adjust the spatial planning accordingly, with the goal to reduce the chance that COVID-19 is spread within their region. For this application to be useful, it should adhere to the following requirements:

- The runtime of the algorithm on a single frame should be lower than 0.5 sec.
- The accuracy of distance detection should be higher than 90%.
- It should be robust in the sense that the program handles a variety of crowds, perspectives, resolutions, lighting conditions and environmental differences. To measure this, 10 clips containing these different conditions are carefully selected as presented in the method section.
- The operating time should be limited to a 1 time setup with a duration less than 10 minutes.
- The tool should be user-friendly in the sense that it easy to operate and also outputs a variety of data.

This tool was developed during times of the COVID-19 epidemic, however its use is much more versatile than crime prevention and law enforcement only. For example, one can use it to adjust infrastructure or urban planning based on pedestrian flows and measure the effect of events on specific parts of their domain.

# Related Work

**Other implementations • Underlying principles • YOLO**

To go from idea to a working tool, inspiration was taken from existing variants and widely used techniques were used to execute specific phases in the program such as the YOLO neural network to detect pedestrians. These other implementations and underlying principles will be set out below. 

### Other Implementations

- Landing AI, a famous start-up in the field of Artificial Intelligence, built a [social distancing detector](https://landing.ai/landing-ai-creates-an-ai-tool-to-help-customers-monitor-social-distancing-in-the-workplace/). The detector works roughly the same as the one presented in this blog post, as they use the video surveillance input, detect pedestrians and evaluate the social distancing violations from a bird's-eye perspective. Where the algorithms differ is in the sense of object detection. Landing AI uses Faster R-CNN, where our program uses YOLOv3, which will be explained later. Landing AI's tool does only show the violations as opposed by our tool, where the locations of occurrence are also given as output. This is where our program provides novel insights.
- Aqeel Anwar also came up with a [likewise tool](https://towardsdatascience.com/monitoring-social-distancing-using-ai-c5b81da44c9f), again using the bird's eye perspective to measure the violations of social distancing. His implementation of transforming the perspective to bird's eye view was used as basis for the tool in this post. A limitation in his work is the way the distance between identified persons is calculated. This problem is addressed in the method section.
- Lastly, a [social-distancing-analyser](https://github.com/Ank-Cha/Social-Distancing-Analyser-COVID-19) by Ankush Chaudhari was used as inspiration for our algorithm. The way the YOLO neural net in implemented is equal to that of this blog post. This implementation does not make use of bird's eye view and thus makes up for a interesting comparison to the other implementations including that proposed this post.

### Underlying Principles

- To calculate the distance between two people, the program first need to know where the persons are located. To classify humans from video, a trained neural network called [You Only Look Once (YOLOv3)](https://arxiv.org/pdf/1506.02640v5.pdf) is widely used in industry. YOLO is a real-time object detection system. The single neural network it consists of looks at the image once, divides it into regions, predicts bounding boxes and probabilities for each region. Finally, the bounding boxes are weighted by the predicted probabilities. The pretrained weights available are trained using the [COCO-dataset](http://cocodataset.org/#home), which consists of images of common objects (including persons/pedestrians).
- As the problem introduced lies within the field of computer vision, this project makes use of [OpenCV](https://opencv.org/). The cv2 module is an renowned image and video processing library providing many capabilities. In this project, mainly the deep neural network modules, as well as the video processing tools were used.

# Methods

**Algorithm walkthrough • Motivation for current implementation with reference to problem statement • Training YOLO for person detection**

## Social Distance Monitor

The algorithm of the Social Distance Monitor consists of a number of constituent parts (phases) that will be explained separately in an input-to-output manner.

The overarching goal of the algorithm is to provide the user with insightful indicators indicating the amount and location of social distance violations for an input of pedestrian video footage. On the highest level, the input-to-output relation can thus be visualised as follows.

*Image that shows input video footage and output indicators and in between the “algorithm”.*

When we descend into a more detailed level, the algorithm’s phases shown in the table below can be considered separately. The algorithm progresses through these phases for every frame of a video separately and linearly.

[Untitled](https://www.notion.so/5f97ba769d9b47269c82fdce862f66bb)

![Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/Block_diagram_phases.png](Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/Block_diagram_phases.png)

Block diagram indicating the individual phases of the Social Distance Monitor

[Using Python to Monitor Social Distancing in a Public Area](https://towardsdatascience.com/monitoring-social-distancing-using-ai-c5b81da44c9f)

![Phases](/Blog/Images/Phases.eps | width=100)

![MovingOutput](https://github.com/timovijn/SocialDistanceMonitor/blob/master/Blog/Images/MovingOutput.gif | width=50)
![Phases](https://github.com/timovijn/SocialDistanceMonitor/blob/master/Blog/Images/Phases.eps?raw=true)



### (Phase 1) **Video processing**

To obtain (output) a processed video frame from (input) raw video footage, the algorithm contains phase (1) Video processing.

This step in the algorithm is a straightforward one which starts by loading a file of any video format for the timestamps set by the user, and loading a single frame from this video. Every input video can have its own dimensions. Output is standardised such that frames are always shown in the same horizontal dimension. Furthermore, a frame is always resized to 416 x 416 resolution before continuing to the next step.  Maybe we can implement further video processing such as contrast/colour etc.

Motivated by our requirement of user-friendliness, the algorithm output for this step provides the user with information about the selected video and clip.

```python
...

Started at 12:56:50

...

Checkpoint Initialisation

...

Path: ./Videos/TownCentreXVID.avi
Width: 1920 px
Height: 1080 px
Framerate: 25.0 fps
Duration: 300.0 s
Frames: 7500

...

New frame 0 (0 → 250) (1 of 251) (0%)

...
```

### (Phase 2) **Object detection**

To obtain (output) person locations in 3D from (input) a processed video frame, the algorithm contains phase (2) Object detection.

As stated before, object detection is implemented through the YOLO object detection system. The three required files are stored locally to allow configuration before being loaded for the algorithm to use. The processed frame from one phase earlier is further processed by `blob = cv2.dnn.blobFromImage()`. In our case, this function is used to create a 4-dimensional blob from the frame with swapped Blue and Red channels.

The artificial network is initialised by `net = cv2.dnn.readNetFromDarknet()` which takes the weight file and configuration file as input. The input to this network is then set by `net.setInput()` and layer outputs are obtained by `net.forward()`. This output is used to determine bounding boxes of the objects that have been detected to be of class ‘person’ with a confidence value of at least a user set value.

### (Phase 3) Perspective change

To obtain (output) person locations in 2D from (input) person locations in 3D, the algorithm contains phase (3) Perspective change.

The phase to change perspective is to obtain a top view (bird’s eye view) of the scene. This phase is motivated by the fact that – given that this view is a realistic representation of reality – social distance violations can be determined easily by circles. Furthermore, some users may find this view to be more insightful, and it may help to create additional visualisations at a later stage.

At the foundation of this phase is `cv2.perspectiveTransform()`, a function that requires two equal-sized source and destination arrays to determine the transformation matrix. The source or “to-be-transformed’ array is obtained manually. At the start of the algorithm, there is a popup that directs the user to mark four points that indicate the perspective of the scene. These points are then used as source array. After obtaining the transformation matrix, it is straightforward to map the centre point of the bounding box of a person from the original 3D view to a “warped” centre point on the 2D bird’s eye view.

![Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/SocialDistancePedestrian.gif](Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/SocialDistancePedestrian.gif)

### (Phase 4) Violation detection

To obtain (output) violations from (input) person locations in 2D, the algorithm contains phase (4) Violation detection.

Because of the bird’s eye view from the previous phase, the phase of violation detection contains only basis mathematical operations. A relationship matrix is computed that contains the distance from every point to all other points. All relationships in the upper triangular part of this symmetric matrix that do not exceed the given social distance are marked as being a violation and a line is drawn between the two people that are in violation on both 2D and 3D views.

### (Phase 5) Indicators

To obtain (output) indicators from (input) violations, the algorithm contains phase (5) Indicators.

One of the most informative visualisations is that of the Violation Heatmap, which exactly fits the original scene. To establish this heatmap, the centre-point of a violation – that is, the point that is exactly in between two people that are in violation – is recorded as a violation.

![Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/heatmap.png](Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/heatmap.png)

![Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/combined.png](Social%20Distance%20Monitor%207d35bd1776c6429bb1f55d258b90a908/combined.png)

## Training YOLO for person detection

(...)

# Experiments & Data

**What type of data • Data source • How much data • Preprocessing steps • Explicit experiments (Experiment description → Hypothesis → Result) / Performance / Robustness • Ablation study • Shortcomings**

## Data

### Input Data

In order to build the social distance monitor a variety of data was used. First of all, the input of the program is a video (mostly from surveillance cameras). For these videos as input data, footage from a variety of video datasets was used. The goal was to select a range of videos with differing characteristics such as lighting, amount of pedestrians, perspectives and surrounding objects in the environment. The selection of these videos was done manually with careful consideration. Footage from the following datasets was used: [Oxford Town Centre](https://megapixels.cc/oxford_town_centre/), [CAVIAR](http://homepages.inf.ed.ac.uk/rbf/CAVIARDATA1/), [Virat Video Dataset](https://viratdata.org/), [EPFL](https://www.epfl.ch/labs/cvlab/data/data-pom-index-php/) and an additional test video was provided by [BriefCam](https://www.youtube.com/watch?v=aUdKzb4LGJI). 

### Data for object detection

As this is the data used as input of the program, it is not used in building the tool. For the different phases, different datasets were used to be able to get an accurate and working tool. For person recognition, this can be split up in two parts, being the data used for the pre-trained weights of the YOLO network and the data used for the self-trained YOLO network. 

The **COCO-dataset** was used to train and craft the original YOLO network

For the custom self-trained variant of the YOLO network, the **Caltech dataset** was introduced

## Performance and robustness experiments

A number experiments that give an indication of the performance and robustness of the Social Distance Monitor are to vary the type of input video footage. All experiments are shortly addressed below in terms of description, hypothesis, and result.

### (Experiment a) Lower resolution

Downscale a video to a lower resolution. Run SDM for the original video and downscaled video for the exact same frames.

The hypothesis for this experiment is that the person detection will suffer somewhat from downscaling the video, and more with decreased resolution. Other aspects of the SDM have no further dependency on the resolution of the clip. Social distance identification is thus expected to have decreased accuracy due to incorrect recognition of persons.

(...)

### (Experiment b) Different height perspective

Record two videos of same scene, from different height perspectives. Run SDM for both videos.

The hypothesis for this experiment is that the person detection will be exactly the same for both clips, as long as both clips are recorded at a reasonable camera height. The perspective transformation is expected to be of a lower accuracy for the clip that has been recorded at a lower height. Therefore, the social distance identification is expected to have decreased accuracy due to reduced accuracy in the phase of perspective transformation.

(...)

### (Experiment c) People farther away

In same scene, identify two sets of people that are more or less at the same distance.

It is expected that the set of people that are more in the back of the scene will be closer together in the bird’s eye view due to inaccuracies in the method of perspective transformation.

(...)

## Shortcomings

As demonstrated in the experiments, the SDM has shortcomings with regard to perspective. The transformation from a 3D scene to a 2D scene introduces inaccuracies that ripple on in the social distance identification. (... Discuss how and why these inaccuracies arise ...)

# Conclusion

**Summary • Learning curve • Future work**

## Summary

(...)

## Future work

Based upon our findings during the development of the Social Distance Monitor, we would like to discuss some aspects that could be improved in future work.

### Couple detection (filtering)

Something that introduces a significant amount of noise in the resulting data is couples (or families) that are walking close to each other or even holding hands. Many restrictive measures do not limit the distance that such persons are allowed to have and they should thus not be identified as a violation.

At this point, the SDM contains no procedure to ignore couples. Such a procedure would probably contain a methodology that ignores all violations between two people that have been in violation for at least an X amount of frames.

### Improved perspective

(...)

### Automatic perspective

At this point, the SDM has one step that severely limits the applicability of such a tool; the perspective of a scene has to be given manually. Introducing methodology to automatically detect the perspective of a scene would be essential before deploying such a tool on large scale. A limitation that related tools that omit the bird’s eye view altogether need not address.
