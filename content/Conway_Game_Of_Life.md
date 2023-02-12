---
title: "Conway's Game of Life"
date: 2022-09-29T15:10:04+08:00
description: "Recreation of the famous simulation using PIXI.js"
featured_image:
tags: ['Algorithm', 'Game', 'Simulation', 'Computer Science', 'Turing machine']
---

> The **Game of Life**, also known simply as **Life**, is a cellular automaton devised by the British mathematician John Horton Conway in 1970. It is a zero-player game, meaning that its evolution is determined by its initial state, requiring no further input. One interacts with the Game of Life by creating an initial configuration and observing how it evolves. It is Turing complete and can simulate a universal constructor or any other Turing machine. \-\- Wikipedia

I was introduced to Conway's Game of Life by my college roomate who was also studying computer science. During my college study, one of the courses taught us about Turing machines, a fascinating concept of what makes a 'computer'. It is both simple and complicated at the same time. One thing led to another and soon I found myself researching
"turing complete languages", and Conway's game of life immediately stood out as one of if not the most interesting one. 

The entire simulation can be summarized by the following rules:

>1. Any live cell with two or three live neighbours survives.
>2. Any dead cell with three live neighbours becomes a live cell.
>3. All other live cells die in the next generation. Similarly, all other dead cells stay dead.

In my code, the three rules translate into this:

```javascript
function recalculate() {
    for(let x = 0; x < grid.length; x++) {
        for(let y = 0; y < grid[x].length; y++) {
            let count = calculateNeighbor(x, y);
            if(grid[x][y][0] === 1) { // Alive
                if (count < 2 || count > 3) {
                    grid[x][y][1] = 0; // Mark for deletion
                }  
            }
            else { // Dead
                if(count === 3) {
                    grid[x][y][1] = 1; // Mark for creation
                }
            }
        }
    }
}
```

![Game_Of_Life](/conway-gif.gif)

A remarkably simple set of rules create such complex behaviors.

