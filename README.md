import java.util.Scanner;

public class TicTacToeWithProbability {

    private static final int SIZE = 3;
    private static char[][] board = new char[SIZE][SIZE];

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        initBoard();

        char currentPlayer = 'X';
        char winner = ' ';

        System.out.println("=== Tic Tac Toe with Win Probability (Java) ===");

        while (true) {
            printBoard();

            // Calculate win probabilities from current state
            double[] probs = simulateProbabilities(board, currentPlayer);
            double pXWin = probs[0];
            double pOWin = probs[1];
            double pDraw = probs[2];

            System.out.println();
            System.out.println("Win Probability (assuming random valid moves):");
            System.out.printf("  X wins: %.2f%%\n", pXWin * 100);
            System.out.printf("  O wins: %.2f%%\n", pOWin * 100);
            System.out.printf("  Draw  : %.2f%%\n", pDraw * 100);

            if (currentPlayer == 'X') {
                System.out.printf(">> Current Player: X (Win chance: %.2f%%)\n", pXWin * 100);
            } else {
                System.out.printf(">> Current Player: O (Win chance: %.2f%%)\n", pOWin * 100);
            }

            // Ask for input
            int row, col;
            while (true) {
                System.out.print("Enter row (1-3) and column (1-3) for player " + currentPlayer + ": ");
                row = scanner.nextInt() - 1;
                col = scanner.nextInt() - 1;

                if (isValidMove(row, col)) {
                    board[row][col] = currentPlayer;
                    break;
                } else {
                    System.out.println("Invalid move. Try again.");
                }
            }

            // Check for winner or draw
            winner = checkWinner();
            if (winner == 'X' || winner == 'O') {
                printBoard();
                System.out.println("Player " + winner + " wins!");
                break;
            } else if (isBoardFull()) {
                printBoard();
                System.out.println("It's a draw!");
                break;
            }

            // Switch player
            currentPlayer = (currentPlayer == 'X') ? 'O' : 'X';
        }

        scanner.close();
        System.out.println("Game Over.");
    }

    // Initialize the board with spaces
    private static void initBoard() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                board[i][j] = ' ';
            }
        }
    }

    // Print the current board
    private static void printBoard() {
        System.out.println();
        System.out.println("Current Board:");
        for (int i = 0; i < SIZE; i++) {
            System.out.print(" ");
            for (int j = 0; j < SIZE; j++) {
                System.out.print(board[i][j] == ' ' ? '-' : board[i][j]);
                if (j < SIZE - 1) System.out.print(" | ");
            }
            System.out.println();
            if (i < SIZE - 1) {
                System.out.println("-----------");
            }
        }
    }

    // Check if a move is valid
    private static boolean isValidMove(int row, int col) {
        return row >= 0 && row < SIZE &&
               col >= 0 && col < SIZE &&
               board[row][col] == ' ';
    }

    // Check who has won or if game continues
    // Returns 'X', 'O', 'D' (draw) or ' ' (no result yet)
    private static char checkWinner() {
        // rows
        for (int i = 0; i < SIZE; i++) {
            if (board[i][0] != ' ' &&
                board[i][0] == board[i][1] &&
                board[i][1] == board[i][2]) {
                return board[i][0];
            }
        }

        // columns
        for (int j = 0; j < SIZE; j++) {
            if (board[0][j] != ' ' &&
                board[0][j] == board[1][j] &&
                board[1][j] == board[2][j]) {
                return board[0][j];
            }
        }

        // diagonals
        if (board[0][0] != ' ' &&
            board[0][0] == board[1][1] &&
            board[1][1] == board[2][2]) {
            return board[0][0];
        }

        if (board[0][2] != ' ' &&
            board[0][2] == board[1][1] &&
            board[1][1] == board[2][0]) {
            return board[0][2];
        }

        // draw or not finished
        if (isBoardFull()) {
            return 'D';
        }

        return ' ';
    }

    // Check if board is full
    private static boolean isBoardFull() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (board[i][j] == ' ') return false;
            }
        }
        return true;
    }

    /**
     * Recursively simulate all possible future games from the current board
     * assuming both players choose valid moves randomly.
     *
     * Returns array: [probabilityXWins, probabilityOWins, probabilityDraw]
     */
    private static double[] simulateProbabilities(char[][] state, char playerToMove) {
        char result = evaluateState(state);
        if (result == 'X') {
            return new double[]{1.0, 0.0, 0.0};
        } else if (result == 'O') {
            return new double[]{0.0, 1.0, 0.0};
        } else if (result == 'D') {
            return new double[]{0.0, 0.0, 1.0};
        }

        int moves = 0;
        double sumX = 0.0;
        double sumO = 0.0;
        double sumD = 0.0;

        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (state[i][j] == ' ') {
                    // make move
                    state[i][j] = playerToMove;
                    char nextPlayer = (playerToMove == 'X') ? 'O' : 'X';

                    double[] child = simulateProbabilities(state, nextPlayer);
                    sumX += child[0];
                    sumO += child[1];
                    sumD += child[2];

                    // undo move
                    state[i][j] = ' ';
                    moves++;
                }
            }
        }

        if (moves == 0) {
            // should not normally happen because draw is handled above
            return new double[]{0.0, 0.0, 1.0};
        }

        // average probabilities across all possible moves
        return new double[]{sumX / moves, sumO / moves, sumD / moves};
    }

    // Evaluate state for terminal condition:
    // 'X' if X wins, 'O' if O wins, 'D' if draw, ' ' if game not over
    private static char evaluateState(char[][] state) {
        // rows
        for (int i = 0; i < SIZE; i++) {
            if (state[i][0] != ' ' &&
                state[i][0] == state[i][1] &&
                state[i][1] == state[i][2]) {
                return state[i][0];
            }
        }

        // columns
        for (int j = 0; j < SIZE; j++) {
            if (state[0][j] != ' ' &&
                state[0][j] == state[1][j] &&
                state[1][j] == state[2][j]) {
                return state[0][j];
            }
        }

        // diagonals
        if (state[0][0] != ' ' &&
            state[0][0] == state[1][1] &&
            state[1][1] == state[2][2]) {
            return state[0][0];
        }

        if (state[0][2] != ' ' &&
            state[0][2] == state[1][1] &&
            state[1][1] == state[2][0]) {
            return state[0][2];
        }

        // draw?
        boolean full = true;
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (state[i][j] == ' ') {
                    full = false;
                    break;
                }
            }
            if (!full) break;
        }

        if (full) return 'D';
        return ' ';
    }
}
