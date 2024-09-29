---
title: Poisson Disc Sampling Algorithm in Godot 4
published: 2024-09-28
description: 'An algorithm that can be used to place objects down randomly without overlap and without the need for raycasts or collisions.'
image: ''
tags: [Godot, Tutorial, Game Programming, Algorithm]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

![](src/assets/images/godot_4_tutorials/PoissonDiscSamplingAlgorithmPreviewV1.png)

# What Is The Poisson Disc Sampling Algorithm?

An algorithm that can be used to place objects down randomly without overlap and without the need for raycasts or collisions.

---

# Godot Tutorial

- First let's go over the variables

```gdscript
# Variables
@export var minDistance : float = 30.0 ## Minimum distance between points
@export var sampleAttempts : int = 30   ## Number of attempts to place a point around an active point, avoids infinite loops
@export var maxDistanceMultiplier : float = 2 ## Multiplied to the minimum distance to get the max distance it'll look for a location from the point
@export var chunkSize : Vector2 = Vector2(200, 200)
@export var pointOffset : Vector2 = Vector2(100, 100) ## Adds an offset to the final position if needed

@export_group("Debug")
@export var debugMode : bool = true
@export var pointDebug : PackedScene ## This is just a Node3D circle mesh with a collider that does nothin, but simply is there to help visualize the distance between the points and make sure there's no overlap

var grid : Array = []
var gridCellSize : float
var activeList : Array = [] ## Used for when generating the list of points
var points : Array = [] ## Stores the final points
var usedPoints : Array = []
```

- `minDistance`: Minimum distance required between the two points
- `sampleAttempts`: Number of attempts to place a point, stopping any chances of an infinite while loop
- `maxDistanceMultiplier`: Used to determined the max distance from a point another point can be placed, calculated by multiplying it by the `minDistance`
- `chunkSize`: Size of the chunk (aka the floor)
- `pointOffset`: Adds an offset to the final positions if needed depending on world structure
- `activeList`: Used to loop through all the points to check for more valid points when getting a list of points
- `points`: Stores all the final and valid points
- The script is made up of a few functions

```gdscript
func poisson_disc_sampling_algorithm() -> void:
	_initialize_grid()
	_generate_points()
	
	if debugMode:
		_draw_points()
```

- Let's start with initializing the grid

```gdscript
func _initialize_grid() -> void:
	## Get grid sizing
	gridCellSize = minDistance / sqrt(2)
	var gridWidth : int = int(ceil(chunkSize.x / gridCellSize))
	var gridHeight : int = int(ceil(chunkSize.y / gridCellSize))
	
	## Create 2D grid array
	grid = []
	for i in range(gridWidth):
		grid.append([])
		for j in range(gridHeight):
			grid[i].append(null)
```
- Why do we divide by `sqrt(2)` to find the grid cell size?
	- The equation to find the longest distance across the cell aka the diagonal is `diagonal = side length * sqrt(2)*
	- To ensure the entire cell is within range of r (minimum distance), you want the diagonal cell to be les than or equal to r
	- So `r / sqrt(2)` ensures that if two points are more then r apart, they won't be in the same cell or neighboring cells
- Next let's actually determine our points

```gdscript
func _generate_points():
	## Start with a random point
	var firstPoint : Vector2 = Vector2(randf_range(0, chunkSize.x), randf_range(0, chunkSize.y))
	_add_point(firstPoint)

	
	## Find all valid points
	while activeList.size() > 0:
		var point : Vector2 = activeList.pick_random()
		var isFound : bool = false
		
		for i in range(0, sampleAttempts):
			var newPoint : Vector2 = _generate_random_point_around(point)
			
			if _is_valid_point(newPoint):
				_add_point(newPoint)
				isFound = true
				break
		
		if !isFound:
			activeList.erase(point)
```

- Let's break down some of the functions getting called
- First let's go over the function to add the point

```gdscript
func _add_point(_point: Vector2):
	points.append(_point)
	activeList.append(_point)
	var gridPos = _point_to_gridPos(_point) ## Convert the position into a grid cell ID
	grid[gridPos.x][gridPos.y] = _point
```

- First things first, it stores the point in the points list and the active list of points to get a point from
- Then to add it to the grid we need to convert the points location to a spot in the grid to store it in the array

```gdscript
func _point_to_gridPos(_point: Vector2) -> Vector2:
	return Vector2(int(_point.x / gridCellSize), int(_point.y / gridCellSize))
```

- Then, we search for all the valid points
- The variable `sampleAttempts` gets used to make sure we don't get stuck in an infinite loop
- When getting a new point we generate that point based on the current point we're basing it off of

```gdscript
func _generate_random_point_around(_point: Vector2) -> Vector2:
	var r = randf_range(minDistance, maxDistanceMultiplier * minDistance)
	var angle = randf() * TAU
	return _point + Vector2(cos(angle), sin(angle)) * r
```

- First, let's breakdown the line `var r = randf_range(minDistance, maxDistanceMultiplier * minDistance)`
	- This generates a point (`r`) from the existing point to be placed
	- It gets a point anywhere from the minimum distance to the max distance, which is calculated via the `maxDistanceMultiplier` and the `minDistance`, ensuring it's not too close and not too far
		- **NOTE**: You could always give a specific max distance value instead of just multiplying by the `minDistance` if you want!
- Next, let's breakdown the line `var angle = randf() * TAU`
	- This gets an angle which determines the direction the new point will be placed compared to the existing point
	- TAU is the same as 2π radians (360 degrees), so it gets a random point within a circle
- Lastly, let's breakdown the line `return _point + Vector2(cos(angle), sin(angle)) * r`
	- This gets the new position of the point based on the distance and angle
	- cos/sin gets a vector that points in the correct direction (between -1 and 1)
	- Then we multiply that be `r` to get the desired distance
	- Lastly, we add that offset to the existing point to get the new point location
- Now that we have our point, we have to make sure it's valid

```gdscript
func _is_valid_point(_point: Vector2) -> bool:
	## Check if the point is within chunk bounds
	if _point.x < 0 or _point.x >= chunkSize.x or _point.y < 0 or _point.y >= chunkSize.y:
		return false
	
	var gridPos = _point_to_gridPos(_point)
	
	## Check neighboring cells in the grid
	for i in range(-1, 2):
		for j in range(-1, 2):
			var neighborPos = gridPos + Vector2(i, j)
			if neighborPos.x >= 0 and neighborPos.x < grid.size() and neighborPos.y >= 0 and neighborPos.y < grid[0].size():
				var neighborPoint = grid[neighborPos.x][neighborPos.y]
				if neighborPoint != null:
					if _point.distance_to(neighborPoint) < minDistance:
						return false
	
	return true
```

- First we check if it's within bounds of our chunk
- Then we convert the point to it's location on the grid
- Next we go through 2 for loops to check all the surrounding neighbors
	- We store the grid position to check and make sure it exists within the grid array
	- If it does, we then get the value stored in that location, if it's null then it's valid and we return true, otherwise we check the distances between the points to make sure they area/greater then the minimum distance required and if they're not then we return false
- Alright, now that we have all the points let's continue
- In order to better visualize the program, we can place a bunch of circle meshes at each point

```gdscript
func _draw_points():
	## Draw the generated points as circle mesh instances
	for point in points:
		var newPoint = pointDebug.instantiate()
		self.add_child(newPoint)
		newPoint.global_position = Vector3(point.x - pointOffset.x, 0, point.y - pointOffset.y) ## Add an offset to the location
```

- When placing the point, we may want to add an offset depending on the structure of the world
- Then we can simple call the function `poisson_disc_sampling_algorithm()` in the `_ready()` function and quickly test out different variables to see what fits your needs!

---

And that's it! Now you have a list of points where you can place your obstacles at. There's a few ways to load them in, whether it be adding a function to this script or just referncing the list of points from another script, I'll leave that up to you since it'll vary by game. There are also some other things we could do to expand upon our algorithm:

1. **Support for Multiple Generations:** Let's say you want to use this algorithm to place obstacles but to ALSO place enemies. Your obstacles (like a building) is probably much bigger then an enemy so your distance checks will be different. Adding in an extra system that can check previously placed chunks with it's own distance value on top of the current system is one add-on that can be done.
2. **Storing Previous Runs:** Let's say you mix this with, oh I don't know, a chunk loading system not too different from my own *cough* you can see the tutorial [here](https://minoqi.vercel.app/posts/coordinate-based-chunk-checking-in-godot-4/) *cough*, adding in a system to store the locations of the obstacles so that they can be reloaded when needed can be another QoL feature to add.

Alright, that's it for now. Next week we'll combine that chunk system from [last weeks post](https://minoqi.vercel.app/posts/godot-4-tutorials/coordinate-based-chunk-checking-in-godot-4/) with out poisson disc sampling algorithm! Please look forward to it :) If you'd like to support me you can get the project files [here](https://minoqi.itch.io/poisson-disc-sampling-algorithm-in-godot-4).