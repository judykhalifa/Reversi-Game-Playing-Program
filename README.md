# Reversi-Game-Playing-Program
A C program that extends a Reversi game to allow a human to play against the computer, including an AI that selects moves based on scoring strategies and can be improved for smarter gameplay.


#include <stdio.h>
#include <stdbool.h>
#include "lab8part1.h"


void printBoard(char board[][26], int n) {
  printf("  ");
  for (int j = 0; j < n; j++) printf("%c", 'a' + j);
  printf("\n");

  for (int i = 0; i < n; i++) {
    printf("%c ", 'a' + i);
    for (int j = 0; j < n; j++) printf("%c", board[i][j]);
    printf("\n");
  }
}

bool positionInBounds(int n, int row, int col) {
  return row >= 0 && row < n && col >= 0 && col < n;
}

bool checkLegalInDirections(char board[][26], int n, int row, int col,
                           char colour, int dr, int dc) {

  char opponent;

if (colour == 'B') {
  opponent = 'W';
} else {
  opponent = 'B';
}

  int r = row + dr;
  int c = col + dc;

  if (!positionInBounds(n, r, c) || board[r][c] != opponent)
    return false;

  r += dr;
  c += dc;

  while (positionInBounds(n, r, c)) {
    if (board[r][c] == 'U') return false;
    if (board[r][c] == colour) return true;
    r += dr;
    c += dc;
  }

  return false;
}

int isValidMove(char board[][26], int n, int row, int col, char colour) { 
  if (board[row][col] != 'U') return 0;

  for (int dr = -1; dr <= 1; dr++) {
    for (int dc = -1; dc <= 1; dc++) {
      if (dr == 0 && dc == 0) continue;

      if (checkLegalInDirections(board, n, row, col, colour, dr, dc))
        return 1;
    }
  }
  return 0;
}

void flipInDirection(char board[][26], int n, int row, int col,
                     char colour, int dr, int dc) {

  char opponent;

if (colour == 'B') {
  opponent = 'W';
} else {
  opponent = 'B';
}

  int r = row + dr;
  int c = col + dc;

  while (board[r][c] == opponent) {
    board[r][c] = colour;
    r += dr;
    c += dc;
  }
}

int countFlips(char board[][26], int n, int row, int col,
               char colour, int dr, int dc) {

  char opponent;

if (colour == 'B') {
  opponent = 'W';
} else {
  opponent = 'B';
}

  int r = row + dr;
  int c = col + dc;
  int count = 0;

  while (positionInBounds(n, r, c) && board[r][c] == opponent) {
    count++;
    r += dr;
    c += dc;
  }

  if (!positionInBounds(n, r, c) || board[r][c] != colour)
    return 0;

  return count;
}

void findBestMove(char board[][26], int n, char colour, int *bestRow, int *bestCol) {
  int bestScore = -1;

  for (int i = 0; i < n; i++) {



    for (int j = 0; j < n; j++) {

      if (board[i][j] != 'U') continue;

      int score = 0;
      int valid = 0;

      for (int dr = -1; dr <= 1; dr++) {
        for (int dc = -1; dc <= 1; dc++) {
          if (dr == 0 && dc == 0) continue;

          if (checkLegalInDirections(board, n, i, j, colour, dr, dc)) {
            valid = 1;
            score += countFlips(board, n, i, j, colour, dr, dc);
          }
        }
      }

      if (valid && score > bestScore) {
        bestScore = score;
        *bestRow = i;
        *bestCol = j;
      }
    }
  }
}

int allValidMove(char board[][26], int n, char colour) { // this checks for multiple valid moves instead of one like the isValidMove function
  for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
      if (isValidMove(board, n, i, j, colour)) return 1; // t
    }
  }
  return 0;
}

void Score(char board[][26], int n, int *b, int *w) {
  *b = *w = 0;

  for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
      if (board[i][j] == 'B') (*b)++;
      if (board[i][j] == 'W') (*w)++;// we count the score for both instead of just one because we need to know the score for both players at the end of the game to determine the winner
    }
  }
}

int main(void) {
  int n;
  char computer;
  char board[26][26];

  printf("Enter the board dimension: ");
  scanf("%d", &n);

  printf("Computer plays (B/W) : ");
  scanf(" %c", &computer);
  

  for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
      board[i][j] = 'U';

  board[n/2 - 1][n/2 - 1] = 'W';
  board[n/2][n/2] = 'W';
  board[n/2 - 1][n/2] = 'B';
  board[n/2][n/2 - 1] = 'B';

  printBoard(board, n);

  char turn = 'B';

  while (1) {

    if (!allValidMove(board, n, 'B') && !allValidMove(board, n, 'W'))
      break;//break means that it will be brought to the end of the game and the score will be counted to determine the winner

    if (!allValidMove(board, n, turn)) {
      printf("%c player has no valid move.\n", turn);
      turn = (turn == 'B') ? 'W' : 'B';
      continue;
    }

    int row, col;

    if (turn == computer) {

      findBestMove(board, n, turn, &row, &col);
      printf("Computer places %c at %c%c.\n", turn, 'a'+row, 'a'+col);

    } else {

      char r, c;
      printf("Enter move for colour %c (RowCol): ", turn);
      scanf(" %c%c", &r, &c);

      row = r - 'a';
      col = c - 'a';

      if (!isValidMove(board, n, row, col, turn)) {
        printf("Invalid move.\n");
        if (turn == 'B') {
  printf("W player wins.\n");
} else {
  printf("B player wins.\n");
}
        return 0;
      }
    }

    board[row][col] = turn;

    for (int dr = -1; dr <= 1; dr++) {
      for (int dc = -1; dc <= 1; dc++) {
        if (dr == 0 && dc == 0) continue;

        if (checkLegalInDirections(board, n, row, col, turn, dr, dc))
          flipInDirection(board, n, row, col, turn, dr, dc);
      }
    }

    printBoard(board, n);

    turn = (turn == 'B') ? 'W' : 'B';
  }

  int b, w;
  Score(board, n, &b, &w);

  if (b > w) 
  {printf("B player wins.\n");}
  else if (w > b) {printf("W player wins.\n");}
  else {printf("Draw!\n");}

  return 0;
}
