---
title: (Partially) Recreating Genshin Impacts Movement System Postmortem
published: 2022-05-02
description: 'The partial recreation of genshin impacts movement system in Unity'
image: ''
tags: [Postmortem]
category: 'College'
draft: false 
lang: ''
---

![](src/assets/images/unity_genshin_impact_movement_system/GIMSStatesGIF.gif)
*Gameplay from Genshin Impact, Footage from Indie Waffles*

For my final project in portfolio, I wanted to follow this tutorial I had found online that went in depth to recreate the major part of Genshins movement system. While the tutorial is a "still-in-progress" series, the first part of it had finished which goes over the main chunk of the movement system (with the other parts focused on swimming and flying). While I don't mind coding movement in 2D games, I've never been quite amazing at it, especially in 3D games, so I thought Id' use this class and the tutorial as a chance to develop my skills when it comes to programming more gameplay type programming like movement. While I wasn't able to finish the entire series on time, I did only have half the time to finish this assignment since I switched to it partway through. Taking that into consideration I'm happy with what got developed in time and plan on finishing it in my free time.

---

# Recreating (Part of) The Movement System

![](src/assets/images/unity_genshin_impact_movement_system/GIMSStatesList.png)
*Graphic by Indie Waffles*

Genshin Impact uses a hierarchal state machine to program their movement, among other things. I was able to get in the idle, dash, walk, run and sprint states in time. Due to all the states in Genshin, there were quite a lot of scripts and code going on. I wasn't sure just how to quite break down so many scripts talking to each other but I tried to pull out the essentials.

![](src/assets/images/unity_genshin_impact_movement_system/GIMSScriptsList.png)

Each state has it's own data script which holds all the variables needed for each state. Below is an example of what these data scripts look like. All the movement states are pretty similar in terms of the variables they store. The capsule collider data is used to create a floating capsule so the player can go up slopes. The player layer data stores what layers should be ignored (part of the floating capsule system) and the slope data of course holds data to help regulate the player walking/falling/sliding on slopes.

![](src/assets/images/unity_genshin_impact_movement_system/GIMSScriptableObjectValues.png)

![](src/assets/images/unity_genshin_impact_movement_system/GIMSCurve.png)

The player scriptable object is where all the real data gets stored and adjusted for the states. As you can see each state has it's own section for its data file. The walk, run, sprint and dash state all have their own speed modifier. The sprint and dash data also have variables that help adjust the time between transitioning states. For slopes an animation curve was implemented.

![](src/assets/images/unity_genshin_impact_movement_system/GIMSPlayerInsecptor.png)

For other things like the collider and slopes that information is dealt with in the Player.cs script. While there's a lot of code and math going on along all these different scripts, I've included snippets of the main movement code to give an overview of the movement.

```csharp
private void Move()
{
    // Not moving
    if (stateMachine.ReusableData.MovementInput == Vector2.zero || stateMachine.ReusableData.MovementSpeedModifier == 0)
    {
        return;
    }

    // Move
    Vector3 movementDirection = GetMovementDirection();
    float targetRotationYAngle = Rotate(movementDirection);
    Vector3 targetRotationDirection = GetTargetRotationDirection(targetRotationYAngle);
    float movementSpeed = GetMovementSpeed();
    Vector3 currentPlayerHorizontalVelocity = GetPlayerHorizontalVelocity();
    stateMachine.Player.PlayerRigidbody.AddForce(targetRotationDirection * movementSpeed - currentPlayerHorizontalVelocity, ForceMode.VelocityChange);

    // NOTE :: AddForce() happens next update, changing velocity is instantaneous
    // NOTE :: Vector multiplication takes resources, so for optimization make sure you always multiply the vector last (ex. float * float * vector)
}
```

```csharp
protected void RotateTowardsTargetRotation()
{
    float currentYAngle = stateMachine.Player.PlayerRigidbody.rotation.eulerAngles.y;

    if (currentYAngle != stateMachine.ReusableData.CurrentTargetRotation.y)
    {
        float smoothedYAngle = Mathf.SmoothDampAngle(currentYAngle, stateMachine.ReusableData.CurrentTargetRotation.y, ref
            stateMachine.ReusableData.DampedTargetRotationCurrentVelocity.y, stateMachine.ReusableData.TimeToReachTargetRotation.y -
            stateMachine.ReusableData.DampedTargetRotationPassedTime.y);

        return;
    }

    stateMachine.ReusableData.DampedTargetRotationPassedTime.y += Time.deltaTime;
    Quaternion targetRotation = Quaternion.Euler(0f, smoothedYAngle, 0f);

    stateMachine.Player.PlayerRigidbody.MoveRotation(targetRotation);
}
```

```csharp
protected float UpdateTargetRotation(Vector3 direction, bool shouldConsiderCameraRotation = true)
{
    // Variables
    float directionAngle = Mathf.Atan2(direction.x, direction.z) * Mathf.Rad2Deg;

    // Convert negative degrees to positive
    if (directionAngle < 0)
    {
        directionAngle += 360f;
    }

    // Add camera rotation to angle
    if (shouldConsiderCameraRotation)
    {
        directionAngle += stateMachine.Player.MainCameraTransforn.eulerAngles.y;
        if (directionAngle > 360f)
        {
            directionAngle -= 360;
        }
    }

    if (directionAngle != stateMachine.ReusableData.CurrentTargetRotation.y)
    {
        stateMachine.ReusableData.CurrentTargetRotation.y = directionAngle;
        stateMachine.ReusableData.DampedTargetRotationPassedTime.y = 0;
    }

    return directionAngle;
}
```

## Move()

The `Move()` function is the main function that calls all the other functions that take care of all the math. While it calls multiple functions (most being just one line of code and a return) I've included the two biggest ones below it. The most important part is that at the very end it takes all the data accumulated and adds it to the rigidbody. 

## RotateTowardsTargetRotation()

he `RotateTowardsTargetRotation()` first checks to see if the new rotation is different the last rotation and if not it exits out of the function. Otherwise, it moves onto using `Mathf.SmoothDampAngle()` and the variables created in the data scripts to calculate the y angle which is then used to create the target location and called in `MoveRotation()`.

## UpdateTargetRotation()

The `UpdateTargetRotation()` first calculates the direction of the angle. From there it checks if the value is less than 0 and if it is it converts the negative number into the positive equivalent. From there it takes into account the cameras rotation and if it doesn't equal the latest target rotation then it updates the target rotation. 

All together it helps create a movement system that looks like this:

![](src/assets/images/unity_genshin_impact_movement_system/GIMSMoveAndCameraGIF.gif)
*Demo of the movement and camera together*

![](src/assets/images/unity_genshin_impact_movement_system/GIMSMoveGIF.gif)
*Demo of just the movement*

![](src/assets/images/unity_genshin_impact_movement_system/GIMSCameraGIF.gif)
*Demo of just the camera*

For the floating capsule collider, along side the data script, there's also 2 main functions being used to calculate the updates to the collider. `CalculateCapsuleColldierDimensions()` is being used to not only calculate the new collider dimensions but also ensure the height of the capsule is at least twice it's radius to reduce the chances of bugs caused by odd numbers that break the collider. 

```csharp
public void CalculateCapsuleColliderDimensions()
{
    SetCapsuleColliderRadius(DefaultColliderData.Radius);
    SetCapsuleColliderHeight(DefaultColliderData.Height * (1f - SlopeData.StepHeightPercentage));

    RecalculateCapsuleColliderCenter();

    // Keep height at least double of the radius
    float halfColliderHeight = CapsuleColliderData.Collider.height / 2f;
    if (halfColliderHeight < CapsuleColliderData.Collider.radius)
    {
        SetCapsuleColliderRadius(halfColliderHeight);
    }

    CapsuleColliderData.UpdateColliderData();
}
```

The `RecalculateCapsuleColliderCenter()` is used to update the center of the collider before the colliders dimensions are checked and adjusted further if necessary.

```csharp
public void RecalculateCapsuleColliderCenter()
{
    float colliderHeightDifference = DefaultColliderData.Height - CapsuleColliderData.Collider.height;
    Vector3 newColliderCenter = new Vector3(0f, DefaultColliderData.CenterY + (colliderHeightDifference/2f), 0f);
    CapsuleColliderData.Collider.center = newColliderCenter; // Measured in local space
}
```

By using these functions, we're able to create a slope system that looks like this:

![](src/assets/images/unity_genshin_impact_movement_system/GIMSSlopesGIF.gif)

While the dash has been implemented, it's not perfect since there's no exit state currently (since animations have not been implemented) so you can go into a dash you'll just also continuously dash forever.

![](src/assets/images/unity_genshin_impact_movement_system/GIMSInfiniteDashGIF.gif)

---

# What Have I Learned?

While I didn't have the chance to finish it all in time, I've definitely learned a lot so far. Out of everything I'd say there's 4 main things I've learned.

## State Machines:

While I knew roughly what state machines were before starting this project, I had never implemented one before. I had always heard of it in the context of AI so I had never even thought that it could be used outside of that. But now that I've learned more about state machines and how to not only implement a state machine but also a hierarchal state machine I can definitely see myself using them much more in the future. Especially seeing how useful it is when creating even a basic movement system. I've already started implementing state machines for my AI final which is about recreating a simplified version of the enemy system in Genshin (which you can read about here once it's available).

## Floating Capsule:

Besides learning about creating a nice movement system in general, I also learned about how to properly implement a floating capsule which would've been really useful last semester when we kept messing with the code to get the player to move on slopes smoothly. It's definitely a method I'll be referring back to in the future whenever I need it. (If you're interested in the game I was referring to, it's called Warplight Wanderer and you can read about what I did here).

## Regions:

I don't think I had ever really heard about # regions before this project. If I remember right I recently did a freelance project called Chimera Hunt where I had to work with other people's code that I didn't know at all and I believe I might've seen this implemented there, but in general I had never really learned what this was before. It's just a simple way to help keep your code organized, especially for really long scripts with lots of functions. I use it all the time now!

## Different Coding Styles:

Lastly, the style of coding this person has is rather different from my own. I've worked with lots of other programmers but their style in general has never been drastically too different usually. This time though he tended to do things backwards from how I would usually do them. It was very interesting and always took me an extra second to wrap my head around what was happening but I got more used to it as time went on. It's not the biggest deal or anything, and I don't think I'll be coding the way they do necessarily, but I just thought it was interesting.

---

## References

![Indie Waffles Genshin Impact Movement Tutorial Series](https://www.youtube.com/playlist?list=PL0yxB6cCkoWKuPoh_9dSvdItQENVx7YTW)