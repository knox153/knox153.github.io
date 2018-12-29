---
layout: post
title: Flappy Bird
author: Chee Yew Lim
keywords: Flappy Bird, Medal, Random, pipe, pause, game development, CS50, GD50 
date:   2018-12-28 22:55:22 -0700
summary: "Randomize pipe in Flappy Bird, implement pause feature, adding medal, and some bugs fix"
categories: blog
---

On the second lecture ([flappy bird][flappy-bird]), we learned how to use image/spirit and implement infinity scrolling (by looping the image at one point). Games are illusions in a way that we only see what the games want us to see (many games used this to improve performance and save memory by only rendering visible object). [Procedural Generation][procedural-generation] was introduced to generate the pipes. A state machine class was also used to manage the states instead of using a string. 

Just like last time, I will start with the assignment portion then moved on to bugs I discovered and some changes I made. 

1. [Randomizing the pipe](#randomizing-the-pipe)
2. [Medal](#medal)
3. [Pause feature](#pause-feature)
4. [Bugs & Changes](#bugs--changes)

# Randomizing the pipe
The first task in the assignment is to make the pipes generation even more random by changing the gap between the upper and lower pipes and the time to spawn the next pipes pair.   

When changing the gap height (the distance between upper and lower pipe), be cautious not to change the gap height constant outside of the game loop because this will only randomize it once and all the pipes will have the same gap height afterwards. We will want to randomize it everytime a pipe pair is spawned/initialized instead.

![Gif of random gap height]({{ site.url }}/assets/img/gapHeight.gif)
*Randomize gap height*

Similar concept to the last one, to randomize the time to spawn the next pipes pair, we can just use a variable that will be randomized whenever a pipe pair is spawned. Friendly reminder that "math.random(lower, upper)" will only produce an integer. Float number can be achieved with "lower + math.random() * (upper - lower)".  

![Gif of random time to spawn]({{ site.url }}/assets/img/timeToSpawn.gif)
*Randomize time to spawn the next pipes* 

# Medal
This is quite simple, all we have to do is to add the sprites and render the correct medal on the score state screen depending on the player's score. 

![Picture of medal on score screen]({{ site.url }}/assets/img/medal.png)
*Bronze medal on the score screen*

*Bonus*! I made three medals sprites using [Aseprite][aseprite] and you can download them for free [**here**]({{ site.url }}/assets/file/medals.zip). :)

![Picture of gold, silver, bronze medal]({{ site.url }}/assets/img/allMedal.png)
*All three medals*
 
# Pause feature
I have considered two ways to do this. Starting with the easier one, we can just add a boolean variable to keep track of the game is being paused or not. If it is paused, nothing will be done in the update function. Otherwise, it will update all the entities as usual.  

Another more elegant solution is by adding a new state - PauseState. I decided to went with this route because I wanted to have more practice with the state machine class, and this felt like a cleaner way to do it. The code will be encapsulated in a different file making it easier to read and modify! Whenever the player press "p", the game will transition from play state to pause state, and vice versa.

Whenever we change state, everything from the previous state will be "deleted" unless it is being passed as a parameter in the changeState function. We will use this to save the previous state's variable so that the game can be continued from where it was paused. The main steps breakdown: 

1. When going from the PlayState to PauseState, store all variables (bird, pipes etc...) of the PlayState in a table and pass it as a parameter in the changeState function. 
2. Now the enterState function that will be executed upon entering the PauseState will receive the table from step 1. 
3. We can then store the table in a variable for the PauseState
4. Whenever we are going back to PlayState, the table that is being stored in a variable can be passed back to it as a parameter.  
5. In the enterState function of the PlayState, we can check if there is any parameter. If there is none (which means this is the first time entering the PlayState), we then intialize everything to default. On the other hands, if there is a parameter (which means it is coming from pauseState), we would want to initialize it to the value in the table. 

{% include video.html file="pause.mp4" attr="autoplay controls loop muted" %}
*Pause feature!*

> Side story: Initially, I couldn't get the pause feature to work even though all the codes looks fine. Eventually, I found out that the PlayState class already has an enterState function at the bottom of the code, thus replacing my enterState function at the top of the code <sup><sup>facepalm</sup></sup>. Don't be me haha, always check if there is an existing function before writing a new one.  

# Bugs & Changes

**Speed of the bird is depending on the frame**
```lua
-- Original code
self.dy =  self.dy + GRAVITY * dt
if love.keyboard.wasPressed('space') then
	self.dy =  -5
end
self.y =  self.y +  self.dy
```
{: data-title="/Bird.lua" .code-title }

The original last line of the code does not rely on "dt" (time passed since last frame) to calculate the y position. This is an issue because the bird will moves faster on a faster machine, and a slower machine will have a slower bird. Frame independent physic is always an important thing to consider in game development. Just look at this speed:   

![Gif of bird moving super duper fast when frame is high]({{ site.url }}/assets/img/vsyncOff.gif)
*This is running at around 400+fps*  

This can be easily fixed by adding "dt" into the equation. The value now is going to be a lot smaller (because dt is usually a small fraction of a second). So we need to scale the gravity constant and the "self.dy" so that it will match the previous speed. 

```lua
-- adding dt and chaging the constant
local GRAVITY = 1000

self.dy =  self.dy + GRAVITY * dt
if love.keyboard.wasPressed('space') then
	self.dy =  -275
end
self.y =  self.y +  self.dy * dt
```
{: data-title="/Bird.lua" .code-title }

Of course, it doesn't have to be 1000 and -275, but these two values are what I found to be very similar to the previous movement.

**Bird can fly over the pipe**  
In the current implementation of the game, the collision between the bird and pipe is done using [AABB][aabb]. However, because the pipe has a limited height, the bird can fly all the way above the pipe and avoid all the pipes easily. 

![Picture of bird flying over the pipes]({{ site.url }}/assets/img/flyOver.png)
*Flying over the pipes!*

One solution is to change how the collision is being calculated and make sure the bird will collide with the pipe despite the height. 

Another solution that can be used is by adding a skybox (by just flipping the ground upside down and render it on top). We already have pipes coming out of the sky, so adding a "ground" to the sky isn't that ridiculous, right? 

```lua
GROUND_HEIGHT = ground:getHeight() -- this line is in main.lua
love.graphics.draw(ground, -groundScroll, GROUND_HEIGHT, 0, 1, -1)

-- reset if we reach the sky. -10 so that the hitbox is more forgiving. 
if  self.bird.y < GROUND_HEIGHT -  10  then
	sounds['explosion']:play()
	sounds['hurt']:play()

	gStateMachine:change('score', {
	score =  self.score
	})
end
```
{: data-title="/PlayState.lua" .code-title }

Ground height is stored in a variable so that we only have to call the getHeight function once (slight performance increase). Don't forget to move the skybox down once we flipped it upside down. And be sure to move down the scoreboard and the upper pipes so that the skybox doesn't cover them. Bonus: we are reusing the asset as well! 

If you have read everything until the end, well done and thank you!

[flappy-bird]: https://en.wikipedia.org/wiki/Flappy_Bird
[aseprite]: https://www.aseprite.org/
[aabb]: https://en.wikipedia.org/wiki/Minimum_bounding_box#Axis-aligned_minimum_bounding_box
[procedural-generation]: https://en.wikipedia.org/wiki/Procedural_generation