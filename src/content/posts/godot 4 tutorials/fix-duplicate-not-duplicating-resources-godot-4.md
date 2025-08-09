---
title: FIX duplicate() Not Copying Variable Values from Resources Godot 4
published: 2025-03-26
description: 'This is a fix and detailed breakdown of the code needed to fix GOdots suplicate() issue where it does not copy over values properly'
image: ''
tags: [Game Programming, Godot, Tutorial]
category: 'Godot 4 Tutorials'
draft: false
lang: ''
---

# Why use a custom `duplicate()` function over Godot's?
Recently, I ran into an error when trying to duplicate a custom resource that had subresources attatched to it. Neither the subresources or normal varibles would duplicate, it would just created a new file but didn't copy any of the values. After some googling, I found [this github issue](https://github.com/godotengine/godot/issues/37222) from Godot 3. After some scrolling I saw a response about how to make your own duplicate function to fix this issue. I'll be breaking down the code and how it works here!

---

# Tutorial
- We'll need a single script, which I called `CustomDuplicate`
- We just need a single function, which you can name whatever you'd like, I've called mine `deep_clone()`
- Our function will take in an `Object` (which is what we'll want to clone) and return an `Object` (the final copy)
- First, we use godots built-in `duplicate()` function
- Then, we'll loop through all the properties in the clone, and set them equal to their values in the original
    - This overrides every variable, making sure that none get missed from the original `duplicate()` call
- We might be done here in some cases, but in the case we're duplcating a node with children, we'll copy those too
- After making sure the `_object` is a `Node`, we'll remove all the children on the `clone`
- Once we've remove all the children, we'll use an `assert` to make sure they did in fact all get removed
- Lastly, we'll get all the children from the `object` and call `deep_clone` (to make sure they get copied properly too) on all of them before adding them as a child to our `_object`
- And of course return our `clone` at the end

```gdscript
## Code from https://github.com/godotengine/godot/issues/37222
extends Node
class_name CustomDuplicate

static func deep_clone(_object_ : Object) -> Object:
	## Create a shallow copy
	var clone : Object = _object_.duplicate()

	## get_property_list() returns an array of all the properties
	## loop through all the properties and store their values
	## makes sure any data the duplicate() misses gets copied over
	for property in _object_.get_property_list():
		var propertyName = property["name"]
		clone.set(propertyName, _object_.get(propertyName))

	## properly duplicate all children nodes 
	if _object_ is Node:
		## delete any previously cloned children
		for child in clone.get_children():
			clone.remove_child(child)
			child.free()

		## make sure all children were removed
		assert(clone.get_child_count() == 0)

		## go through each child and deep clone them
		for child in _object_.get_children():
			clone.add_child(deep_clone(child))

	return clone
```

## Why is it a `singleton` instead of an `autoload`?
An autoload will create a node for it inside the scene tree, but we have no need for a node inside the tree, we just need access to the function so using a `static` function is enough and aviods unecessary nodes.

---

And that's it! Since this is a static function, we can call it anywhere in our codebase like an `autoload` by doing `var duplicateObject : Object =  CustomDuplicate.deep_clone(objectHere)`.

---
## References
[Solution from Github Issue](https://github.com/godotengine/godot/issues/37222)