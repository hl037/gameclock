# GameclocK

Go to https://hl037.github.io/gameclock for a live demo.

GameclocK is a sober clock for chess, go etc. that can be used either on a laptop :
 - <kbd>Ctrl</kbd> : Player 1 played
 - <kbd>Space</kbd> : Pause both clocks (draw proposal in a chess game)
 - <kbd>Numpad Enter</kbd> : Player 2 played

Or on a phone (in this case, the display is rotated 180° for both player to see the game.
 - Touch upper screen area : Player 1 played
 - Touch center area ("Pause") : Pause both clock (and show setting / reset buttons)
 - Touch lower screen area : Player 2 played

When a player reaches 0, the clock is locked to prevent any other unwanted input, and the player who timed out has its clock in red.
on mobile, you need to press "Pause" prior to click the reset button and start again.

Features : 

 - [x] Simple clock (set increment to 0, and max time to match initial time in seconds)
 - [x] Fisher clock (set the increment to the number of seconds to be added on each move. The increment does not occurs on the first move).
 - [x] Max time (can be disable by setting a very large time) : The increment can't give more time than this value.
 - [ ] Translation (comming soon)
 - [ ] Different settings for each player (comming soon)
 - [ ] Better configuration UI
 - [ ] Byo-yomi (mostly used in go)
 - [ ] Time periods (n moves in given time)
 - [ ] Multi player (to play 3 player abalone or Chinese checkers
  
## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

### Lints and fixes files
```
yarn lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
