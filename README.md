#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>
#include <iomanip>
#include <algorithm>
#include <string>

class Cell {
public:
    bool isMine = false;
    bool isRevealed = false;
    bool isFlagged = false;
    int adjacentMines = 0;

    char display() const {
        if (isFlagged) return 'F';
        if (!isRevealed) return '.';
        if (isMine) return '*';
        return adjacentMines + '0';
    }
};

class Board {
private:
    int width, height, numMines;
    std::vector<std::vector<Cell>> grid;

    bool isValid(int x, int y) const {
        return x >= 0 && x < height && y >= 0 && y < width;
    }

    void placeMines() {
        int placed = 0;
        while (placed < numMines) {
            int x = std::rand() % height;
            int y = std::rand() % width;
            if (!grid[x][y].isMine) {
                grid[x][y].isMine = true;
                ++placed;
            }
        }
    }

    void calculateAdjacents() {
        for (int x = 0; x < height; ++x) {
            for (int y = 0; y < width; ++y) {
                if (grid[x][y].isMine) continue;
                int count = 0;
                for (int dx = -1; dx <= 1; ++dx)
                    for (int dy = -1; dy <= 1; ++dy)
                        if (isValid(x + dx, y + dy) && grid[x + dx][y + dy].isMine)
                            ++count;
                grid[x][y].adjacentMines = count;
            }
        }
    }

public:
    Board(int w, int h, int m) : width(w), height(h), numMines(m), grid(h, std::vector<Cell>(w)) {
        placeMines();
        calculateAdjacents();
    }

    void reveal(int x, int y) {
        if (!isValid(x, y) || grid[x][y].isRevealed || grid[x][y].isFlagged) return;
        grid[x][y].isRevealed = true;
        if (grid[x][y].adjacentMines == 0 && !grid[x][y].isMine) {
            for (int dx = -1; dx <= 1; ++dx)
                for (int dy = -1; dy <= 1; ++dy)
                    if (dx != 0 || dy != 0)
                        reveal(x + dx, y + dy);
        }
    }

    void toggleFlag(int x, int y) {
        if (isValid(x, y) && !grid[x][y].isRevealed)
            grid[x][y].isFlagged = !grid[x][y].isFlagged;
    }

    bool isMine(int x, int y) const {
        return isValid(x, y) && grid[x][y].isMine;
    }

    void display() const {
        std::cout << "  ";
        for (int y = 0; y < width; ++y)
            std::cout << y << " ";
        std::cout << "\n";
        for (int x = 0; x < height; ++x) {
            std::cout << x << " ";
            for (auto it = grid[x].begin(); it != grid[x].end(); ++it)
                std::cout << it->display() << " ";
            std::cout << "\n";
        }
    }

    bool allSafeCellsRevealed() const {
        for (const auto& row : grid)
            for (const auto& cell : row)
                if (!cell.isMine && !cell.isRevealed)
                    return false;
        return true;
    }
};

class Game {
private:
    Board board;
    bool gameOver = false;

public:
    Game(int width, int height, int mines) : board(width, height, mines) {
        std::srand(static_cast<unsigned int>(std::time(nullptr)));
    }

    void run() {
        while (!gameOver) {
            board.display();
            std::cout << "操作を入力（形式: O x y または F x y）: ";
            std::string command;
            int x, y;
            std::cin >> command >> y >> x;

            if (command == "F" || command == "f") {
                board.toggleFlag(x, y);
            }
            else if (command == "O" || command == "o") {
                if (board.isMine(x, y)) {
                    std::cout << "ゲームオーバー！地雷を踏みました！\n";
                    gameOver = true;
                    break;
                }
                board.reveal(x, y);
                if (board.allSafeCellsRevealed()) {
                    std::cout << "おめでとう！全ての安全なマスを開けました！\n";
                    gameOver = true;
                    break;
                }
            }
            else {
                std::cout << "間違ったコマンドです。OまたはFを使用してください。\n";
            }
        }
        board.display();
    }
};

void printDifficultyMenu() {
    std::cout << "=== マインスイーパー 難易度を選択 ===\n";
    std::cout << "1. 初級 (5x5, 地雷5個)\n";
    std::cout << "2. 中級 (10x10, 地雷15個)\n";
    std::cout << "3. 上級 (16x16, 地雷40個)\n";
    std::cout << "難易度の番号を入力してください！: ";
}

int main() {
    int choice;
    int width, height, mines;

    printDifficultyMenu();
    std::cin >> choice;

    switch (choice) {
    case 1:
        width = height = 5;
        mines = 5;
        break;
    case 2:
        width = height = 10;
        mines = 15;
        break;
    case 3:
        width = height = 16;
        mines = 40;
        break;
    default:
        std::cout << "無効な選択肢です。初級を設定します。\n";
        width = height = 5;
        mines = 5;
    }

    Game game(width, height, mines);
    game.run();
    return 0;
}
