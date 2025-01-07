/*
 * This program implements a simple snake game.
 *
 * Author: Yash Gandhi, Jwal Patel and Claricel Ramos
 * Date: March 6, 2024
 * GitHub Repository link:- https://github.com/Rio-The-Yash/Snake-game/tree/proj_II_w_6
 */

/*
 * Header files needed for the project
 */
#include <stdio.h>
#include <conio.h> // _kbhit() _getch()
#include <windows.h> // Sleep()
#include <unistd.h> // system("cls")
#include <time.h> // srand and rand

// defining numbers for the directions
#define UP    1
#define DOWN  2
#define LEFT  3
#define RIGHT 4


// defining the number of the rows and columns
#define ROWS    30
#define COLS    30

char userInput;
// declaring the integer for the head of the snake and the array for the tail... we make it 100 so that the tail will only grow until 100 inputs/food beyond that the code is running wrongly and putting the tails randomly across the play the area
int headX, headY;
int tailX[100], tailY[100];
int nTail;

// defining the sleep time and speed for the snake
int updater = 100;
int level = 1;

int foodX ,foodY;

// end game flag  : 0 or 1
int gameover;
int direction;

// grow snake flag -> to mimic if food eaten
int grow;

int gameStarted = 0;  // Variable to track whether the game has started or not

int score=0;

// Function to generate a random food location
void generateFood() {
    int overlap = 1;
    // From the start food will be placed randomly but then it will check if the food is either on the head or on the body of the snake 
    while (overlap) {
        overlap = 0;
        foodX = rand() % (COLS - 2) + 1;
        foodY = rand() % (ROWS - 2) + 1;

        // Check if food overlaps with the snake's head or tail
        if (foodX == headX && foodY == headY)
            overlap = 1;
        for (int i = 0; i < nTail; i++) {
            if (foodX == tailX[i] && foodY == tailY[i]) {
                overlap = 1;
                break;
            }
        }
    }
}

// initialization for the random position of the head and initial direction of the snake 
void setup() {
    headX = rand() % (COLS - 2) + 1;
    headY = rand() % (ROWS - 2) + 1;

    generateFood();

    direction = RIGHT;
    nTail = 0;
}

// function to resume the game oncce 
void resumeGame() {
    gameStarted = 1;
}

// function to pause the game
void pauseGame() {
    gameStarted = 0;
    // if the user hit c it will again resume the game
    while (1) {
        if (_kbhit()) {
            userInput = _getch();
            if (userInput == 'c') {
                resumeGame();
                break;
            }
        }
    }
}

// function to draw the playing area, head and the tail of the snake
void draw() {
    system("cls");

	// upper wall
    for (int i = 0; i < ROWS+1; i++)
        printf("*");
    printf("\n");
	
	// Draws right,left wall and head, tail of the snake
    for (int i = 0; i < COLS; i++) {
        for (int j = 0; j < ROWS; j++) {
            if (j == 0 || j == ROWS - 1)
                printf("*");
            if (i == headY && j == headX)
                printf("@");
            else if (i == foodY && j == foodX)
                printf("0");
            else {
                int print = 0;
                for (int k = 0; k < nTail; k++) {
                    if (tailX[k] == j && tailY[k] == i) {
                        printf("o");
                        print = 1;
                    }
                }
                if (print == 0)
                    printf(" ");
            }
        }
        printf("\n");
    }
    
	// for loop to make the lower border of  the play area
    for (int i = 0; i < ROWS + 1; i++)
        printf("*");
    printf("\n");
    printf("\n");

    printf("Level: %d\nScore: %d\n", level, score); // updates score after successfully eating the food
    printf("Position = (%d , %d)\n", headX, headY);
    printf("Press 'p' to pause the game\nPress 'c' to resume the game\nPress 'x' to quit the game");
}

// function to get input fromt the user to get the directions for the movement of the snake and to exit or start the program 
void input() {
    if (_kbhit()) {
        userInput = _getch();
        switch (userInput) {
            case 'w':
                if (direction != DOWN)
                    direction = UP;
                break;
            case 's':
                if (direction != UP)
                    direction = DOWN;
                break;
            case 'a':
                if (direction != RIGHT)
                    direction = LEFT;
                break;
            case 'd':
                if (direction != LEFT)
                    direction = RIGHT;
                break;
            case 'x':
                gameover = 1;
                break;
            case 'p':
            	pauseGame();
            	break;
            case 'c':
            	resumeGame();
            	break;
            default:
                break;
        }
    }
}

void levelUp() {
    if (score % 100 == 0 && score > 0) {
        updater -= 10; // Increase the game speed
        score += 10;
		level ++;   // Increase score as a reward for leveling up
        printf("Level Up! Game Speed Increased!\n");
    }
}

// function on which the main logic will work for the movement of the head and tail of the snake and how the tail will grow after giving the input 
void algorithm() {
    int prevX = tailX[0];
    int prevY = tailY[0];
    int prev2X, prev2Y;

    if (nTail > 0) {
        tailX[0] = headX;
        tailY[0] = headY;
    }

    for (int i = 1; i < nTail; i++) {
        prev2X = tailX[i];
        prev2Y = tailY[i];
        tailX[i] = prevX;
        tailY[i] = prevY;
        prevX = prev2X;
        prevY = prev2Y;
    }

	// logic for the movement of the snake in all directions
    switch (direction) {
        case UP:
            headY--;
            break;
        case DOWN:
            headY++;
            break;
        case LEFT:
            headX--;
            break;
        case RIGHT:
            headX++;
            break;
        default:
            break;
    }

	// game will end once the snake touches the border of the play area
    if (headX >= ROWS - 1 || headX < 0 || headY < 0 || headY >= COLS)
        gameover = 1;
	
	// logic for the self-collision end game
    for (int i = 1; i < nTail; i++) {
        if (tailX[i] == headX && tailY[i] == headY) {
            gameover = 1;
            break;
        }
    }

	// logic for eating the food and the growth of the tail 
    if (foodX == headX && foodY == headY) {
    grow = 1;
    score += 10;
    generateFood();
	}

	// logic for the growth of the tail
    if (grow) {
        tailX[nTail] = tailX[nTail - 1];
        tailY[nTail] = tailY[nTail - 1];
        nTail++;
        grow = 0;
    }
}

// initialization which will pop-up once we run the program
void Init() {
    printf("+------------+\n");
    printf("| SNAKE GAME |\n");
    printf("+------------+\n");
    printf("Press 'j' to start the game and 'x' to quit.\n");
    printf("Press 'w' to make the snake go UP\n");
    printf("Press 's' to make the snake go DOWN\n");
    printf("Press 'a' to make the snake go LEFT\n");
    printf("Press 'd' to make the snake go RIGHT\n");
    printf("Press 'c' to restart the game\n");

	// code will only accepts the value 'j' and 'x' as an input and the rest will be discarded
    while (userInput != 'j' && userInput != 'x') {
        userInput = _getch();
    }

    if (userInput == 'j') {
        gameStarted = 1; // Start the game if 'j' is pressed initially
        setup();
    }
    system("cls");
}

int main() {
    srand(time(NULL)); // Seed for random number generation

    Init();

    while (!gameover){
    
		while (!gameover) {
    	    if (gameStarted) {
        	    draw();
            	input();
	            algorithm();
	            levelUp();
				Sleep(updater);
        	}
	    }
    	system("cls");
	    printf("GAME OVER!!\nScore:- %d\n",score);
    	printf("Do you want to quit the game or restart the Game??\nPress 'r' to restart and 'x' to quit");
    
    	userInput = _getch();
    // if the user will hit 'r' or 'x' the system will get it do appropriate function
    	while(userInput != 'r' && userInput != 'x'){
			userInput = _getch();
		}
    	
	    if (userInput == 'r') {
    	    gameover = 0;
    	    system("cls");
        	Init();
    	}
    	
    	if (userInput == 'x'){
    		gameover = 1;
		}
	}

    return 0;
}

