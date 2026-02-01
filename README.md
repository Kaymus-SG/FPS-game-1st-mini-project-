# FPS-game-1st-mini-project-
FPS game don’t expect much this is prototype btw
import pygame, math, sys, random

pygame.init()
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()

# -------- Map --------
world_map = [
    "1111111111",
    "1000000001",
    "1000010001",
    "1000000001",
    "1000000001",
    "1000100001",
    "1000000001",
    "1000000001",
    "1000000001",
    "1111111111",
]

TILE = 64
FOV = math.pi / 3
HALF_FOV = FOV / 2
NUM_RAYS = 200
MAX_DEPTH = 800
DELTA_ANGLE = FOV / NUM_RAYS
DIST = NUM_RAYS / (2 * math.tan(HALF_FOV))
SCALE = WIDTH // NUM_RAYS

# -------- Player --------
player_x, player_y = 150, 150
player_angle = 0
player_speed = 2

# -------- Waves --------
wave = 1
enemies = []

def spawn_wave():
    enemies.clear()
    count = 3 + wave * 2
    for _ in range(count):
        enemies.append([
            random.randint(2, 8) * TILE,
            random.randint(2, 8) * TILE,
            100
        ])

spawn_wave()

# -------- Shooting --------
last_shot = 0
shoot_delay = 300

def shoot():
    global last_shot
    now = pygame.time.get_ticks()
    if now - last_shot < shoot_delay:
        return
    last_shot = now

    for enemy in enemies[:]:
        dx = enemy[0] - player_x
        dy = enemy[1] - player_y
        angle = math.atan2(dy, dx) - player_angle

        if abs(angle) < 0.05:
            enemy[2] -= 50
            if enemy[2] <= 0:
                enemies.remove(enemy)
            break

# -------- Raycasting --------
def cast_rays():
    start_angle = player_angle - HALF_FOV

    for ray in range(NUM_RAYS):
        angle = start_angle + ray * DELTA_ANGLE
        sin_a = math.sin(angle)
        cos_a = math.cos(angle)

        for depth in range(MAX_DEPTH):
            x = player_x + depth * cos_a
            y = player_y + depth * sin_a

            map_x = int(x // TILE)
            map_y = int(y // TILE)

            if world_map[map_y][map_x] == "1":
                depth *= math.cos(player_angle - angle)
                proj_height = DIST * TILE / (depth + 0.0001)
                shade = 255 / (1 + depth * depth * 0.00001)

                pygame.draw.rect(
                    screen,
                    (shade, shade, shade),
                    (ray * SCALE,
                     HEIGHT // 2 - proj_height // 2,
                     SCALE,
                     proj_height),
                )
                break

# -------- Enemies --------
def move_enemies():
    for enemy in enemies:
        dx = player_x - enemy[0]
        dy = player_y - enemy[1]
        dist = math.hypot(dx, dy)

        if dist > 20:
            enemy[0] += dx / dist * 0.7
            enemy[1] += dy / dist * 0.7

def draw_enemies():
    for enemy in enemies:
        dx = enemy[0] - player_x
        dy = enemy[1] - player_y

        distance = math.hypot(dx, dy)
        angle = math.atan2(dy, dx) - player_angle

        if -HALF_FOV < angle < HALF_FOV:
            screen_x = (angle + HALF_FOV) / FOV * WIDTH
            size = DIST * 40 / (distance + 0.0001)

            pygame.draw.rect(
                screen,
                (50, 200, 200),
                (screen_x - size // 2,
                 HEIGHT // 2 - size // 2,
                 size,
                 size),
            )

# -------- Main Loop --------
font = pygame.font.SysFont(None, 36)

running = True
while running:
    screen.fill((30, 30, 30))
    pygame.draw.rect(screen, (70, 70, 70),
                     (0, HEIGHT // 2, WIDTH, HEIGHT // 2))

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.MOUSEBUTTONDOWN:
            shoot()

    keys = pygame.key.get_pressed()

    if keys[pygame.K_a]:
        player_angle -= 0.04
    if keys[pygame.K_d]:
        player_angle += 0.04

    dx = math.cos(player_angle) * player_speed
    dy = math.sin(player_angle) * player_speed

    if keys[pygame.K_w]:
        player_x += dx
        player_y += dy
    if keys[pygame.K_s]:
        player_x -= dx
        player_y -= dy

    cast_rays()
    move_enemies()
    draw_enemies()

    # crosshair
    pygame.draw.circle(screen, (255,255,255),
                       (WIDTH//2, HEIGHT//2), 4)

    # wave progression
    if not enemies:
        wave += 1
        spawn_wave()

    wave_text = font.render(f"Wave {wave}", True, (255,255,255))
    screen.blit(wave_text, (10, 10))

    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
