---
title: 'Your Forever Home Alpha Build & Our First QA'
published: 2021-04-08
description: 'The alpha build of Your Forever Home'
image: ''
tags: [Devblog]
category: 'College'
draft: false 
lang: ''
---

Your Forever Home is a metroidvania style, psychological 3D horror puzzle game. I've been documenting it's progress from being a basic prototype, to getting it past greenlight to now making an alpha version of the game. In this blog I'll be discussing the journey to getting it from a prototype to an alpha state.

![](src/assets/images/college_prototypes/YFHDirtyLogo.png)
*Your Forever Home Logo by Sarah Shaw*

---

# Weeks Leading Up to the Alpha Build

![](src/assets/images/college_prototypes/YFHBaseDogModel.png)
*Base 3D Model for the Dog by Sarah Shaw*

We spent a lot of the first week getting everyone up-to-date with the game and our plans for it. In terms of programming, there wasn't too much we could start on besides working on the shaders we're going to use to help get the look we're going for.  While the other two programmers were working on the shaders, I started work on developing a new technical risk assessment document (tech doc) for everyone to be able to reference. Besides the typical things you'd see in a tech doc, I also added a section I called "related scripts", this goes underneath each mechanic and is a list of all the scripts related to it making it easier for programmers to go back and find each script they need as well as making it easier for designers to figure out what goes where. While the first week was slow on the programming side, the designers were on full blast working out the mechanics of the game and figuring out details, as well as working on level designs to go with said mechanics. Following this week things started to pick up as artist worked on getting art into the game, designers worked on fleshing out levels and programmers worked on getting everything coded into the game.

While most things have gone at the speed I expected them to go to, one thing that took longer than expected was the camera. We've been having many issues with it and still haven't fully committed to a camera system. Our current system isn't too far off from what we need, it's just got a lot of bugs that need fixing. Originally we had another programmer working on the camera, but since they've been using GRID, they can't use cinemachine as for some reason this has been causing issues. I've instead picked up working on the camera from here on out. I've spent some time researching different methods for a 3D camera system and I think I found a system that could work well. 

My goal now is to work on getting that into the game as well as finishing up our save system. It's mostly done but there's some bugs that need fixing. Besides this, I also implemented the dialogue system. As of right now, we only needed a basic dialogue system so the player can talk to some NPCs and each NPC has one set of dialogue that they repeat every time you talk to them. I made sure to make my system easy for designers to use and edit. The dialogue itself in Unity is saved as a prefab, and all the NPC needs is an empty gameobject with a collider with the range of where the player can talk to the NPC. It also has a system so that the profile image that appears when talking can be updated to match whatever NPC is speaking.

![](src/assets/images/college_prototypes/YFHSadDog.png)
*Art by Sarah Shaw*

We've only spent about 3 (kind-of 4) weeks on the game since the prototype was done so far. Technically it's been 4 weeks, but since the school was doing a "pseudo" spring break (or what was supposed to be a pseudo spring break which just really depended on if your professors gave you one or not) and since the majority of the team was feeling burnt out from all of our other classes, we decided to take that week off. It wasn't mandatory to work on anything unless you really wanted to. While we lost a week of work doing this, I think it was a good idea in the end since if we didn't people might've just gotten even more burnt out which would lead to a less productive rest of the semester. We were also almost done on the programming side of things, since the shaders had been finished as well as the AI, movement (which was actually finished in the prototype) and dialogue plus the core mechanics like pushing/pulling items (among other abilities) was almost done we decided it wouldn't damage us too much. Now we just have to update the camera system, finish the save system and add the final ability to the game and the major programming will be done. We'll then be focusing on adding polish to the game. 

Our final alpha build is at a pretty good state as most things have been implemented and we have 2 out of the 3 parts of the first level implemented as well. 

---

# QA (Testig)

We were finally able to take out alpha game to QA recently and the feedback overall wasn't bad. The feedback we did receive was more or less what we had been expecting. The majority of the complaints came from the camera, which was still rather wonky and its sensitivity not only was way too high but for some reason the controls had been reset to inverted at some point which made it unnatural for most people since there was no way to switch it back to normal. 

Another really common complaint was that people didn't understand how the controls worked, which was really our fault as the person in charge of QA never told the participants what the controls were in the first place and currently the game doesn't tell you the controls either. 

Our third most common complaint was on the abilities. Currently there's no way to rotate an object once you pick it up which makes placing it down the right way rather difficult, so we need to go in and add the ability to do that. 

Someone had also mentioned that the breakable objects should be more distinguishable which I agreed on, as the only thing distinguishing it is a slightly different color and one of the brackets looks different from the others, I keep saying we should just slant it which would make it much more clear that it's breakable but the designers seem hesitant to do that for some reason so the solution is still in discussion.

Overall, the QA went rather well in the end. Most people really liked our concept as well as our dog model as they thought he was very cute. We were struggling with the scaling of everything but we seem to have gotten it right as the majority said the scaling felt right. 

![](src/assets/images/college_prototypes/YFHToilet.png)
*Art by Sarah Shaw... why do I even have this?*

---

# Final Thoughts

Overall, while I think we've managed to stay on track of what I was expecting to have done by alpha, I do think our communication needs work. Especially in the beginning, designers were making lots of changes and updates without clearly updating the programmers and some of them didn't even seem to all be on the same page sometimes as well. The miscommunication on top of the fact that they weren't keeping their design doc up-to-date made it difficult to keep track of what we were supposed to be adding as well as what had gotten scrapped so we wouldn't have to worry about it. Since then, the mechanics have more or less been settled (with minor adjustments here and there) so this isn't an issue as much anymore but our communication as a whole still feels rather lacking. I've definitely realized that the way we would communicate on a team of 3-4 just doesn't work as well with 9 people. Before, we only really had 1 of every discipline so there was only ever 1 person you needed to talk to and keep track of, but now each discipline has 2-3 people involved which has made keeping track of what everyone's doing much more difficult and I definitely think this is something we need to work on for the remainder of the development period. 

While I'm glad we finally made it to QA, and have plans on attending weekly from now on, I definitely think we probably could've gone earlier in retrospect. The designers seemed to always want a full level done and all these other mechanics in prior to taking the game to QA, but we definitely could've made do with just 1 or 2 puzzles since the mechanics for those had been completed early on. While we would've had to make due with our old camera system, I think response on things like movement, puzzle difficulty and mechanics would've been useful. 

Lastly, I wish I could've gotten more done. However, with my other 4 classes (2 of them being programming related) keeping up with those as well has been quite the experience. Especially since my graphics class has been quite the experience in itself this semester. While I'm annoyed at the fact I didn't get more done, I also realize I still got a decent amount of stuff done in what was essentially 2 weeks. (Since one week was mainly finalizing design, and one week was taken off, that left us with about 4 weeks of work.)

![](src/assets/images/college_prototypes/YFHHappyDog.png)
*Good Doggo Saying Adios by Sarah Shaw*
