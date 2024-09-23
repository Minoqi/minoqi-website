---
title: 'Achromatic Devblog #1'
published: 2022-03-10
description: 'The first devblog for my capstone project Achromatic!'
image: ''
tags: [Devblog]
category: 'Achromatic'
draft: false 
lang: ''
---

![](src/assets/images/achromatic/AchromaticOldMainMenu.png)

Mini postmortem on the first 5 and a half sprints (5 and a half weeks) of the development of Achromatic. What I've done, what I plan on doing, what's gone well and what we need to improve on.

---

# What I've Done

Being the designated tools programmer and second AI programmer, I've mainly been working on our QA tool as well as helping implement some AI. 

## QA Tool

The QA Tool is a tool that's being made to help the team gather more accurate testing data from players. While it's planned to have multiple parts, I've currently added in an FOV Tool, to see what the player is seeing, as well as a Time Tracking Tool. 

The FOV Tool is a simple field-of-view gameobject, that can be attached as a child of the parent. A mesh is generated for the tool based on a list of variables that adjusts the distance, angle and height of the object. From there, it's set to a trigger and anytime an object enters the trigger it's considered "seen" by the player. While this current system works, I also plan on adding an extra check using raycasts to see if the object is obstructed by the players view (ex. like a key in a drawer, enemy behind a wall etc.)

![](src/assets/images/unity_obstacle_avoidance_basic/UnityBasicObstacleAvoidanceFOVDemo.gif)

The Time Tracking Tool is the tool that actually allows us to know how long the player is staring at something. While the FOV Tool is used to check if it's within the players FOV, the Time Tracking Tool records how long the player is looking at each object. Currently, this tool records during the entire duration of the player in the scene it's in. Once the game gets closed, the data gets automatically saved onto a CSV file locally. This will get changed in the future to getting saved on google drive instead.

![](src/assets/images/achromatic/AchromaticTimeToolGIF.gif)

## Mannequin AI

Our game has a few enemies, with one of them being the mannequins. These guys chase the player when the player isn't looking and stop when they are. Using the FOV Tool as well as a Behavior Tree plug-in my team had bought, I was able to create a simple AI. 

![](src/assets/images/achromatic/AchromaticFOVToolDEBUG.gif)

![](src/assets/images/achromatic/AchromaticFOVToolDEMO.gif)

---

# What I Plan On Doing Next

While both of these things are at a solid point, I still plan on further developing both the QA Tool and Mannequin AI to fit our needs.

## QA Tool

When it comes to the FOV Tool, while I'd consider this almost done, I do want to add raycasts. Right now if an object was behind something like a wall, the tool wouldn't know and would just think the player can still see it which is something I plan on fixing in the future.

For the Time Tracking Tool, adding in an extra section where the object not only records when it's within the players FOV but also is within the range of the camera in general could be useful for some more accurate data. This can be used to let the designers know how long the player may have had an object inside the camera range before it made it to their FOV (if it ever did or if maybe they never took a closer look).  I also plan on implementing part of a tool a friend made to make it so the CSV file will get sent to google drive and create a google sheet of the data automatically instead of saving locally. This would also include a custom code each player has to make it easier to track the data from the google forms to the testing data.

Outside of these tools, I also plan on adding some other tools including a heatmap and player ghost. The heatmap would get saved as a mesh that designers can then bring into Unity to get a quick view at where the player spent their most time, where they didn't visit much and even places they just never visited. The player ghost would ideally record the players movement and allow the designers to playback this to get an easier idea at what the player was doing at different times.

## Mannequin AI

While the mannequin AI is at a decent state, there are some touch-ups that will most likely get added. Mainly making it so the mannequins can't just run to the player from anywhere, but instead make it so they have to be a certain distance or maybe as long as they can't see the player then they won't chase after them. This will of course vary on the designers, but it is something that may get implemented to make sure the AI isn't unfair.

---

# What's Gone Well

## As A Team

As a team, I think the best thing we dealt with was with how the team reacted after meeting with Rockstar about our game. After the meeting we decided it'd be best to revert the game back  to the way it was during Greenlight the semester prior. To be fair, most of the programming wasn't scraped since it could still be used and I believe a lot of the art is still useable but I still think everyone was very professional when it came to just accepting to let things go and move on to better the game. 

## Personally

In terms of my own personal work I do like how the QA Tool has come out. Because I'm always thinking about the efficiency of code while also thinking about making it reusable in many scenarios, it's allowed us to save a lot of time already. The FOV Tool has been easily reused for the mannequin AI as well as other AI and it's even already come in handy in projects outside of Achromatic. I'm also rather proud on how the Time Tracking Tool has turned out so far and I already know it'll be useful for tons of future projects.


I also think I've done a good job at recognizing the signs of starting to feel burnt out and giving myself the time to rest to avoid the burnout, instead of forcing myself to keep working even if it means I didn't get to do as much as I normally would've liked. 

---

# What Could Go Better

## As A Team

As a team, I think our biggest weakness is communication. This mainly breaks down to two issues of the entire team not always being on the same page about what everyone else is doing and where the game is going as well as people talking over each other, interrupting people while they're talking and ignoring people when they're trying to get the entire teams attention for an announcement or important statement.

Early on we divided the team into two smaller scrum teams since it was impossible to get everyone to have a meeting outside of class time with everyone's schedules. So while splitting the team into two isn't necessarily a bad idea and I think was the right thing to do, we haven't done a good job at making sure both teams are aware of what the other team is doing. We do have leads from both teams who go to meetings to update each other but this information doesn't usually get back to the other team members whether it be from a document or a team meeting. I think if we could have some sort of document or meeting time where the leads update us on what's happening, it would greatly help solve this issue.

The other issue is when everyone talks over each other, a lot. We'll break off into groups during class but whenever (usually a producer) tries to get the teams attention again to discuss things, a lot of people will just keep talking and it takes someone yelling to get everyone's attention and even then some people will still just keep talking which is very rude. I've noticed this problem seems to boil down to a few people who will keep talking and the people around them will just keep listening instead of telling them that they need to continue the conversation later. There's also the issue of someone talking and then someone just starts talking over them. I think these issues could be resolved by the old talking stick, only those with the stick can talk and everyone else should be quite. While I think getting everyone's attention can understandably be difficult, setting set times like "you will have 15 min to discuss things in groups but then we're meeting back here to discuss things", could work well since I've heard it works for other teams. Having that statement with a timer that goes off to get everyone's attention I think could be a good tactic.  

## Personally

Even though preventing burnout is of course a very good idea, it is also annoying when it can take away time from making the game. Overall, I don't feel like I've accomplished as much as I normally would've by now which may just be me having too high standards for myself, but it's personally annoying and something I want to fix in the following weeks.


Also, even though I believe I'm usually pretty good at scraping, reworking or putting an idea on hold when it starts taking too long, I missed the mark this time when it came to the FOV Tool. Like I said earlier, I plan on adding raycasts to get a more accurate FOV for the player. I have however already attempted to do this and ran into a lot of issues that were taking me a very long time to figure out. I switched between multiple methods and spent a while trying to parse through my code logic and see where it is that it could be screwing up. While this isn't terrible, I did spend 2 sprints really focused on trying to get this right when I think after a sprint and a half I should've just put it on the back burner, called what we had good enough for now and work towards other parts of the tool we needed to get it to a useable state sooner for QA sessions.