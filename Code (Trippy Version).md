# Code:
*I dont really know why you would really need this code but here you go:*
Main.py
```python
import pygame, random, enemy, sprite_sheet
import time
from pygame.locals import *
from player import Player
from enemy import Enemy
from background import Background
from sprite_sheet import Sprite_Sheet


pygame.init()

# get screen info
screen_info = pygame.display.Info()
width = screen_info.current_w
height = screen_info.current_h
# Sets up screen according to height and width of device's screen (responsive).
size = (width, height)

screen = pygame.display.set_mode(size)

clock = pygame.time.Clock()

background = Background("game_backdrop.jpg")
# Sets up the background

speed = random.randint(3, 7)
#Random num. for speed for each asteroid.

spriteNum = 0
#spriteNum which will be used to locate the exact spaceship the user chose

enemies = pygame.sprite.Group()
enemy_count = 7


def init():
    global enemies, enemy_count, background, player
    player = Player((width/2, height), spriteNum, width, height)
    player.reset((width/2, height))
    enemies.empty()
    for i in range(enemy_count):
        enemies.add(Enemy((
            random.randint(0, width),#Finds a random number for the x position
            50),
            random.choice([0.1, 0.2, 0.3, 0.4, 0.5]),
            speed
        ))
    player.speed[1] = 5
    player.speed[0] = 0
    #The above 2 lines are used to prevent a bug when the user presses retry. Try it out and see if you find it ( ͡~ ͜ʖ ͡°).


def main():
    color = (135,206,235)
    pygame.font.init()
    defaultFont = pygame.font.get_default_font()
    largeFont = pygame.font.SysFont(defaultFont, 80)
    choosePlayer = largeFont.render('Choose your player:', True, (0, 0, 0))
    while True:
        screen.fill(color)
        sprite_sheet = Sprite_Sheet(screen, width, height, (width/8, height/8), True)
        pygame.draw.line(screen, (0, 0, 0), (width/2, height/4), (width/2, (3*height)/4), 1)
        pygame.draw.line(screen, (0, 0, 0), (width/4, height/2), ((3*width)/4, height/2), 1)
        # Sets up the sprite sheet.
        screen.blit(choosePlayer, (width/8, height/12))
        pygame.display.flip()
        while True:
            pygame.event.get()
            pressed = pygame.mouse.get_pressed()
            pos = pygame.mouse.get_pos()
            if True in pressed:
                x = pos[0]
                y = pos[1]
                if ((width//4)<= x <= (width//2) and ((height//4) <= y <= (height//2))):
                    spriteNum = 2
                    break
                elif ((((width//2) <= x <= ((3 * width)//4))) and ((height//4) <= y <= (height//2))):
                    spriteNum = 1
                    break
                elif ((((width//4) <= x <= (width//2))) and ((height//2) <= y <= ((3 * height)//4))):
                    spriteNum = 3
                    break
                elif ((((width//2) <= x <= ((3 * width)//4))) and ((height//2) <= y <= ((3 * height)//4))):
                    spriteNum = 4
                    break
        if spriteNum > 0:
            break
    init()
    while True:
        player.update()
        enemies.update(speed)
        clock.tick(60)
        player_hit = pygame.sprite.spritecollide(player, enemies, False)
        screen.fill([255, 255, 255])
        bigger_img = pygame.transform.scale(background.image, (width, height))
        screen.blit(bigger_img, (0, 0))
        enemies.draw(screen)
        screen.blit(player.image, player.rect)
        pygame.display.flip()
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_RIGHT:
                    player.speed[0] = 5
                if event.key == pygame.K_LEFT:
                    player.speed[0] = -5
                if event.key == pygame.K_UP:
                    player.speed[1] = -5
                if event.key == pygame.K_DOWN:
                    player.speed[1] = 5
            if event.type == pygame.KEYUP:
                if event.key == pygame.K_RIGHT:
                    player.speed[0] = 0
                if event.key == pygame.K_LEFT:
                    player.speed[0] = 0
                if event.key == pygame.K_UP:
                    player.speed[1] = 0
                if event.key == pygame.K_DOWN:
                    player.speed[1] = 0
        if player_hit:
            enemies.update(speed)
            enemies.empty()
            smallFont = pygame.font.SysFont(defaultFont, 50)
            mediumFont = pygame.font.SysFont(defaultFont, 65)
            color1 = (200, 100, 0)
            screen.fill(color1)
            enemyString = "Score: " + str(enemy.score)
            score = smallFont.render(enemyString, True, (0, 0, 0))
            youLost = largeFont.render('You Lost', True, (0, 0, 0))
            retry = mediumFont.render('Retry', True, (0, 0, 0))
            quit = mediumFont.render('Quit', True, (0, 0, 0))
            retryx = width/8
            quitx = width/1.6
            retryy = height/1.5
            retryRect = pygame.draw.rect(screen, (200, 200, 200), (retryx - 2, retryy - 2, width/6.6, height/12))
            quitRect = pygame.draw.rect(screen, (200, 200, 200), (quitx - 2, retryy - 2, width/8, height/12)) 
            screen.blit(retry, (retryx, retryy))
            screen.blit(quit, (quitx, retryy))
            screen.blit(youLost, ((3 * width)/10, height/3))
            screen.blit(score, ((3 * width)/8, (11 * height)/20))
            pygame.display.flip()
            done = None
            while True:
                pygame.event.get()
                pressed = pygame.mouse.get_pressed()
                pos = pygame.mouse.get_pos()
                if True in pressed:
                    x = pos[0]
                    y = pos[1]
                    if (((retryx - 2) <= x <= ((width/6.6) + (retryx - 2))) and ((retryy - 2) <= y <= ((height/12) + (retryy - 2)))):
                        done = False
                        break
                    elif (((quitx - 2) <= x <= ((width/6.6) + (quitx - 2))) and ((retryy - 2) <= y <= ((height/12) + (retryy - 2)))):
                        done = True
                        break
            if (done == False):
                init()
            elif (done == True):
                color2 = (120, 212, 30)
                screen.fill(color2)
                thanks = largeFont.render('Thanks For Playing :)', True, (0, 0, 0))
                screen.blit(thanks, ((width/8), (height/2)))
                pygame.display.flip()
                break
    #The below code is needed because repl.it erases the console after executing so the user never sees the thank you screen becuase the execution is so fast.
    start = time.clock()
    while True:
        end = time.clock()
        if (end - start) >= 1:
            break
    #The above code makes sure the thank you screen stays up for some time.

if __name__ == '__main__':
    main()
```
Enemy.py
```python
import pygame, random
score = 0


class Enemy(pygame.sprite.Sprite):
    def __init__(self, pos, scale, speed):
        global score
        #imports score
        score = 0
        #resets score to 0
        super().__init__()
        self.image = pygame.image.load("Asteroid.png")
        self.scale = 0.5
        self.image = pygame.transform.smoothscale(
            self.image,
            (int(self.image.get_rect().width * self.scale),
             int(self.image.get_rect().height * self.scale))
        )
        self.rect = self.image.get_rect()
        # Getting the rectangle around the image lets you move it.
        self.rect.center = pos
        self.speed = pygame.math.Vector2(1, speed)
        self.speed.rotate_ip(random.randint(-50, 50))


    def update(self, speed):
        global score
        screen_info = pygame.display.Info()
        # Aqquires the screen info
        self.rect.move_ip(self.speed)
        if self.rect.left < 0 or self.rect.right > screen_info.current_w:
            self.speed[0] *= -1
            self.rect.move_ip((self.speed[0], 0))
        if self.rect.bottom > screen_info.current_h:
            score = score + 1
            self.rect.bottom = 0
            self.rect.left = random.randint(0, screen_info.current_w)
            self.speed = pygame.math.Vector2(1, speed)
            self.speed.rotate_ip(random.randint(30, 50))
        # Make sure asteroid is in bounds of screen.
```
Player.pu
```python
import pygame
from sprite_sheet import Sprite_Sheet


class Player(pygame.sprite.Sprite):
    # Constructor function:
    def __init__(self, pos, spriteNum, width, height):
        super().__init__()
        sprite_sheet = Sprite_Sheet(1, 1, 1, (1, 1), False)
        #Just a bunch of random values since we dont want to call init() but we have to initalize an object of Sprite_Sheet to use the get_image method.
        self.image = sprite_sheet.get_image(spriteNum, width, height)
        self.scale = 0.1
        self.image = pygame.transform.smoothscale(
            self.image,
            (int(self.image.get_rect().width * self.scale),
             int(self.image.get_rect().height * self.scale)),
        )
        self.rect = self.image.get_rect()
        self.rect_width = self.image.get_rect().width
        self.rect_height = self.image.get_rect().height
        # Getting the rectangle around the image lets you move it.
        self.rect.center = pos
        self.speed = pygame.math.Vector2(0, 0)


    def update(self):
        self.rect.move_ip(self.speed)
        screen_info = pygame.display.Info()
        # Gathers information about the screen.
        if self.rect.right > screen_info.current_w:
            self.rect.right = screen_info.current_w
        if self.rect.left < 0:
            self.rect.left = 0
        if self.rect.bottom > screen_info.current_h:
            self.rect.bottom = screen_info.current_h
        if self.rect.top < 0:
            self.rect.top = 0
        # All if statements make sure the player doesnt go off the screen. By
        # readjusting the rect of the player into different x and y positions.

    def reset (self, pos):
        self.rect.center = pos
```
Sprite_Sheet.py
```python
import pygame
player_sheet = None
bigger_sheet = None
spriteNum = 0
#Sets up the image the user chos3

class Sprite_Sheet:
    def __init__(self, screen, width, height, location, boolean):
        if boolean == True:
            global spriteNum, bigger_sheet
            spriteNum = 0
            self.spaceship_sheet = pygame.image.load("actual_spaceships.PNG")
            bigger_sheet = pygame.transform.scale(self.spaceship_sheet, ((width//2), (height//2)))
            screen.blit(bigger_sheet, ((width//4), (height//4)))


    def get_image(self, spriteNum, width, height):
        global player_image, bigger_sheet
        player_image = bigger_sheet
        spaceship_sheet_width = player_image.get_rect().width
        spaceship_sheet_height = player_image.get_rect().height
        spaceship_width = spaceship_sheet_width/2
        spaceship_height = spaceship_sheet_height/2
        screen = pygame.Surface((width, height))
        x = 0
        y = 0
        if spriteNum == 1:
            x = 2 * spaceship_width
            y = spaceship_height
        elif spriteNum == 2:
            x = spaceship_width
            y = spaceship_height
        elif spriteNum == 3:
            x = spaceship_width
            y = 2 * spaceship_height
        elif spriteNum == 4:
            x = 2 * spaceship_width
            y = 2 * spaceship_height
        screen.blit(player_image, (0, 0), (x, y, spaceship_width,spaceship_height))
        screen.set_colorkey(screen.get_at((0, 0)))
        return screen
```
Background.py
```python
import pygame


class Background(pygame.sprite.Sprite):
    def __init__(self, image_file):
        super().__init__()
        self.image = pygame.image.load(image_file)
        #self.rect = self.image.get_rect()
        #self.rect.left, self.rect.top = location

```
