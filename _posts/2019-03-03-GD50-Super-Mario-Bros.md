---
layout: post
title: GD50 Lecture 04 - Super Mario Bros.
author: Chee Yew Lim
keywords: Super Mario Bros, spawn above ground, generate key, generate lock, spawn goal post, regenerate longer level, game development, CS50, GD50 
date:   2019-3-03 22:55:22 -0700
summary: "always spawn a player above the ground, generate a key that can be used to unlock a lock, unlocking lock will spawn a flag post, touching flag post regenerate a new longer level"
categories: blog
---

> This is part of a series where I talk about how I approach the assignment portion of the [GD50 lecture][online-course]. These are written so that I can review them in the future without going through the code! Since this is an assignment, I won't be posting the code unless it is not related to the assignment. If you are stuck at one of these assignments, these posts should contain enough information to help you progress. Feel free to let me know if there is an error. :)

[Super Mario Bros.][SMB] is the game we take a look this time! Topics that were discussed include [tile map][TileMap], animation, platformer physic, basic enemy AI, and procedural level generation.

1. [Spawn the player above the ground](#spawn-the-player-above-the-ground)
2. [Generate a key and a lock](#generate-a-key-and-a-lock)
3. [Spawn a goal post](#spawn-a-goal-post)
4. [Regenerate a new longer level](#regenerate-a-new-longer-level)
5. [Bugs & Changes](#bugs--changes)

# Spawn the player above the ground

At the beginning of a level, the player is spawned at the top left corner of the screen. But because the level generation is random, the player might be spawned above a chasm and fall to death. Thus, the first task of the assignment is to always spawn the player above the ground. This can be done easily by using a loop to check each column and see if there is any tile along the Y-axis. We can make this more efficient by only checking the bottom tile of the column! Once a tile is found, the loop will break, and this will be the x position the player spawn at.

![Gif of player spawning on ground]({{ site.url }}/assets/img/GD50 - Super Mario Bros/spawnOnGround.gif)
*Always spawn on top of the ground*

# Generate a key and a lock

Next up, we will generate a key and a lock block on the map. If the player doesn't have a key, colliding with the lock block will do nothing. On the other hand, the lock block will disappear if the player has the key. All of this is done in the `Level.generate()` function. This function generates the level column by column starting from left to right.

Breaking it down:

- Determine the location to spawn the key and lock block at the start of `Level.generate()` function
- While generating the level, if the location to spawn the key/lock is a chasm, increment it by 1
- Spawn the key/lock when the level generator reaches the correct column
- If a lock block has been spawned, don't spawn the normal block to replace it
- Make sure the lock disappears if the player has the key and collide with it

The first step is to determine where to spawn the key and lock. We could have spawned them randomly on the map, but then the player might have to do some backtracking (especially annoying when the key is at the end of the level and the lock block is at the beginning). So, what I did to reduce the backtracking is by only spawning the key on the first half of the level and spawning the lock block around the middle of the level. This can be done via `math.random(math.floor(width / 2))`.

Since retrieving a key or lock block can be difficult when they are above a chasm, I decided not to spawn the key/lock on top of a chasm. If the location to spawn a key/lock is on a chasm, I simply increment their location by 1. Once the level generator reaches the location to spawn the key/lock, we generate them by creating a GameObject (like how a normal block and gem is generated in the level). And a boolean is used to keep track whether the lock has been spawned as we don't want to spawn another normal block on top of the lock.

Let's talk about how the gem disappear when the player collides with it. Every time a player consumes a gem, the `onConsume()` function will be called. After the `onConsume()` function is called, the game will remove the object from the objects table. This means we can 'piggyback' on this and implement this into the lock block! If the player has the key, the lock block becomes consumable and will disappear if the player collides with it.

![Gif of key and lock spawning and their interaction]({{ site.url }}/assets/img/GD50 - Super Mario Bros/keyAndLock.gif)
*The key unlocks the lock!*

# Spawn a goal post

After the lock block disappears, we want to spawn a goal post at the end of the level. This can be done in the `onConsume()` function of our lock block. Once the lock block is consumed, GameObjects that represents the goal post will be created. Since the goal post is made up of multiple parts, more than one GameObject will be created (i.e. create a flag GameObject and multiple poles GameObject)!

# Regenerate a new longer level

Finally, when the player collides with the goal post, a new longer level will be generated. All we have to do is to reload the play state in the `onCollide()` function of the flag GameObject to make a new level. Remember to pass in the player's score and the current level length as a parameter into the new Playstate to retain them. In the `PlayState:enter(params)` function, we can define that if there is no parameter (which means it is the first level), then it would call the `LevelMaker.generate(length)` with a default length. Otherwise, `LevelMaker.generate(params.length)` will be called and generate a longer level.

![Gif of regenerating new level after touching the goal post]({{ site.url }}/assets/img/GD50 - Super Mario Bros/newLevel.gif)
*Generating a longer level after touching goal post*

# Bugs & Changes

**Unwinnable level**

![Picture of the bug]({{ site.url }}/assets/img/GD50 - Super Mario Bros/bugBlock.png#center)
*Lock spawning in between two pillars*

Random level generation can be fun, but we also need to be careful about generating an unwinnable level. This bug does not happen very often, but it still needs to be fixed regardless. Since we already decided the location of the lock block at the beginning of the level generation, what we can do then is to explicitly 'tell' the program to not generate a pillar on the previous column of the lock block. 

```lua
-- a chance to generate a pillar
    if not (x == width - 2) and not (x == blockLocation - 1) and math.random(8) == 1 then
        -- spawn a pillar
        ...
    end
```
{: data-title="/LevelMaker.lua" .code-title }

`width - 2` is the location of the goal post and `blockLocation - 1` is the column right before the lock block. These two locations are the spot where we don't want a pillar to spawn. Otherwise, there is a 1/8 chance for the level generator to spawn a pillar on a column. A similar approach is also used to prevent the key to spawn in between two pillars with a normal block on top.

This post is slightly shorter than usual, and I think that is because most of the stuff here felt straight forward and does not require much explanation. But I still hope this is informative! Till next time!

[SMB]: https://en.wikipedia.org/wiki/Super_Mario_Bros.
[TileMap]: https://developer.mozilla.org/en-US/docs/Games/Techniques/Tilemaps
[online-course]: https://courses.edx.org/courses/course-v1:HarvardX+CS50G+Games/course/