# 1024-game
Group Project Description
This project's goal is to use Python and the Pygame module to create a graphical version of the well-known 1024 game. The game involves sliding numbered tiles on a grid to combine them and create a tile with the number 1024.  To win, players can choose from various target scores and play the game to reach that score. 

Objective
1. Implement the game mechanics: Develop the essential gameplay elements, such as moving and combining tiles and determining if the round is over or won.
2. Add graphical interface: To display the game board, tiles, and scores, using Pygame.
3. Add sound effects: Include sound effects for different actions such as moving tiles, winning, and losing.
4. Add a menu system: Implement a menu where players may choose their goal score, see their previous high scores, and restart the game.
5. Save high scores: Keep a record of and show the players' highest scores.

Approach
Board Class: Manages the tile movements, tile combinations, and game grid.
HighScore Class: Controls loading, saving, and displaying high scores.
Game Class: Integrates the board and high score management, handles user input, draws the game graphics, and manages the game loop.

Libraries Used
Pygame: A set of Python modules designed for writing video games. It includes computer graphics and sound libraries.
Random: Python library which provides functions to generate random numbers and perform random operations

Input Files
1. Sound files: The program uses the following sound files:
2. button.mp3: Sound for button clicks.
3. swipe.mp3: Sound for tile movements.
4. win.mp3: Sound for winning the game.
5. lose.mp3: Sound for losing the game.
6. High scores file: The program reads from and writes to a file named high_scores.txt to save and load high scores.

Running the Game
Initialize Pygame: The script begins by initializing Pygame and setting up the display.
Main Menu: Display the player the main menu where they can select to start the game, see their score, or quit.
Game Loop: Once the game starts, the main game loop handles user inputs, updates the game state, and refreshes the display.
Game Over/Win Screens: When the game ends, appropriate screens are shown with options to restart or quit.

Code
The provided code is a complete implementation of the 1024 game with all the described features. Here are key points from the code:

Initialization: Initializes Pygame, sets screen dimensions, and defines colors and fonts.
Board Class: Controls the rotations, movements, and game grid.
HighScore Class: Loads, stores, and manages high scores.
Game Class: Manages user input, handles game logic, implements graphics, and maintains the state of the game.

Limitations
Single Player: The game is designed for single-player mode only.
Fixed Grid Size: The grid size is fixed at 4x4. 

Instructions to Run
Ensure you have Python and Pygame installed.
Place the sound files (button.mp3, swipe.mp3, win.mp3, lose.mp3) in the same directory as the script.
Run the script using Python
