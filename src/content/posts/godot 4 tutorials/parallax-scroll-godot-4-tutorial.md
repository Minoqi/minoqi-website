---
title: Parallax Camera Tutorial for Godot 4
published: 2024-11-06
description: 'Create a parallax camera controller in Godot 4 for both 2D and 3D'
image: ''
tags: [Tutorial, Godot, Game Programming]
category: 'Godot 4 Tutorials'
draft: false
lang: ''
---
Godot has a built in parallax system, but only for 2D games, so we want one that can work in both 2D and 3D. Plus, we have more customizable control over it!

**Sudo Code**:
- Parallax Controller
	- Stores reference to the camera
	- Stores an array of all the affected background layers
- Parallax Layer
	- Placed on the background layer to move
	- Contains variables to control what kind of movement it does:
        1. up and down
        2. left and right
        3. even depth! (forward and back)

![](src/assets/images/godot_4_tutorials/ParallaxBackgroundDrawing.excalidrawBK.png)

---

# Godot Tutorial

## 3D

**Parallax Controller**

- First, we want to store a list of all our layers affected by the parallax as well as a reference to the camera
- Then, as long as it's enabled, we want the parallax layers to move with the camera by calling the `_move_parallax_layers()` function in the `_process()` function
- Inside our `_move_parallax_layers()` function we:
    1. Get the cameras updated position
    2. Check that the camera has moved at all and exit early if it hasn't
    3. Get the direction the camera moved in
    4. Call each layer to move based on the cameras direction
    5. Save the cameras position for next frame

```gdscript
extends Node3D
class_name ParallaxController

# Variables
@export var parallaxLayers : Array[CustomParallaxLayer]
@export var camera : Camera3D

var enabled : bool = true
var lastCameraPos : Vector3 = Vector3.ZERO


func _process(delta):
	if enabled:
		_move_parallax_layers()


func _move_parallax_layers() -> void:
	var newCameraPos : Vector3 = camera.global_position

	if newCameraPos - lastCameraPos == 0:
		return

	var direction : Vector3 = lastCameraPos.direction_to(newCameraPos).normalized()
	for layer in parallaxLayers:
		layer.move(direction)

	lastCameraPos = newCameraPos
```

**Parallax Layer**

- Here we store the parallax speed we want as well as what directions we want it affected by
    - This system supports all axis so you can make some cool effects!
- The `move()` function takes in the direction of the camera and we use that to calculate which direction the parallax should happen in
    - We subtract since we want it to move in the *opposite* direction of the camera

```gdscript
extends Sprite3D
class_name CustomParallaxLayer

# Variables
@export var parallaxSpeed : float
@export var moveLeftRight : bool = true
@export var moveUpDown : bool = false
@export var moveDepth : bool = false


func move(_direction : Vector3) -> void:
	if moveLeftRight:
		global_position.x -= _direction.x * parallaxSpeed * get_process_delta_time()

	if moveUpDown:
		global_position.z -= _direction.z * parallaxSpeed * get_process_delta_time()

	if moveDepth:
		global_position.y -= _direction.y * parallaxSpeed * get_process_delta_time()
```

## 2D

- Our 2D system works the same way, but without the extra axis

**Parallax Controller**
```gdscript
extends Node2D
class_name ParallaxController

# Variables
@export var parallaxLayers : Array[CustomParallaxLayer]
@export var camera : Camera2D

var enabled : bool = true
var lastCameraPos : Vector2 = Vector2.ZERO


func _process(delta):
	if enabled:
		_move_parallax_layers()


func _move_parallax_layers() -> void:
	var newCameraPos : Vector2 = camera.global_position

	if newCameraPos - lastCameraPos == 0:
		return

	var direction : Vector2 = lastCameraPos.direction_to(newCameraPos).normalized()
	for layer in parallaxLayers:
		layer.move(direction)

	lastCameraPos = newCameraPos
```

**Parallax Layer**
```gdscript
extends Sprite2D
class_name CustomParallaxLayer

# Variables
@export var parallaxSpeed : float
@export var moveLeftRight : bool = true
@export var moveUpDown : bool = false


func move(_direction : Vector2) -> void:
	if moveLeftRight:
		global_position.x -= _direction.x * parallaxSpeed * get_process_delta_time()

	if moveUpDown:
		global_position.y -= _direction.y * parallaxSpeed * get_process_delta_time()
```

---

**Resources:**

[Video by Raycastly](https://www.youtube.com/watch?v=MEy-kIGE-lI)

[Parallax Background Code Snippet by Raycastly](https://pastebin.com/DG5jcAMZ)

[Parallax Layer Code Snipper by Raycastly](https://pastebin.com/ZniykeGz)