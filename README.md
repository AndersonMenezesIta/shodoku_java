# shodoku_java

import java.util.Random;
import java.util.Scanner;

public class SudokuGame {
    private int[][] board;
    private boolean[][] fixedCells; // Para saber quais células são parte do quebra-cabeça inicial

    public SudokuGame() {
        board = new int[9][9];
        fixedCells = new boolean[9][9];
        generateSudoku();
    }

    // --- Métodos de Geração e Validação (Simplificados para o Exemplo) ---

    private void generateSudoku() {
        // Inicializa o tabuleiro vazio
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                board[i][j] = 0;
                fixedCells[i][j] = false;
            }
        }

        // Preenche uma célula aleatoriamente para iniciar o processo de resolução
        // Em um gerador mais robusto, você preencheria mais células ou usaria um algoritmo mais complexo
        Random rand = new Random();
        board[rand.nextInt(9)][rand.nextInt(9)] = rand.nextInt(9) + 1;

        // Tenta resolver o tabuleiro (isso é uma simplificação, idealmente um SudokuSolver
        // geraria um tabuleiro completo e depois removeria células)
        if (!solve(board)) {
            System.out.println("Erro ao gerar Sudoku. Tentando novamente...");
            generateSudoku(); // Tentar novamente se a primeira tentativa falhar
            return;
        }

        // Remove algumas células para criar o quebra-cabeça
        // O número de células a remover determina a dificuldade
        int cellsToRemove = 40; // Exemplo: 40 células removidas
        int countRemoved = 0;
        while (countRemoved < cellsToRemove) {
            int row = rand.nextInt(9);
            int col = rand.nextInt(9);
            if (board[row][col] != 0) {
                int temp = board[row][col];
                board[row][col] = 0; // Remove o número

                // Verifica se ainda há uma solução única (complexo de implementar)
                // Para este exemplo, apenas removemos e não verificamos unicidade
                fixedCells[row][col] = false; // Não é uma célula fixa
                countRemoved++;
            }
        }

        // Marca as células restantes como fixas
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] != 0) {
                    fixedCells[i][j] = true;
                }
            }
        }
    }


    // --- Lógica do SudokuSolver (simplificada e integrada aqui) ---
    // Este é um algoritmo de backtracking para resolver o Sudoku
    private boolean solve(int[][] currentBoard) {
        for (int row = 0; row < 9; row++) {
            for (int col = 0; col < 9; col++) {
                if (currentBoard[row][col] == 0) {
                    for (int num = 1; num <= 9; num++) {
                        if (isValidMove(currentBoard, row, col, num)) {
                            currentBoard[row][col] = num;

                            if (solve(currentBoard)) {
                                return true;
                            } else {
                                currentBoard[row][col] = 0; // Backtrack
                            }
                        }
                    }
                    return false; // Nenhum número funciona para esta célula
                }
            }
        }
        return true; // Tabuleiro resolvido
    }

    private boolean isValidMove(int[][] currentBoard, int row, int col, int num) {
        // Verifica a linha
        for (int c = 0; c < 9; c++) {
            if (currentBoard[row][c] == num) {
                return false;
            }
        }

        // Verifica a coluna
        for (int r = 0; r < 9; r++) {
            if (currentBoard[r][col] == num) {
                return false;
            }
        }

        // Verifica o bloco 3x3
        int startRow = row - row % 3;
        int startCol = col - col % 3;
        for (int r = 0; r < 3; r++) {
            for (int c = 0; c < 3; c++) {
                if (currentBoard[startRow + r][startCol + c] == num) {
                    return false;
                }
            }
        }
        return true;
    }

    // --- Métodos de Jogo ---

    public void printBoard() {
        System.out.println("-------------------------");
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (j % 3 == 0 && j != 0) {
                    System.out.print("| ");
                }
                System.out.print(board[i][j] == 0 ? ". " : board[i][j] + " ");
            }
            System.out.println();
            if ((i + 1) % 3 == 0 && i != 8) {
                System.out.println("-------------------------");
            }
        }
        System.out.println("-------------------------");
    }

    public boolean placeNumber(int row, int col, int num) {
        // Validação básica da entrada
        if (row < 0 || row >= 9 || col < 0 || col >= 9 || num < 1 || num > 9) {
            System.out.println("Entrada inválida. Linha e coluna devem ser de 1 a 9, número de 1 a 9.");
            return false;
        }

        // Ajusta para índices baseados em 0
        row--;
        col--;

        if (fixedCells[row][col]) {
            System.out.println("Você não pode alterar uma célula predefinida.");
            return false;
        }

        if (isValidMove(this.board, row, col, num)) {
            this.board[row][col] = num;
            System.out.println("Número " + num + " colocado em (" + (row + 1) + "," + (col + 1) + ").");
            return true;
        } else {
            System.out.println("Movimento inválido. O número " + num + " não pode ser colocado em (" + (row + 1) + "," + (col + 1) + ").");
            return false;
        }
    }

    public boolean isGameWon() {
        // Para simplificar, o jogo é ganho se não houver células vazias e o tabuleiro estiver válido.
        // Uma verificação mais robusta seria resolver o tabuleiro atual e comparar com a solução.
        for (int i = 0; i < 9; i++) {
            for (int j = 0; j < 9; j++) {
                if (board[i][j] == 0) {
                    return false; // Ainda há células vazias
                }
            }
        }

        // Verifica se o tabuleiro preenchido é válido
        // Uma maneira simples é tentar resolvê-lo e ver se o solver retorna true
        int[][] tempBoard = new int[9][9];
        for(int i = 0; i < 9; i++) {
            for(int j = 0; j < 9; j++) {
                tempBoard[i][j] = board[i][j];
            }
        }
        return solve(tempBoard); // Se puder ser resolvido (e já está completo), então é válido
    }


    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        SudokuGame game = new SudokuGame();

        System.out.println("Bem-vindo ao Sudoku!");

        while (!game.isGameWon()) {
            game.printBoard();
            System.out.println("\nDigite sua jogada (linha coluna numero) ou 'sair' para sair:");
            System.out.print("Exemplo: 1 2 3 (coloca 3 na linha 1, coluna 2): ");

            String input = scanner.nextLine();

            if (input.equalsIgnoreCase("sair")) {
                System.out.println("Saindo do jogo. Até a próxima!");
                break;
            }

            try {
                String[] parts = input.split(" ");
                if (parts.length == 3) {
                    int row = Integer.parseInt(parts[0]);
                    int col = Integer.parseInt(parts[1]);
                    int num = Integer.parseInt(parts[2]);
                    game.placeNumber(row, col, num);
                } else {
                    System.out.println("Formato de entrada inválido. Tente novamente.");
                }
            } catch (NumberFormatException e) {
                System.out.println("Entrada inválida. Certifique-se de usar números para linha, coluna e valor.");
            }
        }

        if (game.isGameWon()) {
            game.printBoard();
            System.out.println("\nParabéns! Você venceu o Sudoku!");
        }

        scanner.close();
    }
}
