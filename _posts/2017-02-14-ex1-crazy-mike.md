---
layout: post
title: "Example 1 - Crazy Mike's"
author: "Rob Attorri"
tags: []
---

_(Eve is a new programming language, and this is our development blog. If you’re new to Eve, [start here](http://play.witheve.com))_


### What is this?

This small app is pretty straightforward, consisting of a simple webpage with four subpages. The purpose is to demonstrate some basic webpage structure, show how a navigation bar could be implemented, how it changes the view between the different subpages, and how to inject page contents into the page view as you navigate from one subpage to another. You can play with this example in your browser [here](http://play.witheve.com/#gist:0049b5b77a1e01b0124c96c820ff3374-crazy-mikes.eve).

### Page Layout

#### Containers

I want this app to have three containers: a hero image and nav bar above the main page, and page contents which are going to change. I draw the basic page structure here and worry about details like drawing the individual tabs for the subpages later. The hero image and the nav bar also have their classes set here for CSS because their style never changes. The individual pages may require different styles, so their classes are bound later.

```
bind @browser
  [#div class:"app-wrapper" children:
    [#div class:"hero-image"]
    [#div class:"nav-bar" #nav-bar]
    [#div #page-contents]]
```

#### Subpages

Crazy Mike sells a modest selection of repossessed electronics. I can set all the pages the site is going to have right here, and because I separated this from the page containers above, if I want to add another page and have it appear as a tab on the navigation bar, I can simply add it to this list.

```
commit
  [#page page:"homepage" name:"Home"]
  [#page page:"computers" name:"Computers"]
  [#page page:"televisions" name:"Televisions"]
  [#page page:"stereos" name:"Stereos"]
```

#### Initial Landing Site

The #`app` record is where I've decided to keep track of which page is being viewed. I've also set `page` to `homepage` to begin with so that new customers will land there when they visit the site.

```
commit
  [#app page:"homepage"]
```

### The Navigation Bar

#### Drawing the Nav Bar

While the hero image was easy, the nav bar gets its own section because it needs a little more love than a background. It needs to make a button for each page of the website, which were committed in the Subpages section, so I take all the #`page` records and add a child #`div` to the #`nav-bar` for each of them.

```
search @session @browser
  [#page page name]
  nav-bar = [#nav-bar]

bind @browser
  nav-bar.children += [#div class:"nav-btn" page text:name]

```

#### Navigation

We start on the home page, but when you click a button on the nav bar, we want to navigate to that page. This listens for a click on any nav button, whose record has a page attached to it, then sets the `page` attribute of the #`app` record to match the `page` attribute of the button that was clicked.

```
search @browser @session @event
  click = [#click element:[#div class:"nav-btn" page]]
  view = [#app]

commit
  view.page := page
```

#### Highlighting the Active Page

Purely as a style issue, I want to change the background color of the nav bar button of whichever page we're on. You could also use this block to bind a new class to `nav-btn` and use CSS to set the new background color, but this is a little more terse without obfuscating the goal.

```
search @session @browser
  [#app page]
  nav-btn = [#div class:"nav-btn" page]

bind @browser
  nav-btn.style += [background:"#606060"]
```

### Subpages

#### Home Page

When the app specifies that we should be looking at the home page, the contents of this block are injected into the #`page-contents` record that was bound to the browser at the start of the Page Layout section.

```
search @session @browser
  [#app page:"homepage"]
  view = [#page-contents]

bind @browser
  view <- [class:"main-page" children:
    [#h1 text:"Welcome to Crazy Mike's!"]
      [#p text:"Located on the scenic Pulaski Highway in East Baltimore, Crazy Mike's has the region's best selection of used electronics, and our prices are INSANE!"]
    [#p text:"Hours: Tue-Sat 2pm-4am"]
    [#p text:"Contact: (410) 768-7000, Ask for Mike"]
  ]
```

#### Computers

Much like the home page, when the app specifies that we want to navigate to the Computers tab, we want to inject this page into #`page-contents`.

```
search @session @browser
  [#app page:"computers"]
  view = [#page-contents]

bind @browser
  view <- [class:"main-page" children:
    [#p text:"Need to compute things? We can help."]
    [#div class:("computer", "pic")]
    [#p text:"One of our many fine products, this War Operations Plan Response supercomputer was repossessed from the US Dept. of Defense in 1984. Comes with classic games such as chess, checkers, backgammon, poker, tic-tac-toe, and Global Thermonuclear War, though it has been known not to play. Open box, comes as-is. Strict no return policy."]]
```

#### Televisions

Once more, when we navigate to the Televisions tab, it gets injected into #`page-contents`.

```
search @session @browser
  [#app page:"televisions"]
  view = [#page-contents]

bind @browser
  view <- [class:"main-page" children:
    [#p text:"This is where the TVs live - get you one!"]
    [#div class:"tv pic"]
    [#p text:"Forget the internet, this baby is the real series of tubes. Perfect for your LaserDisc collection."]
  ]

```

#### Stereos

When we navigate to the Stereos tab, it gets injected into #`page-contents`.

```
search @session @browser
  [#app page:"stereos"]
  view = [#page-contents]

bind @browser
  view <- [class:"main-page" children:
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
  cursor: pointer;
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