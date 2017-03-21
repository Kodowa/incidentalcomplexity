---
layout: post
title: "Example 5 - Battle School"
author: "Rob Attorri"
tags: []
---

![Example 5]({{ site.url }}/images/ex5.png)

### What is this?

This example demonstrates saving state across different pages, and advanced sorting using the `range[]` function. On the "control" page, you can set a schedule of army matchups for the day. Click on an army and then click on the slot you want to place that army. This matchup is saved, and displayed on the "broadcast" page.

### Page Layout

#### Containers

This one's pretty simple; I want a nav bar at the top to let me manually go to the different pages, and an app window where those pages are rendered.

```
commit @browser
  [#div class:"nav-bar"]
  [#div class:"app-window"]
```

#### Pages

For every page I want, I make a record with an attribute of `target` to specify which each page is called.

```
commit
  [#page target:"Broadcast"]
  [#page target:"Control"]
```

#### Nav Bar Buttons

By abstracting this step out, I can define all the pages I want as in the block before, and then here find those pages and create a button on the nav bar for each of them.

```
search @session @browser
  nav-bar = [#div class:"nav-bar"]
  [#page target]

bind @browser
  nav-bar <- [children: [#button target text:target]]
```

This partner block works with the nav buttons by simply setting the app window to whichever nav button is clicked.

```
search @session @browser @event
  [#click element: [#button target]]
  window = [#app-window]

commit
  window.target := target
```

#### Starting Page

This commits a record called #`app-window` whose attribute `target` specifies which page gets rendered. In this case, when the app starts up, I want the landing view to be the Broadcast page.

```
commit
  [#app-window target:"Broadcast"]
```

#### Sci-Fi Font

It's not the future if there's not futuristic-looking text.

```
commit @browser
  [#link href:"https://fonts.googleapis.com/css?family=Orbitron|Play" rel:"stylesheet"]
```

### Broadcast

#### Drawing the Page

The Broadcast page is meant to serve much like the departures or arrivals screen at an airport - it is purely an informative screen showing which armies have been scheduled in which rooms. There are 9 total battle rooms, so I use the `range` function to generate 9 rooms, each of which has two sides labeled A and B. There's also a #`versus` block whose text will change based on whether or not there are armies scheduled to that room.

```
search @session @browser
  [#app-window target:"Broadcast"]
  window = [#div class:"app-window"]
  i = range[from: 1, to: 9]

bind @browser
  window.class += "broadcast"
  window.children += [#div i class:"battle-room" children:
      [#div class:"room-title" text:"{{i}}"]
      [#div class:"matchup" children:
        [#room class:"teamA" room:i side:"A"]
        [#versus room:i]
        [#room class:"teamB" room:i side:"B"]
      ]
    ]
```

#### Drawing Upcoming Battles

This block checks to see if there's an army assigned to a room and will inject a card for that army into its room slot, but only if there's another army that has been assigned to the opposing side in that room. Having only an A side or a B side will fail the search. If both sides are accounted for however, the search will pass for both of those armies, and so both cards will get injected.

```eve
search @session @browser
  slot = [#room room side]
  army = [#army room side name color1 color2 color3 uniform]
  other-side = if side = "A" then "B"
               else if side = "B" then "A"
  [#army room side:other-side]
  versus = [#versus room]

bind @browser
  slot.class += ("army-card")
  slot <- [#div name children:
    [#div class:"army-name" text:name]
    [#div class:"ribbon" children:
      [#div class:color1]
      [#div class:color2]
      [#div class:color3]]
  ]
```

If a room has two armies assigned to it, one to each side, then the #`versus` block becomes just the text "VS" between the two army cards.

```
search @session @browser
  versus = [#versus room]
  [#army room side:"A"]
  [#army room side:"B"]

bind @browser
  versus <- [#div class:"versus-text" text:"VS"]
```

If a room is missing either an A or a B side, or both, I want a message to be displayed saying that a battle hasn't been scheduled yet. Since this comprises a union in set theory, I need two blocks to explicity handle all my possibilities. This first one looks to see if both sides have yet to be assigned, and injects the message into the #`versus` block if that happens to be the case.

```
search @session @browser
  versus = [#versus room]
  not([#army room side:"A"])
  not([#army room side:"B"])

bind @browser
  versus <- [#div class:"empty-room" text:"No battle scheduled"]
```

This second block gives us the other half of the union by checking to see if either an A side or a B side has been assigned, but not its complement, and will inject the same message into the #`versus` block.

```
search @session @browser
  versus = [#versus room]
  army = [#army room side]
  other-side = if side = "A" then "B"
               else if side = "B" then "A"
  not([#army room side:other-side])

bind @browser
  versus <- [#div class:"empty-room" text:"No battle scheduled"]
```

### Control

#### Drawing the Page

The Control page is laid out to be displayed on a smaller device - perhaps a smart phone or a tablet - and is used to assign which armies will fight in which rooms. Again, for the nine rooms, the range function is used to identify and number them, and each room is given two sides. A list of all the available armies - that is, those which have not been assigned to a room - is also drawn.

```
search @session @browser
  [#app-window target:"Control"]
  [#army name color1 color2 color3 uniform not(room)]
  i = range[from: 1, to: 9]
  window = [#div class:"app-window"]

bind @browser
  window.class += "control"
  window.children += ([#img class:"logo" src:"http://vignette2.wikia.nocookie.net/ansible/images/6/69/InternationalFleetLogo.png"],
    [#div class:"control-lists" children:
    [#div class:"all-rooms" children:
      [#div class:"room" i children:
        [#div class:"title" text:"Battle Room {{i}}"]
        [#div room:i side:"A" class:"battle-slot"]
        [#div class:"vs-line" text:"vs"]
        [#div room:i side:"B" class:"battle-slot"]
      ]
    ],
    [#div class:"all-armies" children:
      [#div class:"title" text:"Armies"]
      [#div sort:name name class:("army-tab", uniform) text:name]
    ]
    ])
```

As an addition to main drawing block for the Control page, this block checks to see if there are any armies assigned to a slot in a battle room. If it is, that army has its tab drawn in the corresponding slot in the browser.

```
search @session @browser
  window = [#div class:"battle-slot" room side]
  [#army name uniform room side]

bind @browser
  window.children += [#div class:("army-tab", uniform) text:name]
```

#### Selecting

In order to be able to assign armies to slots, there needs to be a mechanism to select both elements. I've chosen to demonstrate selection in two different ways to cover some of the possibilities of how this might be achieved, and because armies each have a record stored in the session database, I used them to show selection by modifying a session record. The intended workflow here is to click an army to highlight it, then choose a slot to assign them to, or go the other way around and click a battle slot to highlight it, then choose an army to assign there. That means I only want to highlight an element if there's nothing else already selected - otherwise, I'm probably trying to assign an army. This block searches for a click on an army tab and, as long as that army isn't already highlighted, nor any battle slots, adds the #`highlighted` tag to the record of the clicked army.

```
search @session @event @browser
  [#click element: [#div class:"army-tab" name]]
  army = [#army name not(#highlighted)]
  not([#div #highlighted class:"battle-slot" room])

commit
  army += #highlighted
```

This block adds the #`highlighted` tag to a record in browser instead of session. If a battle slot gets clicked and isn't already highlighted and doesn't have an army already assigned to it, then that battle slot becomes highlighted.

```
search @session @event @browser
  battle-slot = [#div class:"battle-slot" room side]
  [#click element: battle-slot]
  not([#div #highlighted class:"battle-slot" room side])
  not([#click element:[#div class:"army-tab"]])
  not([#army #highlighted])

commit @browser
  battle-slot += #highlighted
```

#### Assigning

Once an army is #`highlighted`, if a battle slot gets clicked, the highlighted army gets assigned to that particular slot, which corresponds to both a `room` and a `side`.

```
search @session @event @browser
  army = [#army #highlighted]
  [#click element: [#div class:"battle-slot" room side]]

commit
  army.room := room
  army.side := side
```

On the flip side, if a battle slot is #`highlighted` and an army gets clicked, the clicked army gets assigned to the highlighted slot.

```
search @session @event @browser
  [#div #highlighted class:"battle-slot" room side]
  army = [#army name]
  [#click element: [#div class:"army-tab" name]]

commit
  army.room := room
  army.side := side
```

#### Unselecting

A click anywhere deselects anything. This works if you're just clicking around the page and want to deselect something, or if you click to assign an army somewhere. The assigning workflow still occurs, but the final click deselects everything so that no residual highlights are left over.

```
search @session @event @browser
  [#click]
  highlighted = [#highlighted]

commit @session @browser
  highlighted -= #highlighted
```

#### Unassigning

If there's an army in a slot that's clicked, this block removes that army from that slot.

```
search @session @event @browser
  [#click element: [#div class:"battle-slot" room side]]
  in-slot = [#army room side]

commit
  in-slot.room := none
  in-slot.side := none
```

#### Highlight Styling

When either an army or a battle slot is #`highlighted`, I want to add a class to it so I can use CSS to add a visual marker to it. Because I add the #`highlighted` tag to armies and battle slots differently, I need two different blocks to handle those classes. In the case of a highlighted army, the army is highlighted but its corresponding army tab gets the new class apended.

```
search @session @browser
  army-tab = [#div class:"army-tab" name]
  army = [#army #highlighted name]

bind @browser
  army-tab.class += "highlighted"
```

In the case of a highlighted battle slot, the #`highlighted` tag is already on the #`div` that needs the new class, which gets apended with this block.

```
search @session @browser
  battle-slot = [#highlighted class:"battle-slot" room]

bind @browser
  battle-slot.class += "highlighted"
```

### Army Data

Each army is listed here with its name, its three colors, and a uniform color.

```
commit
  [#army name:"Manticore" color1:"gray" color2:"yellow" color3:"green" uniform:"yellow"]
  [#army name:"Asp" color1:"lightgreen" color2:"blue" color3:"green" uniform:"green"]
  [#army name:"Scorpion" color1:"purple" color2:"orange" color3:"red" uniform:"orange"]
  [#army name:"Flame" color1:"red" color2:"yellow" color3:"red"  uniform:"red"]
  [#army name:"Tide" color1:"blue" color2:"lightblue" color3:"blue" uniform:"lightblue"]
  [#army name:"Salamander" color1:"green" color2:"lightgreen" color3:"brown" uniform:"lightgreen"]
  [#army name:"Rat" color1:"black" color2:"brown" color3:"black" uniform:"brown"]
  [#army name:"Hound" color1:"blue" color2:"brown" color3:"purple" uniform:"brown"]
  [#army name:"Condor" color1:"black" color2:"white" color3:"black" uniform:"gray"]
  [#army name:"Squirrel" color1:"green" color2:"gray" color3:"blue" uniform:"gray"]
  [#army name:"Rabbit" color1:"white" color2:"gray" color3:"red" uniform:"red"]
  [#army name:"Leopard" color1:"orange" color2:"brown" color3:"orange" uniform:"brown"]
  [#army name:"Centipede" color1:"orange" color2:"blue" color3:"red" uniform:"blue"]
  [#army name:"Phoenix" color1:"yellow" color2:"orange" color3:"red" uniform:"yellow"]
  [#army name:"Dragon" color1:"gray" color2:"orange" color3:"gray" uniform:"orange"]
  [#army name:"Ferret" color1:"white" color2:"lightblue" color3:"black" uniform:"lightblue"]
  [#army name:"Badger" color1:"red" color2:"white" color3:"black" uniform:"red"]
  [#army name:"Griffin" color1:"yellow" color2:"brown" color3:"purple" uniform:"purple"]
  [#army name:"Tiger" color1:"orange" color2:"black" color3:"white" uniform:"orange"]
  [#army name:"Spider" color1:"green" color2:"black" color3:"purple" uniform:"purple"]
```

### Styles

There's a lot of CSS this time around because of a greater need for media queries on this example, so it's been split up into the style sheets for each page

```css
{for a good time, leave this here}
```

#### Broadcast Page
```css

.program {
  margin: 0px;
  padding: 0px;
}

.broadcast {
  height: 100%;
  background: #333;
  display: flex;
  flex-direction: column;
  overflow: scroll;
  user-select: none;
}

.battle-room {
  color: white;
  font-family: "Play", sans-serif;
  text-transform: uppercase;
  text-align: center;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  padding: 5px 0px;
  position: relative;
}

.room-title {
  color: #316282;
  position: absolute;
}

.matchup {
  display: flex;
  flex-direction: row;
  flex: 0 0 auto;
}

.army-card {
  display: flex;
  flex-direction: row;
  align-items: center;
  font-family: "Play", sans-serif;
  transform: skew(-20deg);
  background: #777;
}

.empty-room {
  color: #555;
}

.teamB > .army-name {
  order: 2;
}

.teamB > .ribbon {
  order: 1;
}

.teamB > .ribbon div {
  border-left: 0px solid #222;
  border-right: 1px solid #222;
}

.army-name {
  flex: 1 0 auto;
  text-transform: uppercase;
  transform: skew(20deg);
}

.ribbon {
  box-sizing: border-box;
  display: flex;
  flex-direction: row;
}

.ribbon div {
  flex: 1 0 auto;
  border-left: 1px solid #222;
}

.red {
  background: #EE0034;
  color: white;
}

.orange {
  background: #FF7900;
  color: black;
}

.yellow {
  background: #FCCC0A;
  color: black;
}

.green {
  background: #00933C;
  color: white;
}

.lightgreen {
  background: #6CBE45;
  color: black;
}

.blue {
 background: #0039A6;
 color: white;
}

.lightblue {
  background: #00A1DE;
  color: white;
}

.purple {
  background: #A626AA;
  color: white;
}

.black {
  background: #000000;
  color: white;
}

.white {
  background: #ffffff;
  color: black;
}

.gray {
  background: #A7A9AC;
  color: white;
}

.brown {
  background: #996633;
  color: white;
}

@media (max-width: 1279px) {

.broadcast {
  width: 100%;
  max-width: 250px;
  min-width: 220px;
  overflow: scroll;
}

.battle-room {
  border-bottom: 1px solid #999;
  flex: 0 0 67px;
}

.room-title {
  font-size: 20px;
  line-height: 20px;
  left: 8px;
}

.matchup {
  flex-direction: column;
}

.army-card {
  height: 20px;
  width: 130px;
}

.versus-text {
  font-size: 10px;
  line-height: 16px;
}

.empty-room {
  font-size: 14px;
  line-height: 14px;
}

.army-name {
  font-size: 12px;
}

.ribbon {
  height: 20px;
  width: 30px;
}

}

@media (min-width: 1280px) and (max-width: 1499px) {

.broadcast {
  width: 100%;
  max-width: 550px;
  overflow: scroll;
}

.battle-room {
  flex: 1 0 55px;
}

.room-title {
  font-size: 28px;
  line-height: 24px;
  left: 8px;
}

.army-card {
  height: 30px;
  width: 200px;
}

.versus-text {
  font-size: 16px;
  line-height: 30px;
}

.empty-room {
  font-size: 14px;
  line-height: 14px;
}

.army-name {
  font-size: 18px;
}

.ribbon {
  height: 30px;
  width: 50px;
}

.teamA {
  margin-right: 10px;
}

.teamB {
  margin-left: 10px;
}

}

@media (min-width: 1500px) and (max-width: 1679px) {

.broadcast {
  width: 100%;
  max-width: 750px;
}

.battle-room {
  flex: 1 0 100px;
}

.room-title {
  font-size: 48px;
  line-height: 48px;
  left: 10px;
}

.army-card {
  height: 40px;
  width: 280px;
}

.versus-text {
  font-size: 20px;
  line-height: 40px;
}

.empty-room {
  font-size: 24px;
}

.army-name {
  font-size: 22px;
}

.ribbon {
  height: 40px;
  width: 90px;
}

.teamA {
  margin-right: 12px;
}

.teamB {
  margin-left: 12px;
}

}

@media (min-width: 1680px) {

.broadcast {
  width: 100%;
  max-width: 960px;
}

.battle-room {
  flex: 1 0 120px;
}

.room-title {
  font-size: 60px;
  line-height: 40px;
  color: #316282;
  position: absolute;
  left: 20px;
}

.army-card {
  height: 40px;
  width: 320px;
  display: flex;
  flex-direction: row;
  align-items: center;
  font-family: "Play", sans-serif;
  transform: skew(-20deg);
  background: #777;
}

.versus-text {
  font-size: 30px;
  line-height: 40px;
}

.empty-room {
  font-size: 30px;
  line-height: 40px;
  color: #555;
}

.teamA {
  margin-right: 20px;
}

.teamB {
  margin-left: 20px;
}


.army-name {
  font-size: 24px;
}

.ribbon {
  height: 40px;
  width: 100px;
}

}
```

#### Control Page
```css
.control {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #333;
  width: 450px;
  overflow: scroll;
  position: relative;
  user-select: none;
}

.logo {
  width: 100px;
  margin: 25px 0px;
}

.control-lists {
  display: flex;
  width: 400;
}

.title {
  font-size: 20px;
  line-height: 20px;
  text-transform: uppercase;
  text-align: center;
  font-family: "Play", sans-serif;
  order: -10;
  margin-bottom: 5px;
}

.all-rooms {
  color: white;
  flex: 1 0 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.room {
  width: 200px;
  margin-bottom: 30px;
  flex: 0 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.battle-slot {
  height: 26px;
  width: 162px;
  border-radius: 4px;
  border: 1px solid white;
  cursor: pointer;
}

.vs-line {
  line-height: 16px;
  font-size: 16px;
}

.all-armies {
  color: white;
  flex: 1 0 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.army-tab {
  width: 160px;
  height: 24px;
  font-size: 20px;
  line-height: 24px;
  text-align: center;
  text-transform: uppercase;
  border-radius: 4px;
  margin-bottom: 10px;
  font-family: "Play", sans-serif;
  cursor: pointer;
}

.highlighted {
  box-shadow: 0px 0px 10px 5px #bbb;
}

.battle-slot.highlighted {
  background: #bbc;
}

@media (max-width: 1279px) {

.battle-slot {
  height: 22px;
  width: 82px;
}

.logo {
  width: 80px;
  margin: 18px 0px;
}

.title {
  font-size: 12px;
  line-height: 14px;
  width: 70px;
  margin-bottom: 2px;
}

.control {
  width: 200px;
}

.control-lists {
  width: 200;
}

.all-rooms {
  flex: 1 0 80px;
}

.room {
  width: 80px;
  margin-bottom: 20px;
}

.vs-line {
  line-height: 12px;
  font-size: 10px;
}

.all-armies {
  flex: 1 0 80px;
  margin-top: 14px;
}

.army-tab {
  width: 80px;
  height: 20px;
  font-size: 12px;
  line-height: 20px;
}

}
```