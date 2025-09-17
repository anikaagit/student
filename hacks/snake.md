---
layout: default
title: Snake Game
permalink: /snake/
---

<style>
    body {
        /* Optional background or font styling */
    }
    .wrap {
        margin-left: auto;
        margin-right: auto;
    }

    canvas {
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: #FFFFFF;
        /* Add glowing border */
        box-shadow: 0 0 15px 3px #00FF00AA;
        transition: box-shadow 0.3s ease-in-out;
    }

    canvas:focus {
        outline: none;
        box-shadow: 0 0 25px 6px #00FF00FF; /* stronger glow when focused */
    }

    #gameover p,
    #setting p,
    #menu p {
        font-size: 20px;
        text-shadow: 0 0 5px #00FF00AA;
    }

    #gameover a,
    #setting a,
    #menu a {
        font-size: 30px;
        display: block;
        text-shadow: 0 0 7px #00FF00CC;
    }

    #gameover a:hover,
    #setting a:hover,
    #menu a:hover {
        cursor: pointer;
        text-shadow: 0 0 12px #00FF00FF;
    }

    #gameover a:hover::before,
    #setting a:hover::before,
    #menu a:hover::before {
        content: ">";
        margin-right: 10px;
    }

    #menu {
        display: block;
    }

    #gameover {
        display: none;
    }

    #setting {
        display: none;
    }

    #setting input {
        display: none;
    }

    #setting label {
        cursor: pointer;
        text-shadow: 0 0 5px #00FF00AA;
        transition: background-color 0.3s ease;
    }

    #setting input:checked + label {
        background-color: #FFF;
        color: #000;
        box-shadow: 0 0 10px #00FF00FF;
    }

    /* New UI overhaul for game over and settings */

    #gameover {
        background-color: #111;
        border-radius: 15px;
        padding: 20px;
        max-width: 300px;
        margin: 40px auto;
        box-shadow: 0 0 20px #00FF00CC;
        text-align: center;
    }

    #gameover p {
        font-size: 24px;
        margin-bottom: 15px;
    }

    #gameover .lives-remaining {
        font-size: 18px;
        color: #66FF66;
        margin-bottom: 15px;
        text-shadow: 0 0 8px #00FF00AA;
    }

    #gameover a {
        font-size: 26px;
        margin: 10px 0;
        padding: 10px 0;
        border-radius: 10px;
        background-color: #002200;
        text-decoration: none;
        color: #00FF00;
        display: block;
        box-shadow: 0 0 10px #00FF00CC;
        transition: background-color 0.3s ease;
    }

    #gameover a:hover {
        background-color: #005500;
        box-shadow: 0 0 20px #00FF00FF;
    }

    #setting {
        background-color: #111;
        border-radius: 15px;
        padding: 20px;
        max-width: 400px;
        margin: 40px auto;
        box-shadow: 0 0 20px #00FF00CC;
        color: #00FF00;
        font-family: monospace;
    }

    #setting p {
        margin: 12px 0;
    }

    #setting label {
        display: inline-block;
        margin: 0 15px 10px 0;
        padding: 5px 12px;
        border-radius: 6px;
        border: 1px solid #00FF00;
        box-shadow: 0 0 8px #00FF00AA;
    }

    #setting input:checked + label {
        background-color: #00FF00;
        color: #000;
        box-shadow: 0 0 15px #00FF00FF;
        border-color: #00CC00;
    }

    #setting a {
        font-size: 28px;
        color: #00FF00;
        text-decoration: none;
        display: block;
        margin-top: 20px;
        text-align: center;
        box-shadow: 0 0 10px #00FF00CC;
        border-radius: 10px;
        padding: 10px 0;
    }

    #setting a:hover {
        background-color: #005500;
        box-shadow: 0 0 20px #00FF00FF;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <p class="fs-4">Score: <span id="score_value">0</span> &nbsp;&nbsp; Lives: <span id="lives_value">3</span></p>

    <div class="container bg-secondary" style="text-align:center;">
        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over</p>
            <p class="lives-remaining">Lives Remaining: <span id="lives_remaining">0</span></p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="120" checked />
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75" />
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35" />
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked />
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0" />
                <label for="walloff">Off</label>
            </p>
            <p>Obstacles:
                <input id="obstacleon" type="radio" name="obstacle" value="1" checked />
                <label for="obstacleon">On</label>
                <input id="obstacleoff" type="radio" name="obstacle" value="0" />
                <label for="obstacleoff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
    (function () {
        const canvas = document.getElementById("snake");
        const ctx = canvas.getContext("2d");

        const SCREEN_SNAKE = 0;
        const SCREEN_MENU = -1;
        const SCREEN_GAME_OVER = 1;
        const SCREEN_SETTING = 2;

        const screen_snake = document.getElementById("snake");
        const screen_menu = document.getElementById("menu");
        const screen_game_over = document.getElementById("gameover");
        const screen_setting = document.getElementById("setting");

        const button_new_game = document.getElementById("new_game");
        const button_new_game1 = document.getElementById("new_game1");
        const button_new_game2 = document.getElementById("new_game2");
        const button_setting_menu = document.getElementById("setting_menu");
        const button_setting_menu1 = document.getElementById("setting_menu1");

        const ele_score = document.getElementById("score_value");
        const ele_lives = document.getElementById("lives_value");
        const ele_lives_remaining = document.getElementById("lives_remaining");
        const speed_setting = document.getElementsByName("speed");
        const wall_setting = document.getElementsByName("wall");
        const obstacle_setting = document.getElementsByName("obstacle");

        const BLOCK = 10;
        let SCREEN = SCREEN_MENU;
        let snake;
        let snake_dir;
        let snake_next_dir;
        let snake_speed;
        let foods = [];
        let score;
        let wall;
        let obstacles = [];
        let obstaclesEnabled;
        let lives;

        let gameInterval = null;

        const OBSTACLE_COUNT = 10;
        const FOOD_COUNT = 3;

        let showScreen = function (screen_opt) {
            SCREEN = screen_opt;
            screen_snake.style.display = (screen_opt === SCREEN_SNAKE || screen_opt === SCREEN_GAME_OVER) ? "block" : "none";
            screen_menu.style.display = (screen_opt === SCREEN_MENU) ? "block" : "none";
            screen_game_over.style.display = (screen_opt === SCREEN_GAME_OVER) ? "block" : "none";
            screen_setting.style.display = (screen_opt === SCREEN_SETTING) ? "block" : "none";
        };

        window.onload = function () {
            button_new_game.onclick = () => newGame();
            button_new_game1.onclick = () => newGame();
            button_new_game2.onclick = () => newGame();
            button_setting_menu.onclick = () => showScreen(SCREEN_SETTING);
            button_setting_menu1.onclick = () => showScreen(SCREEN_SETTING);

            // Initialize speed and event listeners
            setSnakeSpeed(120); // Default slow speed
            for (let i = 0; i < speed_setting.length; i++) {
                speed_setting[i].addEventListener("click", function () {
                    if (speed_setting[i].checked) {
                        setSnakeSpeed(parseInt(speed_setting[i].value));
                        startGameLoop();
                    }
                });
            }

            setWall(1);
            for (let i = 0; i < wall_setting.length; i++) {
                wall_setting[i].addEventListener("click", function () {
                    if (wall_setting[i].checked) setWall(wall_setting[i].value);
                });
            }

            // Obstacle toggle
            setObstacles(true);
            for (let i = 0; i < obstacle_setting.length; i++) {
                obstacle_setting[i].addEventListener("click", function () {
                    if (obstacle_setting[i].checked) {
                        setObstacles(obstacle_setting[i].value === "1");
                        if (SCREEN === SCREEN_SNAKE) {
                            generateObstacles();
                        }
                    }
                });
            }

            window.addEventListener("keydown", function (evt) {
                if (evt.code === "Space") {
                    if (SCREEN !== SCREEN_SNAKE) newGame();
                    else showScreen(SCREEN_SNAKE);
                }
            }, true);
        };

        let mainLoop = function () {
            let _x = snake[0].x;
            let _y = snake[0].y;
            snake_dir = snake_next_dir;

            switch (snake_dir) {
                case 0: _y--; break;
                case 1: _x++; break;
                case 2: _y++; break;
                case 3: _x--; break;
            }

            // Add new head position first
            snake.unshift({ x: _x, y: _y });

            // Check collisions
            if (checkCollision()) {
                lives--;
                altLives(lives);
                if (lives <= 0) {
                    gameOver();
                    return;
                }
                // Reset snake position & direction after losing life
                resetSnake();
                return;
            }

            // Check if snake eats food
            let ateFoodIndex = -1;
            for (let i = 0; i < foods.length; i++) {
                if (snake[0].x === foods[i].x && snake[0].y === foods[i].y) {
                    ateFoodIndex = i;
                    break;
                }
            }

            if (ateFoodIndex !== -1) {
                score += 1;
                altScore(score);
                // Remove eaten food and generate new one
                foods.splice(ateFoodIndex, 1);
                generateFood();
            } else {
                // Remove tail if not eating
                snake.pop();
            }

            draw();
        };

        let checkCollision = function () {
            let head = snake[0];

            // Wall collision
            if (wall) {
                if (head.x < 0 || head.x >= canvas.width / BLOCK ||
                    head.y < 0 || head.y >= canvas.height / BLOCK) {
                    return true;
                }
            } else {
                // Wrap around edges
                if (head.x < 0) head.x = (canvas.width / BLOCK) - 1;
                if (head.x >= canvas.width / BLOCK) head.x = 0;
                if (head.y < 0) head.y = (canvas.height / BLOCK) - 1;
                if (head.y >= canvas.height / BLOCK) head.y = 0;
                snake[0] = head;
            }

            // Self collision
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) return true;
            }

            // Obstacle collision
            if (obstaclesEnabled) {
                for (let i = 0; i < obstacles.length; i++) {
                    if (head.x === obstacles[i].x && head.y === obstacles[i].y) return true;
                }
            }

            return false;
        };

        let resetSnake = function () {
            snake = [
                { x: 12, y: 12 },
                { x: 11, y: 12 },
                { x: 10, y: 12 }
            ];
            snake_dir = 1; // moving right
            snake_next_dir = 1;
        };

        let newGame = function () {
            score = 0;
            altScore(score);
            lives = 3;
            altLives(lives);

            resetSnake();

            // Generate foods
            foods = [];
            for (let i = 0; i < FOOD_COUNT; i++) {
                generateFood();
            }

            // Generate obstacles
            generateObstacles();

            showScreen(SCREEN_SNAKE);

            startGameLoop();

            canvas.focus();
        };

        let gameOver = function () {
            showScreen(SCREEN_GAME_OVER);
            ele_lives_remaining.innerText = lives;
            clearInterval(gameInterval);
            gameInterval = null;
        };

        let startGameLoop = function () {
            if (gameInterval !== null) {
                clearInterval(gameInterval);
            }
            gameInterval = setInterval(mainLoop, snake_speed);
        };

        let generateFood = function () {
            let fx = Math.floor(Math.random() * (canvas.width / BLOCK));
            let fy = Math.floor(Math.random() * (canvas.height / BLOCK));

            // Make sure food does not spawn on snake, obstacles, or existing foods
            while (onSnake(fx, fy) || (obstaclesEnabled && onObstacle(fx, fy)) || onFood(fx, fy)) {
                fx = Math.floor(Math.random() * (canvas.width / BLOCK));
                fy = Math.floor(Math.random() * (canvas.height / BLOCK));
            }
            foods.push({ x: fx, y: fy });
        };

        let onSnake = function (x, y) {
            for (let i = 0; i < snake.length; i++) {
                if (snake[i].x === x && snake[i].y === y) return true;
            }
            return false;
        };

        let onObstacle = function (x, y) {
            for (let i = 0; i < obstacles.length; i++) {
                if (obstacles[i].x === x && obstacles[i].y === y) return true;
            }
            return false;
        };

        let onFood = function (x, y) {
            for (let i = 0; i < foods.length; i++) {
                if (foods[i].x === x && foods[i].y === y) return true;
            }
            return false;
        };

        let generateObstacles = function () {
            obstacles = [];
            if (!obstaclesEnabled) return;
            for (let i = 0; i < OBSTACLE_COUNT; i++) {
                let ox = Math.floor(Math.random() * (canvas.width / BLOCK));
                let oy = Math.floor(Math.random() * (canvas.height / BLOCK));
                while (onSnake(ox, oy) || onFood(ox, oy) || onObstacle(ox, oy)) {
                    ox = Math.floor(Math.random() * (canvas.width / BLOCK));
                    oy = Math.floor(Math.random() * (canvas.height / BLOCK));
                }
                obstacles.push({ x: ox, y: oy });
            }
        };

        let altScore = function (val) {
            ele_score.innerText = val;
        };

        let altLives = function (val) {
            ele_lives.innerText = val;
            ele_lives_remaining.innerText = val;
        };

        let setSnakeSpeed = function (speed) {
            snake_speed = speed;
            startGameLoop();
        };

        let setWall = function (val) {
            wall = Number(val);
        };

        let setObstacles = function (val) {
            obstaclesEnabled = (val === true || val === "1");
            if (!obstaclesEnabled) obstacles = [];
        };

        // Draw snake, food, obstacles
        let draw = function () {
            // Clear canvas
            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw obstacles
            ctx.fillStyle = "#444444";
            for (let i = 0; i < obstacles.length; i++) {
                ctx.fillRect(obstacles[i].x * BLOCK, obstacles[i].y * BLOCK, BLOCK, BLOCK);
            }

            // Draw foods - glowing red circles
            for (let i = 0; i < foods.length; i++) {
                let foodX = foods[i].x * BLOCK + BLOCK / 2;
                let foodY = foods[i].y * BLOCK + BLOCK / 2;
                let radius = BLOCK / 2 - 1;

                // Glow effect with shadow
                ctx.beginPath();
                ctx.shadowColor = "red";
                ctx.shadowBlur = 8;
                ctx.fillStyle = "red";
                ctx.arc(foodX, foodY, radius, 0, 2 * Math.PI);
                ctx.fill();
                ctx.shadowBlur = 0;
                ctx.closePath();
            }

            // Draw snake
            for (let i = 0; i < snake.length; i++) {
                ctx.fillStyle = (i === 0) ? "#00FF00" : "#00AA00";
                ctx.fillRect(snake[i].x * BLOCK, snake[i].y * BLOCK, BLOCK, BLOCK);
            }
        };

        window.addEventListener("keydown", function (evt) {
            switch (evt.keyCode) {
                case 37: // left arrow
                    if (snake_dir !== 1) snake_next_dir = 3;
                    break;
                case 38: // up arrow
                    if (snake_dir !== 2) snake_next_dir = 0;
                    break;
                case 39: // right arrow
                    if (snake_dir !== 3) snake_next_dir = 1;
                    break;
                case 40: // down arrow
                    if (snake_dir !== 0) snake_next_dir = 2;
                    break;
            }
        }, true);
    })();
</script>
