# Rock-Paper-Scissors
import pygame
import random
import sys
pygame.init()

try:
    win_sound = pygame.mixer.Sound('win.wav')
    tie_sound = pygame.mixer.Sound('tie.wav')
    lose_sound = pygame.mixer.Sound('lose.wav')
    pygame.mixer.music.load('background.mp3')
except Exception as e:
    print(f"Error loading sound files: {e}")
    win_sound = None
    tie_sound = None
    lose_sound = None

PINK = (255, 225, 225)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
PURPLE = (187, 171, 219)
BLUE = (0, 0, 255)

settings = {
    'font_color': BLACK,
    'background_color': PINK,
    'sound_effects': True,
    'background_music': True,
    'difficulty': 'easy'
}

WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Rock Paper Scissors")

font = pygame.font.Font(None, 40)
large_font = pygame.font.Font(None, 60)

class Button:
    def __init__(self, color, x, y, width, height, text='', image=None):
        self.color = color
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.text = text
        self.image = image

    def draw(self, screen, outline=None):
        if outline:
            pygame.draw.rect(screen, outline, (self.x - 2, self.y - 2, self.width + 4, self.height + 4))
        pygame.draw.rect(screen, self.color, (self.x, self.y, self.width, self.height))

        if self.image:
            loaded_image = pygame.image.load(self.image).convert_alpha()
            scaled_image = pygame.transform.scale(loaded_image, (self.width, self.height))
            screen.blit(scaled_image, (self.x, self.y))
        elif self.text != '':
            font_size = 30
            font = pygame.font.Font(None, font_size)
            text_surface = font.render(self.text, True, BLACK)

            while text_surface.get_width() > self.width - 20:
                font_size -= 1
                font = pygame.font.Font(None, font_size)
                text_surface = font.render(self.text, True, BLACK)

            text_x = self.x + (self.width - text_surface.get_width()) // 2
            text_y = self.y + (self.height - text_surface.get_height()) // 2
            screen.blit(text_surface, (text_x, text_y))

    def is_over(self, pos):
        return self.x < pos[0] < self.x + self.width and self.y < pos[1] < self.y + self.height

def text(text, font, color, x, y):
    text_surface = font.render(text, True, color)
    text_rect = text_surface.get_rect(center=(x, y))
    screen.blit(text_surface, text_rect)

def main_menu():
    menu_running = True
    start_button = Button(PURPLE, 300, 250, 200, 50, "Start")
    settings_button = Button(PURPLE, 300, 350, 200, 50, "Settings")
    back_button = Button(PURPLE, 300, 450, 200, 50, "Back")

    while menu_running:
        screen.fill(settings['background_color'])
        text("Main Menu", large_font, settings['font_color'], WIDTH // 2, 150)

        start_button.draw(screen, BLACK)
        settings_button.draw(screen, BLACK)
        back_button.draw(screen, BLACK)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()
                if start_button.is_over(pos):
                    return "start"
                elif settings_button.is_over(pos):
                    return "settings"
                elif back_button.is_over(pos):
                    return "back"

        pygame.display.update()

def settings_menu():
    global settings
    settings_running = True
    sound_effects_button = Button(PURPLE, 300, 250, 200, 50, "Sound Effects: ON" if settings['sound_effects'] else "Sound Effects: OFF")
    background_music_button = Button(PURPLE, 300, 350, 200, 50, "Background Music: ON" if settings['background_music'] else "Background Music: OFF")
    back_button = Button(PURPLE, 300, 450, 200, 50, "Back")

    while settings_running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()
                if sound_effects_button.is_over(pos):
                    settings['sound_effects'] = not settings['sound_effects']
                    sound_effects_button.text = "Sound Effects: ON" if settings['sound_effects'] else "Sound Effects: OFF"
                elif background_music_button.is_over(pos):
                    if settings['background_music']:
                        pygame.mixer.music.pause()
                    else:
                        pygame.mixer.music.unpause()
                    settings['background_music'] = not settings['background_music']
                    background_music_button.text = "Background Music: ON" if settings['background_music'] else "Background Music: OFF"
                elif back_button.is_over(pos):
                    return "back"

        screen.fill(settings['background_color'])
        text("Settings Menu", large_font, settings['font_color'], WIDTH // 2, 50)
        sound_effects_button.draw(screen, BLACK)
        background_music_button.draw(screen, BLACK)
        back_button.draw(screen, BLACK)

        pygame.display.update()

def sound(player_choice, computer_choice):
        if settings['sound_effects']:
            if player_choice == computer_choice:
                if tie_sound: tie_sound.play()
                return "Tie"
            elif (player_choice == 'rock' and computer_choice == 'scissors') or \
                    (player_choice == 'paper' and computer_choice == 'rock') or \
                    (player_choice == 'scissors' and computer_choice == 'paper'):
                if win_sound: win_sound.play()
                return "Win"
            else:
                if lose_sound: lose_sound.play()
                return "Lose"
        else:
            if player_choice == computer_choice:
                return "Tie"
            elif (player_choice == 'rock' and computer_choice == 'scissors') or \
                    (player_choice == 'paper' and computer_choice == 'rock') or \
                    (player_choice == 'scissors' and computer_choice == 'paper'):
                return "Win"
            else:
                return "Lose"

player_moves = {'rock': 0, 'paper': 0, 'scissors': 0}

def computer(difficulty, player_last_move):
    if difficulty == 'easy':
        return random.choice(['rock', 'paper', 'scissors'])
    elif difficulty == 'medium':
        if player_last_move:
            return random.choices(
                ['rock', 'paper', 'scissors'],
                weights=[1.5 if move == player_last_move else 1 for move in ['rock', 'paper', 'scissors']],
                k=1
            )[0]
        else:
            return random.choice(['rock', 'paper', 'scissors'])
    elif difficulty == 'hard':
        if max(player_moves.values()) > 0:
            most_common_move = max(player_moves, key=player_moves.get)
            return {'rock': 'paper', 'paper': 'scissors', 'scissors': 'rock'}[most_common_move]
        else:
            return random.choice(['rock', 'paper', 'scissors'])

def start(difficulty='easy'):
    player_score = 0
    computer_score = 0
    choices = ['rock', 'paper', 'scissors']
    end = False
    player_last_move = None
    back_button = Button(PURPLE, 300, 500, 200, 50, "Back")

    rock = Button(PURPLE, 150, 350, 150, 100, image='rock.png')
    paper = Button(RED, 350, 350, 150, 100, image='paper.png')
    scissors = Button(BLUE, 550, 350, 150, 100, image='scissors.png')

    computer_choice = None

    while not end:
        screen.fill(PINK)
        text("Rock Paper Scissors", large_font, BLACK, WIDTH // 2, 50)
        text(f"Player: {player_score} | Computer: {computer_score}", font, BLACK, WIDTH // 2, 150)

        rock.draw(screen, BLACK)
        paper.draw(screen, BLACK)
        scissors.draw(screen, BLACK)
        back_button.draw(screen, BLACK)

        if computer_choice is not None:
            text(f"Computer chose: {computer_choice.capitalize()}", font, BLACK, WIDTH // 2, 250)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()
                player_choice = None
                if rock.is_over(pos):
                    player_choice = 'rock'
                elif paper.is_over(pos):
                    player_choice = 'paper'
                elif scissors.is_over(pos):
                    player_choice = 'scissors'
                elif back_button.is_over(pos):
                    return "back"

                if player_choice:
                    player_moves[player_choice] += 1
                    player_last_move = player_choice
                    computer_choice = computer(difficulty, player_last_move)
                    result = sound(player_choice, computer_choice)
                    if result == "Win":
                        player_score += 1
                    elif result == "Lose":
                        computer_score += 1

        pygame.display.update()

def main():
    if settings['background_music']:
        pygame.mixer.music.play(-1)  
    while True:
        choice = main_menu()
        if choice == "start":
            start()
        elif choice == "settings":
            settings_menu()

if __name__ == "__main__":
    if settings['background_music']:
        pygame.mixer.music.play(-1)
    main()