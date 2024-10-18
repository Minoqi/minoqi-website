---
title: Utility AI Tutorial for Godot 4
published: 2024-10-18
description: 'Creating a simple yet modular Utility AI in Godot 4'
image: ''
tags: [AI, Game Programming, Godot, Tutorial]
category: 'Godot 4 Tutorials'
draft: false 
lang: ''
---

![](src/assets/images/godot_4_tutorials/UtilityAIGodotExample.gif)

**What is the Utility AI?**

*Utility AI* is a type of AI that takes multiple factors into account when considering a list of actions an AI could do. It averages the value of all the factors to the get the utility score for that action and the action with the highest value is what the AI should do next.

*First, let's go over some terminology:*
1. **Utility Decision:** The final decision the AI makes
2. **Utility Action:** A possible action the AI could take
	1. Ex. Walk, Idle, Eat, Sleep, Attack etc.
3. **Utility Consideration/Factor:** Some sort of factor the action considers when deciding on it's value
	1. Ex. An attack action may have a consideration for the players health, the enemies health, how much damage the enemy can do, how much damage the player can do etc.
	2. Ex. A walk action on a tile based game may have a consideration for how far the player is, the total cost of the movement compared to how much the AI can move etc.
4. **Utility Aggregate:** This just refers to how the values are calculated. Most commonly it's done with multiplication, both other methods like addition, subtraction, division etc. can also be used
5. **Curves:** Curves are often used when calculating the actions value, with curves allowing for more then just a linear approach.
	1. Ex. When deciding if an AI should sleep, you may not want to consider the sleep action at all until they're sleep score is below 20. In this case you can use a curve so the sleep action will be set to a score of 0 (and therefore ignored) until the sleep score goes below 20, which it will then return a value based on the sleep score and the curve

:::warning[WARNING: Utility Scores]
When creating your utility scores, you **MUST** make sure the scale is the same across the board. If the scale is different, then the algorithm will not work correctly. This is why *normalizing* (converting a value to be between 0-1) is quite common (but of course not mandatory).
:::

---

# Godot Tutorial
- First, let's look at the list of scripts we'll need
	- `UtilityDecision.gd`
		- This will provide us with the final decision on what the best action is as well as our calculation methods
	- `UtilityAction.gd`
		- This will represent an action our AI can take
	- `UtilityConsideration.gd`
		- This will be the base class that's used for all the sub classes
	- `UtilityAggregrate.gd`
		- This will allow use to calculate one set of considerations differently from the rest
	- `UtilityFactor.gd`
		- This will be the base class our `UtilityAggregrate` and `UtilityConsideration` classes
- First let's start with the `UtilityFactor.gd`
- This scripts simply acts as a base class for others to inherit from
- It stores a reference to the `UtilityDecision` node in our scene as well as a function called `calculate_factor_score`
```gdscript
extends Node
class_name UtilityFactor

var utilityDecision : UtilityDecision

func calculate_factor_score() -> float:
	print("WARNING: Called factor calculation from base class which is meant to be overwritten, returns 0 (UtilityFactor -> calculate_factor_score)")
	return 0
```
- As you can see, I've left in a print statement to let us know that whatever class that's inheriting from this hasn't had their own `calculate_factor_score()` function made
- Next let's move onto the `UtilityConsideration` which also acts as a base class
- Our consideration adds 3 things
	1. A `contextID` that we can use to grab information we need from the `UtilityDecision` class
	2. A `curve` variable that allows us to make our factor get applied to a curve (we'll see how this is useful in our examples later)
	3. A `calculate_consideration_score()` function that is what stores our calculation code
- Since this is a base class, there are no calculations being calculated in the function, simply a warning to let us know we haven't set one up yet
```gdscript
extends UtilityFactor
class_name UtilityConsideration

#region Variables
@export var contextID : String
@export var curve : Curve
#endregion


func calculate_factor_score() -> float:
	return calculate_consideration_score()


func calculate_consideration_score() -> float:
	print("WARNING: Called consideration calculation from base class which is meant to be overwritten, returns 0 (UtilityConsideration -> calculate_consideration_score)")
	return 0
```
- Next let's look at our `UtilityAction.gd` script
- This has a few variables:
	1. The `actionID` is what we use to determine what action to take
		- ! *Each action should have it's own **unique** ID*
	2. The `calculationMethod` will be discussed more later but basically it's how we want to calculate all our scores together (add, subtract, multiply etc.)
	3. The `factors` array stores all our different factors (like the `UtilityConsideration` class)
	4. `scores` is used to track all the different scores from our `factors`
- In our `calculate_utility_score() -> float` function we go through each of our factors, calculating their score and then sending the list of values to our `utilityDecision` that calculates them all together for us before finally returning the final score
```gdscript
extends Node
class_name UtilityAction

#region Variables
@export var actionID : String
@export var calculationMethod : UtilityDecision.CalculationMethod
@export var factors : Array[UtilityFactor]

var scores : Array[float] = []
var utilityDecision : UtilityDecision
#endregion


func calculate_utility_score() -> float:
	scores.clear()
	
	for factor in factors:
		scores.append(factor.calculate_factor_score())
	
	return utilityDecision.calculate_final_score(calculationMethod, scores)
```
- Finally let's look at our `UtilityDecision.gd` script
- It's a bit of a long one, but in reality that's all our math functions
- First off, here is where we set-up our `CalculationMethod` enum. My version supports:
	1. Add
	2. Subtract
	3. Multiply
	4. Divide
	5. Average
- Feel free to adjust this as you need to
- Now let's look at it's variables:
	1. The `actions` array stores all the different actions our AI can do
	2. The `context` dictionary stores data needed by our `UtilityConsiderations` to calculate their scores (we'll go over this more in my examples)
		- For example, to determine if an AI should attack we probably need to store things like it's health, the players health, the damage it does, the damage the player does etc.
		- Here I have `context` as an export, how you decide to set-up your context is up to you. You always have to update the context keys with the updated values, but being able to easily see your list of context from the editor as such may be useful for you. Feel free to remove the `@export` if you don't need it
- Next lets go over our functions:
	1. Our `_ready()` function simply stores a reference of our `UtilityDecision` node to all our `UtilityActions` and `UtilityFactors`
	2. The `get_best_action() -> UtilityAction` is what is actually called in our code to determine the best action for us to take. It has two variables, `bestAction` to store a reference to the actual action and `highestScore` to track the scores. We go through all our `actions`, calculating it's score and then checking if it's higher then the previous one and updating our variables accordingly
	3. Lastly, you'll have noticed our call to the function `calculate_final_score` from our previous scripts. This takes in the type of calculation to be done as well as the list of scores to calculate, returning the final value. We use a match statement to call the appropriate function for that calculation method
```gdscript
extends Node
class_name UtilityDecision

#region Vairables
enum CalculationMethod{
	ADD,
	SUBTRACT,
	MULTIPLY,
	DIVIDE,
	AVERAGE
}

@export var actions : Array[UtilityAction]
@export var context : Dictionary = {} ## This stores the references to any needed variables, with a STRING being used as the key and the value being whatever the context is
#endregion


func _ready() -> void:
	for action in actions:
		action.utilityDecision = self
		
		for factor in action.factors:
			factor.utilityDecision = self


func get_best_action() -> UtilityAction:
	var bestAction : UtilityAction = actions[0]
	var highestScore : float = 0
	
	for action in actions:
		var actionScore : float = action.calculate_utility_score()
		context[action.actionID] = actionScore
		
		if actionScore > highestScore:
			highestScore = actionScore
			bestAction = action
	
	return bestAction


#region Formulas
func calculate_final_score(_calculationMethod : CalculationMethod, _scores : Array[float]) -> float:
	match _calculationMethod:
		CalculationMethod.ADD:
			return calculate_via_add(_scores)
		CalculationMethod.SUBTRACT:
			return calculate_via_subtraction(_scores)
		CalculationMethod.MULTIPLY:
			return calculate_via_multiply(_scores)
		CalculationMethod.DIVIDE:
			return calculate_via_division(_scores)
		CalculationMethod.AVERAGE:
			return calculate_via_average(_scores)
		_:
			print("CALCULATION METHOD INVALID OR NOT SET UP (UtilityAICalculator -> calculate_final_score)")
			return 0


func calculate_via_add(_scores : Array[float]) -> float:
	var totalScore : float = _scores[0]
	
	for i in range(1, _scores.size()):
		totalScore += _scores[i]
	
	return totalScore


func calculate_via_subtraction(_scores : Array[float]) -> float:
	var totalScore : float = _scores[0]
	
	for i in range(1, _scores.size()):
		totalScore -= _scores[i]
	
	return totalScore


func calculate_via_multiply(_scores : Array[float]) -> float:
	var totalScore : float = _scores[0]
	
	for i in range(1, _scores.size()):
		totalScore *= _scores[i]
	
	return totalScore


func calculate_via_division(_scores : Array[float]) -> float:
	var totalScore : float = _scores[0]
	
	for i in range(1, _scores.size()):
		totalScore /= _scores[i]
	
	return totalScore


func calculate_via_average(_scores : Array[float]) -> float:
	var totalScore : float = _scores[0]
	
	for i in range(1, _scores.size()):
		totalScore += _scores[i]
	
	totalScore /= (_scores.size() - 1)
	
	return totalScore
#endregion
```
- Okay, you may be asking at this point, what's the point of the `UtilityFactor` base class? Well that's for our `UtilityAggregrate` class!
- Our `UtilityAggregrate` exists to give us more freedom. Let's say we have 3 considerations for our action, our action will multiply all the final values but there are two of our considerations that we want to add together first, this is where the `UtilityAggregrate` comes in! It let's us say, hey for these considerations we want to add them *first and then* let our `UtilityAction` add that value to it's calculations and do whatever with it
- Let's go over it's variables:
	1. `calculationMethod` is just like in the `UtilityAction` script
	2. `factors` is also just like the `UtilityAction`, an array to store all the considerations and even other `UtilityAggregrate` nodes
- Just like the `UtilityConsideration` script it also has the `calculation_factor_score()` function (since it inherits from `UtilityFactor`) which calls the `calculate_aggregration_scores()` function which uses a for loop just like the `UtilityAction` script to calculate the score
![](src/assets/images/godot_4_tutorials/UtiliityAIDemoAIGodot4.png)
```gdscript
extends UtilityFactor
class_name UtilityAggregrate

#region Variables
@export var calculationMethod : UtilityDecision.CalculationMethod
@export var factors : Array[UtilityFactor]
#endregion


func calculate_factor_score() -> float:
	return calculate_aggregration_score()


func calculate_aggregration_score() -> float:
	var factorScores : Array[float] = []
	
	for factor in factors:
		factorScores.append(factor.calculate_factor_score())
	
	return utilityDecision.calculate_final_score(calculationMethod, factorScores)
```
- Ok, but *why* do I have a `calculate_factor_score()` and then a separate function it calls for both the `UtilityConsideration` and `UtilityAggregrate` scripts?
	- Good question! I just like it that way as I think the naming is more clear. There's no real reason for it, and the code could simply all be placed in the `calculate_factor_score()` if you prefer
- And that's it!
- Now, I'll show you how I created some considerations for my demo gif (seen at the top of the post)
- Let's take a look at my AI node in the scene tree:

- As you can see, we have a few actions:
	1. *Eat:* Our AI can choose to eat based on it's hunger score and if there's food available
	2. *Sleep:* Our AI can take a nap if it gets real tired
	3. *Fishing:* If we're not tired or hungry, we can just fish to pass the time
- As you can see, each action has one or two considerations. The `UtilityConsideration` scripts you have will vary based on your game, but I'll go over the three types I have here
- First, is the `UtilityConsiderationStat` script. This takes in a specific stat value and compares it on the curve to give it a score. Which also brings back up the `curve` variable we have, the curves `x` is set between 0 and 1 while we can set our `y` to whatever, in my case I set it's max value to 100. We can then use this values to determine what value to return back
	- **Formula:** `curve.sample(utilityDecision.contect[contextID] / curve.max_value)`
	- `normalizedStat` normalizes the value while `utility` plots it on our curve
```gdscript
extends UtilityConsideration
class_name UtilityConsiderationStat


func calculate_factor_score() -> float:
	return calculate_consideration_score()


func calculate_consideration_score() -> float:
	var normalizedStat : float = utilityDecision.context[contextID] / curve.max_value
	var utility : float = curve.sample(normalizedStat)
	return utility
```
- The curve for my fatigue state check is actually at 0 until we hit 50% tiredness, since I don't want my goblin to sleep if it's not really that tired. You'll need to mess with your curves to see what works best
- Next let's look at my `UtilityConsiderationBoolean` script
- It's pretty simple, since it simply takes in a `bool` value, converting it into an int value (0 is false and 1 is true). I used this to check if there was food, since if there wasn't I want to set the food action to 0 even if we're hungry, so I make sure to multiply the state value by this value
```gdscript
extends UtilityConsideration
class_name UtilityConsiderationBoolean


func calculate_factor_score() -> float:
	return calculate_consideration_score()


func calculate_consideration_score() -> float:
	return int(utilityDecision.context[contextID])
```
- Lastly let's look at my `UtilityConsiderationActionIsZero` script
- This is done last, since I need the other actions to be calculated first and is used to know when we should fish. If a `UtilityAction` has a score of 0 or less, I want to return one and zero otherwise. I add these values together, so if my fatigue and eat action come back as zero, then this will return at least a 1 thus making it the best action to take
```gdscript
extends UtilityConsideration
class_name UtilityConsiderationActionIsZero


func calculate_factor_score() -> float:
	return calculate_consideration_score()


func calculate_consideration_score() -> float:
	if utilityDecision.context[contextID] <= 0:
		return 1
	else:
		return 0
```
- And now that's really it! You can see it in action in the gif at the top of the blog post. The top bar is our hunger and the bottom is our fatigue
- *"But wait!"* (You might be saying) "What about that debug console thing??" Don't worry I got you :P
- For easy debugging, you can create a `TextEdit` node and make it a variable to our `UtilityDecision` script. Then whenever a new `UtilityAction` is calculated simply add the action score to the `TextEdit` and print out the final score once the loop is over

---

And that’s it! (for real lol) Now you have the building blocks of Utility AI. Now that hard part is figuring out how you want your calculations for all your considerations to go. Of course that doesn't mean it ends here, there are further adjustments you can do.

1. For instance I only normalize the consideration scores and not the final action scores, this may be something you'll want to do. 
2. You're game may also include *a lot* of options, like the sims 4, in which case adding a `UtilityBucket` can be helpful. This is where you group a bunch of actions into a certain type, you first decide what *exactly* it is you need to do *and then* choose an action from there. So our goblin can deicde between being more hungry or tired, and then there can be a list of possible actions. Like maybe our goblin wants to sleep on a healing pad if he's injured over a normal bed

Okay, that's it for now, maybe I'll make a future post that includes the `UtilityBucket` as well but until then you can view my other `Godot 4` tutorials by selecting it under the `Categories` tab on the left! (or possibly on the bottom if you're on mobile)

---

**Resources**

[AI and Games Video](https://www.youtube.com/watch?v=p3Jbp2cZg3Q)

[This Is Vini Godot Ex](https://www.youtube.com/watch?v=d63hbJYYqM8)

[git-amend Unity Ex](https://www.youtube.com/watch?v=S4oyqrsU2WU)