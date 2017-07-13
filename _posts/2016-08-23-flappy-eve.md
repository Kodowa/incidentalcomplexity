---
layout: post
title: "Flappy Bird (in Eve)"
author: "Joshua Cole"
tags: []
---

_(Editor's Note: Keep in mind as you're reading, this post is an executable Eve program. [Try it](https://github.com/witheve/Eve) yourself!)_

![Eve - Flappy Bird]({{ site.url }}/images/flippy-flop.gif)

When a player starts the game, we commit a `#world`, a `#player`, and some `#obstacles`. These will keep all of the essential state of the game. All of this information could have been stored on the world, but for clarity we break the important bits of state into objects that they effect.

- The `#world` tracks the distance the player has travelled, the current game screen, and the high score.
- The `#player` stores his current y position and (vertical) velocity.
- The `obstacles` have their (horizontal) offset and gap widths. We put distance on the world and only keep two obstacles; rather than moving the player through the world, we keep the player stationary and move the world past the player. When an obstacle goes off screen, we will wrap it around, update the placement of its gap, and continue on.

Add a flappy eve and a world for it to flap in

```eve
search
  [#session-connect]
commit
  [#player #self name: "eve" x: 25 y: 50 velocity: 0]
  [#world screen: "menu" frame: 0 distance: 0 best: 0 gravity: -0.061]
  [#obstacle gap: 35 offset: 0]
  [#obstacle gap: 35 offset: -1]
```

Next we draw the backdrop of the world. The player and obstacle will be drawn later based on their current state. Throughout the app we use resources from [#bhauman's flappy bird demo in clojure][1]. Since none of these things change over time, we commit them once when the player starts the game.

[1]: https://github.com/bhauman/flappy-bird-demo

Draw the game world!

```eve
search
  [#session-connect]
commit
  [#div style: [user-select: "none" -webkit-user-select: "none" -moz-user-select: "none"]  children:
    [#svg #game-window viewBox: "10 0 80 100", width: 480 children:
    [#rect x: 0 y: 0 width: 100 height: 53 fill: "rgb(112, 197, 206)" sort: 0]
    [#image x: 0 y: 52 width: 100 height: 43 preserveAspectRatio: "xMinYMin slice" href: "https://cdn.rawgit.com/bhauman/flappy-bird-demo/master/resources/public/imgs/background.png" sort: 1]
    [#rect x: 0 y: 95 width: 100 height: 5 fill: "rgb(222, 216, 149)" sort: 0]]]
```

### Screens

These following blocks handle drawing the game's other screens (such as the main menu and the game over scene).

The main menu displays a message instructing the player how to start the game.

```eve
search
  [#world screen: "menu"]
  svg = [#game-window]
bind
  svg.children += [#text x: 50 y: 45 text-anchor: "middle" font-size: 6 text: "Click the screen to begin!" sort: 10]
```

The "game over" screen displays the final score of the last game, the high score of all games, and a message inviting the player to play the game again.

```eve
search
  [#world screen: "game over" score best]
  svg = [#game-window]
bind
  svg.children += [#text x: 50 y: 30 text-anchor: "middle" font-size: 6 text: "Game Over :(" sort: 10]
  svg.children += [#text x: 50 y: 55 text-anchor: "middle" font-size: 6 text: "Score {{score}}" sort: 10]
  svg.children += [#text x: 50 y: 65 text-anchor: "middle" font-size: 6 text: "Best {{best}}" sort: 10]
  svg.children += [#text x: 50 y: 85 text-anchor: "middle" font-size: 4 text: "Click to play again!" sort: 10]
```

We haven't calculated the score yet, so let's do that. We calculate the score as the `floor` of the distance, meaning we just round the distance down to the nearest integer. If the distance between pipes is changed, this value can be scaled to search.

```eve
search
  world = [#world distance]
bind
  world.score := floor[value: distance]
```

When the game is on the "menu" or "game over" screens, a click anywhere in the application will (re)start the game. Additionally, if the current score is better than the current best, we'll swap them out now. Along with starting the game, we make sure to reset the distance and player positions in the came of a restart.

Start a new game

```eve
search
  [#click #direct-target]
  world = if world = [#world screen: "menu"] then world
          else [#world screen: "game over"]
  new = if world = [#world score best] score > best then score
        else if world = [#world best] then best
  player = [#player]
commit
  world <- [screen: "game" distance: 0 best: new]
  player <- [x: 25 y: 50 velocity: 0]
```

### Run the game

Now we need some logic to actually play the game. We slide obstacles along proportional to the distance travelled, and wrap them around to the beginning once they're entirely off screen. Additionally, we only show obstacles once their distance travelled is positive. This allows us to offset a pipe in the future, without the modulo operator wrapping it around to start off halfway through the screen.

Every 2 distance a wild obstacle appears

```eve
search
  [#world distance]
  obstacle = [#obstacle offset]
  obstacle-distance = distance + offset
  obstacle-distance >= 0
bind
  obstacle <- [x: 100 - (50 * mod[value: obstacle-distance, by: 2])]
```

When the obstacle is offscreen (`x > 90`), we randomly adjust the height of its gap to ensure the game doesn't play the same way twice. Eve's current random implementation yields a single result per seed per evaluation, so you can ask for `random[seed: "foo"]` in multiple queries and get the same result in that evaluation. In practice, this means that for every unique sample of randomness you care about in a program at a fixed time, you should use a unique seed. In this case, since we want one sample per obstacle, we just use the obstacle UUIDs as our seeds. The magic numbers in the equation just keep the gap from being at the very top of the screen or underground.

Readjust the height of the gap every time the obstacle resets

```eve
search
  [#world screen: "game"]
  obstacle = [#obstacle x > 90]
  height = random[seed: obstacle] * 30 + 5
commit
  obstacle.height := height
```

Next we draw the `#player` at its (x,y) coordinates. Since the player is stationary in x, setting his x position here dynamically is just a formality, but it allows us to configure his position on the screen when we initialize. We create the sprite first, then set the x and y positions to let us reuse the same element regardless of where the player is.
Draw the player

```eve
search
  svg = [#game-window]
  player = [#player x y]
  imgs = "https://cdn.rawgit.com/bhauman/flappy-bird-demo/master/resources/public/imgs"
bind
  sprite = [#image player width: 10 height: 10 href: "http://i.imgur.com/sp68LtM.gif" sort: 8]
  sprite.x := x - 5
  sprite.y := y - 5
  svg.children += sprite
```

Drawing obstacles is much the same process as drawing the player, but we encapsulate the sprites into a nested SVG to group and move them as a unit.

Draw the obstacles

```eve
search
  svg = [#game-window]
  obstacle = [#obstacle x height gap]
  bottom-height = height + gap
  imgs = "https://cdn.rawgit.com/bhauman/flappy-bird-demo/master/resources/public/imgs"
bind
  sprite-group = [#svg #obs-spr obstacle sort: 2 overflow: "visible" children:
    [#image y: 0 width: 10 height, preserveAspectRatio: "none" href: "{{imgs}}/pillar-bkg.png" sort: 1]
    [#image x: -1 y: height - 5 width: 12 height: 5 href: "{{imgs}}/lower-pillar-head.png" sort: 2]
    [#image y: bottom-height width: 10 height: 90 - bottom-height, preserveAspectRatio: "none" href: "{{imgs}}/pillar-bkg.png" sort: 1]
    [#image x: -1 y: bottom-height width: 12 height: 5 href: "{{imgs}}/lower-pillar-head.png" sort: 2]]
  sprite-group.x := x
  svg.children += sprite-group
```

When a player clicks during gameplay, we give the bird some lift by setting its velocity.

```eve
search
  [#click #direct-target]
  [#world screen: "game"]
  player = [#player #self]
commit
  player.velocity := 1.17
```

Next, we scroll the world in time with frame updates. Eve is currently locked to 60fps updates here, but this will probably be configurable in the future. Importantly, we only want to update the world state once per frame, so to ensure that we note the offset of the frame we last computed in `world.frame` and ensure we’re not recomputing for the same offset.

```eve
search
  [#time frames]
  world = [#world screen: "game" frame != frames gravity]
  player = [#player y velocity]
  not([#click])
commit
  world.frame := frames
  world.distance := world.distance + 1 / 60
  player <- [y: y - velocity, velocity: velocity + gravity]
```

Checking collision with the ground is very simple. Since we know the y height of the ground, we just check if the player's bottom (determined by center + radius) is below that point.

The game is lost if the player hits the ground.

```eve
search
  world = [#world screen: "game"]
  [#player y > 85] // ground height + player radius
commit
  world.screen := "game over"
```
Collision with the pipes is only slightly harder. Since they come in pairs, we first determine if the player is horizontally in a slice that may contain pipes and if so, whether we're above or below the gap. If neither, we're in the clear, otherwise we've collided.

The game is lost if the player hits an `#obstacle`

```eve
search
  world = [#world screen: "game"]
  [#player x y]
  [#obstacle x: obstacle-x height gap]
  ∂x = abs[value: obstacle-x + 5 - x] - 10 // distance between the edges of player and obstacle (offset of 1/2 obstacle width because origin is on the left)
  ∂x < 0
  collision = if y - 5 <= height then true
              else if y + 5 >= gap + height then true
commit
  world.screen := "game over"
```
