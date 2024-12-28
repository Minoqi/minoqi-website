---
title: Modular Stat/Attribute System Tutorial for Godot 4
published: 2024-12-28
description: 'Creating a modular stat/attribute system in Godot 4 with support for temp stats'
image: ''
tags: [Game Programming, Godot, Tutorial]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

I've been working on a lot of games recently like roguelikes and RPGs/JRPGs and one thing they both share is the need for a decently robust stat system. After a bunch of trial and error, trying different systems and researching, I've come up with a system that's modular and easily to build off of. It makes use of 3 scripts total all quite simple, so let's get started.

---

# Godot Tutorial

- First off, we need a system to store our stat value, which we'll do with a `Stat.gd` script
- It inherits from `Resource` and will store the base value, adjusted value and list of stat modifiers
- It'll also have a signal that we can use to send out when the stat has been updated

:::tip 
Using a signal can be real useful for updating UI
:::

```gdscript
extends Resource
class_name Stat

#region Variables
@export var baseValue : float = 0

var statModifiers : Array[StatModifier] = []
var adjustedValue : float = 0
#endregion

#region Signals
signal stat_adjusted(_stat : Stat)
#endregion
```

- You've probably noticed that the array of `statModifiers` is an array of another class called `StatModifier`, let's go over that now
- Since we have no need to access the `StatModifier` information in the inspector, and will only be created via code, it will inherit from the `Object` base class

:::tip
The `Object` base class in Godot is the simplest class and *does not* contain support for the `_ready()` or `_process()` function. It can be great when we need something to act kind of like a resource, but we don't need to be creating files for it in the editor as well as not needing to see the variables from the editor
:::

- The `StatModifier` holds the data related to whatever modifier we need to add to a `Stat`. It includes a few variables:
    1. `value`: The value of the stat modifier
    2. `modifierType`: An enum value that stores what type of calculation the modifier is (in this example we have support for 8 different types, but you can adjust as you need)
    3. `duration`: This is how long the modifier lasts, (so in other words it supports temporary stats!) leave it at 0 for a "permanent" stat
        - "permanent" here just means there's no set duration for the stat to last, it can always still be removed later it's just not dependent on a certain time frame
- We'll also want a signal to let us know when the temporary modifier is over and needs to be removed
- We make use of a `set` function on the duration to know when the stat has finished
- We also have an `initialize()` function that stores the data for the stat modifier which includes the value, modifier type and duration (can be left blank if there's no time limit)

```gdscript
extends Object
class_name StatModifier

#region Variables
enum StatModifierType {
	ADD,
	SUB,
	MULT,
	DIVIDE,
	PERCENT_ADD,
	PERCENT_SUB,
	PERCENT_MULT,
	PERCENT_DIVIDE
}

var value : float = 0
var modifierType : StatModifierType
var duration : float = 0 : set = set_duration
#endregion


#region Signals
signal modifier_over(_modifier : StatModifier)
#endregion


#region SET/GET
func set_duration(_newDuration : float) -> void:
	if _newDuration <= 0:
		duration = 0
		modifier_over.emit(self)
	else:
		duration = _newDuration
#endregion


func initialize(_value : float, _modifierType : StatModifierType, _duration : float = 0) -> void:
	value = _value
	modifierType = _modifierType
	duration = _duration
```

- Now let's go back to our `Stat` script and add in the abiltity to add, remove and calulate stat modifiers
- Firstly we want an `initialize()` function that you call when creating the `Stat`, since anything extending `Object` has no `_ready()` function but we don't necessarily want it to act like a node in a scene and clutter the tree/take extra resources, I had decided to just make an `initialize()` function that has to be called at the first reference of it. All it does is set the `adjustedValue` variable to be equal to the `baseValue` variable
- Next we have the `add_stat_modifier()` which adds the stat to the list of stat modifiers and  it recalculates the stats value
- The `remove_stat_modifier()` does the opposite, removing the stat modifier from the list updating the stats value (no extra overhead for us!)
- There's also `add_temp_stat_modifier()` which does the same as the `add_stat_modifier()` but this connects the signal when the stat modifier is over which calls our `remove_temp_stat_modifier()` which does the same as `remove_stat_modifier()` function but also disconnects the signal
- Last but not least is the `_calculate_stat_modifiers()` function which does as it says, calculates the stats value with all the other modifiers added. This is done *in the order the stat modifiers were applied*
    - Ex. Let's say you have a base stat of 5 and then have a few modifiers [+5, temp -5 for 3 seconds, +5], it will go one by one and apply these stats (so first adding 5, then subtracting 5 then adding 5 resulting in a final value of 10). Once 3 seconds has passed, it will be removed from the list making it looke like this instead [+5, +5] and will recalculate the stat from the beginning (so it does +5 and +5 resulting in 15)
- We use a `match` to determine what formula to use to apply the modifier

```gdscript
func initialize() -> void:
	adjustedValue = baseValue


func add_stat_modifier(_newStatModifier : StatModifier) -> void:
	statModifiers.append(_newStatModifier)
	_calculate_stat_modifiers()


func add_temp_stat_modifier(_newTempStatModifier : StatModifier, _tempStatManager : TempStatManager) -> void:
	statModifiers.append(_newTempStatModifier)
	_newTempStatModifier.modifier_over.connect(remove_stat_modifier)
	_tempStatManager.add_temp_stat(_newTempStatModifier)
	_calculate_stat_modifiers()


func remove_stat_modifier(_modifierToRemove : StatModifier) -> void:
	statModifiers.erase(_modifierToRemove)
	_calculate_stat_modifiers()


func remove_temp_stat_modifier(_modifierToRemove : StatModifier) -> void:
	statModifiers.erase(_modifierToRemove)
	_modifierToRemove.modifier_over.disconnect(remove_stat_modifier)
	_calculate_stat_modifiers()


func _calculate_stat_modifiers() -> void:
	adjustedValue = baseValue
	
	for statModifier in statModifiers:
		match statModifier.modifierType:
			StatModifier.StatModifierType.ADD:
				adjustedValue += statModifier.value
			StatModifier.StatModifierType.SUB:
				adjustedValue -= statModifier.value
			StatModifier.StatModifierType.MULT:
				adjustedValue *= statModifier.value
			StatModifier.StatModifierType.DIVIDE:
				adjustedValue /= statModifier.value
			StatModifier.StatModifierType.PERCENT_ADD:
				adjustedValue += (adjustedValue * statModifier.value) / 100
			StatModifier.StatModifierType.PERCENT_SUB:
				adjustedValue -= (adjustedValue * statModifier.value) / 100
			StatModifier.StatModifierType.PERCENT_MULT:
				adjustedValue *= (adjustedValue * statModifier.value) / 100
			StatModifier.StatModifierType.PERCENT_DIVIDE:
				adjustedValue /= (adjustedValue * statModifier.value) / 100
	
	stat_adjusted.emit(self)
```

- Lastly is the ability to track temporary stats and remove them as needed
- In order to implement this, we'll make a script called `TempStatManager` which inherits from `Node` so we can access the `_process()` function
- This script holds an array of temporary stat modifiers and loops through them in the `_process()` function and subtracts the time from the stats duration. It then checks if the stat is over (so it's duration is 0 or less) and then stores that in an array of stats to remove which is then looped through afterwards and removes the stats from the array. When adding a stat to the list of temporary stat modifiers, it'll check that the duration is greater then 0 otherwise it'll keep going but it'll print an error to let us know a stat is trying to get added to this list that should not be added

```gdscript
extends Node
class_name TempStatManager

#region Variables
var tempStats : Array[StatModifier]
#endregion


func add_temp_stat(_newTempStatModifier : StatModifier) -> void:
	if _newTempStatModifier.duration > 0:
		tempStats.append(_newTempStatModifier)
		return
	
	printerr("ERROR: Tried to add a temp stat modifier to StatManager that was not a temp state modifier!")


# Called every frame. 'delta' is the elapsed time since the previous frame.
func _process(delta):
	if !tempStats.is_empty():
		update_temp_stat_modifiers()


func update_temp_stat_modifiers() -> void:
	var statsToRemove : Array[StatModifier] = []
	
	for tempStat in tempStats:
		tempStat.duration -= get_process_delta_time()
		
		if tempStat.duration <= 0:
			statsToRemove.append(tempStat)
	
	for statToRemove in statsToRemove:
		tempStats.erase(statToRemove)
    
    statsToRemove.clear()
```

- And that's it! This is all the code necessary to function, let's look at an example script using this system

# Example Implementation

- This is an example of this system being implemented in a simple UI scene, where it tracks one stat known as `strength` and updates it's value whenever a button is pressed

:::note
Here the "remove strength" button simple removes the first item in the array of modifiers, but in an actual game you can store the reference to the modifier to then call it specifically to remove if needed
:::

```gdscript
extends CanvasLayer
class_name Example

#region Variables
@export var tempStatManager : TempStatManager
@export var strength : Stat

@export var strengthLabel : Label
@export var addStrengthButton : Button
@export var removeStrengthButton : Button
@export var addTempStrengthButton : Button
#endregion


# Called when the node enters the scene tree for the first time.
func _ready():
	strength.initialize()
	strength.stat_adjusted.connect(_update_strength_label)
	
	addStrengthButton.pressed.connect(_add_strength)
	removeStrengthButton.pressed.connect(_remove_strength)
	addTempStrengthButton.pressed.connect(_add_temp_strength)
	strengthLabel.text = "Strength: " + str(strength.baseValue)


func _add_strength() -> void:
	var strengthStatModifier : StatModifier = StatModifier.new()
	strengthStatModifier.initialize(5, StatModifier.StatModifierType.ADD)
	strength.add_stat_modifier(strengthStatModifier)
	strengthLabel.text = "Strength: " + str(strength.adjustedValue)


func _remove_strength() -> void:
	if strength.statModifiers.is_empty():
		return
	
	strength.remove_stat_modifier(strength.statModifiers[0])
	strengthLabel.text = "Strength: " + str(strength.adjustedValue)


func _add_temp_strength() -> void:
	var strengthStatModifier : StatModifier = StatModifier.new()
	strengthStatModifier.initialize(5, StatModifier.StatModifierType.ADD, 3)
	strength.add_temp_stat_modifier(strengthStatModifier, tempStatManager)
	strengthLabel.text = "Strength: " + str(strength.adjustedValue)
	
	
func _update_strength_label(_stat : Stat) -> void:
	strengthLabel.text = "Strength: " + str(_stat.adjustedValue)
```

---

- And that's it! The code needed and an example of it's implementation. The example is a quick and basic one though, you can easily create a `Resource` to store all the stats for a player so it looks something like this:

```gdscript
extends Resource
class_name StatData

#region Variables
@export var strength : Stat
@export var speed : Stat
@export var attack : Stat
#endregion
```

- There are other adjustments that can be made, like the order of the application of modifiers and how it's tracked but I hope this at least gives you a good headstart that you can build off of if it doesn't fully suite your need

---

# Referenes

[Character Stats Addon on Unity Asset Store by Kryzarel](https://assetstore.unity.com/packages/tools/integration/character-stats-106351)

[Character Stats Unity Tutorial (Youtube Ver) by Kryzarel](https://www.youtube.com/watch?v=SH25f3cXBVc)

[Character Stats Unity Tutorial (Forum Ver) by Kryzarel](https://discussions.unity.com/t/tutorial-character-stats-aka-attributes-system/682458)