## What's this?

A complete introduction to Rust taking a simple implementation of Tetris as an example to expand on (thanks to @mikejquinn https://github.com/mikejquinn/rust-tetris).

## What's in it?

* in `src/`, a simple but working Tetris that we will try to improve as we learn the language. 
* in `curriculum/`, the text of the course over 10 weeks. See the `intro.md` file for a detailed overview of the content.
* branches that work as bookmarks to easily checkout the state of the repo for each week

## How?

* Read the code to assimilate the basic concepts and flow of Rust programming.
* Expand on it by adding the following:
    * Game over
    * Levels and difficulty
    * Scoring
    * wall-kick
* For advanced learners, try the following, harder improvements:
    * Enhance the graphics:
        * ghost different from actual piece
        * animation when the lines are cleared
        * ASCII Art
    * Add a screen title, a menu, options...
    * Write tests
    * Read the Tetris specification and make it more compliant
    * Modularize the game by segregating the game engine and the display
    * Add a real graphical engine
    * Rework the game engine according to the ECS principles using a library like Legion
