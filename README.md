import pygame
from pygame.locals import *
import random


pygame.init()
# Window dimensions
width, height = 500, 500
screen = pygame.display.set_mode((width, height))
pygame.display.set_caption('Car Game')

# Colors
gray, green, red, white, yellow = (100, 100, 100), (76, 208, 56), (200, 0, 0), (255, 255, 255), (255, 232, 0)

# Road settings
road_width = 300
lane_marker_height = 50
lanes = [150, 250, 350]

# Initialize player position
player_x, player_y = 250, 400

# Frame settings
clock = pygame.time.Clock()
fps = 120

# Game variables
gameover = False
speed = 2
score = 0


# Create font for text display
font = pygame.font.Font('fonts/MorrisRomanAlternate-Black.ttf', 28)

class Vehicle(pygame.sprite.Sprite):
    def __init__(self, image, x, y):
        super().__init__()
        image_scale = 45 / image.get_rect().width
        new_size = (int(image.get_rect().width * image_scale), int(image.get_rect().height * image_scale))
        self.image = pygame.transform.scale(image, new_size)
        self.rect = self.image.get_rect(center=(x, y))
    
      class PlayerVehicle(Vehicle):
    def __init__(self, x, y):
        image = pygame.image.load('images/car2.png')
        super().__init__(image, x, y)

def reset_game():
    global gameover, speed, score
    gameover = False
    speed = 2
    score = 0
    vehicle_group.empty()
    player.rect.center = [player_x, player_y]

def display_score():
    score_text = font.render(f'Score: {score}', True, white)
    screen.blit(score_text, (10, 10))
    

def game_over_screen():
    screen.blit(crash, crash_rect)
    pygame.draw.rect(screen, red, (0, 50, width, 100))
    over_text = font.render('Game Over. Play again? (Y/N)', True, white)
    screen.blit(over_text, (width // 6, 100))
    pygame.display.update()

# Load assets
player = PlayerVehicle(player_x, player_y)
player_group = pygame.sprite.GroupSingle(player)
vehicle_group = pygame.sprite.Group()
image_filenames = ['pickup_truck.png', 'semi_trailer.png', 'taxi.png', 'van.png', 'car3.png', 'car4.png']
vehicle_images = [pygame.image.load(f'images/{img}') for img in image_filenames]
crash = pygame.image.load('images/crash.png')
crash_rect = crash.get_rect()

# Game loop
running = True
while running:
    clock.tick(fps)
    
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False

        if event.type == KEYDOWN:
            if event.key == K_LEFT and player.rect.centerx > lanes[0]:
                player.rect.x -= 100
            if event.key == K_RIGHT and player.rect.centerx < lanes[2]:
                player.rect.x += 100  
                screen.fill(green)
    pygame.draw.rect(screen, gray, (100, 0, road_width, height))
    
    # Add lane markers
    for y in range(0, height, lane_marker_height * 2):
        for lane in lanes:
            pygame.draw.rect(screen, white, (lane + 45 - 10, y, 10, lane_marker_height))

    # Draw player and vehicles
    player_group.draw(screen)
    vehicle_group.draw(screen)

    # Vehicle movement and collision check
    for vehicle in vehicle_group:
        vehicle.rect.y += speed
        if vehicle.rect.top > height:
            vehicle.kill()
            score += 1
            if score % 5 == 0:  # Increase speed every 5 points
                speed += 1
    
    if len(vehicle_group) < 2:
        add_vehicle = True
        for vehicle in vehicle_group:
            if vehicle.rect.top < vehicle.rect.height * 1.5:
                add_vehicle = False
        if add_vehicle:
            lane = random.choice(lanes)
            image = random.choice(vehicle_images)
            vehicle_group.add(Vehicle(image, lane, -height // 2))
    
    # Display score
    display_score()
    
    # Check for collisions
    if pygame.sprite.spritecollide(player, vehicle_group, True):
        gameover = True
        crash_rect.center = player.rect.center

    if gameover:
        game_over_screen()
        while gameover:
            for event in pygame.event.get():
                if event.type == QUIT:
                    gameover = False
                    running = False
                if event.type == KEYDOWN:
                    if event.key == K_y:
                        reset_game()
                    elif event.key == K_n:
                        running = False
                        gameover = False
    
    pygame.display.update()

pygame.quit()
