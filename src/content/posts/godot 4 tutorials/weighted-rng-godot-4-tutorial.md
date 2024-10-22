---
title: Weighted RNG Tutorial in Godot 4
published: 2024-10-23
description: ''
image: ''
tags: [Algorithm, Godot, Tutorial]
category: 'Godot 4 Tutorial'
draft: false 
lang: ''
---

![](src/assets/images/godot_4_tutorials/WeightedRNGOutcomeGodot4.png)
*An image showing the outcome of a weighted RNG done 100 times, with common getting generated 71 times. uncommon 21 times and rare 8 times*

# Why Weighted RNG?

Using weighted RNG can be useful when you don't want the odds of something to happen to all be equal. For example, the most common application is rogueliles, with their different rarity levels for upgrades but it can also be used in games like pokemon for generating the chance of encountering a certain pokemon

---

# Godot
- First, we'll need a dictionary with our item and it's weight
	- ***Remember!*** The higher the number the more likely it is to get chosen
- First, let's look at just a normal dictionary setup

```gdscript
@export var weightedItems : Dictionary = {
	"ITEM": 12,
	"ITEM2": 45,
	"ITEM3": 65
}
```

- Of course, the key is whatever the item is whether it be a string, resource, enum etc.
- But what if we're making an upgrade system where we have a set upgrade chances? Like common, uncommon, rare etc. It might be annoying to set the value for each upgrade especially if you decide to change the numbers later. In this case we can create the dictionary at runtime.
- This set up includes 3 types of variables:
	1. Values to hold the chances value
	2. Arrays to hold the items connected to each chance
	3. Our weighted dictionary (generated)

```gdscript
extends Node

#region Variables
@export var commonChance : int = 60
@export var uncommonChance : int = 30
@export var rareChance : int = 10
@export var commonItems : Array
@export var uncommonItems : Array
@export var rareItems : Array

var weightedDict : Dictionary = {}
#endregion


func _ready():
	weightedDict = WeightedRNGGenerator.generated_weighted_dictionary(commonItems, commonChance)
	weightedDict.merge(WeightedRNGGenerator.generated_weighted_dictionary(uncommonItems, uncommonChance))
	weightedDict.merge(WeightedRNGGenerator.generated_weighted_dictionary(rareItems, rareChance))
```

```gdscript
extends Node

func generate_weighted_dictionary(_items : Array, _weightedValue : float) -> Dictionary:
	var weightedDict : Dictionary = {}
	
	for item in _items:
		weightedDict[item] = _weightedValue
	
	return weightedDict
```

- This will create a weighted dictionary at runtime for us and it's scalable :D
- As you can see it takes up two scripts, one that holds all our calculations for the weighted rng and one that holds all the upgrade data. This makes it easily reusable, compared to if we stored the weighted values in the weighted rng script
- Now let's create the weighted rng algorithm:

```gdscript
extends Node
class_name WeightedRNGGenerator

#region Variables
static var rng : RandomNumberGenerator = RandomNumberGenerator.new()
#endregion

...

static func generate_item(_weightedDict : Dictionary):
	## Calculate the total weights
	var totalWeights : float = 0
	for key in _weightedDict:
		totalWeights += _weightedDict[key]
	
	var keyGenerated : bool = false
	while !keyGenerated:
		## Generate a random weight
		var randomWeight : float = rng.randi_range(0, totalWeights)
		
		## Pick a random item based on the random weight
		for key in _weightedDict:
			randomWeight -= _weightedDict[key]
			
			if randomWeight < 0:
				keyGenerated = true
				return key
		
		print("NO KEY MADE CHOSEN: REPEAT")
```

- Now let's go over the algorithm
- First we create a variable to store the random number generator at the start of the game
- Then in our function `weightedRNG`, we take in the weighted dictionary we made
- We loop through this dictionary to calculate all the weights together
- Then we generate a random weight between 0 and the `totalWeights`
- Lastly, we loop through each item in the dictionary, subtracting it's weight from the random weight and if we go below zero we return that item in the dictionary, hence why we're using a `foreach` for the keys of the dictionary rather then the values
- We wrap this up in a while loop, since sometimes you may not get a key which will cause an error, so we want it to loop until it does return a key, which almost always happens the second time.
	- Technically, the `keyGenerated` is unnecessary, but I like to have it *just* to be safe
- And that's it! You can take the key it returned and use it however you need.
- *BUT WAIT!* This is great but it seems a pain for testing, how can I calculate the expected chances of everything? That's easy, lets make a function to for that:

```gdscript
extends Node

#region Variables
#endregion

...

...

func calculate_expected_chance(_weight : float, _weightedDict : Dictionary, _totalWeights : float = -1) -> void:
	var totalWeights : float = 0
	if _totalWeights == -1:
		for key in _weightedDict:
			totalWeights += _weightedDict[key]
	else:
		totalWeights = _totalWeights
	
	var expectedChance : float = (_weight / totalWeights) * 100
	print("EXPECTED CHANCE FOR WEIGHT ", _weight, " IS ", expectedChance)


func calculate_every_expected_chance(_weightedDict : Dictionary) -> void:
	var totalWeights : int = 0
	for key in _weightedDict:
		totalWeights += _weightedDict[key]
	
	for key in _weightedDict:
		calculate_expected_chance(_weightedDict[key], _weightedDict, totalWeights)
```

- As you can see, we made two functions. One to calculate the expected chance and another that will do it for every item in the dictionary. The two functions give us more freedom between just wanting to get the chance of a specific item or for all of them :)
- To calculate the `expectedChance`, we simply add up all the total weights again before dividing the `_weight` we want to check by the `totalWeights` and lastly multiple it by 100
- We also added in a check, since if we want to calculate the weights for *everything*, then we don't need to be recalculating the weight every time, so instead we check if we've been given a `totalWeights` already and if we do skip recalculating weights.
- Hm, I noticed we need to get the `totalWeights` in these functions and the one from earlier, I'll turn that into it's own function for easy reuse

```gdscript
extends Node

#region Variables
#endregion

...

func generate_item(_weightedDict : Dictionary):
	## Calculate the total weights
	var totalWeights : float = calculate_totale_weights(_weightedDict)

	var keyGenerated : bool = false
	while !keyGenerated:
		## Generate a random weight
		var randomWeight : float = rng.randi_range(0, totalWeights)
		
		## Pick a random item based on the random weight
		for key in _weightedDict:
			randomWeight -= _weightedDict[key]
			
			if randomWeight < 0:
				keyGenerated = true
				return key
		
		print("NO KEY CHOSEN: REPEAT")

func calculate_expected_chance(_weight : float, _weightedDict : Dictionary, _totalWeights : float = -1) -> void:
	var totalWeights : float = 0
	if _totalWeights == -1:
		totalWeights = calculate_total_weight(_weightedDict)
	else:
		totalWeights = _totalWeights
	
	var expectedChance : float = (_weight / totalWeights) * 100
	print("EXPECTED CHANCE FOR WEIGHT ", _weight, " IS ", expectedChance)

func calculate_every_expected_chance(_weightedDict : Dictionary) -> void:
	var totalWeights : float = calculate_total_weight(_weightedDict)
	for weight in _weightedDict:
		calculate_expected_chance(_weightedDict[weight], _weightedDict, totalWeights)

func calculate_total_weight(_weightedDict : Dictionary) -> float:
	var totalWeights : float = 0
	for key in _weightedDict:
		totalWeights += _weightedDict[key]
	return totalWeights
```

- Lastly, make sure to add this to your autoloads to be able to access it form anywhere
- Now with it all done, here's an example output. This example takes into account the rare items as makes sure you can't have a rare item get chosen again if it already had been chosen in the last 5 generations:

```text
MADE WEIGHTED DICTIONARY: { "common1": 60, "common2": 60, "common3": 60, "uncommon1": 30, "uncommon2": 30, "rare1": 10, "rare2": 10, "rare3": 10 }
EXPECTED CHANCE FOR WEIGHT 60 IS 22.2222222222222
EXPECTED CHANCE FOR WEIGHT 60 IS 22.2222222222222
EXPECTED CHANCE FOR WEIGHT 60 IS 22.2222222222222
EXPECTED CHANCE FOR WEIGHT 30 IS 11.1111111111111
EXPECTED CHANCE FOR WEIGHT 30 IS 11.1111111111111
EXPECTED CHANCE FOR WEIGHT 10 IS 3.7037037037037
EXPECTED CHANCE FOR WEIGHT 10 IS 3.7037037037037
EXPECTED CHANCE FOR WEIGHT 10 IS 3.7037037037037
GENERATED ITEM: common1
GENERATED ITEM: common1
GENERATED ITEM: uncommon1
GENERATED ITEM: common1
GENERATED ITEM: common2
GENERATED ITEM: common3
GENERATED ITEM: rare3
GENERATED ITEM: uncommon1
GENERATED ITEM: common2
GENERATED ITEM: common1
GENERATED ITEM: uncommon2
GENERATED ITEM: uncommon1
GENERATED ITEM: common2
GENERATED ITEM: common1
GENERATED ITEM: common3
GENERATED ITEM: common2
GENERATED ITEM: common1
GENERATED ITEM: common2
GENERATED ITEM: uncommon2
GENERATED ITEM: common2
GENERATED ITEM: common2
GENERATED ITEM: rare2
REGENERATED; FAILED MULTI RARE CHANCE CHECK
REGENERATED; FAILED MULTI RARE CHANCE CHECK
GENERATED ITEM: uncommon2
GENERATED ITEM: common2
GENERATED ITEM: common2
GENERATED ITEM: common1
...
```

---

- Okay, nice! This could still be improved in multiple ways, but here's a few ideas if you want to continue to enhance it to suit your specific needs
	1. **More Accurate Expected Chances:** Right now this just calculates the `expectedChance` once, but if you want a more holistic view, you could create a function that runs it multiple times and gets the average of that value
	2. **Optimization:** This current solution uses an O-Notation of O(N) (where N is the number of items in the dictionary). For most cases this is fine. But through the use of a *binary search* instead of a *linear scan* (like what we're currently doing), you can get it to O(log(N)). This should be something to consider if you're dealing with 100s to 1000s of weights but otherwise it's fine. You could also use the *Hopscotch algorithm* by Bruce Hill, which is even better. Or for even MORE performance, you can use the *Aliased algorithm* (? not sure that's the official name) that was made by A.J. Walker in 1974. I'll include a link to Bruce Hills post that includes more detail if you're interested, it's a great reference! Maybe I'll look into optimizing this in future posts 👀
	3. **Add Extra Checks:** Maybe you want a hard rule where you can only get a rare upgrade again after 5 loops. You can go in and add extra checks that takes this into account. One way is to add it to the `weightedRNGGenerator` or a better way to keep it modular, is to add a check to where you're calling the generation and checking all your checks from there, if any return false then you just run the generation again. *This can mess with how the rng feels for the player, so use it with caution!*

---

## References

[Harvey Limbo Medium Post](https://limboh27.medium.com/implementing-weighted-rng-in-unity-ed7186e3ff3b)

[Bruce Hill Blog Post](https://blog.bruce-hill.com/a-faster-weighted-random-choice)