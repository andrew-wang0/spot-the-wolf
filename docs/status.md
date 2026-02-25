
layout: default
title: Status
---

<video width="100%" controls>
  <source src="./img/vid.mp4" type="video/mp4">
</video>


## Project summary

<!--
A short paragraph outlining the main idea of the project (see further
instructions here: https://royf.org/crs/CS175/W26/proposal.pdf). You now have
a better sense of what your project is about than you did in the proposal, so update and clarify
beyond that version. Do not change your proposal.md page unless you completely changed
your topic since your proposal.
-->
Our project aims to build a real-time wolf detection system for Minecraft that locates wolves in a live gameplay environment. The input to the system consists of image frames captured from Minecraft.

These images are collected through a semi-automated data generation process that takes screenshots and associates each wolf with 2D bounding box coordinates in YOLO format through a custom made Minecraft mod.

The output of the model is a set of axis-aligned bounding boxes (AABB), each corresponding to a detected wolf in the frame, along with a confidence score. Each bounding box specifies the predicted location of a wolf in image coordinates.

In practice, running our trained model in a live environment will be able to detect wolves in gameplay frames, enabling live visual highlighting of wolves during gameplay.

## Approach

<!--
Give a detailed description of your approach, in a few of paragraphs (at least a
couple). You should summarize the main method you are using, such as by overviewing the
structure of its data, how it samples that data, and the equations of the loss(es) it optimizes
(you can copy this information from scientific publications or online resources, in which case
cite them clearly. The default GitHub Pages we previously shared includes an example of
redering math within Markdown). You should also provide details about the approach as it
applies to your scenario, such as how you set up inputs and outputs (e.g. states / observations,
actions, and rewards), how much data you use (e.g. for how many interaction steps you train),
and the values of any hyperparameters (cite your source for default hyperparameter values,
and for any changed values detail if and how you tune them and the numbers you end up
using). A good guideline is to incorporate sufficient details so that most of your approach is
reproducible by a reader. You’re encouraged to use figures and tables for this, as appropriate,
e.g. as we used in the exercises
-->
**Model Architecture**

We fine-tune YOLO26 model on a custom dataset of labeled Minecraft wolf screenshots.

**Data Collection**

To generate labeled training data, we implemented a Minecraft mod using NeoForged. The mod automatically captures screenshots and generates bounding box annotations for wolves.

At runtime, the mod automatically spawns wolves directly in front of the player at random coordinates. Before spawning new wolves, it removes previously spawned wolves to prevent a buildup of wolves. We capture an spawn wolves and capture images on a 1 second interval.

For each wolf entity, the mod accesses the mob’s 3D bounding hitbox provided by the game engine. It computes the extreme corner points of the hitbox and projects them into screen space. To ensure that only visible wolves are labeled, the we use ray tracing from the camera position to each corner point of the hitbox. If the rays indicate that the wolf is not fully occluded by terrain or other objects, the wolf is considered visible.

This automated pipeline allows us to generate labeled data directly from the game engine without labelling manually.

**Data Setup**

Our dataset consists of Minecraft screenshots containing wolves, paired with bounding box annotations in YOLO format. Each annotation file contains one line per object:

`<class_id> <center_x> <center_y> <width> <height>`

All coordinates are normalized to the range [0, 1] relative to image dimensions.

The `wolf_yolo26_uno` training script operates on the pre-split dataset layout (train/val directories).

**Hyperparameters.** We train for 60 epochs with batch size 16 and image size 640x640. The confidence threshold for inference is 0.25 and the IoU threshold for TP matching is 0.5.

**Evaluation script.** After training, `evaluation/main.py` loads the saved `best.pt` weights and runs inference on the snow test set (`test_dataset/snow/`). It computes per-image TP/FP/FN counts using greedy highest-confidence-first matching at IoU ≥ 0.5, then aggregates the results.

We generate side-by-side expected-vs-predicted images that are exported to `evaluation/output/snow/` for qualitative visual analysis.


## Evaluation

We compare a v1 model against our improved v2 model across two environments: a controlled grass superflat world and a natural overworld. All evaluations use an IoU threshold of 0.50 and a confidence threshold of 0.25.
In the future, we will add more model comparisons.

### v1 Model
v1 used a dataset size of 4 based on randomly generated superflat images.

**Grass (superflat)**

| TP | FP | FN | Precision | Recall |
|----|----|----|-----------|--------|
| TODO | TODO | TODO | TODO | TODO |

TODO: brief interpretation of baseline grass results.

**Overworld**

| TP | FP | FN | Precision | Recall |
|----|----|----|-----------|--------|
| TODO | TODO | TODO | TODO | TODO |

TODO: brief interpretation of baseline overworld results.

---

### v2 Model 
v2 used a dataset size of 250 based on randomly generated superflat images.

**Grass (superflat)**

| TP | FP | FN | Precision | Recall |
|----|----|----|-----------|--------|
| 129 | 6 | 0 | 0.9556 | 1.0000 |

The model achieves perfect recall (1.00), meaning every ground-truth wolf instance in the grass biome dataset was successfully detected with no missed detections. Precision is 95.56%, with only 6 false positive bounding boxes. These results demonstrate that the improved model performs extremely well in controlled test data, both in terms of detection coverage and prediction accuracy.

<img width="1920" height="1192" alt="image" src="https://github.com/user-attachments/assets/d8f24730-43e8-4cfa-8397-9c41c14897be" />

**Overworld**

| TP | FP | FN | Precision | Recall |
|----|----|----|-----------|--------|
| TODO | TODO | TODO | TODO | TODO |

TODO: brief interpretation of improved model overworld results.

<!--
An important aspect of your project, as we mentioned in the beginning, is
evaluating your project. Be clear and precise about describing the evaluation setup, for both
quantitative and qualitative results. Present the results to convince the reader that you have a
working implementation. Use plots, charts, tables, screenshots, figures, etc. as appropriate.
For each type of evaluation that you perform, you’ll probably need at least 1 or 2 paragraphs
(excluding figures etc.) to describe it.
-->

## Remaining Goals and Challenges

<!--
In a couple of paragraphs, describe your goals for the
remainder of the quarter. At the very least, describe in what ways your current prototype is
more limited than your final goal, and what you want to add to make it a complete contribution.
Note that if you think your method is working well enough, but have not performed sufficient
evaluation to gain insight, doing this should be a goal. Similarly, you may propose comparing
with other methods / approaches / manually tailored solutions (when feasible) that you did not
get a chance to implement, but can enrich your discussion in the final report. Finally, given
your experience so far, describe some of the challenges you anticipate facing by the time your
final report is due, to what degree you expect them to become roadblocking obstacles, and
what you might try in order to overcome them.
-->

The current prototype is functional but limited in several ways. Model performance is a reasonable baseline but has room for improvement. The dataset size and diversity are the main bottleneck. Expanding the dataset with programmatically generated images from a range of biomes, lighting levels, and wolf poses is the top priority.

A key challenge for our model is domain shift, particularly across different Minecraft biomes. For example, early results show that performance drastically drops in the snow biome compared to grass environments. Snow introduces high brightness, reduced contrast, and background textures that more closely resemble wolf fur, making detection more difficult. 
We plan to address this by collecting targeted snow-biome data and diversifying our dataset.

A major remaining goal is transitioning from offline evaluation to live deployment. Currently, the model operates on saved screenshots. A future milestone is integrating the trained model into a real-time tool that:

- Captures frames directly from Minecraft during gameplay.
- Feeds each frame through the trained YOLO model.
- Overlays predicted bounding boxes and confidence scores onto the game display.

Using NeoForged, we plan to implement a mod that captures live frame data and renders detection results directly onto the screen.

## Resources Used

<!--
Mention all the resources that you found useful in implementing your
method, experiments, and analysis. This should include everything like code documentation,
AI/ML libraries, source code that you used, StackOverflow, etc. You do not have to include
every tiny (or commonplace) thing you used, but it is important to report the sources that are
crucial to your project. One aspect that does need to be comprehensive is a description of any
use you made of AI tools.
-->

Ultralytics YOLO26 (Previously also used YOLOv8) - model architecture, inference API
https://docs.ultralytics.com

OpenCV (cv2) - image loading, preprocessing, and drawing bounding boxes for evaluation

Minecraft - game environment used to generate training and evaluation screenshots

NeoForged - modding framework used to develop a Minecraft mod for automated data capture and eventual integration with gameplay

Python Standard Library - argparse, pathlib, random, shutil, datetime

AI Assistance - ChatGPT was used for code clarification and minor implementation support. System design, other implementation, and integration were implemented manually.
