---
title: Creating a Dialogue Tool in Unity
published: 2024-07-11
description: 'AN overview of a dialogue tool I made for a project in Unity'
image: ''
tags: [Devlog, Unity]
category: ''
draft: false 
---

![](src/assets/images/UnityDialogueToolCurrentState.png)

I recently had to make a very simple dialogue tool for Warplight Wanderer. I had never made a tool in Unity before but found it surprisingly fun so I decided to keep working on my dialogue tool and finish it to later upload as a unity asset on the asset store. It uses a node system rather than scriptable objects, so it’s easier to customize and expand for narrative driven games, but I’m also making it as flexible as possible so you can use it for dialogue, sign text etc. Currently, I’ve added 3 nodes.

# Dialogue Node

The dialogue node is quite expansive since you can store the speakers name, the dialogue, a sprite for the speaker as well as an audio file. You can also add as many options for the player to respond with as you’d like.

![](src/assets/images/UnityDialogueNode.png)

I’ve done this to make it as expandable as possible. I’m hoping this system not only allows for a wide variety in NPC dialogue options but also makes it useful for other things like reading signs or tutorials etc. I’m also thinking of adding a checkbox option for those with sprites so you can change where the sprite appears if you want a sprite on the right or left which is commonly seen in games.

# Comment Node

The comment node is exactly as it sounds, a node that doesn’t do anything but allows you to leave yourself comments as your making your dialogue to make it easier to keep track of what’s going on, especially for bigger dialogue trees. Hopefully this will help people keep track of what’s going on as well as what they were doing so when they come back to a file later they’re not lost or as lost.

# Chance Node

The chance node has a slider on it, that adjusts the values of two possible output options. So depending on the value that’s generated randomly, changes which output option it’ll go with. This is almost implemented, right now it shows up and the slider works, it just needs input and output ports.

# Future Planned Nodes

So far these are the 3 nodes that I’ve implemented/began implementing. Other nodes that I plan on implementing are:

1. *Random Node*: Choose the next output at 
random.

2. *Repeat Node*: Repeats a node/section of nodes X amount of times.

3. *Execute Node*: Executes a certain piece of code.

4. *Audio Play Node*: Plays an audio file, think of it like an event trigger in a way.

5. *If/Else Node*: Chooses the next output depending what criteria it meets.

# Other Planned Add-Ons

Outside of nodes a few extra things I want to implement are:

1. *Text Parser*: A function that will parse through all the dialogue text that you can then use to look for certain markers to customize the text (<b></b> means to bold that text etc.)

2. *Minimap*: A minimap to help keep track of the overall tree and not get lost as easily.

3. *Groups*: A way to group a set of nodes and give them some sort of title to make organization for bigger trees easier.

4. *Localization*: This one I’m not sure how I’ll do yet and will probably be the final thing I add, but I’d like to make it so you can save the same dialogue tree in different languages. I’m thinking each file will have a tag that gets auto added to the end of the file name like “Narrative_ENG” or “Narrative_KOR” that can then be used by another script to decide which file to use based on the language the player has chosen. This is a WIP idea though so it’ll be a while before this gets really worked on and fleshed out.