---
layout: post
title: "Example 1 - Crazy Mike's"
author: "Rob Attorri"
tags: []
---

_(Eve is a new programming language, and this is our development blog. If you’re new to Eve, [start here](http://play.witheve.com))_


### What is this?

This small app is pretty straightforward, consisting of a simple webpage with four subpages. The purpose is to demonstrate some basic webpage structure, show how a navigation bar could be implemented, how it changes the view between the different subpages, and how to inject page contents into the page view as you navigate from one subpage to another. You can play with this example live [here](http://play.witheve.com/#gist:0049b5b77a1e01b0124c96c820ff3374-crazy-mikes.eve).

### Page Layout

#### Containers

Since I want this app to have a constant hero image and nav bar above the page, that means the nav bar and image are going to be static while the page contents are going to change.

```
bind @browser
  [#div class:"app-wrapper" children:
    [#div class:"hero-image"]
    [#div class:"nav-bar"]
    [#page-contents]]
```

#### Subpages

We also need to know which pages we're going to have. Crazy Mike sells a modest selection of repossessed electronics:

```
commit
  [#page page:"homepage" name:"Home"]
  [#page page:"computers" name:"Computers"]
  [#page page:"televisions" name:"Televisions"]
  [#page page:"stereos" name:"Stereos"]
```

#### Initial Landing Site

We also want to make sure that when a user reaches the site, they start on the homepage.

```
commit
  [#app page:"homepage"]
```

### The Nav Bar

#### Drawing the Nav Bar

While the hero image was easy, the nav bar gets its own section because it needs a little more love than a background. It needs to make a button for each page of the website (those pages were committed in the Page Template section)

```
search @session @browser
  [#page page name]
  nav = [#div class:"nav-bar"]

bind @browser
  nav <- [#div children:
    [#div #nav class:"nav-btn" page text:name]]

```

#### Navigation

We start on the home page, but when you click a button on the nav bar, we want to navigate to that page.

```
search @browser @session @event
  click = [#click element:[#div class:"nav-btn" page]]
  view = [#app]

commit
  view.page := page
```

#### Highlighting the Active Page

Purely as a style issue, let's change the background color of the nav bar button of whichever page we're on. You could also use this block to add a new class to nav-btn and use CSS to set the new background color, but this is a little more terse without obfuscating the goal.

```
search @session @browser
  [#app page]
  nav-btn = [#div class:"nav-btn" page]

bind @browser
  nav-btn.style += [background:"#606060"]
```

### Subpages

#### Home Page

When the app specifies that we're on the home page, we want to inject the home page contents into the `#page-contents` from the page template.

```
search @session @browser
  [#app page:"homepage"]
  view = [#page-contents]

bind @browser
  view <- [#div class:"main-page" children:
    [#h1 text:"Welcome to Crazy Mike's!"]
    [#p text:"Located on the scenic Pulaski Highway in East Baltimore, Crazy Mike's has the region's best selection of used electronics, all for CHEAP CHEAP CHEAP!"]
    [#p text:"Hours: Tue-Sat 2pm-4am"]
    [#p text:"Contact: (410) 768-7000, Ask for Mike"]
  ]
```

#### Computers

Much like the home page, when the app specifies that we want to navigate to the Computers tab, we want to inject that page into `#page-contents`.

```
search @session @browser
  [#app page:"computers"]
  view = [#page-contents]

bind @browser
  view <- [#div class:"main-page" children:
    [#p text:"Need to compute things? We can help."]
    [#div class:("computer", "pic")]
    [#p text:"One of our many fine products, this War Operations Plan Response supercomputer was repossessed from the US Dept. of Defense in 1984. Comes with classic games such as chess, checkers, backgammon, poker, tic-tac-toe, and Global Thermonuclear War, though it has been known not to play. Open box, comes as-is. Strict no return policy."]]
```

#### Televisions

Once more, when we navigate to the Televisions tab, it gets injected into `#page-contents`.

```
search @session @browser
  [#app page:"televisions"]
  view = [#page-contents]

bind @browser
  view <- [#div class:"main-page" children:
    [#p text:"This is where the TVs live - get you one!"]
    [#div class:"tv pic"]
    [#p text:"Forget the internet, this baby is the real series of tubes. Perfect for your LaserDisc collection."]
  ]

```

#### Stereos

When we navigate to the Stereos tab, it gets injected into `#page-contents`.

```
search @session @browser
  [#app page:"stereos"]
  view = [#page-contents]

bind @browser
  view <- [#div class:"main-page" children:
    [#p text:"The hottest audio equipment in town!"]
    [#div class:"radio pic"]
    [#p text:"New stock arriving daily, priced to move."]
  ]
```

### Styles

A little CSS to clean things up and make the page more readable.

```css
.app-wrapper {
  display: flex;
  flex-direction: column;
  position: absolute;
  align-self: center;
  width: 432px;
  height: 768px;
  overflow-y: auto;
  background: #fff;
}

.hero-image {
  height: 300px;
  background-image: url(http://i.imgur.com/L8suaDZ.jpg);
  background-size: cover;
}

.nav-bar {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: 50px;
  align-items: center;
}

.nav-btn {
  flex: 1;
  text-align: center;
  background: #404040;
  line-height: 50px;
  color: #fff;
  user-select: none;
}

.main-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 20px;
  background: ;
}

.main-page h1 {
  margin: 10px 0px;
  font-size: 26px;
}

.main-page p {
  margin: 5px 25px;
  font-size: 16px;
  text-align: center;
}

.pic {
  margin: 20px 0px;
}

.computer {
  height: 150px;
  width: 320px;
  background: url(http://i.imgur.com/2EbQYGA.jpg) center no-repeat;
  background-size: cover;
}

.tv {
  height: 250px;
  width: 320px;
  background: url(http://i.imgur.com/0kD08WU.jpg) center no-repeat;
  background-size: cover;
}

.radio {
  height: 250px;
  width: 320px;
  background: url(http://i.imgur.com/rs1NW31.jpg) center no-repeat;
  background-size: cover;
}
```