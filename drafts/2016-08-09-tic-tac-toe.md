```
---
layout: post
title: "Breaking Down Tic-Tac-Toe"
author: "Joshua Cole"
tags: []
---
```

_(Editor's Note: Keep in mind as you're reading, this post is an executable Eve program. See the raw text [here](https://raw.githubusercontent.com/Kodowa/incidentalcomplexity/gh-pages/_posts/2016-08-09-tic-tac-toe.md))_

![Tic-tac-toe in Eve]({{ site.url }}/images/tic-tac-toe.gif)

Last week on the [mailing list](https://groups.google.com/forum/#!forum/eve-talk), [RubenSandwich](https://github.com/rubensandwich) posted an interactive demo capable of [playing and scoring tic-tac-toe matches][1]. He provided some great feedback about the issues he ran into along the way. Now that the language is becoming more stable, our first priority is seeing it used and addressing the problems which surface. To that end, his troubles became our guide to making Eve a little friendlier for writing interactive applications in general. Today we'll look at how we went about implementing tic-tac-toe in Eve.

[1]: https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!topic/eve-talk/UQkW7KDdz3M

This analysis (and future breakdowns) will be written inline in Eve to make the discussion flow more naturally. Since our blog is capable of rendering Markdown, we can provide a pleasant reading experience directly from the source code. Right now, Eve is compatible with [GitHub-Flavored Markdown][2].

[2]: https://guides.github.com/features/mastering-markdown/

###  Game logic

Tic-Tac-Toe is a classic game played by two players, "X" and "O", who take turns marking their letter on a 3x3 grid. The first player to mark 3 adjacent cells with their letter wins. The game can potentially result in a draw, where all grid cells are marked, but neither player has 3 adjacent cells. To build this game in Eve, we need several parts:

- A game board with cells
- A way to mark a cell as 'X' or 'O'
- A way to recognize that a player has won the game.

To begin, we initialize the board. We commit an object named `@board` to hold our global state and create a set of `#cell`s to represent each grid square. These `#cell`s will keep track of the moves players have made. Common connect-N games (a generalized tic-tac-toe for any NxN grid) are scored along 4 axes (horizontal, vertical, and the two diagonals). We group cells together along each axis up front to make scoring easier later. We've also pulled `width`, `height`, and `n-in-a-row` out because, with a few minor changes, our program is generic enough to play any connect-N game. This process is made much cleaner by the addition of new math expressions, including `range[from, to]`, `floor[value]`, and `mod[value, by]`. This is a small part of our effort to expand the standard library based on usage. If you're interested in helping shape this, stop by our [RFCs repository][3] or jump right in on our discussion of [standard string expressions][4].

[3]: https://github.com/witheve/rfcs/
[4]: https://github.com/witheve/rfcs/issues/5

Initialize the board
```
  match
    [#session-connect]

    // board constants
    width = 3
    height = 3
    n-in-a-row = 3
    starting-player = "x"

    // generate the cells
    i = range[from: 0, to: width * height]
    column = mod[value: i, by: width]
    row = floor[value: i / width]
    diag-left = column - row
    diag-right = (width - column) - row

  commit
    board = [@board width height n-in-a-row player: starting-player]
    [#cell board row column diag-left diag-right]
```

Next, we handle user input. Any time a cell is directly clicked, we:

1. Ensure the cell hasn't already been played
2. Check for a winner
3. Switch to the next player

Then update the cell the reflect its new owner, and the board's `player` to the next player.

Click on a cell to make your move
```
  match
    [#click #direct-target element: [#div cell]]
    not(cell.player)
    board = [@board player: current not(winner)]
    next_player = if current = "x" then "o"
                  else "x"
  commit
    board.player := next_player
    cell.player := current
```

Next, we score the board. When writing Eve code, we don't have to worry about triggers and subscriptions to decide when to reevaluate. We simply ask for the data we need and when it changes our block will be reevaluated for us. This is where our extra computation during initialization comes in handy; since we've already separated our cells into winnable groupings, we only need to check if the number of cells owned by a player in the group is equal to the number required to win (called `n-in-a-row`).

One caveat of connect-N games is that there won't always be a winner. We catch this case by determining if every cell on the board has been filled yet no winner has been found. When this happens, we mark the mysterious third player `"nobody"` as our winner and allow the reset handling logic to do the rest.

Get N in a row to win the game!
```
  match
    board = [@board n-in-a-row width height not(winner)]
    winner = if cell = [#cell row player]
                n-in-a-row = count[given: cell, per: (row, player)] then player
            else if cell = [#cell column player]
                n-in-a-row = count[given: cell, per: (column, player)] then player
            else if cell = [#cell diag-left player]
                n-in-a-row = count[given: cell, per: (diag-left, player)] then player
            else if cell = [#cell diag-right player]
                n-in-a-row = count[given: cell, per: (diag-right, player)] then player
            else if cell = [#cell player]
                    width * height = count[given: cell] then "nobody"
  commit
    board.winner := winner
```

Since games of tic-tac-toe are often very short and extremely competitive, it's imperative that it be quick and easy to begin a new match. When the game is over (the board has a `winner` attribute), a click anywhere on the drawing area will reset the game for another round of play. This entails clearing every `cell.player` and removing `board.winner`.

Reset the board state after a win
```
  match
    [#click #direct-target]
    board = [@board winner]
    cell = [#cell player]
  commit
    board.winner -= winner
    cell.player -= player
```

### Drawing the Game

We've implemented the game logic, but now we need to actually draw the board so players have something to see and interact with. Our general strategy will be that the game board is a `#div` with one child `#div` for each cell. Each cell will be drawn with an "X", "O", or empty string as text. We also add a `#status` div, which we'll write game state into later. Our cells have the CSS inlined, but you could just as easily link to an external file.

Draw the board
```
  match
    board = [@board]
    cell = [#cell board row column]
    contents = if cell.player then cell.player
              else ""
  bind
    [#div board @container children:
      [#div #status board class: "status"]
      [#div class: "board" children:
        [#div class: "row" sort: row children:
          [#div class: "cell" cell text: contents sort: column style:
            [display: "inline-block" width: "50px" height: "50px" border: "1px solid rgb(47, 47, 49)" color: "black" background: "white" font-size: "2em" line-height: "50px" text-align: "center"]]]]]
```

Finally, we fill the previously mentioned `#status` div with our current game state. If no winner has been declared, we remind the competitors of whose turn it is, and once a winner is found we announce her newly-acquired bragging rights.

Draw the current player
```
  match
    status = [#status board]
    not(board.winner)
  bind
    status.text += "{{board.player}}'s turn!"
```

Draw the winner
```
  match
    status = [#status board]
    winner = board.winner
  bind
    status.text += "{{winner}} wins! Click anywhere to restart!"
```

Along the way to making this demo, many new standard library expressions were added, the execution strategy for aggregates was overhauled, parser bugs were fixed, and dependency ordering glitches resolved. We even began to appreciate how literate programming will work in Eve (the post you are reading right now is an executable Eve program). 

The human-compiled version of tic-tac-toe was completed in only about half an hour, and required very few changes to get working once the platform caught up. The latest iteration of Eve is still very much in its infancy, but even now its showing a lot of promise for teasing out simple and general solutions to complicated problems. As one of its creators, I'm obviously a biased party when discussing Eve, so feedback such as RubenSandwich's is invaluable in helping us make the language more robust. If you find the time to try Eve yourself, please don't hesitate to share your experiences with us on the [mailing list][5].

[5]: https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!forum/eve-talk