# 1024-game

import pygame
import random

pygame.init()
WIDTH, HEIGHT = 400, 500
GRID_SIZE = 4
TILE_SIZE = WIDTH // GRID_SIZE
FONT = pygame.font.Font(None, 55)
SCORE_FONT = pygame.font.Font(None, 35)
MENU_FONT = pygame.font.Font(None, 45)
BUTTON_FONT = pygame.font.Font(None, 40)
sound1 = pygame.mixer.Sound("button.mp3")
sound1.set_volume(0.5)
sound2=pygame.mixer.Sound("swipe.mp3")
sound2.set_volume(0.5)
sound3=pygame.mixer.Sound("win.mp3")
sound3.set_volume(0.5)
sound4=pygame.mixer.Sound("lose.mp3")
sound4.set_volume(0.5)

BACKGROUND_COLOR = [187, 173, 160]
EMPTY_TILE_COLOR = [205, 193, 180]
TILE_COLORS = {
    2: [238, 228, 218],
    4: [237, 224, 200],
    8: [242, 177, 121],
    16: [245, 149, 99],
    32: [246, 124, 95],
    64: [246, 94, 59],
    128: [237, 207, 114],
    256: [237, 204, 97],
    512: [237, 200, 80],
    1024: [237, 197, 63],
    2048: [237, 194, 46],
}

screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("1024 Game")

class HighScore:
    def __init__(self):
        self.high_score = 0
        self.scores = []
        self.load_scores()

    def load_scores(self):
        try:
            with open("high_scores.txt", 'r') as file:
                self.scores = [int(line.strip()) for line in file]
            if self.scores:
                self.high_score = max(self.scores)
        except (FileNotFoundError, ValueError):
            self.scores = []
            self.high_score = 0

    def save_score(self, score):
        self.scores.append(score)
        self.high_score = max(self.scores)
        with open("high_scores.txt", 'a') as file:
            file.write(str(score) + "\n")

class Board:
    def __init__(self, grid_size):
        self.grid_size = grid_size
        self.board = [[0] * grid_size for i in range(grid_size)]
        self.score = 0

    def add_new_tile(self):
        empty_tiles = [
            (row, col) 
            for row in range(self.grid_size) 
            for col in range(self.grid_size) 
            if self.board[row][col] == 0]
        
        if empty_tiles:
            row, col = random.choice(empty_tiles)
            self.board[row][col] = 2

    def move_left(self):
        moved = False
        for row in range(self.grid_size):
            tiles = [num for num in self.board[row] if num != 0]
            for i in range(len(tiles) - 1):
                if tiles[i] == tiles[i + 1]:
                    tiles[i] *= 2
                    self.score += tiles[i]
                    tiles[i + 1] = 0
            new_row = [num for num in tiles if num != 0]
            new_row.extend([0] * (self.grid_size - len(new_row)))
            if self.board[row] != new_row:
                moved = True
            self.board[row] = new_row
        return moved

    def move_right(self):
        for row in range(self.grid_size):
            self.board[row] = self.board[row][::-1]
        moved = self.move_left()
        for row in range(self.grid_size):
            self.board[row] = self.board[row][::-1]
        return moved

    def move_down(self):
        self.board = self.rotate_clockwise(self.board)
        moved = self.move_left()
        self.board = self.rotate_counterclockwise(self.board)
        return moved

    def move_up(self):
        self.board = self.rotate_counterclockwise(self.board)
        moved = self.move_left()
        self.board = self.rotate_clockwise(self.board)
        return moved

    def rotate_clockwise(self, board):
        new_board = []
        for row in range(self.grid_size):
            new_row = []
            for col in range(self.grid_size):
                new_row.append(board[self.grid_size - 1 - col][row])
            new_board.append(new_row)
        return new_board

    def rotate_counterclockwise(self, board):
        new_board = []
        for row in range(self.grid_size):
            new_row = []
            for col in range(self.grid_size):
                new_row.append(board[col][self.grid_size - 1 - row])
            new_board.append(new_row)
        return new_board

    def sounds(self, key):
        if key == pygame.K_LEFT:
            sound2.play()
            return self.move_left()
        elif key == pygame.K_RIGHT:
            sound2.play()
            return self.move_right()
        elif key == pygame.K_DOWN:
            sound2.play()
            return self.move_down()
        elif key == pygame.K_UP:
            sound2.play()
            return self.move_up()
        return False

    def is_game_over(self):
        for row in self.board:
            if 0 in row:
                return False
        
        for row in range(self.grid_size):
            for col in range(self.grid_size):
                if row < self.grid_size - 1 and self.board[row][col] == self.board[row + 1][col]:
                    return False 
                
                if col < self.grid_size - 1 and self.board[row][col] == self.board[row][col + 1]:
                    return False 
        
        return True

    def is_game_won(self, target_score):
        for row in self.board:
            if target_score in row:
                return True
        return False

class Game:
    def __init__(self):
        self.board = Board(GRID_SIZE)
        self.high_score = HighScore()
        self.target_score = 2048

    def draw_rounded_rect(self, surface, color, rect, radius):
        pygame.draw.rect(surface, color, rect, border_radius=radius)

    def draw_grid(self):
        for row in range(GRID_SIZE):
            for col in range(GRID_SIZE):
                value = self.board.board[row][col]
                color = TILE_COLORS.get(value, EMPTY_TILE_COLOR)
                self.draw_rounded_rect(screen, color, (col * TILE_SIZE, row * TILE_SIZE + 100, TILE_SIZE, TILE_SIZE), 10)
                if value != 0:
                    text = FONT.render(str(value), True, [119, 110, 101])
                    text_rect = text.get_rect(center=(col * TILE_SIZE + TILE_SIZE // 2, row * TILE_SIZE + TILE_SIZE // 2 + 100))
                    screen.blit(text, text_rect)

        for i in range(1, GRID_SIZE):
            pygame.draw.line(screen, [187, 173, 160], (i * TILE_SIZE, 100), (i * TILE_SIZE, HEIGHT), 2)
            pygame.draw.line(screen, [187, 173, 160], (0, 100 + i * TILE_SIZE), (WIDTH, 100 + i * TILE_SIZE), 2)


    def draw_scores(self):
        score_text = SCORE_FONT.render(f"Score: {self.board.score}", True, [255, 255, 255])
        high_score_text = SCORE_FONT.render(f"High Score: {self.high_score.high_score}", True, [255, 255, 255])
        screen.blit(score_text, (10, 10))
        screen.blit(high_score_text, (10, 50))

    def display_menu(self):
        menu = True
        while menu:
            screen.fill(BACKGROUND_COLOR)
            menu_text = MENU_FONT.render("Select Target Score", True, (255, 255, 255))
            screen.blit(menu_text, (50, 50))
            targets = [256, 512, 1024, 2048]
            target_buttons = []

            for i, target in enumerate(targets):
                button = pygame.Rect(150, 150 + i * 60, 100, 50)
                target_buttons.append(button)
                pygame.draw.rect(screen, (255, 255, 255), button)
                button_text = MENU_FONT.render(str(target), True, [0, 0, 0])
                text_rect = button_text.get_rect(center=button.center)
                text_x = button.x + (button.width - button_text.get_width()) // 2
                text_y = button.y + (button.height - button_text.get_height()) // 2
                screen.blit(button_text, (text_x, text_y))

            back_button = pygame.Rect(WIDTH // 2 - 50, HEIGHT - 100, 100, 50)
            pygame.draw.rect(screen, [119,110,101], back_button)
            back_text = BUTTON_FONT.render("Back", True, [255,255,255])
            text_x = back_button.x + (back_button.width - back_text.get_width()) // 2
            text_y = back_button.y + (back_button.height - back_text.get_height()) // 2
            screen.blit(back_text, (text_x, text_y))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    return False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    for i, button in enumerate(target_buttons):
                        if button.collidepoint(mouse_pos):
                            sound1.play()
                            self.target_score = targets[i]
                            return True 
                    if back_button.collidepoint(mouse_pos):
                        sound1.play()
                        return False 
        return True

    def display_scores(self):
        screen.fill(BACKGROUND_COLOR)
        scores_text = MENU_FONT.render("High Scores", True, [255, 255, 255])
        screen.blit(scores_text, (50, 50))

        for i, score in enumerate(self.high_score.scores):
            score_text = SCORE_FONT.render(str(score), True, [255, 255, 255])
            screen.blit(score_text, (50, 100 + i * 40))

        back_button = pygame.Rect(WIDTH // 2 - 50, HEIGHT - 100, 100, 50)
        pygame.draw.rect(screen, [119,110,101], back_button)
        back_text = BUTTON_FONT.render("Back", True, [255,255,255])
        text_x = back_button.x + (back_button.width - back_text.get_width()) // 2
        text_y = back_button.y + (back_button.height - back_text.get_height()) // 2
        screen.blit(back_text, (text_x, text_y))

        pygame.display.flip()

        scores_screen = True
        while scores_screen:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    scores_screen = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    if back_button.collidepoint(mouse_pos):
                        sound1.play()
                        scores_screen = False

    def main_menu(self):
        menu = True
        while menu:
            screen.fill(BACKGROUND_COLOR)
            title_text = MENU_FONT.render("1024 Game", True, [255, 255, 255])
            screen.blit(title_text, (WIDTH // 2 - title_text.get_width() // 2, 50))

            buttons = {
                "Start": pygame.Rect(WIDTH // 2 - 50, 150, 100, 50),
                "Scores": pygame.Rect(WIDTH // 2 - 50, 220, 100, 50),
                "Quit": pygame.Rect(WIDTH // 2 - 50, 290, 100, 50),
            }

            for button_text, rect in buttons.items():
                pygame.draw.rect(screen, [119,110,101], rect)
                text = BUTTON_FONT.render(button_text, True, [255,255,255])
                text_x = rect.x + (rect.width - text.get_width()) // 2
                text_y = rect.y + (rect.height - text.get_height()) // 2
                screen.blit(text, (text_x, text_y))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    menu = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    if buttons["Start"].collidepoint(mouse_pos):
                        sound1.play()
                        if self.display_menu():
                            menu = False
                            self.run()
                    elif buttons["Scores"].collidepoint(mouse_pos):
                        sound1.play()
                        self.display_scores()
                    elif buttons["Quit"].collidepoint(mouse_pos):
                        sound1.play()
                        pygame.quit()
                        menu = False

    def display_restart_window(self):
        restart_window = True
        while restart_window:
            screen.fill(BACKGROUND_COLOR)
            end_text = FONT.render("Game Over", True, [255, 0, 0])
            text_rect = end_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
            screen.blit(end_text, text_rect)

            restart_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2, 200, 50)
            quit_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 60, 200, 50)

            pygame.draw.rect(screen, [119,110,101], restart_button)
            pygame.draw.rect(screen, [119,110,101], quit_button)

            restart_text = BUTTON_FONT.render("Restart", True, [255,255,255])
            quit_text = BUTTON_FONT.render("Quit", True, [255,255,255])

            restart_text_x = restart_button.x + (restart_button.width - restart_text.get_width()) // 2
            restart_text_y = restart_button.y + (restart_button.height - restart_text.get_height()) // 2
            quit_text_x = quit_button.x + (quit_button.width - quit_text.get_width()) // 2
            quit_text_y = quit_button.y + (quit_button.height - quit_text.get_height()) // 2

            screen.blit(restart_text, (restart_text_x, restart_text_y))
            screen.blit(quit_text, (quit_text_x, quit_text_y))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    restart_window = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    if restart_button.collidepoint(mouse_pos):
                        sound1.play()
                        self.board = Board(GRID_SIZE)
                        self.run()
                        restart_window = False
                    elif quit_button.collidepoint(mouse_pos):
                        sound1.play()
                        pygame.quit()
                        restart_window = False
    def display_continue_window(self):
        continue_window = True
        while continue_window:
            screen.fill(BACKGROUND_COLOR)
            end_text = FONT.render("You Win!", True, [255, 0, 0])
            text_rect = end_text.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
            screen.blit(end_text, text_rect)

            continue_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2, 200, 50)
            quit_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 60, 200, 50)
            pygame.draw.rect(screen, [119,110,101], continue_button)
            pygame.draw.rect(screen, [119,110,101], quit_button)

            continue_text = BUTTON_FONT.render("Continue", True, [255,255,255])
            quit_text = BUTTON_FONT.render("Quit", True, [255,255,255])
            continue_text_x = continue_button.x + (continue_button.width - continue_text.get_width()) // 2
            continue_text_y = continue_button.y + (continue_button.height - continue_text.get_height()) // 2
            screen.blit(continue_text, (continue_text_x, continue_text_y))
            quit_text_x = quit_button.x + (quit_button.width - quit_text.get_width()) // 2
            quit_text_y = quit_button.y + (quit_button.height - quit_text.get_height()) // 2
            screen.blit(quit_text, (quit_text_x, quit_text_y))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    continue_window = False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    if continue_button.collidepoint(mouse_pos):
                        sound1.play()
                        self.select_difficulty()
                        self.run()
                        continue_window = False
                    elif quit_button.collidepoint(mouse_pos):
                        sound1.play()
                        pygame.quit()
                        continue_window = False

    def select_difficulty(self):
        menu = True
        while menu:
            screen.fill(BACKGROUND_COLOR)
            menu_text = MENU_FONT.render("Select Target Score", True, [255, 255, 255])
            screen.blit(menu_text, (50, 50))

            targets = [256, 512, 1024, 2048]
            target_buttons = []
            for i, target in enumerate(targets):
                button = pygame.Rect(150, 150 + i * 60, 100, 50)
                target_buttons.append(button)
                pygame.draw.rect(screen, [255, 255, 255], button)
                button_text = MENU_FONT.render(str(target), True, [0, 0, 0])
                text_rect = button_text.get_rect(center=button.center)
                screen.blit(button_text, text_rect)

            back_button = pygame.Rect(WIDTH // 2 - 50, HEIGHT - 100, 100, 50)
            pygame.draw.rect(screen, [119,110,101], back_button)
            back_text = BUTTON_FONT.render("Back", True, [255,255,255])
            text_x = back_button.x + (back_button.width - back_text.get_width()) // 2
            text_y = back_button.y + (back_button.height - back_text.get_height()) // 2
            screen.blit(back_text, (text_x, text_y))

            pygame.display.flip()

            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    return False
                elif event.type == pygame.MOUSEBUTTONDOWN:
                    mouse_pos = event.pos
                    for i, button in enumerate(target_buttons):
                        if button.collidepoint(mouse_pos):
                            sound1.play()
                            self.target_score = targets[i]
                            pygame.display.flip()
                            pygame.time.delay(1000)  
                            self.board = Board(GRID_SIZE) 
                            self.run()
                            menu = False
                            return True 
                    if back_button.collidepoint(mouse_pos):
                        sound1.play()
                        return False  
        return True



    def run(self):
        self.board.add_new_tile()
        self.board.add_new_tile()

        running = True
        while running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.high_score.save_score(self.board.score)
                    running = False
                elif event.type == pygame.KEYDOWN:
                    if self.board.sounds(event.key):
                        self.board.add_new_tile()
                        if self.board.is_game_won(self.target_score):
                            self.high_score.save_score(self.board.score)
                            running = False
                        elif self.board.is_game_over():
                            self.high_score.save_score(self.board.score)
                            running = False

            screen.fill(BACKGROUND_COLOR)
            self.draw_grid()
            self.draw_scores()
            pygame.display.flip()

        if self.board.is_game_won(self.target_score):
            sound3.play()
            self.display_continue_window()
        else:
            sound4.play()
            self.display_restart_window()

game = Game()
game.main_menu()
