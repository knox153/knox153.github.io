---
layout: post
title: GD50 Lecture 05 - The Legend of Zelda
author: Chee Yew Lim
keywords: The Legend of Zelda, enemy drop heart, pick up pot, projectile, game development, CS50, GD50 
date:   2019-5-04 22:55:22 -0700
summary: "implement enemies drop heart randomly upon death, adding pot that the player can pick up, players can throw the pot and use it to hurt the enemy"
categories: blog
---

> This is part of a series where I talk about how I approach the assignment portion of the [GD50 lecture][online-course]. These are written so that I can review them in the future without going through the code! Since this is an assignment, I won't be posting the code unless it is not related to the assignment. If you are stuck at one of these assignments, these posts should contain enough information to help you progress. Feel free to let me know if there is an error. :)

Back with another GD50 post! As the title says, this post is about ["The Legend of Zelda"][Zelda]. In the lecture, we learnt about [top-down perspective][top-down], infinite dungeon generation, [hitbox/hurtbox][hitbox], [events][event], screen scrolling, and data-driven design.

1. [Enemies drop heart randomly](#enemies-drop-heart-randomly)
2. [Pot that can be picked up](#pot-that-can-be-picked-up)
3. [Using pot as a projectile](#using-pot-as-a-projectile)
4. [Bugs & Changes](#bugs--changes)

# Enemies drop heart randomly

The first task in the assignment is to spawn a heart randomly when the player kills an enemy, and the heart heals the player. I started off by defining a heart object in the `game_objects.lua` file. This file contains the properties of the game objects and has no code in it (utilizing the data-driven design we learnt in the lecture!).

The remaining code is then written in a separate file called `Room.lua`. The overall approach is kinda similar to the last assignment. When an enemy's health goes below 1, `math.random()` is used to randomly create a heart object and insert it into our object table.

The heart contains an `onCosume()` function that will be executed when the player collides with the heart. This function will increment the player's health and play a sound. And make sure it does not go over the maximum health (3 full heart).

Finally, once the `onCosume()` function is executed, the heart will be removed from the objects table. This will prevent the player from consuming the heart multiple times.

![Gif of enemy spawning heart and it increases the player's health]({{ site.url }}/assets/img/GD50 - The Legend of Zelda/heart.gif)
*Enemy has a chance to drop heart*

# Pot that can be picked up

Following that, we need to add a pot to the game. This pot can be picked up by the player and move around. There are quite a few pieces that needed to be implemented, so let's break it down:

- Define a pot object in the `game_objects.lua` file just like the heart
- Create a pot object and insert it into the objects table in `Room.lua`
- Define the player's carry animation and texture in `entity_defs.lua`
- Create two new states: `PlayerCarryIdleState` and `PlayerCarryWalkState`
- These two new states will be very similar to `PlayerIdleState` and `PlayerWalkState`, but they will have different animation and the player won't be able to swing the sword while carrying the pot.
- Ensure the player will move to `PlayerCarryIdleState` when "Enter" is pressed and there is a pot in front of them
- The pot will track the player's location if the player is in any of the carry states.

While generating the pot's spawn location, if it collides with the switch, another location will be generated until it no longer collides with the switch. Otherwise, there is a small possibility that the pot will be on the switch and prevents the player from using the switch.

In `Room.lua`, I added code to check if the player is colliding with solid objects (objects that contain the solid property in `game_objects.lua`). If they are indeed colliding, it will reverse the movement to prevent the player from phasing through the solid object. Exactly like what we did to prevent player walking through the wall, except this time we apply it on the objects (e.g. the pot).

The pot has two states in my implementation: `Idle` state and `Carry` state. This allows the game to execute different logic based on the state of the object. Whenever the player carries the pot, the pot will enter the `Carry` state. In this state, the pot's location will always equal to the player's location minus some Y offset (giving the illusion that the player is "carrying" the pot). In addition, the solid property of the pot will be changed to false. As a result, the pot won't collide with anything while it is being carried.

When the player pressed "Enter" and there is a pot in front of them, the player will move to `PlayerCarryIdleState`. But how do we know if a pot is in front of the player? Well, I created a hitbox (basically a rectangle) based on the direction of the player is facing, and used the hitbox to check if it is colliding with the pot. If they did collide, the pickup animation will be played once and the pot will be changed to `Carry` state before moving the player into `PlayerCarryIdleState`. `self.player.currentAnimation.timesPlayed` is used here to check whether the pickup animation is completed.

On the other hand, if the hitbox collides with nothing, the player will simply move back to `PlayerIdleState` after the pickup animation is completed. Since the switch is an object too, the player can pick up it if we don't do something about it. This can be prevented by checking the object's type before picking it up. Alternatively, you can also define a new property "pickable" if you have a bunch of objects with different types.

![Gif of player picking up the pot and move it around]({{ site.url }}/assets/img/GD50 - The Legend of Zelda/pot.gif)
*Player can pick up the pot and move it around*

# Using pot as a projectile

After picking up the pot, the player can choose to throw it in a straight line. The pot will disappear after hitting a wall, travelling more than 4 tiles, or hitting an enemy.

To do this, we gonna add a few more properties to the pot object in `game_objects.lua`: "dx", "dy", and "broke". The first two are the speed of the pot, and `broke` is used to keep track whether the pot should be removed from the game. A new state - `Moving` is added to the pot as well.

Then, if the "Enter" button is pressed in `PlayerCarryIdleState` or `PlayerCarryWalkState`, we will change the pot's state to `Moving` and the player's state to `PlayerIdleState`. The destination and the direction of the projectile will be stored in the pot as well. Both of these will be used in the next part to move the pot.

Once we have the speed, direction, and destination of an object, the `GameObject:update(dt)` function can be updated to move the object if its state is `Moving`. While the object is moving, if it hits a wall or reaches the destination (4 tiles away from the origin), its `broke` properties will be set to `true`.

Don't forget to check if the pot collides with an enemy. This can be done in `room.lua`. If the object's state is `Moving` and the object is not `broke`, it will reduce the enemy's health by one when they collide. Once again, the object is set to `broke` at the end of the function.  

When an object is `broke`, I simply set its coordinate to somewhere off-screen. I felt like this is an acceptable approach since there aren't too many objects in a single room. I could have removed the object from the object table, but the object table will be reset anyway when the player moves to the next room.

![Gif of player throwing the pot and how it breaks]({{ site.url }}/assets/img/GD50 - The Legend of Zelda/projectile.gif)
*Pot has different ways to break!*

# Bugs & Changes

**Objects are one pixel off compared to the player**

While playtesting the game, I noticed that the pot is not exactly above the player even though they have the same x position:

![Picture of a pot slight above and right side of the player]({{ site.url }}/assets/img/GD50 - The Legend of Zelda/bug1.png#center)
*Pot is on the right side of the player*

This is because the object class and the player class have different rendering function! And it can be fixed easily by making them consistent:  

```lua
-- added math.floor to make it consistent with the player's render function
function GameObject:render(adjacentOffsetX, adjacentOffsetY)
    love.graphics.draw(gTextures[self.texture], gFrames[self.texture][self.states[self.state].frame or self.frame],
    math.floor(self.x + adjacentOffsetX), math.floor(self.y + adjacentOffsetY))
end
```
{: data-title="/GameObject.lua" .code-title }

**Animation is played for one extra frame**

This bug is a bit harder to notice, but I managed to record the animation and slow it down for demonstration:

![Gif of animation's first frame being played again]({{ site.url }}/assets/img/GD50 - The Legend of Zelda/bug2.gif)
*First frame of the animation is repeated*

Noticed how the game will play the first frame of the sword swinging animation again before returning to the idle animation.

In the `PlayerSwingSwordState`, the player moves back to the idle state once the animation of the sword swinging is completed once. But the game checks whether the animation is completed in the `self.stateMachine:update(dt)` function *before* the animation's update function is called! To fix this, simply swap the sequence of these update functions.

```lua
-- change the sequence and let the animation update first
if self.currentAnimation then
    self.currentAnimation:update(dt)
end

self.stateMachine:update(dt)
```
{: data-title="/Entity.lua" .code-title }

This is it! These features are always quite fun to implement, and I can't wait to work on the next one.

---

[Zelda]: https://en.wikipedia.org/wiki/The_Legend_of_Zelda
[top-down]: https://en.wikipedia.org/wiki/Video_game_graphics#Top-down_perspective
[hitbox]: https://en.wikipedia.org/wiki/Hitbox
[event]: https://en.wikipedia.org/wiki/Event_(computing)
[online-course]: https://courses.edx.org/courses/course-v1:HarvardX+CS50G+Games/course/
