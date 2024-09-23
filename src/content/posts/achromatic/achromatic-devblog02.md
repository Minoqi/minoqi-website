---
title: 'Achromatic Devblog #2'
published: 2022-03-09
description: 'The halfway point of development'
image: ''
tags: [Devblog]
category: 'Achromatic'
draft: false 
lang: ''
---

![](src/assets/images/achromatic/AchromaticBossRoughAttackGIF.gif)

After reaching the halfway point of the semester, I made another devblog going over what Achromatic is, where the team is now, what the team has done poorly and well as well as some takeaways I've had overall.

---

# What Is Achromatic?

Achromatic is a 3D psychological horror game for the PC where the player has to navigate around a 1950s inspired carnival while avoiding enemies which include mannequins and a terrifying horse which chase the player throughout the game. 

![](src/assets/images/achromatic/AchromaticShrromMannequin.png)

---

# Where Are We Now?

![](src/assets/images/achromatic/AchromaticBossRoughIdleGIF.gif)

After having our first meeting with Rockstar about a week to 2 weeks ago, we've since been busy reworking a decent chunk of the game. Our 3-4 level layout was switched to a single level that happens in one room (possibly two) instead of spread out over 3-4 rooms. While this has been the best decision for the game overall it does mean we're a bit behind where we would ideally be right now. 

When it comes to art, a lot of the environment art has had to get redone due to the new environment.  However, art for the mannequins and horse haven't had to get changed. So, the horse not only has a model but also an animation for standing, walking/running and for killing the player. One of the mannequin models is already in the process of being modeled/textured with other designs being planned with the overall mannequin base (what all the other mannequins models/textures are deriving from) having been modeled.

![](src/assets/images/achromatic/AchromaticMannequinDesigns.png)

On the design end, the designers have been busy designing new levels to go with our new system, with our main mechanics basically all having been transferred over so they're not starting from scratch. 


On the programming end, we've probably been affected the least by the big change. Almost all of our systems have been carried over without us needing to recode much, and all the main systems should be coded in by the end of this sprint, so now it'll mainly be focused on the polishing stage for programmers. The only system that was really affected was our scene loading system, which allowed levels to load while the player was still in a level to keep transitions smooth and loading screens unnecessary. While this isn't being implemented anymore currently, there's still a possibility it'll get reused in the end to make it easier for the designers to put together an entire level without having to do it all in one scene together in Unity. 

---

# What's Gone Well? What's Gone Poorly?

![](src/assets/images/achromatic/AchromaticBossMoveGIF.gif)

I just did another devblog going over this about a week ago, so not much has changed in such a short time in this respect so I'll just do an overall recap of what I said previously. 

In terms of what's gone well, our team was very receptive and quickly adaptable based on Rockstar's advice. Reworking a decent chunk of a game is never fun yet I was surprised to see basically everyone on the team was on board with a rework almost immediately if it meant it made the game better. This definitely made the process much smoother. For me personally, I'm proud of how our QA Tool is coming along. The FOV Tool has been reused many times for not only the player but also the monster and mannequin AI. The time tracking tool works well and the new ghost playback system should make it a million times easier for designers to track what it is the player is doing.  I've also done a good job at recognizing the signs of burnout in myself so I can start taking action to prevent it earlier.

![](src/assets/images/achromatic/AchromaticGhostPlaybackDEMO.gif)

In terms of what's gone poorly, our team still has a major issue with communication. They talk over each other frequently which can make meetings messy and disorganized as well as making it hard to follow what's happening. This should hopefully start to get resolved soon with the use of a talking stick, which I happened to run into one of our producers buying a stuffed doll from target to act as the new talking stick. For me personally, I felt as if taking proper precautions to stop burnout from occurring did make me put in less work overall than I would've liked/am used to sometimes, although after getting out peer evaluations back I think this is more of a me issue rather than an actual issue, since everyone on the team thought I was doing a fine amount of work which was reassuring. Also, even though I'm pretty good at moving on from things that are taking up too much time and can get resolved later, I did spent a little too long at the start trying to debug issues with the FOV Tool which could've been ironed out later on.

![](src/assets/images/achromatic/AchromaticConceptEnvironment.png)

---

# Evaluation Takeaways

Like I briefly mentioned earlier, we got our peer evaluations survey results back, where everyone on the team gave evaluations on anyone they had experience working with. The most surprising part of the evaluations for me was the fact that everyone thought I was doing a fine amount of work, since I thought I was doing too little at times, but I am aware I tend to have high standards for myself when it comes to the amount of work I do for projects so while it was surprising it also wasn't at the same time. 


The main area everyone said I could improve on was in communication, which wasn't too surprising. I am in general a very quiet person and don't talk unless I really have something to say. While in general, I think I could improve on my communication with the designers/other programmers when it comes to properly getting the tools implemented into the actual game, I do think that sometimes I talk without a lot of people realizing. This really just boils down to our overall issue of people talking over each other in general. I'll either be in the middle of a conversation with someone, someone else will start talking to this person and then they'll switch to whatever it is the other person was talking about instead of waiting for us to finish our conversation before moving on, or I'll try talking, someone will cut me off almost immediately and then I can never get out what I wanted to say. This has happened in meetings before where a producer will help find a lull point in the conversation to get everyone's attention back to what it was I was trying to say.  This even happened once about 2-3 times for just one statement once which was very annoying, but I appreciate our producer doing their best to get everyone to stop talking and refocus their attention so I could say what I wanted to say. 

![](src/assets/images/achromatic/AchromaticStall.png)

I've also noticed a huge difference between an extrovert dominated team over an introvert dominated team. In the past, I've either been on teams with a somewhat even split between the amount of extroverts and introverts, or on an introvert dominated team and it's just been very interesting to notice some differences. I won't say these differences always happen since this is the only extrovert dominated team I've been in but it's been interesting none the less. While on an introvert team the main issue with communication was people not expressing their thoughts clearly and often enough, now it's people are almost expressing their thoughts too much to the point that others can't get in their thoughts as well. So when I look around, I often notice it's the loud extroverts talking so much that it can make it hard for the quiet team members to get the attention and focus they need to state their opinions. Even when they try to speak up to get everyone's attention they often get ignored. Again I don't know if this is a common difference, or if this is just a team specific issue, but I just thought it was interesting.  


Lastly, I've really realized how much more tiring it is for me when I'm on a team of people that talk a lot compared to a more quiet team. I'm used to being on a quiet team where our work meetings and other meetings are more on the quiet end. You don't hear people talking 24/7 and not that loudly. Now, this team not only talks 24/7 but also some tend to talk rather loudly. And I find myself way more tired after a meeting than I normally would be from all the extra stimulus. This has made me realize that when looking for a job, I should make sure to ask how talkative and loud their work environment can be since I'm not sure how well I'd work in an environment that's constantly filled with people talking rather loudly. 