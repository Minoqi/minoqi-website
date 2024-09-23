---
title: 'A Small Test with Unreal Engine - A Postmortem'
published: 2022-05-07
description: 'A small test I did with Unreal Engine to get a feel for the engine'
image: ''
tags: [Postmortem, Unreal Engine]
category: 'College'
draft: false 
lang: ''
---

![](src/assets/images/small_unreal_test/SmallUnrealTestDEMO.gif)

For my portfolio class, I decided to use our second project as a chance to explore Unreal Engine since up to this point I only had experience with Unity and Gamemaker Studio 1.4. I wanted to mess with Unreal since it's becoming a popular engine for game development.  Normally I'd include snippets of code I've done, but since I just used the blueprint system, I did my best to take screenshots of any blueprints that weren't too long.

---

# The Player & AI

Starting off with the player, I used the Third Person template to start the project so the movement was already coded in. I added some stats to the player including health, mana and xp. I gave the player 2 abilities, one to heal themselves with and the other to shoot a fire projectile that when it hits the enemy they'll take damage and die if their health hits 0, this will also give the player xp. The player can also die when their health hits 0 from the projectiles the AI shoots.

![](src/assets/images/small_unreal_test/SmallUnrealTestHealthDEMO.gif)
*Healing ability*

![](src/assets/images/small_unreal_test/SmallUnrealTestDamageDEMO.gif)
*Fire projectile ability & AI dying*

![](src/assets/images/small_unreal_test/SmallUnrealTestShootBulletBlueprint.png)
*Snippet of shooting project blueprint*

![](src/assets/images/small_unreal_test/SmallUnrealTestAIShootChaseDEMO.gif)
*AI shooting player and player taking damage*

![](src/assets/images/small_unreal_test/SmallUnrealTestAIBlueprint.png)
*Snippet of AI shooting, chase and wander blueprint*

---

# Health & Mana Powerups

I added 3 pick-up items, 2 of which being a health and mana item that will increase the players health/mana respectively as well as giving some xp to the player. The AI will also drop a mana item when killed.

![](src/assets/images/small_unreal_test/SmallUnrealTestHealthPickupDEMO.gif)
*Health pickup adds health to player*

![](src/assets/images/small_unreal_test/SmallUnrealTestManaPickupsDEMO.gif)
*Mana pickup adds mana to player*

![](src/assets/images/small_unreal_test/SmallUnrealTestHealthPickuoBlueprint.png)
*Health pickup blueprint*

---

# Quest System

Lastly, I created a very basic quest system. In the example I made I showed the player picking up an object that gives them a new quest.

![](src/assets/images/small_unreal_test/SmallUnrealTestQuestPickupDEMO.gif)
*Qiest pickup updates players quest*

![](src/assets/images/small_unreal_test/SmallUnrealTestQuestPickupBlueprint.png)
*Quest pickup blueprint*

---

# Overall Experience & Thoughts

Overall, I see the appeal of the Unreal Engine. It's pretty easy to get a game up and running fast and the blueprint system can make it easier to pick-up for those not experience in programming. It was really nice to be able to create basic systems quicker than it would've taken me in Unity. Since I also have years of experience with Unity, I found it easier to watch a tutorial for how to do one thing and then take that knowledge to create another system without a tutorial in Unreal. By the end of my time with Unreal, I think I could make a simple shooter style game without much need for tutorials. I also think the 3D models are definitely ahead of Unity's default 3D models *cough* capsule *cough*.  That being said I did also have some issues with the engine.

While the blueprint system can make it quicker to create systems fast, I found that it could easily get convoluted and hard to keep track of. I created a very simple game, but it's hard for me to imagine a really big full game made like this as I think I'd get lost in all the lines and blueprints going everywhere. This could just be since I'm used to the normal script system most engines like Unity, Gamemaker or Godot uses, and I'm aware you can program in C++ in Unreal but I didn't have time to mess with that.

I also found the overall tools that the Unreal Engine comes with overwhelming and harder to pick-up than Unity's. Unreal is a very powerful program which can be great for game development, but there were also a lot of things like the particle system, the folder system and the buttons toolbar at the top I found to be rather confusing. The layout of Unity I think has always just come more naturally than the layout Unreal Engine uses for me personally. While their overall layouts are similar, I find the smaller details can be quite different. I also found some ways of doing things in Unreal Engine more of a hassle than it would've been in Unity. Lastly, this is really just a nitpicky thing, I hate that the blueprint editor will always stay above Unreal Engine, even if you go to click on Unreal it keeps the blueprint window above it. I don't know if that can be changed somewhere in the settings, but I just found it annoying and slowed down my process a bit.

In the end, I see the appeal for both engines. I think if I wanted to make a really quick prototype of an idea I might do it in Unreal for it's efficiency in getting an idea up and running fast, but I think for now I'd still stick to Unity for full game development. I will say Unreal definitely seems more well equipped for 3D than Unity is, it's templated being much more fleshed out and polished especially with all the default models and particle effects it comes with (as well as the default level design), but it does fall short for 2D games which Unity wins over. I think given time I could pick-up the blueprint system more and get used to the whole look, but I can't help to feel like it'd get more confusing than keeping track of a normal script would be in something like Unity, Gamemaker or Godot. 