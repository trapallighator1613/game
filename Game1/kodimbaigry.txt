import pygame
import sys
import random

pygame.init()
pygame.mixer.init()

SCREEN_WIDTH = 1920
SCREEN_HEIGHT = 1020
GROUND_HEIGHT = 100
GREEN = (255, 255, 0)
WHITE = (255, 255, 255)

screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.time.set_timer(pygame.USEREVENT, 10)
pygame.display.set_caption("The Elder Meteors")
clock = pygame.time.Clock()
start_ticks = pygame.time.get_ticks()

dova = pygame.image.load("dova_stay.png")
dova_left = pygame.image.load("dova.png")
dova_right = pygame.image.load("dova_right.png")
dragon_image = pygame.image.load("meteor.png")
fon = pygame.image.load("background.jpg")
death = pygame.image.load("death.png")
health_image = pygame.image.load("heart.png")
pygame.mixer.music.load('secunda.mp3')
pygame.mixer.music.play(-1)
pygame.mixer.music.set_volume(0.25)

dova_x = (SCREEN_WIDTH // 2 - dova_left.get_width() // 2)
dova_y = (SCREEN_HEIGHT - GROUND_HEIGHT - dova_left.get_height())

heart_x = (SCREEN_WIDTH // 2 - health_image.get_width() // 2)
heart_y = (SCREEN_HEIGHT // 2 - health_image.get_height())
health = 3
dova_speed = 2.5
game_over = False


def print_text(message, x, y, font_color=(0, 0, 0), font_type='abeme.ttf', font_size=24):
    font_type = pygame.font.Font(font_type, font_size)
    text = font_type.render(message, True, font_color)
    screen.blit(text, (x, y))


def pause():
    paused_game = True
    pygame.mixer.music.pause()
    while paused_game:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()

        print_text('Game paused. Press BACKSPACE to continue.', 700, 300)

        keys = pygame.key.get_pressed()
        if keys[pygame.K_BACKSPACE]:
            paused_game = False
        pygame.display.update()
        clock.tick(15)
    pygame.mixer.music.unpause()


def show_health():
    show = 0
    x = 75
    while show != health:
        screen.blit(health_image, (x, 100))
        x += 40
        show += 1


hearts = []
obstacles = []
score = 0
f = 1
n = 3
all_sprites = pygame.sprite.Group()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    if not game_over:
        keys = pygame.key.get_pressed()
        if keys[pygame.K_a] and dova_x > 0:
            dova_x -= dova_speed
        if keys[pygame.K_d] and dova_x < SCREEN_WIDTH - dova_left.get_width():
            dova_x += dova_speed
        if keys[pygame.K_SPACE]:
            dova_y = dova_y - 2
        elif dova_y < 820:
            dova_y = dova_y + 1.5
        if keys[pygame.K_ESCAPE]:
            game_over = True
        if keys[pygame.K_F9]:
            pause()
        if n > 1:
            f += 2
            n += 1
            if random.randint(0, 30) < 0.5 and len(obstacles) < 30:
                obstacle_x = random.randint(0, SCREEN_WIDTH - dragon_image.get_width())
                obstacle_y = dragon_image.get_height()
                obstacles.append((obstacle_x, obstacle_y))
            if random.randint(0, 30) < 0.5 and len(hearts) < 1:
                heart_x = random.randint(0, SCREEN_WIDTH - health_image.get_width())
                heart_y = health_image.get_height()
                hearts.append((heart_x, heart_y))

        for i in range(len(obstacles)):
            obstacle_x, obstacle_y = obstacles[i]
            obstacle_y += 1
            obstacles[i] = obstacle_x, obstacle_y
        for i in range(len(hearts)):
            heart_x, heart_y = hearts[i]
            heart_y += 1
            hearts[i] = heart_x, heart_y

        obstacles = [(x, y) for x, y in obstacles if y < SCREEN_HEIGHT]
        hearts = [(x, y) for x, y in hearts if y < SCREEN_HEIGHT]


        def check_health():
            global health
            global game_over
            global obstacle
            global obstacle_x
            global obstacle_y

            for obstacle in obstacles:
                obstacle_x, obstacle_y = obstacle
                if (
                        dova_x < obstacle_x + (dragon_image.get_width() - 20)
                        and dova_x + dova_left.get_width() > obstacle_x
                        and dova_y < obstacle_y + (dragon_image.get_height() - 20)
                        and dova_y + dova_left.get_height() > obstacle_y
                ):
                    health -= 1
                    obstacles.clear()
                    hearts.clear()
                if health == 0:
                    game_over = True
                    if game_over:
                        pygame.mixer.music.load('skyrim.mp3')
                        pygame.mixer.music.play(-1)


        screen.blit(fon, (0, 0))
        screen.blit(dova, (dova_x, dova_y))
        show_health()
        check_health()

        if keys[pygame.K_a] and dova_x > 0:
            screen.blit(dova_left, (dova_x - 13, dova_y))
        if keys[pygame.K_d] and dova_x < SCREEN_WIDTH:
            screen.blit(dova_right, (dova_x - 8, dova_y))
        for heart in hearts:
            heart_x, heart_y = heart
            if (
                    dova_x < heart_x + (health_image.get_width() - 20)
                    and dova_x + dova_left.get_width() > heart_x
                    and dova_y < heart_y + (health_image.get_height() - 20)
                    and dova_y + dova_left.get_height() > heart_y
            ):
                health += 1
                hearts.clear()
        screen.blit(health_image, (heart_x, heart_y))
        for obstacle in obstacles:
            obstacle_x, obstacle_y = obstacle

            screen.blit(dragon_image, (obstacle_x, obstacle_y))
    else:
        screen.blit(death, (0, 0))

        keys = pygame.key.get_pressed()
        if keys[pygame.K_r]:
            pygame.mixer.music.load('secunda.mp3')
            pygame.mixer.music.play(-1)
            game_over = False
            dova_x = SCREEN_WIDTH // 2 - dova_left.get_width() // 2
            dova_y = SCREEN_HEIGHT - GROUND_HEIGHT - dova_left.get_height()
            obstacles = []
            health = 3

    pygame.display.flip()
