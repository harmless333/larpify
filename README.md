import pygame
import socket
import json
import threading

pygame.init()

screen = pygame.display.set_mode((900, 600))
clock = pygame.time.Clock()

sock = socket.socket()
sock.connect(("192.168.178.25", 5555))

me = int(sock.recv(10).decode())

game = {"p1": 300, "p2": 300, "x": 450, "y": 300, "s1": 0}

def receive():
    global game
    while True:
        try:

            game = json.loads(sock.recv(4096).decode())

        except:
            break

threading.Thread(target=receive, daemon=True).start()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            quit()

    keys = pygame.key.get_pressed()

    if me == 1:
        if keys[pygame.K_w]:
            game["p1"] -= 7
        if keys[pygame.K_s]:
            game["p1"] += 7
        y = game["p1"]
    else:
        if keys[pygame.K_UP]:
            game["p2"] -= 7
        if keys[pygame.K_DOWN]:
            game["p2"] += 7
        y = game["p2"]

    y = max(50, min(550, y))

    sock.send(json.dumps({"y": y}).encode())

    screen.fill((20, 20, 20))

    pygame.draw.rect(screen, "white", (30, game["p1"] - 50, 15, 100))
    pygame.draw.rect(screen, "white", (855, game["p2"] - 50, 15, 100))
    pygame.draw.circle(screen, "white", (game["x"], game["y"]), 10)

    font = pygame.font.SysFont(None, 50)
    text = font.render(
        f'{game.get("s1", 0)}   {game.get("s2", 0)}',
        True,
        "white"
    )
    screen.blit(text, (400, 30))

    pygame.display.flip()
    clock.tick(60)
