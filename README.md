#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#define MAX_TRIES 10
#define MIN_NUMBER 1
#define MAX_NUMBER 200
#define MAX_LEADERBOARD_ENTRIES 5
#define MAX_NAME_LENGTH 50
#define FILE_NAME "leaderboard.txt"
#define GAME_STATE_FILE "game_state.dat"

// Function Declarations
void displayInstructions();
int generateRandomNumber(int lower, int upper);
void playGame(int difficultyLevel, int *totalScore, int *roundsPlayed, char *playerName);
void showScore(int score);
void showLeaderboard();
void addLeaderboardEntry(int score, const char *name);
void displayMainMenu();
void delay(int seconds);
void provideHint(int targetNumber, int guess);
void saveLeaderboardToFile();
void loadLeaderboardFromFile();
void saveGameState(int roundsPlayed, int totalScore, const char *playerName);
void loadGameState(int *roundsPlayed, int *totalScore, char *playerName);
void startNewRound(int *roundsPlayed, int *totalScore);
void displayExitConfirmation();
void displayHighScore(int score);
void clearScreen();
void getPlayerName(char *playerName);

// Leaderboard structure
typedef struct {
    char name[MAX_NAME_LENGTH];
    int score;
} LeaderboardEntry;

LeaderboardEntry leaderboard[MAX_LEADERBOARD_ENTRIES];

// Structure to save game state
typedef struct {
    int roundsPlayed;
    int totalScore;
    int highScore;
    char playerName[MAX_NAME_LENGTH];
} GameState;

int main() {
    int choice;
    int roundsPlayed = 0;
    int totalScore = 0;
    int highScore = 0;
    char playerName[MAX_NAME_LENGTH] = "";

    // Seed the random number generator with current time
    srand(time(NULL));

    // Load leaderboard from file at the start
    loadLeaderboardFromFile();

    // Load previous game state
    loadGameState(&roundsPlayed, &totalScore, playerName);

    // Display game instructions
    displayInstructions();

    // Main menu loop
    while (1) {
        displayMainMenu();
        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                {
                    getPlayerName(playerName);
                    int difficultyLevel;
                    printf("\nSelect Difficulty Level:\n");
                    printf("1. Easy (1-50)\n");
                    printf("2. Medium (1-100)\n");
                    printf("3. Hard (1-200)\n");
                    printf("Enter your choice: ");
                    scanf("%d", &difficultyLevel);
                    playGame(difficultyLevel, &totalScore, &roundsPlayed, playerName);
                    break;
                }
            case 2:
                showScore(totalScore);
                break;
            case 3:
                showLeaderboard();
                break;
            case 4:
                saveLeaderboardToFile();
                printf("\nLeaderboard saved!\n");
                break;
            case 5:
                saveGameState(roundsPlayed, totalScore, playerName);
                printf("\nGame saved successfully!\n");
                break;
            case 6:
                loadGameState(&roundsPlayed, &totalScore, playerName);
                printf("\nGame loaded successfully!\n");
                break;
            case 7:
                displayExitConfirmation();
                break;
            case 8:
                displayHighScore(highScore);
                break;
            default:
                printf("\nInvalid choice, please try again.\n");
        }
    }

    return 0;
}

// Function to display instructions
void displayInstructions() {
    printf("\nWelcome to the 'Guess the Number' Game!\n");
    printf("In this game, you will need to guess a randomly generated number within a specified range.\n");
    printf("You will have a limited number of attempts based on the difficulty level you choose.\n");
    printf("The closer you guess to the target number, the higher your score will be.\n");
    printf("Try to guess the number within the fewest number of attempts to earn the best score.\n");
    printf("Good luck!\n\n");
}

// Function to generate a random number within a specified range
int generateRandomNumber(int lower, int upper) {
    return (rand() % (upper - lower + 1)) + lower;
}

// Function to play the game
void playGame(int difficultyLevel, int *totalScore, int *roundsPlayed, char *playerName) {
    int targetNumber, guess, attempts = 0;
    int maxAttempts, score = 0;

    // Determine the range and number of attempts based on difficulty
    switch (difficultyLevel) {
        case 1: // Easy
            targetNumber = generateRandomNumber(MIN_NUMBER, 50);
            maxAttempts = 10;
            printf("\nYou have chosen 'Easy' difficulty. Guess a number between 1 and 50.\n");
            break;
        case 2: // Medium
            targetNumber = generateRandomNumber(MIN_NUMBER, 100);
            maxAttempts = 8;
            printf("\nYou have chosen 'Medium' difficulty. Guess a number between 1 and 100.\n");
            break;
        case 3: // Hard
            targetNumber = generateRandomNumber(MIN_NUMBER, 200);
            maxAttempts = 6;
            printf("\nYou have chosen 'Hard' difficulty. Guess a number between 1 and 200.\n");
            break;
        default:
            printf("\nInvalid difficulty level!\n");
            return;
    }

    // Main game loop for making guesses
    while (attempts < maxAttempts) {
        printf("\nAttempt %d/%d. Enter your guess: ", attempts + 1, maxAttempts);
        scanf("%d", &guess);

        // Provide a hint if the user requests
        if (guess == -1) {
            provideHint(targetNumber, guess);
            continue;
        }

        // Check if the guess is correct
        if (guess < targetNumber) {
            printf("Too low! Try again.\n");
        } else if (guess > targetNumber) {
            printf("Too high! Try again.\n");
        } else {
            printf("Congratulations! You've guessed the correct number: %d\n", targetNumber);
            score = maxAttempts - attempts; // Reward based on fewer attempts
            *totalScore += score; // Add score to total score
            printf("Your score for this round is: %d\n", score);
            addLeaderboardEntry(score, playerName);  // Add score to leaderboard
            break;
        }
        
        attempts++;
    }

    // If max attempts are exhausted, show the correct number
    if (attempts == maxAttempts && guess != targetNumber) {
        printf("\nSorry! You've used all your attempts. The correct number was %d.\n", targetNumber);
    }

    // Update rounds played
    (*roundsPlayed)++;
    
    // Display score after the game ends
    showScore(*totalScore);
}

// Function to show the current score
void showScore(int score) {
    printf("\nTotal score: %d\n", score);
}

// Function to show the leaderboard
void showLeaderboard() {
    printf("\n------ Leaderboard ------\n");
    for (int i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
        if (leaderboard[i].score > 0) {
            printf("%d. %s - %d points\n", i + 1, leaderboard[i].name, leaderboard[i].score);
        }
    }
    printf("\n");
}

// Function to add a leaderboard entry
void addLeaderboardEntry(int score, const char *name) {
    int i;
    for (i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
        if (leaderboard[i].score < score) {
            // Shift entries down to make space for the new score
            for (int j = MAX_LEADERBOARD_ENTRIES - 1; j > i; j--) {
                leaderboard[j] = leaderboard[j - 1];
            }
            leaderboard[i].score = score;
            strncpy(leaderboard[i].name, name, MAX_NAME_LENGTH);
            break;
        }
    }
}

// Function to save leaderboard to a file
void saveLeaderboardToFile() {
    FILE *file = fopen(FILE_NAME, "w");
    if (file != NULL) {
        for (int i = 0; i < MAX_LEADERBOARD_ENTRIES; i++) {
            if (leaderboard[i].score > 0) {
                fprintf(file, "%s,%d\n", leaderboard[i].name, leaderboard[i].score);
            }
        }
        fclose(file);
    }
}

// Function to load leaderboard from a file
void loadLeaderboardFromFile() {
    FILE *file = fopen(FILE_NAME, "r");
    if (file != NULL) {
        char name[MAX_NAME_LENGTH];
        int score;
        int i = 0;
        while (fscanf(file, "%49[^,],%d\n", name, &score) != EOF && i < MAX_LEADERBOARD_ENTRIES) {
            strncpy(leaderboard[i].name, name, MAX_NAME_LENGTH);
            leaderboard[i].score = score;
            i++;
        }
        fclose(file);
    }
}

// Function to save game state
void saveGameState(int roundsPlayed, int totalScore, const char *playerName) {
    FILE *file = fopen(GAME_STATE_FILE, "wb");
    if (file != NULL) {
        GameState state = {roundsPlayed, totalScore, 0};
        strncpy(state.playerName, playerName, MAX_NAME_LENGTH);
        fwrite(&state, sizeof(GameState), 1, file);
        fclose(file);
    }
}

// Function to load game state
void loadGameState(int *roundsPlayed, int *totalScore, char *playerName) {
    FILE *file = fopen(GAME_STATE_FILE, "rb");
    if (file != NULL) {
        GameState state;
        fread(&state, sizeof(GameState), 1, file);
        *roundsPlayed = state.roundsPlayed;
        *totalScore = state.totalScore;
        strncpy(playerName, state.playerName, MAX_NAME_LENGTH);
        fclose(file);
    }
}

// Function to get the player's name
void getPlayerName(char *playerName) {
    printf("\nEnter your name: ");
    scanf("%s", playerName);
}

// Function to display the main menu
void displayMainMenu() {
    clearScreen();
    printf("Welcome to the Guess the Number Game!\n");
    printf("1. Start Game\n");
    printf("2. View Score\n");
    printf("3. View Leaderboard\n");
    printf("4. Save Leaderboard\n");
    printf("5. Save Game\n");
    printf("6. Load Game\n");
    printf("7. Exit Game\n");
    printf("8. View High Score\n");
}

// Function to confirm exit
void displayExitConfirmation() {
    char choice;
    printf("\nAre you sure you want to exit? (y/n): ");
    scanf(" %c", &choice);
    if (choice == 'y' || choice == 'Y') {
        exit(0);
    }
}

// Function to show high score
void displayHighScore(int score) {
    printf("\nHigh Score: %d\n", score);
}

// Function to clear the screen (platform dependent)
void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}

// Function to provide a hint if the guess is too far off
void provideHint(int targetNumber, int guess) {
    if (abs(targetNumber - guess) > 50) {
        printf("Hint: You're quite far off! Try a much higher or lower number.\n");
    } else {
        printf("Hint: You're getting closer! Try narrowing your guess.\n");
    }
}

           



