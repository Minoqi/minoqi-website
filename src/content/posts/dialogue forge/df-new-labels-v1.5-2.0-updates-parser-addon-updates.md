---
title: New Labels, V1.5, V2.0 Updates, Parers Addon Updates
published: 2024-08-04
description: ''
image: ''
tags: [Devblog]
category: 'Dialogue Forge'
draft: false 
lang: ''
---

![](src/assets/images/dialogue_forge/DialogueForgeNewLabels.png)

> Hello! I've made some small updates to the program. These are yet to be released, as I'm waiting to finish all the last changes I want to make on my list before preparing to start the V2.0.

# What is V2.0?

Version 2.0 will introduce the ability for you to add your own labels to the autocomplete list along with what color to assign it.  I have some smaller changes I want to make before focusing on this.

# What changes have been made?

I recently added a few QoL features and some new labels. The new labels include:

1. Emit Signal: Emit signal makes it easier to use this program with Godot, although the signal pattern Godot uses is called the Observer Pattern and can be used in any engine, I recommend you read up on it!

2. Background: This is added for those that want to create visual novels. It's pretty common to have backgrounds you need to swap so this label is now built in :)

3. END: Honestly, I never thought to have a built in END label, which in hindsight should definitely be in here, so that's been added as well

4. Insert Additions: All of the above labels have been added to the insert dropdown as well as the translation button to generate an ID if you can't use the shortcut or prefer not to

5. Automatically adds an end label to new files now

# What changes are left?

I have a few changes on my todo list I want to add for a 1.5 release.

1. Adding in the ability to rename dialogue files

2. Having the dialogue files appear in alphabetical order

# Will V1.5 be backwards compatible with V1?

I do want it to be backwards compatible, and ordering the dialogue files should not require changing the save data so I believe it will be backwards compatible.

# Will V2.0 be backwards compatible with V1/V1.5?

I will try, but it depends on the process for adding autocomplete and colors. I believe it should be possible though.

# When will V1.5 be released?

I'm hoping within a week it'll be done since most of these changes are minor code wise.

# When will V2.0 be released?

It'll probably be a while before V2.0 is released. This will require much more work. I also want to create parser addons before moving onto a V2.0 release. 

# When will the parser addons be ready?

I plan on starting with the Godot parser first as it's my primary engine as of writing this. After Godot I plan on making one for Unity as well. I already have a rough idea of how the parser will be structured, but I want each parser to have a few example scenes to make it easier to use, so I'm not sure how long making one will take. I want a Godot and Unity version available before moving onto V2.0. While I plan on making ones for Unreal and Gamemaker, I have little experience with both engines so I'm not sure how long it'll take for me to make one. If you have more experience with these engines and would like to help in making one, let me know!

![](src/assets/images/dialogue_forge/DialogueForgeInsertList.png)