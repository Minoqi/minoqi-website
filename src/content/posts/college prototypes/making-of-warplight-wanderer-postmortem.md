---
title: 'Making of Warplight Wanderer - Postmortem'
published: 2021-11-28
description: 'A postmortem about the development of Warplight Wanderer'
image: ''
tags: [Postmortem]
category: 'College'
draft: false 
lang: ''
---

Warplight Wanderer is a 3D puzzle exploration game with platformer elements, with the goal of being a overall easy game to play. It should be calming/relaxing and the levels aren't meant to be too difficult. It's very beginner friendly and is open world so if the player gets stuck in a level they can move on, do another and come back when they feel ready.

---

# What I Did

For Warplight Wanderer, I was mainly in charge of the systems as well as developing tools. I also helped out with the sound effects, music production and UI design/implementation. While I have considered myself a systems/AI programmer for a while, I had no experience making tools and had never really thought about it before. But for our game, I was put in charge of making our dialogue system, which was something that I really wanted to do after watching a video about Hades dialogue system. I ended up having a lot of fun with it and since capstone I’ve kept working on the tool and developing it further with the plans of releasing it early next year on the Unity Asset Store. I also plan on looking into tool development as well since it was surprisingly fun. In terms of the sound effects and music, I've always had an interest in music production and had recently gotten interested in sound design, so I was glad I was able to produce a song as well as record a sound effect. 

---

# Dialogue Tool

![](src/assets/images/warplight_wanderer/WWDialogueTool.gif)
*Demo of the dialogue tool I made*

When making the dialogue tool, I knew I didn’t want to go the normal route of using scriptable objects which is what most tutorials will show you when looking up dialogue systems for Unity. Instead, I wanted to make a node system that was much easier to expand upon and easier for designers to work with. This, of course, was also much more difficult. However, after a while of searching I was able to find a YouTube tutorial that helped create the basis for a node based dialogue tool in Unity, and with this tutorial I was able to make the basis of the tool and have been able to expand upon it since then (you can read more about it here). While this took a lot of work, especially a lot of googling about Unity’s UI.Elements system, I found it well worth it since I learned a lot about programming in Unity, especially realizing all the stuff you can do using Editor Tools and UI.Elements in Unity. It also worked well with the designers since they were happy to see a more advanced and user-friendly system compared to systems they used before in previous games. 

![](src/assets/images/warplight_wanderer/WWDialogeNPC.gif)
*Example of it in action*

---

# Stars & Sockets System

Moving on from the dialogue tool, one of the major systems I worked on was the stars and sockets and how they interact. In the game, the player needs to be able to place and pick back up stars from sockets and there needed to be a way to keep track of how many sockets had been filled so the lighthouse knew when to light up signifying the level was complete. This system went through multiple iterations, with efficiency being a priority.

While the game had been developed for computers, we had discussed the possibility of porting to mobile in the future so we wanted to make sure we created efficient systems since mobile power is much more limited. While initially having all the stars constantly searching for a socket seemed like a logical idea, I knew it wouldn’t be efficient to have every star in the scene constantly searching when they only needed to look for a socket when the player was holding onto them. This was especially important since at the time, we hadn’t decided how many stars would be in a single scene so we wanted to account for the possibility of a scene having a lot of stars.

To help counteract this, I initially made it so a star would check to see if the player was holding it and if the player was holding it, then it would start searching for sockets nearby. When a socket was found, if the player dropped the star it’d get placed into the socket. While this system worked well initially, we ran into issues when trying to implement UI. The UI would appear fine for picking up and placing a star in a socket, but once you placed a star in the socket and left the UI wouldn’t go away. After a lot of different ideas and trying a lot of different methods, I ended up reworking the system so we used triggers from the player instead of having a star search for a socket nearby.

***Instead the final system ended up working like this:***

**​The player** would check to see if they hit any triggers. From these triggers we were able to easily turn on and off the UI. 

**The stars** would hold information like if the player was hitting it’s trigger, if there was a socket nearby (which was gotten from the player if they hit a sockets trigger) as well as what to do when the player went to grab/drop a star. It checks for things like if it’s in a socket, if the socket it’s trying too be placed in is full, if the player is trying to grab a star while holding a star etc. It also made sure the player wouldn’t pick-up/drop a star while in the air to help reduce the chances of the player soft-locking themselves.

**The sockets** really only hold information about whether or not they have a star on them. The had a function that was called from the star script that played some audio/particle effects to correspond with whether it was getting filled with a star or having a star taken off but overall the sockets didn’t check for too much. Outside of this they also called a script from the lighthouse to check to see if the level was complete.

**The lighthouse** keeps track of all the sockets and when a level is complete and plays the “level complete” function if it is.

---

# Socket Behaviors

![](src/assets/images/warplight_wanderer/WWStarSocket.gif)
*Move Object Socket Behavior*

While the stars had their own behaviors that affect the player (ice star letting you walk on ice platforms and the gravity star letting the player jump higher) the sockets had their own behaviors that affect the world. I took over the behaviors for the sockets. There was multiple ways I could've gone about this but in the end I went with creating a script for each behavior. We ended up with 3 behaviors plus 1 that was what we dubbed the "Zelda Camera". As you can see in the gif above, it makes it so the camera pans to focus on whatever it is in the world that is affected by the socket. I was in charge of making the 3 behaviors while the other programmer made the "Zelda Camera". 

Each behavior worked by attaching the script to the socket that will play the affect when filled. The script would check to see if the socket was marked as "activated" and if it was then it'd run whatever affect it did. The 3 behaviors I made was one that moved an object in the level, one that changed the material of an object and one that played a specific animation. I also made it so if a star is taken out of a socket the behavior is undone and with the Zelda Camera it will show the behavior being undone. This was done with just checking to see if the activated became false.

---

# Journal

![](src/assets/images/warplight_wanderer/WWJournal.gif)

The last main system is the journal. The journal was created as a way to help keep the player on track in case they forget what they're doing or what certain stars do. The first tab of the journal, the "Level Info" gives the player information about the name of the island they're on, the name of the Novaling (NPC) they're helping, what it is you're helping the Novaling with, how many sockets have been filled out of how many they're are in the level and information on the types of stars in the level. I made sure the implementation of the star information was dynamic by using Unity's "Vertical Layout Group" component so the star information expands accordingly, that way the designers don't have to constantly resize it in every scene. This was done by having the designers check off some boxes based on what kind of stars are in the level from the journal script, and from there when the level is loaded it fills the proper star information UI accordingly. 

The second tab of the journal, the "Photobook" stores screenshots of all the levels you've beaten as well as a little sticker that has to do with whatever the quest was in that level. While the other programmer was in charge of creating the screenshot system and placing them in the journal, I put together the stickers appearing after they got made later on.

The final tab of the journal, the "Settings" is exactly as it sounds. Here you can find a list of all the controls as well as some options to invert the x and/or y axis of the camera if wanted.

![](src/assets/images/warplight_wanderer/WWJournal2.gif)
*Star/Level Info Updating with Each Level, Completion Status Updating During Gameplay*

---

# Extra Stuff*

Besides doing a lot of bug fixing and implementing UI, I also helped with the music production and sound design of the game. I created a song that was used in the trailer as well as the song that played in each level. I wanted to make a song that made a good ambient soundtrack, was very calming and sounded spacey. After hearing feedback from teammates, testers as well as other music producers, it sounds like I achieved my goal since most people enjoyed the song. Besides music, I also helped with sound effects at the end. I implemented all the footsteps, journal sfx and the sfx for when you pick-up/drop a star from the ground (not from the socket). For the sound effects I made myself, I made the journal sfx by taking a softcover book of mine and recording me opening, closing and flipping pages. I ended up using the flipping page audio clip for the opening of the journal as well as when changing tabs.

The footsteps and star sfx came from kenney.nl (who makes tons of completely free assets for games). The star sound effect was simple to implement, the footsteps took a little more work. In order to make sure the sound effects weren't playing immediately after one another and making it sound terrible, I added a delay between sound effects that I messed with in the inspector to get it to line up with the feet as well as I could. While I could've done this a more efficient way, this was implemented the night before it was due, so I just went with the first thought that came to mind. The game has 4 different footstep types (snow, ice, concrete and wood aka the dock). In the player script when the player moves, I checked to see if they were on an ice platform or the dock and used a switch statement to play the correct corresponding sound effects if they are. I also used a switch statement for the snow and concrete sound effects except I checked to make sure they weren't walking on an ice platform or the dock as well as checking to see what type of lighthouse was in the level (we had an ice and stone lighthouse for corresponding island types) to know which one to play.

---

# What Went Well?

In terms of programming I think I did a good job at making sure to take efficiency into account when designing my code, creating code that gave the designers less work (so most variables were either found during runtime or with the populate variables tool)​ as well as taking on a challenging goal and completing it. I had no experience making a dialogue tool let alone making an editor tool using UI.Elements, but after a good tutorial series and a lot of googling I was able to figure it out and even expand on it since. I was also conscious of trying to make code as designer friendly as possible so they didn't have to constantly be finding and filling variables for every script in the inspector. I also include details on what scripts are part of each system with the Technical Risk Assessment Document (Tech Doc) as well as what each variable does in detail with the variable sheet we made to help reduce confusion. 

​Outside of programming on a more team aspect, I believe I did a good job at managing the tensions between one of our designers and the programmers. There was a lot of tension and arguments over what comes with a role. There was mainly a lot of arguments over how much programming this designer should get to do. Now I want to clarify I don't mind designers coding, I've worked with other designers where we had a system that worked so they could take over smaller programming tasks when I was busy implementing bigger systems or fixing game breaking bugs. However, in this particular case we would try to make a designer-programmer pipeline only for them not to follow it. Instead they would just go ahead and program what they wanted (which was mainly implementing the audio and effects) which would cause issues later on since it wasn't implemented well with the current code. This was causing a lot of tension and instead of continuing to argue it and halter progress on the game, myself as well as the other programmer decided it was best to just let them keep doing what they were doing and to later go in and fix the code when necessary since it was getting close to the deadline and we didn't want to cause more issues that would halt progress. While this wasn't the ideal outcome, it worked well as we were able to get more work done since we weren't busy arguing over programming as much anymore. So I think in this case I did a good job of trying to lessen tension since the most tensions was between the other programmer and designer as well as compromising for the betterment of the game, even if I still wasn't personally happy with the outcome. 

---

# What Went Poorly?

In terms of programming, there are a lot of things I'd go back and redo if I had the time. I would've spent more time on the dialogue tool if I had had time to really help the dialogue, make it stand out by adding things like text effects. I also would've changed how the journal knew which star info to appear from having the designers check variables in the inspector to instead having the code search for which type of stars were in the scene from the populate variables tool in the inspector. This way it wasn't adding code to runtime but also made it so the designers didn't have to spend time going through variables as much. I also would've adjusted the the way objects move from the socket behavior script and used Lerp instead of MoveTowards ​for more flexibility. 

​In terms of team dynamic, I think our communication definitely could've improved, especially early on. We did get a lot better halfway through development, but it definitely still overall hindered us. Discussing topics could get uncomfortable since you wouldn't want to upset someone else and cause an argument. I believe this hindered us since I don't think we helped each other as much as we could. For example, I would've really liked to add another level or a sound effect for when the objects in the world move but even though I did add a lot of sound effects at the end, I did it because it was at the very end and I thought we really needed more sound effects. If I hadn't felt like I was walking on eggshells when it came to helping others with their roles when I was ahead and other roles were falling a bit behind, then I think it would've really helped flesh out the game. I also believe the compromise between the designers and programmers wasn't the best and just caused more tension underneath the surface between the two parties. 

---

# Overall

Overall, despite our challenges we still made a solid game in the end. I implemented a lot of systems with efficiency and ease of use for designers in mind and even took on a big challenge of not only making an editor tool for the first time but creating a node graph based dialogue tool which I had no idea how to do before hand. I'm really glad I did since it not only opened my eyes to tool development, but it's also led me to continue working on the dialogue tool and eventually releasing it early next year. 

​Despite the tensions that were in the team, I think we still did well at working together and I believe I did well at learning to compromise for the betterment of the game even if it wasn't a compromise I was happy with.