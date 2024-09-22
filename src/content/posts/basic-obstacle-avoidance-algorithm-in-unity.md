---
title: Basic Obstalce Avoidance Algorithm in Unity
published: 2022-01-31
description: This post demonstrates how to include embedded video in a blog post.
tags: [Devlog, AI, Unity]
category: ''
draft: false
---

![Gif showing off the final product](src/assets/images/unity_obstacle_avoidance_basic/UnityBasicObstacleAvoidanceDemoGIF.gif)

I made an obstacle avoidance algorithm for my Advanced Game AI class. By using a mixture of an fov tool I made as well as raycasts, the agents go towards a specific target in the environment while avoiding any obstacles in its path including other agents.

---

# FOV Tool

![Gif showing off the FOV tool in the inspector](src/assets/images/unity_obstacle_avoidance_basic/UnityBasicObstacleAvoidanceFOVDemo.gif)

Originally when I was making the obstacle avoidance algorithm, I was using a set number of raycasts surrounding the object as an obstacle detector. However, I had recently made a field-of-view (fov) tool for my capstone (game production) class that I thought would work well for this algorithm.

GIF

This system has multiple benefits. Instead of being limited to a set number of raycasts, I was able to make it so it'd first check if an obstacle was within range of the agent at all, and if it was from there shoot a raycast from the agent to the obstacle to then use to calculate the math for the algorithm. This increases it's accuracy while also possibly saving on performance costs. Based on my research, raycasts are quite cost expensive while colliders aren't always as cost expensive. For example, a primitive collider (box, sphere etc.) is much more cost efficient than using raycasts since they're not really meshes but mathematical computations making it easier to verify intersections on Unity's end. Now my fov tool doesn't use a primitive collider, it uses a mesh collider which can get cost expensive very easily. But, I'm using a convex mesh collider, which helps in cost, plus the mesh itself is a very simple shape and not a very complex one. Based on this, it seems like this method could possibly be more cost efficient overall, mainly for huge games with hundreds or thousands of agents. 

---

# Avoiding Obstacles

When it comes to the actual avoidance algorithm itself, it first uses a trigger collider from the object that has the fov tool it (which is a child of the agent). While the object is inside the trigger collider, it'll call the `AvoidObstacle()` function from the `ObstacleAvoidance.cs` script (which is attached to the agent). 

```csharp
private void OnTriggerStay(Collider other)
{
    player.GetComponent<ObstacleAvoidance>().AvoidObstacle(other.transform.gameObject);
}
```

From there I take in the object it collided with as a parameter. Using the position of the agent and the object I find the direction. From there I shoot a raycast, store the raycast hit in a variable, and calculate the new direction by using the hit and a repelFroce float variable that adjusts the intensity at which the agent will avoid the obstacle.

```csharp
public void AvoidObstacle(GameObject obstacle)
{
    // Variables
    Vector3 origin = transform.position;

    Vector3 dest = obstacle.transform.position;
    Vector3 direction = (dest - origin).normalized;

    // Repel object
    obstacleHitBool = Physics.Raycast(transform.position, direction, out obstacleHit, fovObject.GetComponent<FOVTool>().distance);
    direction += obstacleHit.normal * repelforce;

    // Debug
    debugLine.enabled = true;
    Debug.DrawRay(transform.position, direction, Color.red);

    // Line renderer
    debugLine.startColor = colorHit;
    debugLine.endColor = colorHit;
    debugLine.SetPosition(0, origin);
    debugLine.SetPosition(1, dest);
}
```

Using the direction calculated in `AvoidObstacle()`, the `Move()` function updates the agents position and rotation. In this case I'm also resetting the x rotation and z rotation to 0 to avoid any unwanted rotation happening.

```csharp
void Move()
{
    // Variables
    if (obstacleHitBool) {
        origin = transform.position;
        dest = targetLoc.transform.position;
        direction = (dest - origin).normalized;
    }

    // Move
    transform.rotation = Quaternion.Slerp(transform.rotation, Quaternion.LookRotation(direction), rotateSpeed * Time.deltaTime);
    transform.eulerAngles = new Vector3(8, transform.eulerAngles.y, 0);
    transform.position += transform.forward * speed * Time.deltaTime;
}
```

Once the object leaves the fov, the debug gets turned off and the direction starts getting set in the `Move()` function instead.

```csharp
private void OnTriggerExit(Collider other)
{
    player.GetComponent<ObstacleAvoidance>().debugLine.enabled = false;
    player.GetComponent<ObstacleAvoidance>().obstacleHitBool = false;
}
```