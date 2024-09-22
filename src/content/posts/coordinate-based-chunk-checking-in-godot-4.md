---
title: 'Coordinate-Based Chunk Checking in Godot 4'
published: 2024-09-22
description: 'A system to check what chunk a character is on efficiently, only requiring one calculation independent from the number of chunks.'
image: ''
tags: [Tutorial, Godot, Game Programming]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---
![](src/assets/images/CoordinateBasedChecksDemoGIF.gif)

# What Is Coordinate-Based Checking?
A system to check what chunk a character is on efficiently, only requiring one calculation 
independent from the number of chunks.

# Why Use This Over Distance Checking?
- **Constant Time Complexity:**
	- **Coordinate-Based Checking:** Using this method uses a time of O(1) meaning it takes a constant amount of time regardless of the amount of chunks to check
	- **Distance Checking:** Checking the distance to each chunk would require calculating the distance of the player to every single chunk, giving a time of O(n), n being the number of chunks
- **Fewer Calculations:**
	- **Coordinate-Based Checking:** Only uses simple arithmetic, no need to calculate multiple distances
	- **Distance Checking:** Each distance involves multiple operations: subtracting coordinates, squaring differences, summing them and then taking a square root. Doing this for more and more chunks quickly adds up
- **Memory Efficient:**
	- **Coordinate-Based Checking:** This method only requires memory of the objects position and chunk size, one of which is already store in itself, no additional data like distances to each chunk are needed
	- **Distance Checking:** You most likely need to store at least one variable and run loops to check distances, using up more memory

# Why Use This Over Collision Checking?
- **Constant Time Complexity:**
	- **Coordinate-Based Checking:** Using this method uses a time of O(1) meaning it takes a constant amount of time regardless of the amount of chunks to check
	- **Collision Checking:** Collision detection checks the player's position against the boundaries of the chunks, which can involve multiple comparisons per frame, as well as having to fix any overlapping issues as well, which can end up in O(n) where n is the number of boundaries being checked.
- **Precision:**
	- **Coordinate-Based Checking:** Since this calculates the objects position directly to find the chunk, making it very precise
	- **Collision Checking:** Collision checks can be prone to issues like tunneling (where the object moves too fast and "skips" the boundary) which can lead to inaccurate collisions
- **Overhead:**
	- **Coordinate-Based Checking:** This method is lightweight with minimal overhead
	- **Collision Checking:** Physics collisions can have a a lot of overhead depending on the use case and implementation
- **Game Specific Needs:**
	- **Coordinate-Based Checking:** Ideal for games that can have a grid layout
	- **Collision Checking:** Might be better (or distance checking) for odd shaped games where a grid approach doesn't work

# Godot Tutorial
- In order to recreate this in Godot, we'll need a few scripts:
	1. `ChunksManager.gd`
	2. `Chunk.gd`
	3. `ChunkChecker.gd`
- Let's start by taking a look at the `ChunksManager`
- The `ChunksManager` updates all the chunks accordingly as needed
- First let's look at the variables:
```gdscript
extends Node3D
class_name ChunksManager

# Variables
@export var startingPos : Vector2 = Vector2.ZERO ## Starting position for the center chunk
@export var chunkSize : Vector2i
@export var chunkGridLocations : Array[Vector2i] = [Vector2i(-1,1), Vector2i(0,1), Vector2i(1,1), Vector2i(-1,0), Vector2i(0,0), Vector2i(1,0), Vector2i(-1,-1), Vector2i(0,-1), Vector2i(1,-1)] ## Locations in grid for all the chunks
@export var chunks : Array[Chunk]
@export var player : Player

var chunkGrid : Dictionary = {}
```
- `startingPos`: Starting position for the center chunk
- `chunkSize`: The size of the chunks
- `chunkGridLocations`: All the initial locations on the grid, doesn't have to be a perfect square!
- `chunks`: All the chunks in the world effected
- `chunkGrid`: A dictionary that actually stores the grid locations (the key) and what chunks are on the location (the value)
- Alright, now let's look at the `_ready()` function:
```gdscript
func _ready():
	## Make sure chunkLocations and chunks are the same length
	assert(chunks.size() == chunkGridLocations.size(), "ERROR: Not enough chunks to fit the grid, or too many!")
	
	player.chunkChecker.initialize(chunkSize)
	player.chunkChecker.update_chunk_grid_position.connect(_update_chunks)
	
	_initialize_chunks()
```
- Then we connect some information with the player, we'll go over this later!
- Lastly we initialize our chunks:
```gdscript
func _initialize_chunks() -> void:
	for i in range(0, chunks.size()):
		chunkGrid[chunkGridLocations[i]] = chunks[i]
		chunks[i].update_chunk(chunkGridLocations[i], chunkGridLocations[i] * chunkSize)
```
- We want to loop through all ours chunks (or chunk locations, remember they're the same size so it doesn't matter!) and then add the location and chunk to the `chunkGrid`
- Afterwards we want to update the chunk so it's position and chunk ID is updated:
```gdscript
extends Node3D
class_name Chunk

# Variables
var chunkID : Vector2i = Vector2i.ZERO

func update_chunk(_id : Vector2i, _targetPos : Vector2) -> void:
	chunkID = _id
	global_position = Vector3(_targetPos.x, global_position.y, _targetPos.y)
```
- This is all the code our `Chunk` needs! A way to update the ID of the current chunk it's on and to update the chunks position
- Now let's go back to our `ChunkManager` and check how to update the chunks:
```gdscript
func _update_chunks(_enteredChunkID : Vector2i) -> void:
	var chunksToUpdate : Array[Vector2i] = []
	var newChunks : Array[Vector2i] = [
		_enteredChunkID, ## Center chunk
		Vector2(_enteredChunkID.x + 1, _enteredChunkID.y + 1),
		Vector2(_enteredChunkID.x - 1, _enteredChunkID.y + 1),
		Vector2(_enteredChunkID.x + 1, _enteredChunkID.y - 1),
		Vector2(_enteredChunkID.x - 1, _enteredChunkID.y - 1),
		Vector2(_enteredChunkID.x, _enteredChunkID.y + 1),
		Vector2(_enteredChunkID.x, _enteredChunkID.y - 1),
		Vector2(_enteredChunkID.x + 1, _enteredChunkID.y),
		Vector2(_enteredChunkID.x - 1, _enteredChunkID.y)
	]
	
	## Store out of bounds chunks
	for chunk in chunks:
		if !newChunks.has(chunk.chunkID):
			chunksToUpdate.append(chunk.chunkID)
	
	## Update out of bounds chunks to new position
	for newChunkID in newChunks:
		if !chunkGrid.has(newChunkID):
			var oldChunk : Vector2i = chunksToUpdate[0]
			chunkGrid[newChunkID] = chunkGrid[oldChunk]
			chunkGrid[newChunkID].update_chunk(newChunkID, newChunkID * chunkSize)
			chunkGrid.erase(oldChunk)
			chunksToUpdate.erase(oldChunk)
	
	chunksToUpdate.clear()
```
- First we want to create an array with all the new positions with the chunk the player hit acting as the center
- Then we want to go through each of the current chunks and store any chunk that is not within the new chunk grid locations we need
- Then we loop through all the new chunk grid locations, checking to make sure the chunk isn't already there, and then take the first chunk from the `chunksToUpdate` and update it's ID to the new spot we need it in
- Alright, now let's move onto our `ChunkChecker`!
```gdscript
# Variables
@export var actor : Node3D

var chunkSize : Vector2i = Vector2i.ZERO
var chunkGridPos : Vector2i = Vector2.ZERO : set = _set_chunkGridPos
```
- We want to store the actor, so we can get it's position
- We need to store the size of the chunks and it's current position on the chunk grid
- You'll notice a *set* function, we'll come back to that!
- Firstly, we want to make sure we have a function we can call to store the chunk size:
```gdscript
func initialize(_chunkSize : Vector2i) -> void:
	chunkSize = _chunkSize
```
- This gets called back in the `ChunksManager`!
```gdscript
player.chunkChecker.initialize(chunkSize) ## This line from the _ready() function!
```
- Next let's look at the code that actually checks for the chunk it's in:
```gdscript
# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	_check_chunk_grid_position()


func _check_chunk_grid_position() -> void:
	var updatedChunkGridPosition : Vector2i = Vector2i(round(actor.global_position.x / chunkSize.x), round(actor.global_position.z / chunkSize.y))
	if chunkGridPos != updatedChunkGridPosition:
		chunkGridPos = updatedChunkGridPosition
```
- We simply want to use the formula: `actors position / chunk size` and make sure to `round()` it! This gets us our position in the chunk grid
- If the chunk grid position is different then it was last frame, we'll want to update our `chunkGridPosition`
```gdscript
# Signals
signal update_chunk_grid_position(_newChunkGridPos : Vector2i)


# SET/GET
func _set_chunkGridPos(_newChunkGridPos) -> void:
	chunkGridPos = _newChunkGridPos
	update_chunk_grid_position.emit(chunkGridPos)
```
- Updating that variable will call our *set* function! This will emit a signal that the `ChunksManager` is connected to to let it know to update the chunks based on the chunk the actor is on
```
player.chunkChecker.update_chunk_grid_position.connect(_update_chunks) ## This line from the _ready() function!
```
- And that's it! You can now run around and the chunks will follow. 
- **NOTE**: This system can support other necessities, like maybe having an array that holds all the enemies on the chunk. You can add something like this by simply adding a `bool` in the `ChunkChecker` that marks if it's an enemy or player, and send a different signal of it's an enemy to the `ChunkManager`

And that’s it! This was made with Godot 4.3 but it should be easy to update any syntax changes Godot makes in the future or for using older versions. You can support me by getting the project files off of my [itchio](https://minoqi.itch.io/coordinate-based-chunk-loading-for-godot-4)! Subscribe for next weeks post about the ***poisson disc sampling algorithm*** which we can use to place obstacles randomly in our chunks!