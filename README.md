# Tetris
Developed a tetris game under a week, game has some bugs will fix in the near future

### Link to download files, click on GIF
[![tetrisGIF](https://github.com/user-attachments/assets/50fbc426-ce63-4473-b75e-fe73d88e2511)](https://github.com/maxwelllokshin1/javaProjects/blob/main/Jar%20File%20games/tetris.zip)

**JAVA**

## Code files written here:

### Class window
```sh
package tetrisMain;

import java.awt.Dimension;

import javax.swing.JFrame;

public class window {
	public static JFrame window;
	public static int scale = 30;
	public static int width = scale*10, height = scale * 20;
	
	
	public static void main(String[] args)
	{
		window = new JFrame("Tetris");
		window.setMinimumSize(new Dimension(width, height));
		window.setResizable(false);
		window.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		
		tetris t = new tetris();
		window.add(t);
		window.pack();
		
		window.setLayout(null);
		window.setLocationRelativeTo(null);
		window.setVisible(true);
	}
}

```

### Class KeyHandler

```sh
package tetrisMain;

import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;

public class KeyHandler implements KeyListener{
	tetris t;
	
	public KeyHandler(tetris t)
	{
		this.t = t;
	}
	
	@Override
	public void keyPressed(KeyEvent e) { 
		int keyPressed = e.getKeyCode();
		
		if(t.gameOver)
		{
			if(keyPressed == KeyEvent.VK_ENTER ) {t.resetGame();}
		}else {
			if(keyPressed == KeyEvent.VK_UP) { t.upPressed = true;}
			if(keyPressed == KeyEvent.VK_LEFT) { t.leftPressed = true;}
			if(keyPressed == KeyEvent.VK_RIGHT) { t.rightPressed = true;}
			if(keyPressed == KeyEvent.VK_DOWN) { t.downPressed = true;}
			if(keyPressed == KeyEvent.VK_SPACE) { t.spacePressed = true;}
		}
	}
	@Override
	public void keyReleased(KeyEvent e) { 
		t.upPressed= false;
		t.leftPressed= false;
		t.rightPressed= false;
		t.downPressed= false;
		t.spacePressed = false;
	}
	
	@Override
	public void keyTyped(KeyEvent e) {
		
	}

	
}

```

### Class tetris

```sh
package tetrisMain;

import java.awt.AlphaComposite;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Graphics;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Iterator;
import java.util.LinkedList;

import javax.swing.JPanel;
import javax.swing.Timer;
import tetrisMain.tetris.Node;

public class tetris extends JPanel implements ActionListener {
	public int screenWidth = window.width, screenHeight = window.height;

	public boolean upPressed, leftPressed, downPressed, rightPressed, spacePressed;

	KeyHandler keyH;

	int nx = window.scale * 5, ny = window.scale;
	int pWidth = window.scale, pHeight = window.scale;

	int gravityY = window.scale;
	int gravityX = window.scale;

	int FPS = 60;

	public Timer gameLoop;

	LinkedList<Block> blocks;

	Iterator<Block> itr;
	Block cur;
	
	int fallingSpeed = 20;
	int fallCounter = 0;
	int moveCounter = 0;
	Block.Type currentType;
	
	boolean gameOver = false;
	
	int level = 1;
	int rowsCleared = 0;

	public tetris() {
		setPreferredSize(new Dimension(screenWidth, screenHeight));
		setBackground(new Color(0, 0, 0));
		setFocusable(true);
		keyH = new KeyHandler(this);
		addKeyListener(keyH);

		blocks = new LinkedList<>();
		generateBlock();
		cur = blocks.getLast();

		startGameThread();
	}

	public void generateBlock() {
		int randBlock = (int) (Math.random() * 7);
		switch (randBlock) {
		case 0: currentType = Block.Type.O; break;
		case 1: currentType = Block.Type.I; break;
		case 2: currentType = Block.Type.S; break;
		case 3: currentType = Block.Type.Z; break;
		case 4: currentType = Block.Type.L; break;
		case 5: currentType = Block.Type.J; break;
		case 6: currentType = Block.Type.T; break;
		}
		blocks.add(new Block(nx, ny, currentType)); // Add a new block
	}

	@Override
	public void paintComponent(Graphics g) {

		super.paintComponent(g);
		drawGameScreen(g);
		

	}

	public void drawGameScreen(Graphics g) {
		for (int i = 0; i < 10; i++) {
			for (int j = 0; j < 20; j++) {
				g.setColor(Color.DARK_GRAY);
				g.fillRect(i * window.scale, j * window.scale, window.scale - 1, window.scale - 1);
			}
		}
		
		for (Block b : blocks) {
			for (Node n : b.nodes) {
				g.setColor(b.color);
				g.fillRect(n.x, n.y, window.scale - 1, window.scale - 1);
			}
		}
		
		drawScore(g);
		
		if(gameOver) drawGameOverScreen(g);
		
	}
	
	public void drawGameOverScreen(Graphics g)
	{
		g.setColor(new Color(255,255,255,255));
		g.setFont(getFont().deriveFont(25F));
		g.drawString("SCORE: " + level, getXforCenteredText("SCORE: " + level, g), screenHeight/2);
		g.drawString("[PRESS ENTER]", getXforCenteredText("[PRESS ENTER]", g), screenHeight/2 + 50);
		
		gameLoop.stop();
	}
	
	public void drawScore(Graphics g)
	{
		g.setColor(new Color(255,255,255,255));
		g.setFont(getFont().deriveFont(20F));
		g.drawString("Rows Cleared: " + rowsCleared, window.scale/2, window.scale+(window.scale/2));
		g.drawString("Level: " + level, window.scale/2, window.scale);
	}
	
	public int getXforCenteredText(String text, Graphics g) {
        int length = (int) g.getFontMetrics().getStringBounds(text, g).getWidth();
        return screenWidth / 2 - length / 2; // Center the text horizontally
    }
	
	public void startGameThread()
	{
		gameLoop = new Timer(1000 / FPS, this);
		gameLoop.start();
	}
	
	public void move() {
	    fallCounter++;
	    moveCounter++;
	    if (moveCounter % 3 == 0) {
	        if (leftPressed) { cur.move(-gravityX, 0); }
	        if (rightPressed) { cur.move(gravityX, 0); }
	    }
	    if (moveCounter % 3 == 0) {
	        if (upPressed) { cur.rotate(); }
	        if (spacePressed) {
	            while (canMoveDown(cur)) { cur.move(0, gravityY); }
	        }
	    }
	    
	    
	    if (fallCounter >= fallingSpeed) {
	    	if(canMoveDown(cur)) cur.move(0, gravityY);
	        fallCounter = 0;
	    }
	    
	    if (!canMoveDown(cur)) {
	    	for (Node n : cur.nodes) {
		        if (n.y <= 0) gameOver = true; 
		    }
	    	
	    	if(!gameOver)
	    	{
	    		blocks.add(cur); // Add the current block to the blocks list
	            generateBlock(); // Create a new block
	            cur = blocks.getLast(); // Get the new current block
	            clearFilledRows(); // Check and clear filled row
	    	}
        }

	    // if (downPressed) { cur.move(0, gravityY); }
	    checkEdges(0, 0, screenWidth, screenHeight);
	}

	public boolean canMoveDown(Block cur) {
	    // Check if the block's nodes are touching the bottom of the screen
	    for (Node n : cur.nodes) {
	        if (n.y >= screenHeight - window.scale) return false; 
	    }

	    // Check for collisions with other blocks (excluding itself)
	    for (Block b : blocks) {
	        if (b != cur) {  // Don't check against the current block itself
	            for (Node nb : b.nodes) {
	                if (collision(cur, nb)) return false;
	            }
	        }
	    }

	    return true; // No collision, can move down
	}

	public boolean collision(Block cur, Node nodeLayer) {
	    // Check for overlap between the current block's nodes and another block's node
	    for (Node n : cur.nodes) {
	        if (n.x < nodeLayer.x + nodeLayer.width && n.x + n.width > nodeLayer.x
	           && n.y <= nodeLayer.y + nodeLayer.height && n.y + n.height >= nodeLayer.y) {
	            return true;  // Collision detected
	        }
	    }
	    return false;  // No collision
	}



	public void clearFilledRows() {
		// Clear full rows and shift down the rest
        for (int row = 0; row < 20; row++) {
            if (isRowFilled(row)) {
            	System.out.println("ROW CLEARED: " + (20 - row));
                removeRow(row);
                shiftRowsDown(row);
                rowsCleared+=1;
                if(rowsCleared % 10 == 0) increaseSpeed(); // Increase speed after clearing a row
            }
        }
	}


	public boolean isRowFilled(int row) {
        for (int i = 0; i < 10; i++) {
            boolean filled = false;
            for (Block b : blocks) {
                for (Node n : b.nodes) {
                    if (n.y == row * window.scale && n.x == i * window.scale) {
                        filled = true;
                        break;
                    }
                }
                if(filled) break;
            }
            if (!filled) return false;
        }
        return true;
    }

    public void removeRow(int row) {
        //blocks.removeIf(block -> block.nodes.stream().anyMatch(node -> node.y == row * window.scale));
			for(Block b : blocks)
			{
				Iterator<Node> nItr = b.nodes.iterator();
				while(nItr.hasNext())
				{
					Node n = nItr.next();
					if(n.y == row*window.scale) nItr.remove();
				}
			}
    }

    public void shiftRowsDown(int row) {
        for (Block b : blocks) {
            for (Node n : b.nodes) {
                if (n.y < row * window.scale) n.y += window.scale;
                //if (n.y == row * window.scale) n.y = screenHeight; 
                //else n.y -= window.scale;
            }
        }
    }

    public void increaseSpeed() {
    	level+=1;
        if (fallingSpeed > 5) fallingSpeed-=3;
    }



	public void checkEdges(int minEdgeX, int minEdgeY, int maxEdgeX, int maxEdgeY) {
	    int minX = Integer.MAX_VALUE;
	    int maxX = Integer.MIN_VALUE;
	    int minY = Integer.MAX_VALUE;
	    int maxY = Integer.MIN_VALUE;

	    for (Node n : cur.nodes) {
	        if (n.x < minX) minX = n.x;
	        if (n.x > maxX) maxX = n.x;
	        if (n.y < minY) minY = n.y;
	        if (n.y > maxY) maxY = n.y;
	    }

	    // Check boundaries and adjust block position
	    if (minX < minEdgeX) cur.move(minEdgeX - minX, 0);
	    if (maxX > maxEdgeX - window.scale) cur.move(maxEdgeX - window.scale - maxX, 0);
	    if (maxY > maxEdgeY - window.scale) cur.move(0, maxEdgeY - window.scale - maxY);
	}
	
	public void resetGame() {
        gameLoop.stop();

        // Clear all rocks and reset score, player position, etc.
        blocks.clear();
        generateBlock();
		cur = blocks.getLast();
        level = 1;
        rowsCleared = 0;
        gameOver = false;
        
        upPressed = false;
        downPressed = false;
        leftPressed = false;
        rightPressed = false;
//        dy = 0;
//        dx = 0;

//        r = null;
        // Restart timers
        gameLoop.start();
    }

	
	public class Node {
		int x, y;
		int width = pWidth, height = pHeight;

		public Node(int nx, int ny) {
			x = nx;
			y = ny;
		}
	}

	public class Block {
		enum Type {
			O, I, S, Z, L, J, T
		}

		LinkedList<Node> nodes;
		Color color;

		public Block(int nx, int ny, Type type) {
			nodes = new LinkedList<>();
			switch (type) {
			case I:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx + (pWidth * 2), ny));
				nodes.add(new Node(nx - pWidth, ny));
				color = Color.CYAN;
				break;
			case J:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx - pWidth, ny));
				nodes.add(new Node(nx - pWidth, ny - pHeight));
				color = Color.BLUE;
				break;
			case L:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx - pWidth, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx + pWidth, ny - pHeight));
				color = Color.ORANGE;
				break;
			case O:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx, ny + pHeight));
				nodes.add(new Node(nx + pWidth, ny + pHeight));
				color = Color.YELLOW;
				break;
			case S:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx - pWidth, ny));
				nodes.add(new Node(nx, ny - pHeight));
				nodes.add(new Node(nx + pWidth, ny - pHeight));
				color = Color.GREEN;
				break;
			case T:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx - pWidth, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx, ny - pHeight));
				color = Color.MAGENTA;
				break;
			case Z:
				nodes.add(new Node(nx, ny));
				nodes.add(new Node(nx + pWidth, ny));
				nodes.add(new Node(nx, ny - pHeight));
				nodes.add(new Node(nx - pWidth, ny - pHeight));
				color = Color.RED;
				break;
			}
		}

		public void move(int dx, int dy) {
			for (Node n : nodes) {
				n.x += dx;
				n.y += dy;
			}
		}
		public void rotate() {
		    if (currentType == Type.O) return; // No rotation for "O" shape

		    int pivotX = nodes.get(0).x;
		    int pivotY = nodes.get(0).y;

		    // Save the new positions temporarily
		    LinkedList<Node> newNodes = new LinkedList<>();
		    for (Node n : nodes) {
		        int dx = n.x - pivotX;
		        int dy = n.y - pivotY;
		        newNodes.add(new Node(pivotX - dy, pivotY + dx)); // Apply rotation formula
		    }

		    // Check if rotated position collides with other blocks
		    Block tempBlock = new Block(cur.nodes.get(0).x, cur.nodes.get(0).y, currentType); // Create a temporary block
		    tempBlock.nodes = newNodes; // Set the temporary rotated nodes

		    boolean collisionDetected = false;
		    for (Block b : blocks) {
		        if (b != cur) { // Avoid checking against the current block
		            for (Node nb : b.nodes) {
		                if (collision(tempBlock, nb)) {
		                    collisionDetected = true; // Collision detected
		                    break;
		                }
		            }
		        }
		        if (collisionDetected) break;
		    }

		    // If no collision, update current block’s nodes with the rotated ones
		    if (!collisionDetected) {
		        cur.nodes = newNodes;
		    }
		}


	}

	@Override
	public void actionPerformed(ActionEvent e) {
		move();
		repaint();

	}
}

```
