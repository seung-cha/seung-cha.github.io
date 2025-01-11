---
layout: post
title: "3D Technical Combat System 0: Input Processing"
date: 2025-01-11 18:30 +1100
categories: Coding
---

A year ago, I wrote a post about input buffering in (2D) fighting game. The solution I introduced works, but it is limited to the 2D space only (not to mention the example code is unscalable). As a fan of 2D/3D action games, I decided to make a series of posts to demonstrate how I implemented a motion-input enabled combat system.

The resulting system will be similar to those you can find in various 3D action games, such as `Devil May Cry` or `Monster Hunter`.

This is not a how-to series. This series solely summarises my findings.

Input Processing - Theory
---
The freedom of camera movement and the expansion to the 3D space bring complications. Unlike 2D fighters where the camera is locked in place and only the left and right keys can be inverted, 3D action games must adjust all raw directional inputs based on the camera and the character to make control more fluid.

Take a look at an example where I perform a launch attack (Back + Left Click) as `Nero` in `Devil May Cry 5`.

<iframe width="1148" height="646" src="https://www.youtube.com/embed/3T_Xtt6dsz4" title="Launch Attack DMC 5" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

As I rotate the camera around, I have to adjust my directional input so that my input (if thought of as a vector) roughly aligns to the opposite of where my character faces.

It is difficult to conceptualise the relation between the camera and the character when it comes to input processing (especially in games without target-locking). In short, raw input vectors are defined in the camera's coordinate system and transformed to the character's coordinate system.

If the camera and the character are both looking in the same direction (that is, the camera is looking at the back of the character), no input processing is needed. Holding `W` moves forwards, away from the camera, `S` backwards, towards the camera, and `A` `D` left and right, respectively.

Suppose the camera and the character orientation are given as follows:
![img](/assets/2025-01-11-technical-combat-system-0/dmc_nero.png)

Now the relation becomes more clear. The character does not move forwards (towards the enemy) upon pressing `W`. Rather, he will step to the right, away from the camera. Without target-locking, the character will still move forwards and away from the camera by rotating the body to align with the input. This is why target-locking is crucial in observing the relation.

Input Processing - Implementation
---
Now that the relation is clear, it is easy to implement solution:
1. Raw input vector is processed in the camera's coordinate system
2. The input vector in the camera space is transformed to the character space
3. The input vector in the character space is processed normally

Suppose the character is looking forwards <0, 0, 1> (hence its forward vector is <0, 0, 1>). If the camera is looking at the side of the character, as shown above, the camera's forward vector will be <1, 0, 0>. If the player holds `W`, they are essentially saying "Move in the direction of <1, 0, 0>", which is to the right of the character. 

Similarly, holding `A` will command the player to move in the direction of <0, 0, 1> which is moving forwards for the character.

In Unity, one can transform vector from local to world using [Transform.transformDirection](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Transform.TransformDirection.html). The resulting code will look as follows:

```cs
Vector2 rawInput = GetXYInput();
// Assume the camera's forwards vector is projected onto the flat surface.
// This will give you the direction in which the character should move, in world space.
Vector3 processedInput = camera.transform.transformDirection(new Vector3(rawInput.x, 0.0f, rawInput.y));

// It is possible to move the character without transforming it to the character space.
// However, this will not give us information as to which button this vector corresponds to for technical input.
Vector3 input = character.InverseTransformDirection(processedInput);
Vector2 finalInput = new Vector2(input.x, input.z);

// You may now use the input as if it's from the keyboard directly.
// For example, to move the character:
character.Move(character.transform.forward * finalInput.y + character.transform.right * finalInput.x);
// Which is equivalent to:
// character.Move(processedInput);
```

The extra step of converting the world space input into the character space input allows us to gain insight on 
what the processed input key would be. This allows us to implement technical motions without worrying about the camera orientation: You would write "If the player is holding back, do A move" instead of "If the player is holding X and the camera orientation is Y, do Z".
 