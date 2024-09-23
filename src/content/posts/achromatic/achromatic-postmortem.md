---
title: Achromatic Postmortem
published: 2022-05-07
description: 'A postmortem for my capstone project Achromatic made in Unity'
image: ''
tags: [Postmortem]
category: 'Achromatic'
draft: false 
lang: ''
---

<iframe width="100%" height="468" src="https://www.youtube.com/watch?v=j-wnhaJXZI4&feature=youtu.be" title="YouTube video player" frameborder="0" allowfullscreen></iframe>

# Achromatic

Achromatic is a 3D psychological horror game for the PC where the player has to navigate around a 1950s inspired carnival while avoiding enemies which include mannequins and a terrifying horse which chase the player throughout the game. 

It was made with a team of 20 people, 16 members and 4 freelance members  including 3 students from Berkeley that helped us with our music and sound and another Champlain student who did all our voice lines. 

The game is already available to play for free on [Steam](https://store.steampowered.com/app/1990210/Achromatic/).

---

# What Did I Do?

## QA Tool - FOV Tool:

Originally, the FOV (field-of-view) tool was made just for the QA tool but ended up getting repurposed for the AI tool as well. The FOV tool can be placed on any object in Unity and with the use of triggers and a spherecast, the tool is able to let you know when something is within the objects field-of-view and if something is blocking that field of view. This was used not just to figure out when the AI could see the player, but also for the player data recording feature.


## QA Tool - Player Data Recording:

Part of the QA tool was the player data recording feature which recorded what objects the player was looking at and for how long through the use of the FOV tool. This gave our designers a better idea as to what the player was spending the most time looking at which helped them iterate on the level design.


## QA Tool - Ghost Playback:
	
Another part of the QA tool was the ghost playback feature. This tool recorded the players location and rotation every X seconds which gets saved and can later be imported into Unity and played back which allowed designers to watch back how the players were playing the game and better correlate the players data from the form they filled out at testing sessions. 


## QA Tool - Unique ID:

The last part of the QA tool was a unique ID feature. This allowed each tester to get a unique ID which would then be saved in the file names (for the player data recording and ghost playback files) and the tester would then copy the ID into their form. This made it easier to correlate all the data we received.


## Monster & Mannequin AI:

As stated earlier, through the use of my FOV tool we were able to help develop a realistic seek and pursuit AI. The FOV tool went through multiple reiterations before we finally landed on a spherecast system to help with the accuracy of the system while giving some leeway to players. You can read more about the development of the FOV tool here.

## Back-Up Captions System:

Towards the end of development, we had very little time left and our ideal caption system wasn't yet working. So, I created a quick caption system that would work as a back-up system if the one we had been developing wasn't able to get finished in time.


## Trailer & Team Intro:

Finally, I also edited together the games trailer and team intro video for the senior show. Along with our designer Benji who got all the footage, I think they both came out rather nicely! You can view the trailer [here](https://www.youtube.com/watch?v=j-wnhaJXZI4&feature=youtu.be) and the team intro [here](https://www.youtube.com/watch?v=AlQz73QMAU0&feature=youtu.be).

---

# What Went Well?

## As A Team

One of the things that we developed towards the end of our development was implementing a scheduling system so everyone when the others were planning to work in-engine. This was not only useful since we were all working in one scene at the end, but also helped hold people accountable to get their work done earlier rather than later.

The way the team adapted to situations throughout development was very good. We were quick to adapt after Rockstars initial feedback and people were able to overcome unexpected challenges without too much struggle or complaining. Especially when we had to overall the entire game since it was way over-scoped for the time limit we had.


## Personally

Personally, I think I did a good job at avoiding crunch most of the time. Definitely not towards the end but throughout the semester I did my best to avoid crunching and tried to give myself a break when I felt like I might be starting to get burnt out.

I'm also rather proud of the tools I developed and how they were used. Especially the FOV tool. Even though it was initially made for the QA tool, it got used in so many other areas of the game and I've been able to reuse it for other projects already. It's definitely one of the most reusable and useful things I've developed.

---

# What Went Wrong?

## As A Team

One of the biggest issues our team had was communication. Others would often talk over people and not give the proper focus that people deserved when we were trying to do group projects like when we played the game as a group or tried to go through a burn down list to make sure no ones overworked. Side conversations would often start about the game that could've happened someplace else at a different time since it didn't require the entire team to be there.

I'd say our other biggest issue was being able to rely on others to get their work done when they said they would. We commonly had issues where others would rely on someone to get their work done at a specific time and that person wouldn't delivery which affected everyone else. This was especially an issue when it came to getting our art pass ready for the trailer, since it resulted in Benji and I having to crunch to get it all done in time since the art was finished way later than was promised. From our in-class postmortem, it sounded like this was an issue that everyone on the team had felt at some point.


## Personally

My main gripe is with how long it took to get the FOV tool to a fully functioning state. It was working for a while but the system we used to stop the AI looking through walls would often either be finicky or work for a while only to suddenly break. It's the one part of the tool that'd I'd like to try to go back and really polish up. I'm still not sure why we had so many issues with it, but I am glad it at least got done properly at the end.

---

Overall I think we made an amazing game that really was scary based on how testers responded and the recruiters/players at the Senior Show reacted. We were also able to fix so many bugs in such a short amount of time.