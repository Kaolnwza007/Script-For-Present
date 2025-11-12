snake game source

Python

import random  
from tkinter import *

# game over eiei
game_over_texts = ["หัวโน", "เอาหม่ายดิ๊", "โถ นิวเดียว", "OH NO!", "จิ๊ ๆ ไม่ไหว ๆ "]

# setting
GAME_WIDTH = 700
GAME_HEIGHT = 700
SPEED = 800
SPACE_SIZE = 78
BODY_PARTS = 3

# my fav color 
SNAKE_COLOR = "green"
FOOD_COLOR = "red"
BACKGROUND_COLOR = "black"

class Snake:
    
    def __init__(self):
        self.body_size = BODY_PARTS
        self.coordinates = []
        self.squares = []
        
        for i in range(0, BODY_PARTS):
            self.coordinates.append([0, 0])

        for x, y in self.coordinates:
            square = canvas.create_rectangle(x, y ,x + SPACE_SIZE,y + SPACE_SIZE, fill=SNAKE_COLOR, tag='snake')
            self.squares.append(square)

class Food:
    def __init__(self):
        while True:
            x = random.randint(0, (GAME_WIDTH // SPACE_SIZE) - 1) * SPACE_SIZE
            y = random.randint(0, (GAME_HEIGHT // SPACE_SIZE) - 1) * SPACE_SIZE

            if (x, y) not in used_food_positions:
                used_food_positions.add((x, y))
                break

        self.coordinates = [x, y]

        canvas.create_oval(x+5, y+10, x + SPACE_SIZE-5, y + SPACE_SIZE-5, fill="#cc0000", outline="#990000", width=2, tag="food")

        stem_x = x + SPACE_SIZE // 2
        stem_y_bottom = y + 10  
        stem_y_top = stem_y_bottom - 10  

        canvas.create_line(
        stem_x, stem_y_bottom,
        stem_x, stem_y_top,
        fill="#663300", width=5, tag="food"
        )

        leaf_tip_x = stem_x + 10   
        leaf_tip_y = stem_y_top + 5

        leaf_base_top_x = stem_x + 2  
        leaf_base_top_y = stem_y_top

        leaf_base_bottom_x = stem_x + 2  
        leaf_base_bottom_y = stem_y_top + 10

        canvas.create_polygon(leaf_tip_x, leaf_tip_y,leaf_base_top_x, leaf_base_top_y,leaf_base_bottom_x, leaf_base_bottom_y,
                              fill="green", outline="darkgreen", tag="food"
    )



used_food_positions = set()

def next_turn(snake, food):

    x, y = snake.coordinates[0]  

    if direction == "up":
        y -= SPACE_SIZE

    elif direction == "down":
        y += SPACE_SIZE

    elif direction == "left":
        x -= SPACE_SIZE

    elif direction == "right":
        x += SPACE_SIZE

    snake.coordinates.insert(0, (x, y))

    square = canvas.create_rectangle(x , y, x + SPACE_SIZE, y + SPACE_SIZE, fill=SNAKE_COLOR)

    snake.squares.insert(0, square)

    if x == food.coordinates[0] and y == food.coordinates[1]:
        global score
        score += 1
        canvas.itemconfig(score_text, text='Score: {}'.format(score))
        canvas.delete('food')
        food = Food()
    else:
        del snake.coordinates[-1]

        canvas.delete(snake.squares[-1])

        del snake.squares[-1]
    
    if check_collisions(snake):
        game_over()

    else:
        window.after(SPEED, next_turn, snake , food)


def change_direction(new_direction):
    
    global direction

    if new_direction == 'left':
        if direction != 'right':
            direction = new_direction

    elif new_direction == 'right':
        if direction != 'left':
            direction = new_direction

    elif new_direction == 'up':
        if direction != 'down':
            direction = new_direction

    elif new_direction == 'down':
        if direction != 'up':
            direction = new_direction


def check_collisions(snake):
    
    x, y = snake.coordinates[0]

    if x < 0 or x >= GAME_WIDTH:
        return True
    elif y < 0 or y >= GAME_HEIGHT:
        return True
    
    for body_part in snake.coordinates[1:]:
        if x == body_part[0] and y == body_part[1]:
            return True

    return False


def game_over():
    canvas.delete(ALL)
    random_text = random.choice(game_over_texts)
    text_id = canvas.create_text(canvas.winfo_width()/2,canvas.winfo_height()/2,font=('consolas', 70),text=random_text,fill="red",
        tag="gameover"
    )
    canvas.tag_bind(text_id, "<Button-1>", restart_game)
    canvas.tag_bind(text_id, "<Enter>", lambda e: canvas.config(cursor="hand2"))
    canvas.tag_bind(text_id, "<Leave>", lambda e: canvas.config(cursor=""))


def restart_game(event=None):
    global snake, food, score, direction, score_text
    canvas.delete(ALL)
    canvas.config(cursor="arrow")  
    score = 0
    direction = 'down'
    score_text = canvas.create_text(
    10, 10,
    anchor='nw',
    font=('consolas', 30),
    fill='white',
    text='Score: 0',
    tag='score'
)
    
    snake = Snake()
    food = Food()
    next_turn(snake, food)

    
window = Tk()
window.title("Snake Game")
window.resizable(False, False)

score = 0
direction = 'down'

# label = Label(window, text='Score: {}'.format(score), font=('Consolas', 40))
# label.pack()

canvas = Canvas(window, bg=BACKGROUND_COLOR, height=GAME_HEIGHT, width=GAME_WIDTH)
canvas.pack()

window.update()  

score_text = canvas.create_text(10, 10,anchor='nw',font=('consolas', 30),fill='white',text='Score: 0',tag='score')

window_width = window.winfo_width()
window_height = window.winfo_height()
screen_width = window.winfo_screenwidth()
screen_height = window.winfo_screenheight()

x = int((screen_width / 2) - (window_width / 2))
y = int((screen_height / 2) - (window_height / 2))

window.geometry(f"{window_width}x{window_height}+{x}+{y}")

window.bind('<Left>', lambda event: change_direction('left'))
window.bind('<Right>', lambda event: change_direction('right'))
window.bind('<Down>', lambda event: change_direction('down'))
window.bind('<Up>', lambda event: change_direction('up'))

snake = Snake()
food = Food()

next_turn(snake, food)

window.mainloop()
