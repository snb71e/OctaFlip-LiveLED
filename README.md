# CSE306 Term Project - OctaFlip Game & LED Visualizer with Server Communication (Spring 2025)
👥 Developed by **Team JUME**. 

![demo](demo.gif)

## 📌 Overview
This project is a client implementation for the **OctaFlip Game**, developed as part of a team assignment at **DGIST**.   
It includes:
- A move-generation algorithm to play OctaFlip intelligently against an opponent.
- Real-time board visualization on a 64×64 LED matrix panel.
- Network communication with a game server over TCP using JSON protocol.

## File Structure
```
OctaFlipGame/
├── libs/
│   ├── rpi-rgb-led-matrix/   # for LED matrix
│   └── cJSON.c               
├── include/  
│   ├── board.h
│   ├── client.h
│   ├── game.h
│   ├── json.h
│   └── server.h          
├── src/                      
│   ├── board.c
│   ├── client.c
│   ├── game.c
│   ├── json.c
│   └── server.c
├── main.c                    
├── README.md
```

## Game Description

**OctaFlip** is a two-player turn-based strategy game played on an 8×8 grid. Each player is assigned a color:
- Red (`R`)
- Blue (`B`)

### Turn Mechanics
- Players alternate turns, starting with **Red**.
  - **Clone**: Move to an **adjacent** cell (1 cell away in any of the 8 directions). The original piece remains.
  - **Jump**: Move to a cell exactly **2 cells away** (in any of the 8 directions). The original piece disappears.

After moving, all **opponent pieces adjacent to the destination** (in 8 directions) are **flipped** to the moving player’s color.

### Valid Move Conditions
- The source cell must contain the current player's piece.
- The destination cell must be empty (`.`).
- Movement must be exactly 1 or 2 steps in a straight or diagonal direction (no irregular leaps).

### Passing
- If no valid moves are available, a player **must pass** by submitting the move `0 0 0 0`.
- If **both players pass consecutively**, the game ends immediately.

### Game Ends When:
- All cells are filled (no `.` left).
- One player has no pieces remaining.
- Both players pass in succession.

### Winning Conditions
- When the game ends, the player with **more pieces** on the board wins.
- If both players have the same number of pieces, the result is a **Draw**.

### Game Logic Implementation
The rules and logic for the OctaFlip game (e.g., move validation, board updates, flipping mechanism, and termination conditions) are implemented in [`game.c`](src/game.c).


### Board Representation
Each cell on the 8×8 board uses the following characters:
- `R`: Red player’s piece
- `B`: Blue player’s piece
- `.`: Empty space
- `#`: Blocked cell (unavailable for movement or placement)

The board state is represented as 8 lines of 8 characters each.


## Features
- **AI Move Generation**: Implements `generate_move()` to select optimal moves under a 3-second timeout using custom algorithms.
- **LED Display**: Displays the 8×8 game board on a 64×64 LED panel using:
  - 8x8 grid cell rendering (each cell = 8×8 pixels)
  - 6×6 piece icons centered in each cell
  - Color-coded pieces
- **Server Communication**:
  - Follows a predefined JSON message protocol to interact with the game server

## Server Interaction
Example interaction:
```json
{"type":"register","username":"temp"}
{"type":"register_ack"}
{"type":"game_start", "players":["temp","TAr"], "first_player":"temp"}
{"type":"your_turn", "board":["R......B", ..., "B......R"], "timeout":3}
```

## Build Instructions

### 1. Build LED Standalone Visualizer
```bash
g++ -DBOARD_STANDALONE src/board.c \
    -Iinclude -Ilibs/rpi-rgb-led-matrix/include \
    -Llibs/rpi-rgb-led-matrix/lib -lrgbmatrix -lpthread -lrt \
    -o board_standalone
```
### 2. Build Full Game Executable
``` bash
g++ -Iinclude -Ilibs/rpi-rgb-led-matrix/include \
    main.c src/server.c src/client.c src/json.c src/game.c src/board.c libs/cJSON.c \
    -Llibs/rpi-rgb-led-matrix/lib -lrgbmatrix -lpthread -lrt \
    -o OctaflipGame
```

## Local Testing Mode

### 1. Start Local Server
``` bash
sudo ./OctaflipGame server -p 8080 \
  --led-rows=64 --led-cols=64 \
  --led-gpio-mapping=regular --led-brightness=75 \
  --led-chain=1 --led-no-hardware-pulse
```

### 2. Run Clients (in order)
- **Client 1**
``` bash
sudo ./OctaflipGame client -i 127.0.0.1 -p 8080 -u user1 \
--led --led-rows=64 --led-cols=64 \
--led-no-hardware-pulse --led-gpio-mapping=regular \
--led-chain=1
```
- **Client 2**
``` bash
sudo ./OctaflipGame client -i 127.0.0.1 -p 8080 -u user2 \
  --led-rows=64 --led-cols=64 \
  --led-no-hardware-pulse --led-gpio-mapping=regular \
  --led-chain=1
```


