---
title: 'Achromatic Devblog #3'
published: 2022-05-05
description: ''
image: ''
tags: [Devblog]
category: 'Achromatic'
draft: false 
lang: ''
---

![](src/assets/images/achromatic/AchromaticDevblog3Cover.png)

For this post, I decided to share a bit about one of the main issues we had during development which was the horses AI. While we're using the same "check to see if the player is in their line of sight" for the fungal mannequins and the horse, the horse caused us the most issues due to the tightness and confined space the maze takes place in.

---

# What Is Achromatic?

Achromatic is a 3D psychological horror game for the PC where the player has to navigate around a 1950s inspired carnival while avoiding fungal mannequins and avoiding the horse monster in the maze.

---

# Overcoming A Challenge - The Monsters AI

![](src/assets/images/achromatic/AchromaticMonsterGIF.gif)

One of the biggest issues when it came to programming that I faced was making the monster horse AI fun yet realistic. We went through multiple iterations, with the biggest being single raycasts and multiple raycasts before ending up on using a spherecast. 


Initially we tried using a single raycast, and while it worked it did pose some issues. The big one being how finnicky it made the AI. Since it was only a single raycast (which is super thin), it wouldn't always be the most accurate. It was especially an issue since it prevented the player from being able to look around corners since the raycats would just barely hit the player despite the natural instinct to be that the monster shouldn't be able to see the player yet.

![](src/assets/images/achromatic/AchromaticSingleRaycast.png)

To combat this, initially we tried using multiple raycasts. With the help of getting the characters collision bounders, I was able to get the top, left, right, center and bottom location of the player/horse. From there I had a raycast shooting from each of those locations. However this caused other issues. One of them being do we say that the monster can see the player if all the raycasts can see him or just one? If it's just one then it'd give us the same issue we we're trying to fix, but if it was for all of them that could result in other situations where he can see the player but since one of the raycasts were false it wouldn't work right. We could say if half of the raycasts are true then the monster can see the player or something like that, but this could be hard to test to make sure it still runs properly.

![](src/assets/images/achromatic/AcharomticMultiRaycast.png)

So, we finally ended on using a spherecast. This allowed for a bigger area for the raycast to hit as well as made it so the player could look around corners without the monster instantly recognizing them.

![](src/assets/images/achromatic/AchromaticSpherecast.png)