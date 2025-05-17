---
layout: post
title: "K-DOP colliders in Unreal Engine"
date: 2025-05-17 16:30 +1100
categories: Coding
---

Unreal Engine's colliders are not as intuitive as those in Unity. Particularly, it has colliders named `N-DOP Simplified Collision`, that may seem confusing at first.  
![collider_types](/assets/2025-05-17-colliders-in-unreal-engine/collider_types.png)

Basically, how the [official document](https://dev.epicgames.com/documentation/en-us/unreal-engine/add-a-k-dop-collision-hull-to-a-static-mesh-in-unreal-engine) describes each option is pretty accurate. However, without background knowledge, it may be difficult to fully understand the behaviour.

The original paper on K-DOP (Discrete Oriented Polytope) is [publicly available](https://ieeexplore.ieee.org/abstract/document/675649) and I think this is sufficient to understand K-DOP in Unreal Engine.

TLDR summary for those without time and my future-self: 
K-DOP first creates an AABB (Axis-Aligned Bounding Box), and bevels its edges only (18-DOP) or vertices and edges altogether (26-DOP). The 10-DOP options only bevel the 4 edges parallel to the X, Y or Z axis.


# Slightly More Detailed Explanation
Suppose in a 2D space, we were to create a bounding-box representation of a shape by doing the following:
1. Locate the 'minimum' and 'maximum' pixel of the shape along the x-axis. Draw vertical lines on those points.
2. Locate the 'minimum' and 'maximum' pixel of the shape along the y-axis. Draw horizontal lines on those points.
3. Create a rectangular bounding box from the intersecting points

This is basically 4-DOF in 2D space, also known as AABB.

the `K` in the name represents the number of orientations. It first encapsulates objects with `H` halfspaces. a halfspace `H_i`, whose normal is parallel to `K_i` direction, is placed on the 'maximum' point along the direction.
It then creates a bounding representation by taking the intersection of all.

As such, 4-DOF in 2D space is AABB if the 4 directions are `<1, 0>, <-1, 0>, <0, 1>, <0, -1>`.

To optimise computation, Half of the `K` directions are the opposite to the other half. This essentially creates `K/2` unique axes and hyperplanes with one axis constrained by the min and max points.

Conventionally, the standard axes of `D` dimensional space are contained in the `K` directions. Therefore, What modifies the AABB created by the standard-axis hyperplanes are the hyperplanes along `(K - 2 * D)/2` different axes. 

In the paper, the author describes 14-DOP (4 add. axes), 18-DOP (6 add. axes) and 26-DOP (10 add. axes) in 3D space.
The 4 additional axes in 14-DOP penetrate the 8 vertices of a cube, 6 additional axes in 18-DOP penetrate the 12 edges and the 10 axes in 26-DOP are the combination of 14-DOP and 18-DOP.
![cube](/assets/2025-05-17-colliders-in-unreal-engine/cube.png)
Each axis represented by a different colour.


As such, Unreal Engine's `18DOF Simplified Collision` cannot accurately encapsulate a cube whose edges and vertices are beveled:
![18DOF](/assets//2025-05-17-colliders-in-unreal-engine/18DOF.png)

Whereas `26DOP Simplified Collision` can because it bevels both vertices and edges:
![26DOF](/assets//2025-05-17-colliders-in-unreal-engine/26DOF.png)

The `10-DOP` options are similar: they bevel the 4 edges of a cube that are parallel to a particular axis.
