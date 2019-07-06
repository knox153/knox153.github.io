---
layout: post
title: GD50 Lecture 06 - Angry Birds
author: Chee Yew Lim
keywords: Angry Birds, split the alien, game development, CS50, GD50 
date:   2019-07-06 22:55:22 -0700
summary: "split the alien that the player launched into three when spacebar is pressed"
categories: blog
---

> This is part of a series where I talk about how I approach the assignment portion of the [GD50 lecture][online-course]. These are written so that I can review them in the future without going through the code! Since this is an assignment, I won't be posting the code unless it is not related to the assignment. If you are stuck at one of these assignments, these posts should contain enough information to help you progress. Feel free to let me know if there is an error. :)

[Angry Birds][AngryBirds] - a classic mobile game where the player slingshots birds to destroy obstacles housing the pigs. A clone of this game was used in the lecture to demonstrate [Box2D][box2d] and Mouse Input.

1. [Split the alien into three](#split-the-alien-into-three)
2. [Bugs & Changes](#bugs--changes)

# Split the alien into three

The one and only task in the assignment is to spilt the alien into three when 'spacebar' is pressed, mimicking the ability of the [blue birds][blue] in the original game.

> TIL that each blue bird has a name: Jay, Jake, and Jim

I started off by creating two flags: `hasCollided` and `hasSplit`. The first flag is used to prevent the player from splitting the alien once it has collided with something, while the second one is to ensure that the player can only split the alien once. `hasCollided` flag will be set to true whenever a collision happens (the collision callback function).

If the alien has not collided/split and the space bar is pressed, we can split the alien by creating two more aliens above and below the original alien. The position of these new aliens is the position of the original alien with some Y offset. After the aliens are created, we have to set their velocity by calling `setLinearVelocity()` on their body in the Box2D's world. Instead of using the same velocity as the original alien, I adjusted the Y velocity by increasing the Y velocity for the alien above and decreasing the Y velocity for the alien below. This way, the aliens will look like they originated from one point.  

![Picture of the alien splitting from one point]({{ site.url }}/assets/img/GD50 - Angry Birds/point.png#center)
*Adjusted Y velocity for the effect on the right*

Similar to the original alien, we also have to set the new alien's restitution (bounciness) and angular damping (to slow down the rotation) using `fixture:setRestitution()` & `body:setAngularDamping()`. At this point, the splitting process is done and we are ready to set the `hasSplit` flag to true.

On every game loop, the game will remove the aliens and destroy their body in the Box2D's world if their velocity is low enough. When all of the aliens on the field are removed, we reset the slingshot by creating another `AlienLaunchMarker` to replace the previous one. At the same time, we reset both of the flags (`hasCollided` and `hasSplit`) back to false.

![Gif of the alien splitting into three]({{ site.url }}/assets/img/GD50 - Angry Birds/split.gif)
*Alien splitting into three!*

# Bugs & Changes

**Alien spawning below the ground**

If the alien is close to the ground when it splits, one of the aliens might spawn beneath the ground:

![Gif of alien spawning underground when it splits]({{ site.url }}/assets/img/GD50 - Angry Birds/bug.gif)
*Alien spawning underground when it splits*

This happens because the ground is just a horizontal line. When we split the alien, the game simply creates another alien below the original alien, and this process can place it below the ground. To fix this bug, `math.min` is used to set the minimum height of the alien to just above the ground.

```lua
-- Added math.min to make sure the alien spawns above the ground when it splits
-- ((VIRTUAL_HEIGHT - 35) - 35) is the ground height minus the alien's height
-- Previous code: Alien(self.world, 'round', xPos, yPos + 30), 'Player')
Alien(self.world, 'round', xPos, math.min((VIRTUAL_HEIGHT - 35) - 35, yPos + 30), 'Player')
```
{: data-title="/Level.lua" .code-title }

A relatively short post as the assignment only has one task in it. For the upcoming post, we will have a look at the infamous Pokemon!

---

[AngryBirds]: https://en.wikipedia.org/wiki/Angry_Birds
[box2d]: https://en.wikipedia.org/wiki/Box2D
[blue]: https://angrybirds.fandom.com/wiki/Jay,_Jake,_and_Jim
[online-course]: https://courses.edx.org/courses/course-v1:HarvardX+CS50G+Games/course/
