/* Constants */
setSize(800, 600); // Make playground larger

const GRAVITY = 0.7;
const JUMP_STRENGTH = 16;
const PLAYER_WIDTH = 40;
const PLAYER_HEIGHT = 60;
const GROUND_HEIGHT = 40;
const MOVE_SPEED = 10;

const ENEMY_WIDTH = 40;
const ENEMY_HEIGHT = 40;
const ENEMY_SPEED = 8;

const PLATFORM_COLOR = "#555555"; // dark gray

/* Global variables */
let player;
let ground;

let platforms = [];
let enemies = [];

let isJumping = false;
let velocityY = 0;

let keys = {};

let score = 0;
let scoreLabel;

let gameOver = false;

function began() {
    sky();
    initPlayer();
    initGround();
    createPlatforms();
    enemyGround();
    createEnemies();
    initScoreLabel();

    keyDownMethod(handleKeyDown);
    keyUpMethod(handleKeyUp);

    setTimer(update, 20);
}

function initPlayer() {
    player = new WebImage("https://codehs.com/uploads/161a979d935937cfe673c76765375c73");
    player.setSize(PLAYER_WIDTH, PLAYER_HEIGHT);
    let startX = getWidth() / 2 - PLAYER_WIDTH / 2;
    let startY = getHeight() - GROUND_HEIGHT - PLAYER_HEIGHT;
    player.setPosition(startX, startY);
    add(player);
}

function initGround() {
    ground = new WebImage("https://codehs.com/uploads/cc727d840c5e1888a62682d99f410a86");
    ground.setSize(getWidth(), GROUND_HEIGHT);
    ground.setPosition(0, getHeight() - GROUND_HEIGHT);
    add(ground);
}

function sky() {
    var sky = new WebImage("https://codehs.com/uploads/7d84713450bf26d8d40c78a9a2adcf88");
    sky.setSize(getWidth(), getHeight());
    sky.setPosition(0, 0);
    add(sky);
}

function createPlatforms() {
    var platformData = [
        {x: 140, y: 130, width: 500, height: 30},
        {x: 100, y: 300, width: 300, height: 30},
        {x: 300, y: 400, width: 400, height: 30}
    ];
    for (var i = 0; i < platformData.length; i++) {
        var data = platformData[i];
        var platform = new WebImage("https://codehs.com/uploads/66ef6792d60484b85bbf2a10ce930cae");
        platform.setSize(data.width, data.height);
        platform.setPosition(data.x, data.y);
        add(platform);
        platforms.push(platform);
    }
}

function createEnemies() {
    for (let i = 0; i < platforms.length; i++) {
        const platform = platforms[i];
        const enemyX = platform.getX() + 10;
        const enemyY = platform.getY() - ENEMY_HEIGHT;
        const directionMultiplier = i % 2 === 0 ? 1 : -1;
        const enemy = createEnemy(
            enemyX,
            enemyY,
            ENEMY_SPEED * directionMultiplier
        );
        enemies.push(enemy);
    }
}

function enemyGround(){
    let EN = new WebImage("https://codehs.com/uploads/f96ea9e2aadb96f99a5104f7ace99458");
    EN.setSize(50, 40);
    let ENX = 100;
    let ENY = getHeight() - GROUND_HEIGHT - 40;
    EN.setPosition(ENX, ENY);
    add(EN);
    EN.vx = ENEMY_SPEED;
    enemies.push(EN);
}

function createEnemy(x, y, vx) {
    let enemy = new WebImage("https://codehs.com/uploads/50e6dc724d06cfceed826839b19082ce");
    enemy.setSize(ENEMY_WIDTH, ENEMY_HEIGHT);
    enemy.setPosition(x, y);
    enemy.vx = vx;
    add(enemy);
    return enemy;
}

function initScoreLabel() {
    scoreLabel = new Text("Score: 0");
    scoreLabel.setFont("20pt Arial");
    scoreLabel.setPosition(10, 30);
    add(scoreLabel);
}

function handleKeyDown(e) {
    let key = e.key || String.fromCharCode(e.charCode || e.keyCode);
    key = key.toLowerCase();
    keys[key] = true;
    if ((key === ' ' || key === 'arrowup' || key === 'w') && !isJumping && !gameOver) {
        isJumping = true;
        velocityY = -JUMP_STRENGTH;
    }
}

function handleKeyUp(e) {
    let key = e.key || String.fromCharCode(e.charCode || e.keyCode);
    key = key.toLowerCase();
    keys[key] = false;
}

function update() {
    if (gameOver) return;

    // Move left/right
    if (keys['arrowleft'] || keys['a']) {
        player.move(-MOVE_SPEED, 0);
        if (player.getX() < 0) {
            player.setPosition(0, player.getY());
        }
    }
    if (keys['arrowright'] || keys['d']) {
        player.move(MOVE_SPEED, 0);
        if (player.getX() + PLAYER_WIDTH > getWidth()) {
            player.setPosition(getWidth() - PLAYER_WIDTH, player.getY());
        }
    }

    // Jump and gravity
    if (isJumping) {
        velocityY += GRAVITY;
        player.move(0, velocityY);

        let playerBottom = player.getY() + PLAYER_HEIGHT;
        let groundTop = getHeight() - GROUND_HEIGHT;

        if (playerBottom >= groundTop) {
            player.setPosition(player.getX(), groundTop - PLAYER_HEIGHT);
            isJumping = false;
            velocityY = 0;
        } else {
            if (velocityY >= 0) {
                for (let plat of platforms) {
                    let platTop = plat.getY();
                    let platLeft = plat.getX();
                    let platRight = platLeft + plat.getWidth();
                    if (playerBottom >= platTop &&
                        playerBottom - velocityY < platTop &&
                        player.getX() + PLAYER_WIDTH > platLeft &&
                        player.getX() < platRight) {
                        player.setPosition(player.getX(), platTop - PLAYER_HEIGHT);
                        isJumping = false;
                        velocityY = 0;
                    }
                }
            }
        }
    } else {
        // Start falling if not on platform
        let standingOnPlatform = false;
        let playerBottom = player.getY() + PLAYER_HEIGHT;
        for (let plat of platforms) {
            let platTop = plat.getY();
            let platLeft = plat.getX();
            let platRight = platLeft + plat.getWidth();
            if (Math.abs(playerBottom - platTop) < 1 &&
                player.getX() + PLAYER_WIDTH > platLeft &&
                player.getX() < platRight) {
                standingOnPlatform = true;
                break;
            }
        }
        let onGround = (playerBottom === getHeight() - GROUND_HEIGHT);
        if (!standingOnPlatform && !onGround) {
            isJumping = true;
            velocityY = 0;
        }
    }

    // Enemy logic
    for (let i = enemies.length - 1; i >= 0; i--) {
        let enemy = enemies[i];
        enemy.move(enemy.vx, 0);

        if (enemy.getX() < 0 || enemy.getX() + ENEMY_WIDTH > getWidth()) {
            enemy.vx = -enemy.vx;
        }

        let standingOn = null;
        for (let plat of platforms) {
            if (Math.abs(enemy.getY() + ENEMY_HEIGHT - plat.getY()) < 1 &&
                enemy.getX() + ENEMY_WIDTH > plat.getX() &&
                enemy.getX() < plat.getX() + plat.getWidth()) {
                standingOn = plat;
                break;
            }
        }

        if (standingOn) {
            if (enemy.getX() < standingOn.getX() ||
                enemy.getX() + ENEMY_WIDTH > standingOn.getX() + standingOn.getWidth()) {
                enemy.vx = -enemy.vx;
            }
        } else {
            let groundY = getHeight() - GROUND_HEIGHT - ENEMY_HEIGHT;
            if (Math.abs(enemy.getY() - groundY) > 1) {
                enemy.setPosition(enemy.getX(), groundY);
            }
        }

        if (rectsCollide(player, enemy)) {
            let playerBottom = player.getY() + PLAYER_HEIGHT;
            let enemyTop = enemy.getY();
            if (velocityY > 0 && (playerBottom - enemyTop) < (PLAYER_HEIGHT / 2)) {
                // Remove the killed enemy and replace it at the same spot
                remove(enemy);
                enemies.splice(i, 1);
                score += 100;
                updateScore();
                velocityY = -JUMP_STRENGTH / 1.5;
                isJumping = true;

                // === SPAWN A NEW ENEMY AT THE SAME SPOT ===
                let newEnemy = createEnemy(enemy.getX(), enemy.getY(), enemy.vx);
                enemies.push(newEnemy);
            } else {
                endGame();
                return;
            }
        }
    }
}

function rectsCollide(rect1, rect2) {
    return !(
        rect1.getX() + rect1.getWidth() < rect2.getX() ||
        rect1.getX() > rect2.getX() + rect2.getWidth() ||
        rect1.getY() + rect1.getHeight() < rect2.getY() ||
        rect1.getY() > rect2.getY() + rect2.getHeight()
    );
}

function updateScore() {
    scoreLabel.setText("Score: " + score);
}

function endGame() {
    gameOver = true;
    let gameOverLabel = new Text("Game Over");
    gameOverLabel.setFont("40pt Arial");
    gameOverLabel.setColor(Color.black);
    gameOverLabel.setPosition(
        getWidth() / 2 - gameOverLabel.getWidth() / 2,
        getHeight() / 2
    );
    add(gameOverLabel);
}

function showStartScreen() {
    let startImage = new WebImage("https://codehs.com/uploads/678a45130afe1cf1e2e78b7cfb3551aa");
    startImage.setSize(getWidth(), getHeight());
    startImage.setPosition(0, 0);
    add(startImage);

    let startButton = new Rectangle(200, 50);
    startButton.setPosition(getWidth()/2 - 100, getHeight() - 100);
    startButton.setColor(Color.green);
    add(startButton);

    let buttonText = new Text("START GAME");
    buttonText.setFont("20pt Arial");
    buttonText.setColor(Color.white);
    buttonText.setPosition(
        startButton.getX() + (startButton.getWidth() - buttonText.getWidth()) / 2,
        startButton.getY() + 35
    );
    add(buttonText);

    // Click handler
    mouseClickMethod(function(e) {
        if (e.getX() >= startButton.getX() &&
            e.getX() <= startButton.getX() + startButton.getWidth() &&
            e.getY() >= startButton.getY() &&
            e.getY() <= startButton.getY() + startButton.getHeight()) {
            
            // Clear start screen
            remove(startImage);
            remove(startButton);
            remove(buttonText);

            // Remove this mouse handler
            mouseClickMethod(null);

            // Start the game
            began();
        }
    });
}

showStartScreen();
