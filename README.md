import pygame
import sys
import numpy as np

# Game Constants
WINDOW_SIZE = (600, 600)
GRID_SIZE = 3
SQUARE_SIZE = WINDOW_SIZE[0] // GRID_SIZE

# Styling Constants
LINE_WIDTH = 15
CROSS_WIDTH = 20
CIRCLE_WIDTH = 15
RADIUS = SQUARE_SIZE // 3
OFFSET = 50

# Colors
BG_COLOR = (28, 170, 156)
LINE_COLOR = (23, 145, 135)
CROSS_COLOR = (66, 66, 66)
CIRCLE_COLOR = (239, 231, 200)

# Symbol Sets (Traditional + Emoji combinations)
SYMBOL_SETS = {
    '1': ('X', 'O'),             # Traditional
    '2': ('X 🌟', 'O 🌙'),        # Traditional + Stars
    '3': ('X 🦁', 'O 🐯'),        # Traditional + Animals
    '4': ('X 🎯', 'O 🎲'),        # Traditional + Games
    '5': ('X ❤️', 'O 💙'),        # Traditional + Hearts
}

class TicTacToe:
    """
    the entire game logic
    """
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode(WINDOW_SIZE)
        pygame.display.set_caption('Tic Tac Toe - Traditional + Emoji')
        self.font = pygame.font.SysFont('segoe ui emoji', 80)
        self.board = np.zeros((GRID_SIZE, GRID_SIZE), dtype=np.int8)
        self.current_player = 1
        self.game_over = False
        self.winner = None
        self.winning_combo = None
        self.current_symbols = SYMBOL_SETS['1']  # Default to traditional
        self.draw_grid() 
    def select_symbols(self):
        """
        Allows the player to choose a symbol set.
        """
        print("\nChoose your symbol set:")
        for key, symbols in SYMBOL_SETS.items():
            print(f"{key}. {symbols[0]} vs {symbols[1]}")
        while True:
            choice = input("\nEnter set number (1-5): ")
            if choice in SYMBOL_SETS:
                self.current_symbols = SYMBOL_SETS[choice]
                break
            print("Invalid choice! Please try again.")
    def handle_click(self, pos):
        """
        Handles mouse clicks and updates the game state.
        """
        if self.game_over:
            return
        row = pos[1] // SQUARE_SIZE
        col = pos[0] // SQUARE_SIZE
        if self.board[row][col] == 0:
            self.board[row][col] = self.current_player
            self.place_symbol(row, col)
            self.check_win()
            if not self.game_over:
                self.switch_player()
    def check_win(self):
        """
        Checks for winning combinations and updates game state accordingly.
        """
        winning_combinations = [
            [(0, 0), (0, 1), (0, 2)],  # Rows
            [(1, 0), (1, 1), (1, 2)],
            [(2, 0), (2, 1), (2, 2)],
            [(0, 0), (1, 0), (2, 0)],  # Columns
            [(0, 1), (1, 1), (2, 1)],
            [(0, 2), (1, 2), (2, 2)],
            [(0, 0), (1, 1), (2, 2)],  # Diagonals
            [(0, 2), (1, 1), (2, 0)]
        ]
        for combo in winning_combinations:
            values = [self.board[row][col] for row, col in combo]
            if len(set(values)) == 1 and values[0] != 0:
                self.game_over = True
                self.winner = values[0]
                self.winning_combo = combo
                self.draw_win_line()
                return
        if np.all(self.board):  # Check for a draw
            self.game_over = True
    def draw_grid(self):
        """
        Draws the game board on the screen.
        """
        self.screen.fill(BG_COLOR)
        # Draw vertical lines
        pygame.draw.line(self.screen, LINE_COLOR, (SQUARE_SIZE, 0), (SQUARE_SIZE, WINDOW_SIZE[1]), LINE_WIDTH)
        pygame.draw.line(self.screen, LINE_COLOR, (2 * SQUARE_SIZE, 0), (2 * SQUARE_SIZE, WINDOW_SIZE[1]), LINE_WIDTH)
        # Draw horizontal lines
        pygame.draw.line(self.screen, LINE_COLOR, (0, SQUARE_SIZE), (WINDOW_SIZE[0], SQUARE_SIZE), LINE_WIDTH)
        pygame.draw.line(self.screen, LINE_COLOR, (0, 2 * SQUARE_SIZE), (WINDOW_SIZE[0], 2 * SQUARE_SIZE), LINE_WIDTH)
    def place_symbol(self, row, col):
        """
        Draws the appropriate symbol (X or O with optional emoji) on the board.
        """
        symbol = self.current_symbols[0 if self.current_player == 1 else 1]
        if 'X' in symbol:
            self._draw_cross(row, col)
            if len(symbol) > 1:
                self._draw_emoji(row, col, symbol.split()[1], self.current_player == 1)
        elif 'O' in symbol:
            self._draw_circle(row, col)
            if len(symbol) > 1:
                self._draw_emoji(row, col, symbol.split()[1], self.current_player == 1)
    def _draw_cross(self, row, col):
        """
        Draws the X symbol.
        """
        start_desc = (col * SQUARE_SIZE + OFFSET, row * SQUARE_SIZE + OFFSET)
        end_desc = (col * SQUARE_SIZE + SQUARE_SIZE - OFFSET, row * SQUARE_SIZE + SQUARE_SIZE - OFFSET)
        pygame.draw.line(self.screen, CROSS_COLOR, start_desc, end_desc, CROSS_WIDTH)
        start_asc = (col * SQUARE_SIZE + OFFSET, row * SQUARE_SIZE + SQUARE_SIZE - OFFSET)
        end_asc = (col * SQUARE_SIZE + SQUARE_SIZE - OFFSET, row * SQUARE_SIZE + OFFSET)
        pygame.draw.line(self.screen, CROSS_COLOR, start_asc, end_asc, CROSS_WIDTH)
    def _draw_circle(self, row, col):
        """
        Draws the O symbol.
        """
        center = (col * SQUARE_SIZE + SQUARE_SIZE // 2, row * SQUARE_SIZE + SQUARE_SIZE // 2)
        pygame.draw.circle(self.screen, CIRCLE_COLOR, center, RADIUS, CIRCLE_WIDTH)
    def _draw_emoji(self, row, col, emoji, is_player_one):
        """
        Draws the emoji beside the X or O symbol.
        """
        text = self.font.render(emoji, True, CROSS_COLOR if is_player_one else CIRCLE_COLOR)
        pos = (col * SQUARE_SIZE + SQUARE_SIZE // 2, row * SQUARE_SIZE + SQUARE_SIZE // 2)
        rect = text.get_rect(center=pos)
        rect.x += 20  # Offset emoji slightly to not overlap with X/O
        self.screen.blit(text, rect)
    def draw_win_line(self):
        """
        Draws the winning line across the board.
        """
        start = (self.winning_combo[0][1] * SQUARE_SIZE + SQUARE_SIZE // 2, self.winning_combo[0][0] * SQUARE_SIZE + SQUARE_SIZE // 2)
        end = (self.winning_combo[-1][1] * SQUARE_SIZE + SQUARE_SIZE // 2, self.winning_combo[-1][0] * SQUARE_SIZE + SQUARE_SIZE // 2)
        color = CROSS_COLOR if self.winner == 1 else CIRCLE_COLOR
        pygame.draw.line(self.screen, color, start, end, LINE_WIDTH)
    def switch_player(self):
        """
        Switches the current player between 1 (player 1) and 2 (player 2).
        """
        self.current_player = 3 - self.current_player 
def main():
    """
    Starts and runs the game.
    """
    game = TicTacToe()
    game.select_symbols()  # Allow player to choose symbols
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                game.handle_click(event.pos)
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_r:  # Reset game
                    game = TicTacToe()
                    game.select_symbols()
                elif event.key == pygame.K_s:  # Change symbols
                    game.select_symbols()
                    game.board = np.zeros((GRID_SIZE, GRID_SIZE), dtype=np.int8)  # Clear the board
                    game.current_player = 1
                    game.game_over = False
                    game.winner = None
                    game.winning_combo = None
                    game.draw_grid()  # Redraw the board
        pygame.display.update()

if __name__ == "__main__":
    main()
