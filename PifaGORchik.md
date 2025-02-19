import javax.imageio.ImageIO;
import java.io.File;
import java.io.IOException;
import java.awt.Image;
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class Main extends JFrame implements ActionListener, KeyListener {

    private final int WIDTH = 1000;
    private final int HEIGHT = 900;
    private final int PLAYER_SIZE = 25;
    private final int INITIAL_OBSTACLE_SPEED = 3;
    private int obstacleSpeed = INITIAL_OBSTACLE_SPEED;
    private final int PLAYER_SPEED = 12;
    private int playerX;
    private int playerY;
    private List<Rectangle> obstacles;
    private Random random;
    private Timer timer;
    private boolean gameOver;
    private int score;
    private int obstacleSpawnCounter; // Счётчик для генерации препятствий.
    private final int OBSTACLE_SPAWN_INTERVAL = 80; // Интервал в кадрах.
    private Image backgroundImage;
    private Color randomBackgroundColor;
    private boolean gameWon = false; //чтобы отслеживать, выиграл ли игрок

    public Main() {
        setTitle("pifaGORchik");
        setSize(WIDTH, HEIGHT);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);
        addKeyListener(this);
        setFocusable(true);
        setFocusTraversalKeysEnabled(false);

        playerX = WIDTH / 30;
        playerY = HEIGHT - 310;
        obstacles = new ArrayList<>();
        random = new Random();
        gameOver = false;
        score = 0;
        obstacleSpawnCounter = 0;

        timer = new Timer(1, this); // 50 FPS
        timer.start();

        generateObstacle();
        setVisible(true);
    }

    private void generateObstacle() {
        int size = 20 + random.nextInt(27); // Размеры от 20 до 50
        int x = WIDTH + random.nextInt(200); // за пределами экрана
        int y = random.nextInt(HEIGHT - size - 100);  // чтобы не на самом низу
        obstacles.add(new Rectangle(x, y, size, size));
    }

    private void moveObstacles() {
        for (int i = 0; i < obstacles.size(); i++) {
            Rectangle obstacle = obstacles.get(i);
            obstacle.x -= obstacleSpeed;
            if (obstacle.x < -obstacle.width) {
                obstacles.remove(i);
                score++; // Увеличиваем счёт за пройденное препятствие
                generateObstacle(); // Создаем новый obstacle
                if (score % 15 == 0) { // Увеличиваем сложность каждые 15 очков
                    obstacleSpeed++; // Увеличиваем скорость
                    System.out.println("Уровень сложности повышен! Скорость: " + obstacleSpeed);
                }
                // Проверяем условие победы
                if (score >= 400) {
                    gameWon = true;
                    timer.stop();
                    System.out.println("ПОБЕДА! Теперь PifaGORchik может сиять в нашей чайхоне");
                }
            }
        }
    }

    private void checkCollisions() {
        Rectangle playerRect = new Rectangle(playerX, playerY, PLAYER_SIZE, PLAYER_SIZE);
        for (Rectangle obstacle : obstacles) {
            if (playerRect.intersects(obstacle)) {
                gameOver = true;
                timer.stop();
                System.out.println("PifaGORchik не выиграл! Ваш счёт: " + score);
                break;
            }
        }
    }

    private void resetGame() {
        gameOver = false;
        score = 0;
        obstacleSpeed = INITIAL_OBSTACLE_SPEED;
        obstacles.clear();
        playerX = WIDTH / 30;
        playerY = HEIGHT - 310;

        Random random = new Random();
        randomBackgroundColor = new Color(random.nextInt(256), random.nextInt(256), random.nextInt(256));

        generateObstacle();
        timer.start();
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (!gameOver) {
            moveObstacles();
            checkCollisions();
            obstacleSpawnCounter++;

            // Создаём новое препятствие с заданным интервалом.
            if (obstacleSpawnCounter >= OBSTACLE_SPAWN_INTERVAL) {
                generateObstacle();
                obstacleSpawnCounter = 1;
            }
        }
        repaint();
    }

    @Override
    public void paint(Graphics g) {
        super.paint(g);
        Graphics2D g2d = (Graphics2D) g;

        if (backgroundImage != null) {
            int imageWidth = backgroundImage.getWidth(this);
            int imageHeight = backgroundImage.getHeight(this);
            for (int x = 0; x < WIDTH; x += imageWidth) {
                for (int y = 0; y < HEIGHT; y += imageHeight) {
                    g2d.drawImage(backgroundImage, x, y, this);
                }
            }
        } else {
            // Используем сохраненный случайный цвет
            if (randomBackgroundColor != null) {
                g2d.setColor(randomBackgroundColor);
            } else {
                g2d.setColor(Color.LIGHT_GRAY); // Если randomBackgroundColor == null (что не должно быть)
            }
            g2d.fillRect(0, 0, WIDTH, HEIGHT);
        }

        g2d.setColor(Color.BLUE);
        g2d.fillRect(playerX, playerY, PLAYER_SIZE, PLAYER_SIZE);

        g2d.setColor(Color.RED);
        for (Rectangle obstacle : obstacles) {
            g2d.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
        }

        g2d.setColor(Color.WHITE);
        g2d.setFont(new Font("Arial", Font.BOLD, 20));
        g2d.drawString("Счёт: " + score, 10, 30);

        if (gameOver) {
            g2d.setColor(Color.BLACK);
            g2d.setFont(new Font("Arial", Font.BOLD, 50));
            String gameOverText = "PifaGORchik проиграл! Счёт: " + score;
            int strWidth = g2d.getFontMetrics().stringWidth(gameOverText);
            g2d.drawString(gameOverText, WIDTH / 2 - strWidth / 2, HEIGHT / 2);

            g2d.setFont(new Font("Arial", Font.BOLD, 25));
            String restartText = "Нажмите 'R' для перезапуска";
            int restartWidth = g2d.getFontMetrics().stringWidth(restartText);
            g2d.drawString(restartText, WIDTH / 2 - restartWidth / 2, HEIGHT / 2 + 50);
        }
        // Отрисовываем сообщение о победе
        if (gameWon) {
            g2d.setColor(Color.GREEN);
            g2d.setFont(new Font("Arial", Font.BOLD, 20));
            String winText = "ПОБЕДА! Теперь PifaGORchik может сиять в нашей чайхоне";
            int strWidth = g2d.getFontMetrics().stringWidth(winText);
            g2d.drawString(winText, WIDTH / 2 - strWidth / 2, HEIGHT / 2);
        }
    }

    @Override
    public void keyPressed(KeyEvent e) {
        if (gameOver) {
            if (e.getKeyCode() == KeyEvent.VK_R) {
                resetGame();
            }
            return;
        }
        int key = e.getKeyCode();
        if (key == KeyEvent.VK_UP && playerX > 0) {
            playerY -= PLAYER_SPEED;
        }
        if (key == KeyEvent.VK_DOWN && playerX < WIDTH - PLAYER_SIZE) {
            playerY += PLAYER_SPEED;
        }
        if (key == KeyEvent.VK_RIGHT && playerX < WIDTH - PLAYER_SIZE) {
            playerX += PLAYER_SPEED;
        }
        if (key == KeyEvent.VK_LEFT && playerX < WIDTH - PLAYER_SIZE) {
            playerX -= PLAYER_SPEED;
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {
    }

    @Override
    public void keyReleased(KeyEvent e) {
    }

    public static void main(String[] args) {
        new Main();
    }
}

<!---
Kykymbir/Kykymbir is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
