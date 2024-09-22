---
title: Dialogue Forge Roadmap Updates
published: 2024-03-28
description: ''
image: ''
tags: [Devlog]
category: 'Dialogue Forge'
draft: false 
lang: ''
---

It's been a few days since the release of the alpha version of Minos Dialogue Manager. It's great to have it finally released and in that time I've taken a short break from working on it directly, but have been mulling over how the remaining features will be implemented and have finally come to a decision.  So for those interested, you can keep reading to hear how to final version will most likely function:

# Save Files

Before, the tool would save one file for each dialogue, however for easier multiple file support, in the final release all dialogues for each project will be stored in a single file. Each dialogue will have a name that gets stored in the JSON as a key to the dictionary which then holds a dictionary of all the data related to that dialogue file. This'll make not only translation support much easier but also showing all the different dialogues on the left hand side for easy access. Having all the dialogues related to a specific project will also make it easier to not get lost in possibly hundreds of files as well as making those working with a framework have an easier time.

# Translations

Like stated above, the new file system will allow for easier translation support. As of right now, the exact way a translation ID will be generated isn't totally set in stone, but it's believed that they'll be generated when you go to export a file. Every time it exports it'll check if an ID is needed and if one isn't yet made for that marker it'll generate one. Any IDs for markers that no longer exist will get removed. Right now it's planned to work for certain markers, although I may look into adding support for custom markers as well.

# Custom Markers

While custom markers already exist, I do plan on adding syntax support so you can add syntax highlighting color for any of your custom markers. You'll also be able to add it to the suggestion box, as well as give it variables it'll automatically be made with. In other words, it'll work basically just like the default markers!

# Addons

As the description for the tool states, this is meant to be usable in any engine/framework as long as it supports reading JSON files. While my main focus right now is getting the final version of the tool ready for release, post that I plan on creating tools for popular engines/frameworks that'll already read in the file for you. It'll be easy to adjust to include the code for custom markers or for those that don't need custom markers you can just plug and play! These will be additional addons you can purchase. My main target is for Godot and Unity, but I plan on offering framework support for pygame, love2D, raylib and SDL in the future as well. If there's any engine/framework you'd like official support for let me know!

---

And that's it! Between work I'll be working on the dialogue manager and plan on releasing small updates for the alpha version as I go before the official version! I'm not sure how long it'll take, but I'm hoping it'll be done given a month or two :)

PS Oh, I also plan on making a discord, I'll make an announcement once it's live!