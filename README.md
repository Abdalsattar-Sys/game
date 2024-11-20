import pygame
import random
import sys

# Initialize pygame
pygame.init()

# Constants
CELL_SIZE = 20
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
BLACK = (0, 0, 0)
OBSTACLE_COLOR = (255, 165, 0)

# Get screen size dynamically
info = pygame.display.Info()
WIDTH = (info.current_w // CELL_SIZE) * CELL_SIZE  # Ensure width is a multiple of CELL_SIZE
HEIGHT = (info.current_h // CELL_SIZE) * CELL_SIZE  # Ensure height is a multiple of CELL_SIZE

# Classes
class Snake:
    def __init__(self):
        self.body = [[100, 100], [80, 100], [60, 100]]
        self.direction = "RIGHT"
        self.new_direction = "RIGHT"

    def move(self):
        head = self.body[0][:]
        if self.new_direction == "UP" and self.direction != "DOWN":
            self.direction = "UP"
        elif self.new_direction == "DOWN" and self.direction != "UP":
            self.direction = "DOWN"
        elif self.new_direction == "LEFT" and self.direction != "RIGHT":
            self.direction = "LEFT"
        elif self.new_direction == "RIGHT" and self.direction != "LEFT":
            self.direction = "RIGHT"

        if self.direction == "UP":
            head[1] -= CELL_SIZE
        elif self.direction == "DOWN":
            head[1] += CELL_SIZE
        elif self.direction == "LEFT":
            head[0] -= CELL_SIZE
        elif self.direction == "RIGHT":
            head[0] += CELL_SIZE

        self.body.insert(0, head)
        self.body.pop()

    def grow(self):
        tail = self.body[-1][:]
        self.body.append(tail)

    def check_collision(self, obstacles):
        head = self.body[0]
        # Collision with walls
        if head[0] < 0 or head[0] >= WIDTH or head[1] < 0 or head[1] >= HEIGHT:
            return True
        # Collision with itself
        if head in self.body[1:]:
            return True
        # Collision with obstacles
        if head in obstacles:
            return True
        return False

class Food:
    def __init__(self):
        self.position = [random.randrange(0, WIDTH, CELL_SIZE),
                         random.randrange(0, HEIGHT, CELL_SIZE)]

    def generate_new_position(self, snake_body, obstacles):
        while True:
            self.position = [random.randrange(0, WIDTH, CELL_SIZE),
                             random.randrange(0, HEIGHT, CELL_SIZE)]
            if self.position not in snake_body and self.position not in obstacles:
                break

class GameScreen:
    def __init__(self):
        self.window = pygame.display.set_mode((WIDTH, HEIGHT))
        pygame.display.set_caption("Snake Game")
        self.clock = pygame.time.Clock()
        self.score = 0
        self.level = 1

    def update(self, snake, food, obstacles):
        self.window.fill(BLACK)
        # Draw obstacles
        for obstacle in obstacles:
            pygame.draw.rect(self.window, OBSTACLE_COLOR, pygame.Rect(obstacle[0], obstacle[1], CELL_SIZE, CELL_SIZE))
        # Draw snake
        for segment in snake.body:
            pygame.draw.rect(self.window, GREEN, pygame.Rect(segment[0], segment[1], CELL_SIZE, CELL_SIZE))
        # Draw food
        pygame.draw.rect(self.window, RED, pygame.Rect(food.position[0], food.position[1], CELL_SIZE, CELL_SIZE))
        # Display score and level
        self.display_score_and_level()
        pygame.display.flip()

    def display_score_and_level(self):
        font = pygame.font.SysFont("arial", 20)
        score_text = font.render(f"Score: {self.score}  Level: {self.level}", True, WHITE)
        self.window.blit(score_text, (10, 10))

class GameControl:
    def __init__(self):
        self.snake = Snake()
        self.food = Food()
        self.screen = GameScreen()
        self.running = True
        self.obstacles = []

    def handle_input(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                self.running = False
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    self.snake.new_direction = "UP"
                elif event.key == pygame.K_DOWN:
                    self.snake.new_direction = "DOWN"
                elif event.key == pygame.K_LEFT:
                    self.snake.new_direction = "LEFT"
                elif event.key == pygame.K_RIGHT:
                    self.snake.new_direction = "RIGHT"

    def check_game_over(self):
        if self.snake.check_collision(self.obstacles):
            self.running = False

    def add_obstacle(self):
        while True:
            obstacle = [random.randrange(0, WIDTH, CELL_SIZE),
                        random.randrange(0, HEIGHT, CELL_SIZE)]
            if obstacle not in self.snake.body and obstacle != self.food.position and obstacle not in self.obstacles:
                self.obstacles.append(obstacle)
                break

    def run(self):
        while self.running:
            self.handle_input()
            self.snake.move()

            # Check if the snake eats the food
            if self.snake.body[0] == self.food.position:
                self.snake.grow()
                self.food.generate_new_position(self.snake.body, self.obstacles)
                self.screen.score += 1
                # Increase level and speed
                if self.screen.score % 5 == 0:
                    self.screen.level += 1
                    self.add_obstacle()

            self.check_game_over()
            self.screen.update(self.snake, self.food, self.obstacles)
            self.screen.clock.tick(10 + self.screen.level * 2)

        print("Game Over! Final Score:", self.screen.score)
        pygame.quit()
        sys.exit()

# Main execution
if __name__ == "__main__":
    game = GameControl()
    game.run()
