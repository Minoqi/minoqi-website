---
title: Using Quick Sort to Determine Player Order
published: 2024-12-30
description: 'Using the quick sort algorithm to set the turn order in a game'
image: ''
tags: [Game Programming, Algorithm, Godot, Tutorial]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

![](src/assets/images/quickSortDiagram.png)

Recently I've been working on a turn-based jrpg game in Godot 4 for a client similar to that of **Fire Emblem** series. I've learned a lot through this project, and one of them included how to determine the move order efficiently through the use of the quick sort algorithm. First let's go over some important terminology.

---

# What is Quick Sort?
*Quick sort* is one of the main sorting algorithms you would learn in a data structures and algorithms class. It's considered to usually be one of the fastest. It uses a divide-and-conquer strategy by constantly splitting the array. It determines a pivot point, then sorts all the smaller values on the left and largest on the right, lastly rinse and repeat until it's all sorted.

:::warning
However, in our case we'll want our sorting to be done greater to least, so the greatest will be on the left and lowest on the left. This is because we're using it for our initiative order, so the highest initiative should go first. Of course, you could always still sort as normal and then just reverse the array if you prefer.
:::

---

# What is a "recursive" function?

A recursive function is a function that calls itself, essentially creating a while loop

:::warning
Make sure every recursive function you make has some sort of check to know when to exit the loop, otherwise you've got an infinite loop in your hands!
:::

---

# What does "partition" mean?

It just means splitting (in this case an array) into two, and in this case into multiple parts

---

# Godot Tutorial

- Before we go over the code there's some pre-reqs about my code to understand:
    1. Each character on the board has a `GridObjectCharacter` script attached, which while containing other things the only thing that matters is it contains their initiative/movement score
    2. I have a variable called `initiativeOrder` in the script that contains the order for the characters to play in
    3. In this scenario, the turn order applies to *all* the characters, not just the enemies like in games like **Fire Emblem**, although the code would still apply in that context, just only do it with the enemies and not the player characters
    3. This is all done in one script
- Okay with that out of the way let's get this started
- Firstly, we have the simplest funtion which takes two lines. One calls the quick sort algorithm and the other updates the UI accordingly once it's done
- You can see we pass in the `initiativeOrder` variable since that's what we want sorted. All the charactesr relevant to the turn order have already been added to it in the `_ready()` function

```gdscript
func calculate_turn_order() -> void:
	quickSort(initiativeOrder)
	gridCombatUI.initialize(initiativeOrder)
```

- Now let's look at the `quickSort()` function
- It takes in a few values, including the array to sort, the low value and high value which default to 0 and the size of the array if nothing is passed in
- First we check to make sure the low value is smaller than the high value, otherwise we're done
- Then inside our if statement, we find the new pivot point based on the data we pass in and then do it again for the left and right sides until we finish

```gdscript
func quickSort(_characterArray : Array[GridObjectCharacter], _low : int = 0, _high : int = _characterArray.size() - 1) -> void:
	if _low < _high:
		## Find each point so that the smaller elements are on the left and larger on the right of said point
		var point : int = partition(_characterArray, _low, _high)
		
		## Recrusive call for left and right sides
		quickSort(_characterArray, _low, point - 1)
		quickSort(_characterArray, point + 1, _high)
```

- Our `partition()` function takes in the same values as our `quickSort()` function
- We want to take in the pivot point value, which in our case is the characters initiative stat
- Then we want to get the greater elements ID which keeps track of the position of the last element that is greater then or equal to the pivot
- We have two variables `previousGreaterElementValue` and `newGreaterElementValue` which simply stores values as we move things around
- Now we can start our for loop, which compares each element with the pivot point
- If the initiative is greater then or equal to the pivot point we want it to move to the left (remember, we want the highest values first and lowest last)
- First we'll increase the `greaterElementID` for the next interation
- Then we want to store the current `greaterElementID` value and the value at `i` so we can swap them right after
- Once our loop is done, we want to swap the pivot point with the element right after the `greaterElementID` element, this makes sure the pivot point is placed in the correct position
- Finally we return the final location of the pivot point which is then used to re-split the array and continue the quick sort

```gdscript
func partition(_characterArray : Array[GridObjectCharacter], _low : int, _high : int) -> int:
	var pivotPoint : int = _characterArray[_high].stats.initiative
	var greaterElementID : int = _low - 1 ## Keeps track of the position of the last element that is greater than or equal to the pivot

	var previousGreaterElementValue : GridObjectCharacter = null
	var newGreaterElementValue : GridObjectCharacter = null

	## Compare each element with pivot point
	for i in range(_low, _high):
		## Swap elemenets if it's larger then the pivotPoint
		if _characterArray[i].stats.initiative >= pivotPoint:
			greaterElementID += 1
			
			previousGreaterElementValue = _characterArray[greaterElementID]
			newGreaterElementValue = _characterArray[i]
			_characterArray[greaterElementID] = newGreaterElementValue
			_characterArray[i] = previousGreaterElementValue
	
	## Swap pivot point with the greater element specified by greaterElementID
	previousGreaterElementValue = _characterArray[greaterElementID + 1]
	newGreaterElementValue = _characterArray[_high]
	_characterArray[greaterElementID + 1] = newGreaterElementValue
	_characterArray[_high] = previousGreaterElementValue
	
	## Return where partition is done
	return greaterElementID + 1
```

- Last but not least for those curious, this is how I'm creating the UI
- I simply loop through and instantiate my UI prefab and set it's values appropriately, adding it as a child to a vbox container

```gdscript
extends CanvasLayer
class_name GridCombatUI

#region Variables
@export var initiativePortraitPrefab : PackedScene
@export var initiativeTower : TextureRect
@export var initiativeList : VBoxContainer

var portraits : Array[InitiativePortrait] = []
#endregion


func initialize(_initialOrder : Array[GridObjectCharacter]) -> void:
	for character in _initialOrder:
		var newPortrait : InitiativePortrait = initiativePortraitPrefab.instantiate()
		initiativeList.add_child(newPortrait)
		newPortrait.initialize(character.stats.characterName, character.stats.initiative)
```

---

**References**

[Quick Sort Algorithm Photo & Explanation](https://workat.tech/problem-solving/tutorial/sorting-algorithms-quick-sort-merge-sort-dsa-tutorials-6j3h98lk6j2w)