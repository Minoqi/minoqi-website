---
title: "Case Study: Recreating Persona 5 Tactica All Out Attack Triangle"
published: 2025-01-01
description: 'A case study about how to recreate the all out attack triangle in Persona 5 Tactica'
image: ''
tags: [Godot, Tutorial, Game Programming]
category: 'Case Study'
draft: true 
lang: ''
---

GIF

Recently I got pretty obsessed with Persona 5 and quickly went to Persona 5 Tactics after finishing it. While playing, it in the middle of a battle I saw the triangle affect that appears when getting ready for an all out attack and thought "huh... I could make that" and now here we are. A few hours later and I've recreated the effect in Godot with a new post for the breakdown of the code :D

---

# Case Study Overview

Before going into how this was implemented let's examine how it's done in Persona 5 Tactica.
1. The fire and filled in triangle only appears when the character that can perform the all out attack is selected

![](src/assets/images/persona5Tactica/persona5TacticaTriangleFire.webp)

2. When a different charcter is selected, the red triangle and fire is gone, instead using blue lines to show no enemies are inside the triangle and red to show that at least one enemy is inside the triangle

![](src/assets/images/persona5Tactica/persona5TacticaTriangleBlue.jpg)

**This case study implements:**
1. The different colored lines based on enemy status and character selected
2. A fire shader and triangle/lines generated via code to represent the triangle on and off fire
3. This example *does not* include the support for multiple levels
    - If anyone can be my tech support and explain how to make the z axis rotate correctly please let me know 😭
4. This example *does not* have the triangle appear *exactly* like in the game, since in the game it seems to render the triangle-fill on the floor of the models so the triangle always appears flat. In this example if the characters are on different y-axis levels then the triangle will no longer appear flat

![](src/assets/images/persona5Tactica/persona5TacticaTriangleFireLevels.jpg)

---

# Implementation

- Alright with all that said let's go over my implementation I came up with
- This project only contains 2 scripts:
    1. `TriangleFormation.gd`: In charge of the code for making the triangle appear
    2. `CharacterManager.gd`: Used to move and switch characters
- There are also 2 other minor scripts to support the animations for the characters but that will not be covered here
- We'll also be making use of a single shader to display our fire
- And with that let's get started, first we'll quickly go over the player controls
- Our `CharacterControls.gd` script will store an array of our different characters, their movement speed, a reference to our `TriangleFormation.gd` and a variable that stores which character in the array we're controlling
- In our `_process()` function we check first if a character has been switched before moving the selected character
- In our `_switch_character()` function we check for a specific input and update the character selected accordingly. We also update the `TriangleFormation.gd` to let it know if our "All Out Attack" character is the one selected. In this case, I've just made it so the first character in the array (Joker) is the one who can do the move. We also redraw the triangle if the character selected has changed
- Secondly is our `_move()` function, it's pretty standard function which again tells the triangle to redraw at the end

:::note
The code for this script was made quickly, I'm not saying this is the best organization method to use for this style of gameplay, heck, I don't even have lerping for the rotations! But it's good enough ¯\\_(ツ)_/¯
:::

```gdscript
extends Node3D
class_name CharacterControls

#region Variables
@export var characters : Array[CharacterManager]
@export var movementSpeed : float = 25
@export var triangleFormation : TriangleFormation

var characterControlling : int = 0
#endregion


# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	_switch_characters()
	_move()


func _move() -> void:
	# Get input
	var direction : Vector3 = Vector3.ZERO
	direction.x = Input.get_action_strength("right") - Input.get_action_strength("left")
	direction.z = Input.get_action_strength("down") - Input.get_action_strength("up")
	direction = direction.normalized()

	if direction == Vector3.ZERO:
		characters[characterControlling].play_idle()
		return

	characters[characterControlling].global_position += direction * movementSpeed * get_process_delta_time()
	characters[characterControlling].play_run()
	
	# Make the character face the direction they are moving
	var new_rotation : Vector3 = Vector3.ZERO
	new_rotation.y = atan2(direction.x, direction.z) # Only need y axis
	characters[characterControlling].rotation = new_rotation
	
	triangleFormation.draw_triangle()


func _switch_characters() -> void:
	if Input.is_action_just_pressed("switch character"):
		if characterControlling + 1 >= characters.size():
			characterControlling = 0
			triangleFormation.characterForFireIsSelected = true
		else:
			characterControlling += 1
			triangleFormation.characterForFireIsSelected = false
	
		triangleFormation.draw_triangle()
```

- Now let's *really* get to the meat and potatoes of the code
- This is gonna be a long one so bear with me as we go through the script
- First let's go over the variables and `_ready()` function:
    1. `meshInstance`: Stores a reference to the mesh that will render the triangle/lines
    2. `triangleColor`: Stores the color of the filled in triangle
    3. `points`: Stores an array to each of the triangles points, which in this case are the three characters
    4. `lines`: Stores an array of each mesh instance which represents an edge of the triangle (this is where our fire shader will be rendered from)
    5. `tolerance`: This is used later on in our algorithm when determining if an enemy is within a triangle, from my tests using a value of 0 is fine but giving a bit of leeway is always a useful variable to have
    6. `fireMaterial`: Stores the material of the fire shader which will be added to the `lines` mesh
    7. `lineColorCharacterSelected/lineColorCharacterNotSelected/lineColorOnFire`: Stores what colors to set the lines to based on the status
    8. `enemies`: Stores references to the enemies
    9. `characterForFireIsSelected`: Status of whether or not the character that can do the "All Out Attack" is selected
- Lastly the `_ready()` function initialize our triangle mesh and draws it

```gdscript
extends Node3D
class_name TriangleFormation

#region Variables
@export var meshInstance : MeshInstance3D
@export var triangleColor : Color = Color.RED
@export var points : Array[CharacterManager]
@export var lines : Array[MeshInstance3D]
@export var tolerance : float = 0
@export var fireMaterial : ShaderMaterial
@export var lineColorCharacterSelected : Color
@export var lineColorCharacterNotSelected : Color
@export var lineColorOnFire : Color
@export var	enemies : Array[Node3D]

var characterForFireIsSelected : bool = true
#endregion


func _ready():
	_initialize_mesh()
	draw_triangle()
```

- Now let's see how the mesh gets initialized first
- First, we want to use an `ImmediateMesh` since we'll need to keep updating it regularly
- Then we need to set up the material for our new mesh, which has a few settings:
    1. `vertex_color_use_as_albedo`: Enable vertex colors in the material
    2. `transparency`: Enable alpha transparency
    3. `cull_mode`: Disable culling so it can be seen at all angles fine
    4. `shading_mode`: We don't want this mesh to be shaded at all

```gdscript
func _initialize_mesh() -> void:
	# Create a new ImmediateMesh
	var mesh = ImmediateMesh.new()
	meshInstance.mesh = mesh
	
	# Create a material that uses vertex colors
	var material = StandardMaterial3D.new()
	material.vertex_color_use_as_albedo = true
	material.transparency = BaseMaterial3D.TRANSPARENCY_ALPHA
	material.blend_mode = BaseMaterial3D.BLEND_MODE_MIX  # Use mix mode for proper blending
	material.cull_mode = BaseMaterial3D.CULL_DISABLED
	material.shading_mode = BaseMaterial3D.SHADING_MODE_UNSHADED
	meshInstance.material_override = material  # Apply the material to the MeshInstance3D
```

- Alright now let's start breaking down our rendering function `draw_triangle()`
    - I'll break down each piece first before showing the entire function at the end
- First we want to determine if any enemies are within the area of the triangle

```gdscript
func draw_triangle() -> void:
	# Check if triangle should be on fire
	var isOnFire : bool = _check_for_enemies_in_triangle()
```

- To do this we're calling another function `_check_for_enemies_in_triangle()`
- First we need to take our list of characters (points of the triangle) and copy the array so that way we can sort it
- Then we want to go through each enemy and check if they're within the triangle
- First we'll convert each characters and the enemies position into a Vector2 since the y-axis plays no roll in our decision, just the x and z axis
- There are multiple ways to determine if something is inside a triangle, but here we'll stay simple and use the *cross product* method
- For each edge of the triangle we'll conduct a cross product of it and the enemies position
- Then we need to make sure that each result has the same sign. Each result can be positive or negative, all that matters is that they're *all* negative or positive, otherwise it means it's *not* within the triangle
- If it is within the triangle we can return true since there's no need to check the other enemies in this scenario

:::tip
If you wanted to include the damage and attacking systems, then you would want to keep going to know what enemies specifically would be taking damage
:::

```gdscript
func _check_for_enemies_in_triangle() -> bool:
	# Create copy of the array to not affect rendering
	var sortedPoints : Array[CharacterManager] = points.duplicate()
	sortedPoints.sort_custom(_compare_triangle_points)
	
	for enemy in enemies:
		# Set variables for easier access
		var point : Vector2 = Vector2(enemy.global_position.x, enemy.global_position.z)
		var trianglePointA : Vector2 = Vector2(points[0].global_position.x, points[0].global_position.z)
		var trianglePointB : Vector2 = Vector2(points[1].global_position.x, points[1].global_position.z)
		var trianglePointC : Vector2 = Vector2(points[2].global_position.x, points[2].global_position.z)
		
		# Determine if the enemy is within each side of the triangle
		var crossProduct01 : float  = (trianglePointB - trianglePointA).cross(point - trianglePointA)
		var crossProduct02 : float  = (trianglePointC - trianglePointB).cross(point - trianglePointB)
		var crossProduct03 : float  = (trianglePointA - trianglePointC).cross(point - trianglePointC)

		# Check if it's inside by taking the results and checking if they're either all negative or positive
		if crossProduct01  >= tolerance and crossProduct02 >= tolerance and crossProduct03 >= tolerance:
			return true
		elif crossProduct01 < tolerance and crossProduct02 < tolerance and crossProduct03 < tolerance:
			return true

	return false
```

- Then we'll want to reset our mesh so we don't render the last frame with the new frame at the same time
- After that we need to check if we need to render the fire and triangle fill or not
- If we don't, then we go through each line and turn them off as well as setting their material to null
- Otherwise, we want to start drawing to a new surface and use the `PRIMITIVE_TRIANGLES` mesh type
- From there we want to loop through each character (point)
- We want to set the normal and uv before moving onto the fire
- If their are enemies within the triangle, then we want to make all line visible, set the fire material to the line and set the color of the triangle mesh

:::note
Since the line and characters (points) are the same size, we don't need to do an extra loop for the lines. While this would break if these values didn't line up, in this case there's always 3 characters so it doesn't matter
:::

- If there are no enemies, then we want to make sure the line is not visible, it's material is set to null and we do not draw the color of the triangle fill
- To finish up our triangle mesh we want to add the point to the vertex

:::tip
We set the normal, uv and color all before adding the vertex since it has to be done before adding the vertex in Godot!
:::

- Now let's move onto drawing the actual fire shader
- The fire shader is done with a quad mesh with a y size of 0.5 and a y center offset of 0.25, the x size of the mesh is changed based on the distance of the two points
- 

```gdscript
	# Begin draw
	meshInstance.mesh.clear_surfaces()
	
	# Draw points of triangle
	if characterForFireIsSelected:
		meshInstance.mesh.surface_begin(Mesh.PRIMITIVE_TRIANGLES)
		
		for i in range(0, points.size()):
			# Draw triangle
			meshInstance.mesh.surface_set_normal(Vector3(0, 0, 1))
			meshInstance.mesh.surface_set_uv(Vector2(0, 0))
			
			if isOnFire:
				lines[i].visible = true
				meshInstance.mesh.surface_set_color(triangleColor)
				lines[i].set_surface_override_material(0, fireMaterial)
			else:
				lines[i].visible = false
				meshInstance.mesh.surface_set_color(Color(0,0,0,0))
				lines[i].set_surface_override_material(0, null)
			
			# Creates vertex and adds the above attributes
			meshInstance.mesh.surface_add_vertex(points[i].global_position)
			
			# Draw fire shader
			var nextPoint : int = i + 1
			if nextPoint >= points.size():
				nextPoint = 0
			
			var direction : Vector3 = points[i].global_position - points[nextPoint].global_position
			var midpoint : Vector3 = (points[i].global_position + points[nextPoint].global_position) / 2
			
			lines[i].global_position = midpoint
			lines[i].look_at(points[i].global_position, Vector3.UP)
			lines[i].global_rotation_degrees.y += 90
			lines[i].mesh.size.x = direction.length()
			
			if isOnFire:
				lines[i].get_surface_override_material(0).set_shader_parameter("texture_scale", lines[i].mesh.size)
			
		#End drawing
		meshInstance.mesh.surface_end()
	else:
		for line in lines:
			line.visible = false
			line.set_surface_override_material(0, null)
```


```gdscript
func draw_triangle() -> void:
	# Check if triangle should be on fire
	var isOnFire : bool = _check_for_enemies_in_triangle()
	
	# Begin draw
	meshInstance.mesh.clear_surfaces()
	
	# Draw points of triangle
	if characterForFireIsSelected:
		meshInstance.mesh.surface_begin(Mesh.PRIMITIVE_TRIANGLES)
		
		for i in range(0, points.size()):
			# Draw triangle
			meshInstance.mesh.surface_set_normal(Vector3(0, 0, 1))
			meshInstance.mesh.surface_set_uv(Vector2(0, 0))
			
			if isOnFire:
				lines[i].visible = true
				meshInstance.mesh.surface_set_color(triangleColor)
				lines[i].set_surface_override_material(0, fireMaterial)
			else:
				lines[i].visible = false
				meshInstance.mesh.surface_set_color(Color(0,0,0,0))
				lines[i].set_surface_override_material(0, null)
			
			# Creates vertex and adds the above attributes
			meshInstance.mesh.surface_add_vertex(points[i].global_position)
			
			# Draw fire shader
			var nextPoint : int = i + 1
			if nextPoint >= points.size():
				nextPoint = 0
			
			var direction : Vector3 = points[i].global_position - points[nextPoint].global_position
			var midpoint : Vector3 = (points[i].global_position + points[nextPoint].global_position) / 2
			
			lines[i].global_position = midpoint
			lines[i].look_at(points[i].global_position, Vector3.UP)
			lines[i].global_rotation_degrees.y += 90
			lines[i].mesh.size.x = direction.length()
			
			if isOnFire:
				lines[i].get_surface_override_material(0).set_shader_parameter("texture_scale", lines[i].mesh.size)
			
		#End drawing
		meshInstance.mesh.surface_end()
	else:
		#meshInstance.mesh.surface_set_color(Color(0,0,0,0))
		for line in lines:
			line.visible = false
			line.set_surface_override_material(0, null)
	
	# Draw lines
	meshInstance.mesh.surface_begin(Mesh.PRIMITIVE_LINES)
	for i in range(0, lines.size()):
		var nextPoint : int = i + 1
		if nextPoint >= lines.size():
			nextPoint = 0
		
		if characterForFireIsSelected and isOnFire:
			meshInstance.mesh.surface_set_color(lineColorOnFire)
		else:
			if !isOnFire:
				meshInstance.mesh.surface_set_color(lineColorCharacterSelected)
			else:
				meshInstance.mesh.surface_set_color(lineColorCharacterNotSelected)
		
		meshInstance.mesh.surface_add_vertex(points[i].global_position)
		meshInstance.mesh.surface_add_vertex(points[nextPoint].global_position)
		
		# End drawing
	meshInstance.mesh.surface_end()
```

---

**Resources**

[Persona 5 Models](https://sketchfab.com/oscar3dmodel/models)

[Godot Model](https://captainripley.itch.io/godot-3d-robot-character)

[Base Fire Shader](https://godotshaders.com/shader/fire-shader-for-godot-engine-4/)