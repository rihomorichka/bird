import pygame
from random import randint

pygame.init()

WIDTH, HEIGHT = 800, 600 #рамка игры
FPS = 60 #частота кадров

window = pygame.display.set_mode((WIDTH, HEIGHT)) #открыть рамку
clock = pygame.time.Clock() #ограничение по времени

pygame.display.set_caption('Flappy bird') #название игры в углу
pygame.display.set_icon(pygame.image.load('images/icon.png')) #иконка игры в углу

font1 = pygame.font.Font(None, 35) #размер шрифта жизни и очки

imgBG = pygame.image.load('images/background.png') #загружаем пикчи
imgBird = pygame.image.load('images/bird.png')
imgPT = pygame.image.load('images/pipe_top.png')
imgPB = pygame.image.load('images/pipe_bottom.png')

pygame.mixer.music.load('sounds/music.mp3') #загружаем саундтрек игры миксер чтобы звук накладывался друг на друга
pygame.mixer.music.set_volume(0.3) #громкость
pygame.mixer.music.play(-1) #бесконечное повторение музыки

sndFall = pygame.mixer.Sound('sounds/fall.wav') #даем название для воспроизведения

py, sy, ay = HEIGHT // 2, 0, 0  #положение птички, скорость, ускорение
player = pygame.Rect(WIDTH // 3, py, 34, 24) #рамки птички
frame = 0 #для анимации

state = 'start' #начальное полжение птичка прост стоит
timer = 10 #нельзя управлять в течение этого времени

pipes = [] #создаем список труб
bges = [] #для фона
pipesScores = []

pipeSpeed = 8 #скорость приближения труб
pipeGateSize = 200 #ширина дырки
pipeGatePos = HEIGHT // 2 #хз пока

bges.append(pygame.Rect(0, 0, 288, 600)) #рамка фон

lives = 5 #жизни
scores = 0 #начальное число очков

play = True #пока значение тру цикл
while play:
    for event in pygame.event.get():
        if event.type == pygame.QUIT: #если пользователь закрыл то кык
            play = False

    press = pygame.mouse.get_pressed() #придаем значение типа
    keys = pygame.key.get_pressed()
    click = press[0] or keys[pygame.K_SPACE] #два варика о тип левая кнопка мыши

    if timer: timer -= 1 #таймер стремится к нулю

    frame = (frame + 0.2) % 4 #смена крыльев

    for i in range(len(bges) - 1, -1, -1): #
        bg = bges[i] #лист со значением ай
        bg.x -= pipeSpeed // 2 #скорость меньше чем у труб

        if bg.right < 0: bges.remove(bg) #удаляем бэкграунд чтобы память не забилась

        if bges[len(bges) - 1].right <= WIDTH: #если 
            bges.append(pygame.Rect(bges[len(bges) - 1].right, 0, 288, 600)) #добавляем фон

    for i in range(len(pipes) - 1, -1, -1):
        pipe = pipes[i]
        pipe.x -= pipeSpeed

        if pipe.right < 0:
            pipes.remove(pipe)
            if pipe in pipesScores:
                pipesScores.remove(pipe)

    if state == 'start':
        if click and not timer and not len(pipes): state = 'play' #начинаем игру если кликнули и нет труб

        py += (HEIGHT // 2 - py) * 0.1 #возвращаем в стартовое положение птицы в центре экрана
        player.y = py #возвращение на стартовое положение

    elif state == 'play': #игра
        if click:
            ay = -0.5 #ускорение и положение меняется птичка вниз оп
        else:
            ay = 0

        py += sy #увеличиваем положение птички на скорость
        sy = (sy + ay + 0.5) * 0.98 #чему равна скорость гравитация сопротивление
        player.y = py #возвращаем на место

        if not len(pipes) or pipes[len(pipes) - 1].x < WIDTH - 200:
            pipes.append(pygame.Rect(WIDTH, 0, 52, pipeGatePos - pipeGateSize // 2))
            pipes.append(
                pygame.Rect(WIDTH, pipeGatePos + pipeGateSize // 2, 52, HEIGHT - pipeGatePos + pipeGateSize // 2))

            pipeGatePos += randint(-100, 100)
            if pipeGatePos < pipeGateSize:
                pipeGatePos = pipeGateSize
            elif pipeGatePos > HEIGHT - pipeGateSize:
                pipeGatePos = HEIGHT - pipeGateSize

        if player.top < 0 or player.bottom > HEIGHT: state = 'fall' #если птичка ниже нуля то возвращаем ее в фолл и оттуда уже старт энд хир уи го эгэйн

        for pipe in pipes:
            if player.colliderect(pipe): state = 'fall' #чекаем на столкновение с рект трубы

            if pipe.right < player.left and pipe not in pipesScores: #чтобы труба удалялась иначе память забьется
                pipesScores.append(pipe) #добавляем в список
                scores += 5
                pipeSpeed = 3 + scores // 100 #скорость появления

    elif state == 'fall':
        sndFall.play()
        sy, ay = 0, 0 #сбрасываем значение чтобы не гнала
        pipeGatePos = HEIGHT // 2

        lives -= 1 #отнимаем жизнь
        if lives:
            state = 'start' #ждем клика
            timer = 60
        else:
            state = 'game over'
            timer = 180

    else:
        py += sy #механика падения птички место плюс скорость
        sy = (sy + ay + 1) * 0.98 #увеличение скорости плюс гравитация
        player.y = py

        if not timer: play = False

    # Отрисовка
    for bg in bges: window.blit(imgBG, bg)

    for pipe in pipes:
        if not pipe.y: #
            rect = imgPT.get_rect(bottomleft=pipe.bottomleft) #
            window.blit(imgPT, rect) #выводим изображение
        else:
            rect = imgPB.get_rect(topleft=pipe.topleft) #совмещение по верхнему левому углу
            window.blit(imgPB, rect) #выводим

    image = imgBird.subsurface(34 * int(frame), 0, 34, 24) #выбираем номер фрэйм для анимации крыльев и из большого изображения получаем маленькое координаты точки и ширина с высотой
    image = pygame.transform.rotate(image, -sy * 2) #исходное изображение и угол для движения носика птички
    window.blit(image, player)

    text = font1.render('Очки: ' + str(scores), 1, 'black')
    window.blit(text, (10, 10))

    text = font1.render('Жизни: ' + str(lives), 1, 'black')
    window.blit(text, (10, HEIGHT - 30))

    pygame.display.update() #чтоб обновляла экран
    clock.tick(FPS)

pygame.quit() #закрывается
