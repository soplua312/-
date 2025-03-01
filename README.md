import pygame
import sys
import speech_recognition as sr
import threading
import random

# Инициализация pygame
pygame.init()

# Настройки окна
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 800
CELL_SIZE = 40  # Размер одной клетки лабиринта
FPS = 30

# Цвета
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
RED = (255, 0, 0)

# Лабиринт (карта)
# 1 - стены, 0 - свободное пространство, 2 - игрок, 3 - выход, 4 - противник
maze = [
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 2, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1],
    [1, 1, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1, 1, 0, 1],
    [1, 0, 0, 0, 0, 0, 1, 4, 0, 1, 0, 0, 1, 0, 1, 0, 0, 1, 0, 1],
    [1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 0, 1, 0, 1],
    [1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 1],
    [1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 1],
    [1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 4, 1],
    [1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1],
    [1, 4, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1],
    [1, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1],
    [1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1],
    [1, 0, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 3, 1]
]

# Начальная позиция игрока
player_pos = [1, 1]

# Счетчик ходов игрока
player_move_count = 0

# Инициализация окна
screen = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Голосовой лабиринт с противниками")

# Функция для отрисовки лабиринта
def draw_maze():
    for y, row in enumerate(maze):
        for x, cell in enumerate(row):
            rect = pygame.Rect(x * CELL_SIZE, y * CELL_SIZE, CELL_SIZE, CELL_SIZE)
            if cell == 1:
                pygame.draw.rect(screen, BLACK, rect)  # Стены
            elif cell == 0:
                pygame.draw.rect(screen, WHITE, rect)  # Свободное пространство
            elif cell == 2:
                pygame.draw.rect(screen, BLUE, rect)   # Игрок
            elif cell == 3:
                pygame.draw.rect(screen, GREEN, rect)  # Выход
            elif cell == 4:
                pygame.draw.rect(screen, RED, rect)    # Противник

# Функция для распознавания голосовой команды
def recognize_speech():
    recognizer = sr.Recognizer()
    while True:
        with sr.Microphone() as source:
            print("Скажите команду (вверх, вниз, влево, вправо):")
            audio = recognizer.listen(source, phrase_time_limit=2)  # Ограничение времени записи
            try:
                command = recognizer.recognize_google(audio, language="ru-RU")
                print(f"Вы сказали: {command}")
                update_position(command)
            except sr.UnknownValueError:
                print("Речь не распознана")
            except sr.RequestError:
                print("Ошибка сервиса распознавания речи")

# Функция для обновления позиции игрока
def update_position(command):
    global player_pos, player_move_count
    x, y = player_pos
    new_x, new_y = x, y

    if command == "вверх" and y > 0 and maze[y - 1][x] != 1:
        new_y = y - 1
    elif command == "вниз" and y < len(maze) - 1 and maze[y + 1][x] != 1:
        new_y = y + 1
    elif command == "влево" and x > 0 and maze[y][x - 1] != 1:
        new_x = x - 1
    elif command == "вправо" and x < len(maze[0]) - 1 and maze[y][x + 1] != 1:
        new_x = x + 1

    # Проверка на столкновение с противником
    if maze[new_y][new_x] == 4:
        print("Вы столкнулись с противником! Возвращаемся на старт.")
        reset_player()

        return

    # Обновление позиции игрока
    maze[y][x] = 0  # Очищаем текущую позицию игрока
    player_pos = [new_x, new_y]
    maze[new_y][new_x] = 2  # Устанавливаем новую позицию игрока

    # Увеличиваем счетчик ходов игрока
    player_move_count += 1

    # Если игрок сделал 2 хода, двигаем противников
    if player_move_count % 2 == 0:
        move_enemies()

# Функция для сброса игрока на стартовую позицию
def reset_player():
    global player_pos
    maze[player_pos[1]][player_pos[0]] = 0  # Очищаем текущую позицию игрока
    player_pos = [1, 1]  # Возвращаем игрока на старт
    maze[1][1] = 2  # Устанавливаем игрока на стартовую позицию

# Функция для перемещения противников
def move_enemies():
    for y, row in enumerate(maze):
        for x, cell in enumerate(row):
            if cell == 4:  # Если это противник
                # Возможные направления движения
                directions = [
                    (x + 1, y),  # Вправо
                    (x - 1, y),  # Влево
                    (x, y + 1),  # Вниз
                    (x, y - 1)   # Вверх
                ]
                # Фильтруем направления, чтобы не выходить за пределы лабиринта и не сталкиваться со стенами
                valid_directions = [
                    (nx, ny) for nx, ny in directions
                    if 0 <= nx < len(maze[0]) and 0 <= ny < len(maze) and maze[ny][nx] == 0
                ]
                if valid_directions:
                    # Выбираем случайное направление
                    new_x, new_y = random.choice(valid_directions)
                    # Перемещаем противника
                    maze[y][x] = 0
                    maze[new_y][new_x] = 4

# Основной игровой цикл
def game_loop():
    global player_pos
    clock = pygame.time.Clock()
    running = True

    # Запуск потока для распознавания речи
    speech_thread = threading.Thread(target=recognize_speech, daemon=True)
    speech_thread.start()

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

        # Отрисовка лабиринта
        screen.fill(WHITE)
        draw_maze()
        pygame.display.flip()

        # Проверка на победу
        if maze[player_pos[1]][player_pos[0]] == 3:
            print("Поздравляем! Вы нашли выход!")
            running = False
            break

        clock.tick(FPS)

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    game_loop()
