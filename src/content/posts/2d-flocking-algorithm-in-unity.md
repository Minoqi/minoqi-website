---
title: 2D Flocking Algorithm in Unity
published: 2022-02-14
description: A simple example of a Markdown blog post.
tags: [Devblog, Unity, AI, Algorithm]
category: ''
draft: false
---

![](src/assets/images/unity_2d_flocking_algorithm/Unity2DFlockingAlgorithmDemoGIF.gif)

I made a 2D flocking algorithm for my Advanced Game AI class, but the code can be easily converted to work in 3D as well. Using a mix of the 3 main behaviors in flocking (cohesion, alignment and avoidance), a few extra behaviors including group flocking and obstacle avoidance, plus composite to put it all together, we get a decently expandable flocking algorithm. Above is an example of it in action, and at the bottom you can see an example of the flocks avoiding each other as well. You can access the code here.

---

# Flock

The `Flock.cs` script has two main functions, `Update()` and `GetNearbyObjects()`.


***Update:*** Get all the data from the behaviors, use it to calculate the movement and tell the agent the final move value.


***GetNearbyObjects:*** Uses physics overlap circle to check for all other objects nearby.

```csharp
// Update is called once per frame
// Unity Message | 0 references
void Update()
{
    foreach (FlockAgent agent in agents)
    {
        List<Transform> context = GetNearbyObjects(agent);

        Vector2 move = behavior.CalculateMove(agent, context, this); // Run scriptable object to calculate move
        move *= driveFactor; // For speedier movement

        if (move.sqrMagnitude > squareMaxSpeed) // Don't pass max speed
        {
            move = move.normalized * maxSpeed;
        }

        agent.Move(move);
    }
}
```

```csharp
private List<Transform> GetNearbyObjects(FlockAgent agent)
{
    List<Transform> context = new List<Transform>();
    Collider2D[] contextColliders = Physics2D.OverlapCircleAll(agent.transform.position, neighborRadius);

    foreach (Collider2D col in contextColliders)
    {
        if (col != agent.AgentCollider) // Not own collider
        {
            context.Add(col.transform);
        }
    }

    return context;
}
```

---

# Behaviors

Flocking is made up of 3 behaviors: cohesion, alignment and avoidance. There's also composite which takes data from all 3 of these and calculates the final movement values. The 3 main behaviors are pretty similar code wise wise, mainly being that they all check to make sure there are neighbors, otherwise it'll just return nothing as well as checking to see if any special filtering was done (This is used for things like finding their own flock as well as obstacle avoidance).

***Cohesion:*** Cohesion adds up all the positions and gets the average plus the offset.

```csharp
[CreateAssetMenu(menuName = "Flock/Behavior/Cohesion")]
using UnityEngine;
using System.Collections.Generic;

public class CohesionBehavior : FilteredFlockBehavior
{
    public override Vector3 CalculateMove(FlockAgent agent, List<Transform> context, Flock flock)
    {
        // If no neighbors, return zero
        if (context.Count == 0 || filter.Filter(agent, context).Count == 0)
        {
            return Vector3.zero;
        }

        // Add all points together and average
        Vector3 cohesionMove = Vector3.zero;
        List<Transform> filteredContext = null;

        if (filter == null) // If not filtering, use original context list
        {
            filteredContext = context;
        }
        else
        {
            filteredContext = filter.Filter(agent, context);
        }

        foreach (Transform item in filteredContext)
        {
            cohesionMove += (Vector3)item.position; // Add all points
        }
        cohesionMove /= filteredContext.Count; // Average point

        // Create offset from agent position
        cohesionMove -= (Vector3)agent.transform.position;

        return cohesionMove;
    }
}
```

***Alignment:*** Alignment adds up all the directions the agents are facing and gets the average.

```csharp
[CreateAssetMenu(menuName = "Flock/Behavior/Alignment")]
public class AlignmentBehavior : FilteredFlockBehavior
{
    public override Vector3 CalculateMove(FlockAgent agent, List<Transform> context, Flock flock)
    {
        // if no neighbors, return current heading
        if (context.Count == 0 || filter.Filter(agent, context).Count == 0)
        {
            return agent.transform.up;
        }
        
        // Add all points together and average
        Vector2 alignmentMove = Vector2.zero;
        List<Transform> filteredContext = null;

        if (filter == null) // If not filtering, use original context list
        {
            filteredContext = context;
        }
        else
        {
            filteredContext = filter.Filter(agent, context);
        }

        foreach (Transform item in filteredContext)
        {
            alignmentMove += (Vector2)item.transform.up; // Get direction facing
        }

        alignmentMove /= filteredContext.Count; // Average direction
        return alignmentMove;
    }
}
```

***Avoidance:*** Avoidance calculates all the differences in positions between the agent and other objects surrounding it, which is then averaged by the number of things the agents avoiding.

```csharp
public override Vector CalculateMove(FlockAgent agent, List<Transform> context, Flock flock)
{
    // If no neighbors, return no adjustment
    if (context.Count == 0 || filter.Filter(agent, context).Count == 0)
    {
        return Vector2.zero;
    }

    Vector avoidancemove = Vector2.zero;

    int numAvoid = 8; // Number of boids to avoid
    List<Transform> filteredContext = null;

    if (filter == null) // If not filtering, use original context list
    {
        filteredContext = context;
    }
    else
    {
        filteredContext = filter.Filter(agent, context);
    }

    foreach (Transform item in filteredContext)
    {
        if (Vector2.SqrMagnitude(item.position - agent.transform.position) < flock.squareAvoidanceRadius)
        {
            Debug.Log(Vector2.SqrMagnitude(item.position - agent.transform.position));
            numAvoider;
            avoidanceMove += (Vector2)(agent.transform.position - item.position);
        }
    }
    return avoidanceMove;
}
```

***Composite:*** Composite takes all this data and calculates the movement off of it as well as the weight set to each behavior. There's also a check just to make sure the behavior doesn't exceed that amount of weight it should have.

```csharp
// Set up move
Vector2 move = Vector2.zero;

// Iterate through behaviors
for (int i = 8; i < behaviors.Length; i++)
{
    Vector2 partialMove = behaviors[i].CalculateMove(agent, context, flock) * weights[i]; // Multiply each behavior by its weight
    if (partialMove != Vector2.zero)
    {
        if (partialMove.sqrMagnitude > weights[i] * weights[i]) // Make sure it doesn’t exceed the weight
        {
            partialMove.Normalize();
            partialMove *= weights[i];
        }
        move += partialMove;
    }
}

return move;
```

---

# Check for Specific Flock

To keep track of agents that are part of their flock, they just check all nearby objects and store all the ones that have the same flock type in a new list. Each agent gets designated a flock at the start of runtime through the Flock.cs script. The Flock.cs script is attached the an empty gameObject that holds all the flocks. The flock type is decided by whatever prefab you use in the Flock.cs script in the inspector.

```csharp
[CreateAssetMenu(menuName = "Flock/Filter/Same Flock")]
using UnityEngine;
using System.Collections.Generic;

public class SameFlockFilter : ContextFilter
{

    public override List<Transform> Filter(FlockAgent agent, List<Transform> originalList)
    {

        List<Transform> filtered = new List<Transform>();

        foreach (Transform item in originalList) // Iterate through original list
        {
            FlockAgent itemAgent = item.GetComponent<FlockAgent>();
            if (itemAgent != null && itemAgent.AgentFlock == agent.AgentFlock) // Make sure it's flock agent & in same flock
            {
                filtered.Add(item);
            }
        }

        return filtered;
    }
}
```

---

# Obstacle Avoidance

To avoid obstacles, there's a `Obstacle Avoidance Filter Scriptable Object`. The scriptable object has it so you can set what layer you want the flock to avoid, and the script will get a list of all objects on that layer based on objects nearby. From there this object is added to the `Avoidance Behavior Scriptable Object` where all the math happens.

```csharp
[CreateAssetMenu(menuName = "Flock/Filter/Obstacle Avoidance")]
using UnityEngine;
using System.Collections.Generic;

public class ObstacleAvoidanceFilter : ContextFilter
{
    // Variables
    public LayerMask layerToAvoid;

    public override List<Transform> Filter(FlockAgent agent, List<Transform> originalList)
    {
        List<Transform> filtered = new List<Transform>();

        foreach (Transform item in originalList) // Iterate through original list
        {
            if (layerToAvoid == (layerToAvoid | (1 << item.gameObject.layer))) // If it's on a layer to avoid, add to list
            {
                filtered.Add(item);
            }
        }

        return filtered;
    }
}
```

You can see from the gif at the start of this devblog, the obstacle avoidance working for walls. Since it's based on layers, as long as you give each flock it's own layer you can make flocks avoid each other as well.

![](src/assets/images/unity_2d_flocking_algorithm/Unity2DFlockingAlgorithmTighterDemo.gif)

---

# Inspector

Lastly, through the inspector, the user can adjust a wide array of values for their flocks.


`Behavior`: A composite behavior scriptable object that has all the behaviors you want the flock to take into account. The composite behavior has a list of all the behaviors and their weight values.


`Starting Count`: Number of agents you want in the flock.


`Agent Density`: Used to calculate spawning.


`Drive Factor`: A value that gets multiplied to the movement to speed up the agent.


`Max Speed`: Max speed the agent can go to after all the calculations have been done.


`Neighbor Radius`: How far the agent is checking for things.


`Avoidance Radius Multiplier`: Used to calculate avoidance.

![](src/assets/images/unity_2d_flocking_algorithm/Unity2DFlockingAlgoFlockInpsector.png)

![](src/assets/images/unity_2d_flocking_algorithm/Unity2dFlockingAlgoScriptable.png)