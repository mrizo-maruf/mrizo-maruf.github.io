---
layout: post
title: "Getting Started with 3D Scene Graphs for Robotics"
---

This is a sample blog post to show how things look. You can delete this file and write your own.

## Why Scene Graphs?

Scene graphs provide a structured representation of a 3D environment — objects, their attributes, and the relationships between them. For robots, this means going beyond raw point clouds to a semantic understanding of the world.

## A Simple Example

Consider a table with a mug on it. A scene graph would encode:

- **Nodes**: table, mug, floor
- **Edges**: mug *on* table, table *on* floor

This lets a robot answer queries like *"What is on the table?"* without re-processing the entire scene.

## Code Snippet

Here's a minimal example of building a graph with NetworkX:

```python
import networkx as nx

G = nx.DiGraph()
G.add_node("table", category="furniture")
G.add_node("mug", category="object")
G.add_edge("mug", "table", relation="on")

print(list(G.successors("mug")))  # ['table']
```

## What's Next?

In future posts I'll cover how to generate these graphs from RGB-D data in real time and integrate them with robot control pipelines.

Stay tuned!
