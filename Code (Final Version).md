Main.py
```python
import pygame, random, enemy, sprite_sheet
import time
from pygame.locals import *
from player import Player
from enemy import Enemy
from background import Background
from sprite_sheet import Sprite_Sheet
from laser import Laser


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

player = Player()
#Player object

lasers = pygame.sprite.Group()
#laser array.

count = 0
#Keeps track of when player.image hasn't formed

spriteNum = 0
#spriteNum which will be used to locate the exact spaceship the user chose

enemies = pygame.sprite.Group()
enemy_count = random.randint(7, 10)


def init():
    global enemies, enemy_count, background, player, speed
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
    mediumFont = pygame.font.SysFont(defaultFont, 65)
    smallFont = pygame.font.SysFont(defaultFont, 50)
    tinyFont = pygame.font.SysFont(defaultFont, 35)
    choosePlayer = mediumFont.render('Choose your spaceship to start:', True, (0, 0, 0))
    backstory = tinyFont.render("Welcome to the Elite Space Academy Entrace Test. You will be", True, (0, 0, 0))
    rules = tinyFont.render("teted on all the skills an Elite Space Academy student requires. A", True, (0, 0, 0))
    rules1 = tinyFont.render("score of 50 or less is deemed a failure and you can't enter the", True, (0, 0, 0))
    rules2 = tinyFont.render("Academy.", True, (0, 0, 0))
    good_luck = smallFont.render("Good luck!", True, (0, 0, 0))
    while True:
        screen.fill(color)
        sprite_sheet = Sprite_Sheet(screen, width, height, (width/8, height/8))
        pygame.draw.line(screen, (0, 0, 0), (width/2, height/4), (width/2, (3*height)/4), 1)
        pygame.draw.line(screen, (0, 0, 0), (width/4, height/2), ((3*width)/4, height/2), 1)
        # Sets up the sprite sheet.
        screen.blit(choosePlayer, (width/16, height/12))
        screen.blit(backstory, (0, (height * 3)/4))
        screen.blit(rules, (0, (height * 16)/20))
        screen.blit(rules1, (0, (height * 17)/20))
        screen.blit(rules2, (0, (height * 18)/20))
        screen.blit(good_luck, (width/2.5, (height * 19)/20))
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
    player.collect((width/2, height), spriteNum)
    lasers_list = []
    for i in range(0, 25):
        lasers_list.append(Laser((width/2, height)))
    lastLaser = len(lasers_list)
    init()
    while True:
        screen.fill([255, 255, 255])
        bigger_img = pygame.transform.scale(background.image, (width, height))
        screen.blit(bigger_img, (0, 0))
        playerPosition = player.update()
        #lasers_list = lasers.sprites()
        for i in range(0, len(lasers_list)):
            if lastLaser > 0 and i < len(lasers_list):
                asteroid_hit = pygame.sprite.spritecollide(lasers_list[i], enemies, False)
                if asteroid_hit:
                    lasers_list.pop(i)
                    for enemy in asteroid_hit:
                        enemy.score += 1
                        enemy.update(speed)
                        enemy.score = 0
                        enemy.rect.bottom = 0
                        enemy.rect.left = random.randint(0, width)
                        enemy.speed = pygame.math.Vector2(1, speed)
                        enemy.speed.rotate_ip(random.randint(30, 50))
                else:
                    kill = lasers_list[i].update(playerPosition, screen)
                    if kill == 1:
                        lasers_list.pop(i)
        enemies.update(speed)
        clock.tick(60)
        if lastLaser > 0:
            last_laser = lasers_list[lastLaser -1]
        player_hit = pygame.sprite.spritecollide(player, enemies, False)
        lasersLeftStr = "Lasers Left: " + str(lastLaser)
        if lastLaser > 15:
            laser_color = (100, 250, 100)
        elif lastLaser > 5:
            laser_color = (255, 100, 100)
        else:
            laser_color = (255, 0, 0)
        lasers_left = smallFont.render(lasersLeftStr, True, laser_color)
        lasers_x = width - 246.5
        lasers_left_rect = pygame.draw.rect(screen, (255, 255, 255), (lasers_x - 2, 0, width/3.25, height/15))
        screen.blit(lasers_left, (lasers_x, 0))
        screen.blit(player.image, player.rect)
        enemies.draw(screen)
        #lasers.draw(screen)
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
                if (event.key == pygame.K_SPACE):
                    if lastLaser > 0:
                        last_laser.rect.center = (player.rect.centerx, player.rect.top)
                        last_laser.track += 1
                        lastLaser -= 1
            if event.type == pygame.KEYUP:
                if event.key == pygame.K_RIGHT:
                    player.speed[0] = 0
                if event.key == pygame.K_LEFT:
                    player.speed[0] = 0
                if event.key == pygame.K_UP:
                    player.speed[1] = 0
                if event.key == pygame.K_DOWN:
                    player.speed[1] = 0
                if event.key == pygame.K_SPACE:
                    abcd = "0"
                    #nothing  
        if player_hit:
            enemies.update(speed)
            random_enemy = enemies.sprites()[0]
            random_enemy.get_score()
            enemy_score = random_enemy.score
            enemies.empty()
            largeFont = pygame.font.SysFont(defaultFont, 80)
            color1 = (200, 100, 0)
            screen.fill(color1)
            enemyString = "Score: " + str(enemy_score)
            score = smallFont.render(enemyString, True, (0, 0, 0))
            if enemy_score >= 50:
                youLost_str = "Congrats! You made it."
                lost_x = width/7
            else:
                youLost_str = "Sadly, you failed."
                lost_x = width/5
            youLost = largeFont.render(youLost_str, True, (0, 0, 0))
            retry = mediumFont.render('Retry', True, (0, 0, 0))
            quit = mediumFont.render('Quit', True, (0, 0, 0))
            retryx = width/8
            quitx = width/1.6
            retryy = height/1.5
            retryRect = pygame.draw.rect(screen, (200, 200, 200), (retryx - 2, retryy - 2, width/6.6, height/12))
            quitRect = pygame.draw.rect(screen, (200, 200, 200), (quitx - 2, retryy - 2, width/8, height/12)) 
            screen.blit(retry, (retryx, retryy))
            screen.blit(quit, (quitx, retryy))
            screen.blit(youLost, (lost_x, (height/3)))
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
                lasers_list = []
                for i in range(0, 25):
                    lasers_list.append(Laser((width/2, height)))
                lastLaser = len(lasers_list)
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
        self.scale = 0.7
        self.image = pygame.transform.smoothscale(
            self.image,
            (int(self.image.get_rect().width * self.scale),
             int(self.image.get_rect().height * self.scale))
        )
        self.rect = self.image.get_rect()
        # Getting the rectangle around the image lets you move it.
        self.rect.center = pos
        self.speed = pygame.math.Vector2(1, speed)
        self.speed.rotate_ip(random.randint(30, 50))
        self.score = 0

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
        score = score + self.score

    def get_score(self):
        global score
        self.score = score
```
Laser.py
```python
import pygame


class Laser(pygame.sprite.Sprite):
    def __init__(self, pos):
        super().__init__()
        self.image = pygame.image.load("laser.png")
        self.scale = 0.05
        self.image = pygame.transform.smoothscale(
            self.image,
            (int(self.image.get_rect().width * self.scale),
             int(self.image.get_rect().height * self.scale)),
        )
        self.rect = self.image.get_rect()
        # Getting the rectangle around the image lets you move it.
        self.rect.center = pos
        self.speed = pygame.math.Vector2(0, -5)
        self.speed.rotate_ip(0)
        self.track = 0

    def update(self, pos, screen):
        num = 0
        if self.track == 0:
            self.rect.move_ip(pos)
        else:
            self.move()
        screen.blit(self.image, self.rect)
        if self.rect.top < 0:
                num = 1
        return num

    def move(self):
        self.rect.move_ip(self.speed)
        self.track += 1
```
Player.py
```python
import pygame


class Player(pygame.sprite.Sprite):
    # Constructor function:
    def __init__(self):
        super().__init__()


    def collect (self, pos, spriteNum):
        if spriteNum == 1:
            image = pygame.image.load("orange_spaceship.png")
        elif spriteNum == 2:
            image = pygame.image.load("PinkSpaceship.png")
        elif spriteNum == 3:
            image = pygame.image.load("turq_spaceship.png")
        elif spriteNum == 4:
            image = pygame.image.load("blue_spaceship.png")
        self.image = image
        self.scale = 0.7
        self.image = pygame.transform.smoothscale(
            self.image,
            (int(self.image.get_rect().width * self.scale),
             int(self.image.get_rect().height * self.scale)),
        )
        self.rect = self.image.get_rect()
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
        return self.rect.left, self.rect.top
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
    def __init__(self, screen, width, height, location):
            global spriteNum, bigger_sheet
            spriteNum = 0
            self.spaceship_sheet = pygame.image.load("actual_spaceships.PNG")
            bigger_sheet = pygame.transform.scale(self.spaceship_sheet, ((width//2), (height//2)))
            screen.blit(bigger_sheet, ((width//4), (height//4)))
```
