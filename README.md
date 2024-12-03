# PrinciplesClass
import pygame
import random

# Initialize pygame
pygame.init()

# Screen dimensions and colors
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 1000
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
LIGHT_GRAY = (211, 211, 211)
DARK_GRAY = (169, 169, 169)
GREEN = (34, 139, 34)
YELLOW = (255, 223, 0)
GRAY = (105, 105, 105)

# Fonts
pygame.font.init()
FONT = pygame.font.Font(pygame.font.get_default_font(), 48)
KEYBOARD_FONT = pygame.font.Font(pygame.font.get_default_font(), 32)

# Load wordbank
wordbank_path = "/Users/orikgashi/PycharmProjects/WordleProject/.venv/lib/wordbank.txt"
with open(wordbank_path, "r") as file:
    words = [line.strip().upper() for line in file if len(line.strip()) == 5]

# Random word selection
if not words:
    raise ValueError("Wordbank is empty or invalid.")
target_word = random.choice(words)

# Initialize pygame screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Enhanced Wordle Game")

# Game variables
input_word = ""
guesses = []
max_attempts = 6
keyboard_keys = [
    "QWERTYUIOP",
    "ASDFGHJKL",
    "ZXCVBNM"
]
keyboard_feedback = {letter: LIGHT_GRAY for row in keyboard_keys for letter in row}
game_over = False  # Flag to indicate game-over state
win_message = ""  # Variable to store the win/lose message

# Layout settings
GRID_START_Y = 100
GRID_SIZE = 90
GRID_SPACING = 20  # Space between the squares in the grid
KEYBOARD_START_Y = 750

# Helper functions
def draw_grid():
    """Draw the grid for guesses, centered horizontally."""
    grid_width = (GRID_SIZE * 5) + (GRID_SPACING * 4)
    grid_start_x = (SCREEN_WIDTH - grid_width) // 2
    for row in range(max_attempts):
        for col in range(5):
            rect = pygame.Rect(grid_start_x + col * (GRID_SIZE + GRID_SPACING), GRID_START_Y + row * (GRID_SIZE + GRID_SPACING), GRID_SIZE, GRID_SIZE)
            pygame.draw.rect(screen, LIGHT_GRAY, rect, border_radius=8)
            pygame.draw.rect(screen, DARK_GRAY, rect, 3, border_radius=8)

def draw_keyboard():
    """Draw the on-screen keyboard with feedback, centered horizontally."""
    for row_idx, row in enumerate(keyboard_keys):
        row_width = len(row) * 60
        x_offset = (SCREEN_WIDTH - row_width) // 2
        for col_idx, key in enumerate(row):
            rect = pygame.Rect(x_offset + col_idx * 60, KEYBOARD_START_Y + row_idx * 60, 50, 50)
            pygame.draw.rect(screen, keyboard_feedback[key], rect, border_radius=8)
            pygame.draw.rect(screen, BLACK, rect, 2, border_radius=8)
            key_text = KEYBOARD_FONT.render(key, True, BLACK)
            screen.blit(key_text, (rect.x + 15, rect.y + 10))

def draw_guesses():
    """Display all guesses centered within the grid."""
    grid_width = (GRID_SIZE * 5) + (GRID_SPACING * 4)
    grid_start_x = (SCREEN_WIDTH - grid_width) // 2
    for row_idx, guess in enumerate(guesses):
        for col_idx, char in enumerate(guess):
            rect = pygame.Rect(grid_start_x + col_idx * (GRID_SIZE + GRID_SPACING), GRID_START_Y + row_idx * (GRID_SIZE + GRID_SPACING), GRID_SIZE, GRID_SIZE)
            # Feedback color for grid
            if char == target_word[col_idx]:
                color = GREEN
            elif char in target_word:
                color = YELLOW
            else:
                color = GRAY
            pygame.draw.rect(screen, color, rect, border_radius=8)
            pygame.draw.rect(screen, BLACK, rect, 3, border_radius=8)
            char_text = FONT.render(char, True, WHITE)
            screen.blit(char_text, (rect.x + 25, rect.y + 20))

def draw_current_input():
    """Display the current input word in the active row, centered within the grid."""
    row_idx = len(guesses)
    grid_width = (GRID_SIZE * 5) + (GRID_SPACING * 4)
    grid_start_x = (SCREEN_WIDTH - grid_width) // 2
    for col_idx, char in enumerate(input_word):
        rect = pygame.Rect(grid_start_x + col_idx * (GRID_SIZE + GRID_SPACING), GRID_START_Y + row_idx * (GRID_SIZE + GRID_SPACING), GRID_SIZE, GRID_SIZE)
        pygame.draw.rect(screen, LIGHT_GRAY, rect, border_radius=8)
        pygame.draw.rect(screen, DARK_GRAY, rect, 3, border_radius=8)
        char_text = FONT.render(char, True, BLACK)
        screen.blit(char_text, (rect.x + 25, rect.y + 20))

def update_keyboard_feedback(guess):
    """Update keyboard color feedback based on guesses."""
    for idx, char in enumerate(guess):
        if char == target_word[idx]:
            keyboard_feedback[char] = GREEN
        elif char in target_word:
            if keyboard_feedback[char] != GREEN:
                keyboard_feedback[char] = YELLOW
        else:
            keyboard_feedback[char] = DARK_GRAY

def display_message(message, color):
    """Display a message on the screen."""
    msg_text = FONT.render(message, True, color)
    screen.blit(msg_text, ((SCREEN_WIDTH - msg_text.get_width()) // 2, SCREEN_HEIGHT // 2))

# Main game loop
running = True
clock = pygame.time.Clock()

while running:
    screen.fill(WHITE)
    draw_grid()
    draw_keyboard()
    draw_guesses()
    draw_current_input()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        if not game_over:  # Only process input if game is not over
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RETURN and len(input_word) == 5:
                    guesses.append(input_word)
                    update_keyboard_feedback(input_word)
                    if input_word == target_word:
                        win_message = "You Win!"
                        game_over = True
                    elif len(guesses) == max_attempts:
                        win_message = f"You Lose! Correct Word: {target_word}"
                        game_over = True
                    input_word = ""
                elif event.key == pygame.K_BACKSPACE and len(input_word) > 0:
                    input_word = input_word[:-1]
                elif event.unicode.isalpha() and len(input_word) < 5:
                    input_word += event.unicode.upper()

    # Display the win/lose message if the game is over
    if game_over:
        display_message(win_message, GREEN if "Win" in win_message else RED)

    pygame.display.flip()
    clock.tick(30)

pygame.quit()