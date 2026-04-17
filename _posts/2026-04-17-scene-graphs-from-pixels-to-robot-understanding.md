---
layout: post
title: "From Pixels to Understanding: Scene Graphs and Their Role in Robotics"
---

What does it mean for a machine to *understand* a scene? Not just to label pixels or draw boxes around objects, but to truly grasp that the mug is *on* the table, the table is *in* the kitchen, and the kitchen is *part of* the apartment. This is the promise of **scene graphs** --- a structured representation that captures not just *what* is in a scene, but *how* things relate to each other.

In this post, I trace the evolution of visual scene understanding from its classical roots to the emergence of scene graphs, survey the key papers that shaped the field, and discuss how scene graphs are becoming the backbone of robotic perception systems.

---

## 1. The Road to Scene Understanding

Computer vision has always been, at its core, about making machines see. But "seeing" has meant very different things over the decades.

### Classical Computer Vision

Before deep learning, scene understanding was built on hand-crafted features. Researchers developed edge detectors (Canny, Sobel), feature descriptors (SIFT, SURF, HOG), and template matching methods. These approaches could match patterns and find correspondences, but they struggled with the variability and complexity of real-world scenes.

The classical pipeline was:

1. **Feature extraction** --- detect keypoints and compute local descriptors
2. **Matching** --- find correspondences across images
3. **Geometric reasoning** --- estimate structure via epipolar geometry, homographies

This worked well for tasks like stereo matching and image stitching, but it couldn't answer the deeper question: *what is in this image?*

### Image Classification: Naming the Scene

The deep learning revolution, ignited by **AlexNet** (Krizhevsky et al., 2012), changed everything. For the first time, a neural network dramatically outperformed hand-crafted methods on ImageNet, achieving a top-5 error rate of 15.3% (vs. 26.2% for the runner-up).

<div class="paper-ref">
<strong>ImageNet Classification with Deep Convolutional Neural Networks</strong><br>
<em>A. Krizhevsky, I. Sutskever, G. Hinton. NeurIPS 2012.</em><br>
The paper that launched modern deep learning in computer vision --- a deep CNN trained end-to-end on 1.2M images.
</div>

Classification answers *"What is in this image?"* --- but only at the image level. A single label for an entire scene. This is a drastic oversimplification. Real scenes contain multiple objects, complex spatial arrangements, and rich interactions.

### Object Detection: What and Where

The next leap came with **R-CNN** (Girshick et al., 2014) and its successors (Fast R-CNN, Faster R-CNN), which combined region proposals with CNN features to detect and localize *multiple* objects in an image. YOLO (Redmon et al., 2016) later pushed this to real-time speeds.

Detection answers *"What objects are present, and where are they?"* --- each object gets a bounding box and a class label. But bounding boxes are coarse. They don't capture object shape, and they say nothing about how objects relate to each other.

### Semantic and Instance Segmentation: Pixel-Level Understanding

**Fully Convolutional Networks** (Long et al., 2015) introduced pixel-level classification --- every pixel gets a semantic label. **Mask R-CNN** (He et al., 2017) extended this to instance segmentation, distinguishing individual object instances with precise masks.

Now we could answer *"Which pixels belong to which object?"* --- but we were still treating each object in isolation. The scene was a bag of labeled regions, with no structure connecting them.

<figure>
<img src="/assets/images/blog/scene-graphs/cv-evolution.svg" alt="Evolution of visual scene understanding from classification to scene graphs">
<figcaption>Figure 1. The evolution of visual scene understanding: from whole-image classification (2012), to object detection with bounding boxes (2014), to pixel-level segmentation (2015), to structured scene graphs that capture relationships (2017).</figcaption>
</figure>

### The Missing Piece: Relationships

Consider the difference between these two descriptions of the same image:

- **Bag of labels**: person, horse, hat, grass, sky
- **Structured description**: a *person* is *riding* a *horse*, the person is *wearing* a *hat*, the horse is *standing on* *grass*, the *sky* is *above* the scene

The first is what detection and segmentation give us. The second is what we actually need for understanding --- and it's what scene graphs provide.

---

## 2. What Is a Scene Graph?

A **scene graph** is a graph-structured representation of a visual scene where:

- **Nodes** represent objects (or regions), annotated with attributes (e.g., color, material, state)
- **Edges** represent pairwise relationships between objects (e.g., spatial: *on*, *next to*; actions: *riding*, *holding*; comparative: *taller than*)

Formally, a scene graph is a tuple `G = (O, R)` where `O = {o_1, ..., o_n}` is a set of objects, each with a class label and attributes, and `R ⊆ O × P × O` is a set of directed relationships, where `P` is a predicate vocabulary.

<figure>
<img src="/assets/images/blog/scene-graphs/scene-graph-example.svg" alt="A scene graph example showing objects as nodes and relationships as edges">
<figcaption>Figure 2. Anatomy of a scene graph: objects (person, dog, park, bench, tree, sky) are nodes with attributes; directed edges encode pairwise relationships (walking, in, near, above). The graph transforms an unstructured image into a queryable semantic structure.</figcaption>
</figure>

Scene graphs are powerful because they make implicit visual knowledge *explicit and queryable*. Given a scene graph, a system can answer:

- *"What is the person doing?"* --- follow edges from `person`
- *"What objects are on the table?"* --- find all nodes with an `on` edge pointing to `table`
- *"Is there a path from the kitchen to the bedroom?"* --- graph traversal

This structured representation is exactly what downstream tasks like visual question answering, image retrieval, image generation, and --- crucially --- robotic reasoning require.

---

## 3. A Brief History of Scene Graphs

The concept of scene graphs originated in computer graphics (SGI's OpenGL Performer, VRML in the 1990s), where they were used to organize the spatial hierarchy of 3D rendering scenes. But the idea of using scene graphs for *visual understanding* --- parsing an image into a structured relational representation --- is a distinctly computer vision contribution that emerged in the mid-2010s.

### The Timeline

<div class="timeline-item">
<span class="timeline-year">2015 --- Visual Relationship Detection</span><br>
<strong>Lu et al., "Visual Relationship Detection with Language Priors" (ECCV 2016, arXiv 2015)</strong> introduced the task of detecting <code>&lt;subject, predicate, object&gt;</code> triplets in images. Rather than treating relationships as a classification problem over all possible triplets (which is combinatorially explosive), they used language priors to project objects and predicates into a shared embedding space. This paper established the core formulation that scene graph generation builds upon.
</div>

<div class="timeline-item">
<span class="timeline-year">2017 --- Visual Genome & Scene Graph Generation</span><br>
<strong>Krishna et al., "Visual Genome: Connecting Language and Vision Using Crowdsourced Dense Image Annotations" (IJCV 2017)</strong> was the watershed moment. The Visual Genome dataset provided 108K images densely annotated with objects, attributes, relationships, region descriptions, and question-answer pairs. For the first time, researchers had large-scale ground-truth scene graphs to train and evaluate on.<br><br>
In the same year, <strong>Xu et al., "Scene Graph Generation by Iterative Message Passing" (CVPR 2017)</strong> proposed the first end-to-end neural model for scene graph generation (SGG). Their model used a Gated Recurrent Unit (GRU) based message passing framework that iteratively refines object and relationship features by passing messages along the graph structure. This was the first paper to generate full scene graphs (not just individual relationships) from images.
</div>

<div class="timeline-item">
<span class="timeline-year">2018 --- Neural Motifs and Graph R-CNN</span><br>
<strong>Zellers et al., "Neural Motifs: Scene Graph Parsing with Global Context" (CVPR 2018)</strong> made a striking observation: most scene graph generation models were essentially learning dataset biases (e.g., "person-riding-horse" is common, so predict it whenever you see a person near a horse). They showed that a simple frequency baseline was surprisingly competitive and proposed a stacked-motif network that captures global context through bidirectional LSTMs.<br><br>
<strong>Yang et al., "Graph R-CNN for Scene Graph Generation" (ECCV 2018)</strong> introduced an efficient alternative using an attentional Graph Convolutional Network (aGCN) for relationship reasoning, with a relation proposal network that prunes unlikely relationships before classification.
</div>

<div class="timeline-item">
<span class="timeline-year">2019 --- Unbiased SGG and 3D Scene Graphs</span><br>
<strong>Tang et al., "Unbiased Scene Graph Generation from Biased Training" (CVPR 2020, arXiv 2019)</strong> tackled the long-tail distribution problem head-on. In Visual Genome, a handful of predicates (on, has, wearing) dominate, causing models to ignore rare but informative relationships. They proposed causal inference methods (TDE, counterfactual analysis) to de-bias predictions.<br><br>
In parallel, <strong>Armeni et al., "3D Scene Graph: A Structure for Unified Semantics, 3D Space, and Camera" (ICCV 2019)</strong> extended scene graphs to 3D, constructing hierarchical graphs from 3D point clouds with layers for cameras, objects, rooms, and buildings. This was the first paper to bring the scene graph idea into 3D spatial understanding.
</div>

<div class="timeline-item">
<span class="timeline-year">2020--2021 --- Incremental and Dynamic Scene Graphs</span><br>
<strong>Wu et al., "SceneGraphFusion: Incremental 3D Scene Graph Prediction from RGB-D Sequences" (CVPR 2021)</strong> tackled the critical challenge of building scene graphs incrementally from streaming sensor data. Rather than requiring a complete 3D reconstruction before graph construction, their method updates the graph as new frames arrive --- essential for real-time robotic applications.<br><br>
<strong>Rosinol et al., "3D Dynamic Scene Graphs: Actionable Spatial Perception with Places, Objects, and Humans" (RSS 2020)</strong> introduced dynamic scene graphs that evolve over time, tracking moving agents and objects.
</div>

<div class="timeline-item">
<span class="timeline-year">2022 --- Hydra: Real-Time 3D Scene Graphs</span><br>
<strong>Hughes et al., "Hydra: A Real-time Spatial Perception System for 3D Scene Graph Construction and Optimization" (RSS 2022)</strong> was a major milestone for robotics. Hydra constructs a hierarchical 3D scene graph in real time from sensor data, with layers for the metric-semantic mesh, objects, places (navigation graph), rooms, and buildings. It runs online on a robot, enabling real-time spatial reasoning.
</div>

<div class="timeline-item">
<span class="timeline-year">2023--2024 --- Foundation Models Meet Scene Graphs</span><br>
<strong>Gu et al., "ConceptGraphs: Open-Vocabulary 3D Scene Graphs for Perception and Planning" (ICRA 2024)</strong> combined vision-language models (CLIP, LLaVA) with 3D mapping to build open-vocabulary scene graphs --- graphs where objects are described with natural language rather than fixed class labels. This enabled zero-shot generalization to novel objects and environments.<br><br>
<strong>Rana et al., "SayPlan: Grounding Large Language Models using 3D Scene Graphs for Scalable Robot Task Planning" (CoRL 2023)</strong> demonstrated that LLMs, when grounded with 3D scene graphs, can perform complex multi-step task planning in large environments, while scene graphs keep the LLM's context focused and factual.
</div>

<div class="timeline-item">
<span class="timeline-year">2025 --- Scene Graphs for Control</span><br>
Recent work has focused on closing the loop from perception to control. Scene graphs are being used not just to understand the environment, but to directly condition robot policies. Methods like knowledge-guided manipulation use scene graph representations to structure reinforcement learning reward functions and guide exploration in manipulation tasks.
</div>

---

## 4. From 2D to 3D: Scene Graphs Enter the Physical World

The transition from 2D to 3D scene graphs was a critical step for robotics. A 2D scene graph parsed from a single image is useful for visual question answering and image retrieval, but robots operate in 3D space. They need to know not just that the mug is *on* the table, but *where* in 3D the mug is, how to reach it, and what's between them.

### Hierarchical 3D Scene Graphs

The key insight behind modern 3D scene graphs is **hierarchy**. A flat graph of objects and relationships doesn't scale to large environments. Instead, 3D scene graphs organize information in layers of increasing abstraction:

<figure>
<img src="/assets/images/blog/scene-graphs/3d-scene-graph-layers.svg" alt="Hierarchical 3D scene graph with layers from mesh to building">
<figcaption>Figure 3. Hierarchical 3D scene graph structure (adapted from Hydra, Hughes et al. 2022). The graph is organized in five layers: (1) a dense metric-semantic mesh, (2) 3D object instances, (3) a free-space places layer for navigation, (4) rooms with connectivity, and (5) building-level topology. Each layer captures a different level of abstraction.</figcaption>
</figure>

This hierarchy is powerful because different tasks require different levels of detail:

- **Navigation**: the places layer provides a topological graph for path planning
- **Object manipulation**: the object layer provides precise 3D poses and semantics
- **High-level task planning**: the room/building layers enable abstract reasoning ("go to the kitchen, then pick up the mug")
- **Geometric reasoning**: the mesh provides dense spatial information

### Building 3D Scene Graphs

Constructing a 3D scene graph from sensor data involves several components:

1. **3D Reconstruction**: build a metric map from RGB-D or LiDAR data (via SLAM, NeRF, or 3D Gaussian Splatting)
2. **Semantic Segmentation**: label regions with object categories (using 2D segmentation models like SAM, projected into 3D)
3. **Object Instance Extraction**: cluster labeled points into distinct object instances
4. **Relationship Inference**: determine spatial and semantic relationships between objects
5. **Hierarchical Structuring**: organize nodes into room and building layers

Systems like Hydra perform all of this incrementally in real time, making scene graphs practical for online robotic perception.

---

## 5. Scene Graphs for Robotics

Why are scene graphs particularly compelling for robotics? Because robots need to *reason* about their environment, not just perceive it. Scene graphs bridge the gap between low-level perception and high-level reasoning.

<figure>
<img src="/assets/images/blog/scene-graphs/robot-scene-graph-pipeline.svg" alt="Robotic perception pipeline using scene graphs">
<figcaption>Figure 4. A scene-graph-based robotic perception pipeline: raw sensor data is processed into detections and 3D reconstructions, which are structured into a 3D scene graph. The graph then supports task planning, spatial queries, and LLM-based reasoning, ultimately driving robot actions like navigation and manipulation.</figcaption>
</figure>

### Task Planning with Scene Graphs

Traditional task planning requires a symbolic state representation (think PDDL). Scene graphs provide exactly this --- a graph-based state description that planners can reason over. Given a scene graph, a planner can:

- **Identify preconditions**: "to pour water, the robot must be holding the pitcher, and the pitcher must contain water"
- **Track state changes**: update the graph after each action (move `mug` from `table` to `counter`)
- **Plan multi-step tasks**: find a sequence of actions that transforms the current graph into a goal graph

SayPlan (Rana et al., 2023) showed that LLMs excel at this when grounded with scene graphs. The scene graph constrains the LLM's output to physically plausible plans and provides the spatial context that language alone cannot capture.

### Manipulation

For object manipulation, scene graphs encode the spatial relationships that matter:

- **Support relationships**: what is stacking on what (critical for pick-and-place)
- **Containment**: what is inside what (important for packing, pouring)
- **Proximity**: what is near what (relevant for collision avoidance)
- **Functional relationships**: what is connected to what (important for articulated objects)

A robot tasked with "clear the table" can query the scene graph for all objects with an `on` edge to `table`, plan a sequence to remove them, and update the graph as it works.

### Navigation

The places layer of a hierarchical 3D scene graph is essentially a topological navigation graph. Combined with room-level semantics, it enables:

- **Semantic navigation**: "go to the kitchen" (find the `kitchen` node, plan a path through the places layer)
- **Object search**: "find the keys" (check which room contains `keys`, navigate there)
- **Multi-room planning**: "bring the book from the study to the bedroom"

### Human-Robot Interaction

Scene graphs provide a natural interface for communication between humans and robots. When a human says "move the red cup next to the plate," the robot can:

1. Parse the language into a graph query (find `cup` with attribute `red`)
2. Locate the target in the scene graph
3. Determine the spatial goal (`next to` `plate`)
4. Plan and execute the action
5. Update the scene graph

---

## 6. Current State and Open Challenges

Scene graphs have matured significantly, but several challenges remain:

**Long-tail relationships.** Most datasets are dominated by a few common predicates (`on`, `has`, `near`). Rare but informative relationships (`leaning against`, `plugged into`, `reflected in`) remain hard to detect. Debiasing methods help, but the fundamental data imbalance persists.

**Dynamic scenes.** Most scene graph methods assume static environments. Real-world scenes change constantly --- objects move, people walk, doors open and close. Dynamic scene graphs (Rosinol et al., 2020) address this, but efficient real-time updates remain challenging.

**Open-vocabulary generalization.** Fixed-vocabulary scene graphs can only represent pre-defined object and relationship classes. ConceptGraphs showed that vision-language models can enable open-vocabulary 3D scene graphs, but the accuracy and consistency of such systems still lag behind closed-vocabulary methods.

**Scalability.** Scene graphs grow with scene complexity. Large buildings with thousands of objects produce massive graphs that are expensive to store, query, and update. Efficient graph representations and pruning strategies are active areas of research.

**Evaluation metrics.** Standard metrics like Recall@K and mean Recall@K have known limitations. They don't capture the downstream utility of a scene graph --- a graph that is "wrong" by recall metrics might still support correct task planning, while a "correct" graph might miss the relationships that actually matter.

**Integration with foundation models.** The convergence of scene graphs with large language models and vision-language models is perhaps the most exciting current direction. Scene graphs provide the structured, grounded representation that LLMs need to reason about the physical world, while LLMs provide the common-sense knowledge and language understanding that scene graphs lack.

---

## 7. Conclusion

The story of scene understanding in computer vision is one of increasing structure. We went from:

- **Classifying** entire images ("this is a kitchen scene")
- to **detecting** individual objects ("there is a mug at position (x, y, w, h)")
- to **segmenting** precise object boundaries (pixel-level masks)
- to **relating** objects to each other ("the mug is on the table, which is in the kitchen")

Scene graphs represent the current frontier of this progression. They transform a scene from a collection of independent detections into a *structured, queryable, and composable* representation of objects, attributes, and relationships.

For robotics, this structure is not a luxury but a necessity. Robots that can build and reason over scene graphs can plan complex tasks, communicate with humans in natural language, and adapt to novel environments --- capabilities that are impossible with raw perception alone.

The field is moving fast. With the integration of foundation models, real-time 3D construction, and direct use in robot control loops, scene graphs are becoming a central data structure in robotic perception. If you work in robotics or vision, this is a space worth watching --- and contributing to.

---

## References

1. Krizhevsky, A., Sutskever, I., & Hinton, G. (2012). ImageNet Classification with Deep Convolutional Neural Networks. *NeurIPS 2012*.

2. Girshick, R., Donahue, J., Darrell, T., & Malik, J. (2014). Rich Feature Hierarchies for Accurate Object Detection and Semantic Segmentation. *CVPR 2014*.

3. Long, J., Shelhamer, E., & Darrell, T. (2015). Fully Convolutional Networks for Semantic Segmentation. *CVPR 2015*.

4. He, K., Gkioxari, G., Dollar, P., & Girshick, R. (2017). Mask R-CNN. *ICCV 2017*.

5. Lu, C., Krishna, R., Bernstein, M., & Fei-Fei, L. (2016). Visual Relationship Detection with Language Priors. *ECCV 2016*.

6. Krishna, R., Zhu, Y., Groth, O., Johnson, J., Hata, K., Kravitz, J., ... & Fei-Fei, L. (2017). Visual Genome: Connecting Language and Vision Using Crowdsourced Dense Image Annotations. *IJCV 2017*.

7. Xu, D., Zhu, Y., Choy, C. B., & Fei-Fei, L. (2017). Scene Graph Generation by Iterative Message Passing. *CVPR 2017*.

8. Zellers, R., Yatskar, M., Thomson, S., & Choi, Y. (2018). Neural Motifs: Scene Graph Parsing with Global Context. *CVPR 2018*.

9. Yang, J., Lu, J., Lee, S., Batra, D., & Parikh, D. (2018). Graph R-CNN for Scene Graph Generation. *ECCV 2018*.

10. Tang, K., Niu, Y., Huang, J., Shi, J., & Zhang, H. (2020). Unbiased Scene Graph Generation from Biased Training. *CVPR 2020*.

11. Armeni, I., He, Z. Y., Gwak, J., Zamir, A. R., Fischer, M., Malik, J., & Savarese, S. (2019). 3D Scene Graph: A Structure for Unified Semantics, 3D Space, and Camera. *ICCV 2019*.

12. Rosinol, A., Gupta, A., Abate, M., Shi, J., & Carlone, L. (2020). 3D Dynamic Scene Graphs: Actionable Spatial Perception with Places, Objects, and Humans. *RSS 2020*.

13. Wu, S. C., Wald, J., Tateno, K., Navab, N., & Tombari, F. (2021). SceneGraphFusion: Incremental 3D Scene Graph Prediction from RGB-D Sequences. *CVPR 2021*.

14. Hughes, N., Chang, Y., & Carlone, L. (2022). Hydra: A Real-time Spatial Perception System for 3D Scene Graph Construction and Optimization. *RSS 2022*.

15. Gu, Q., Kuwajerwala, A., Morin, S., Jatavallabhula, K. M., Sen, B., Agarwal, A., ... & Paull, L. (2024). ConceptGraphs: Open-Vocabulary 3D Scene Graphs for Perception and Planning. *ICRA 2024*.

16. Rana, K., Haviland, J., Garg, S., Abou-Chakra, J., Reid, I., & Suenderhauf, N. (2023). SayPlan: Grounding Large Language Models using 3D Scene Graphs for Scalable Robot Task Planning. *CoRL 2023*.
