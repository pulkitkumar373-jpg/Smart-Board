# Smart-Board
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.AffineTransform;
import java.awt.image.BufferedImage;
import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import javax.imageio.ImageIO;

/**
 * SARASWATI EDU-STUDIO - MASTER PRO EDITION
 * Features: Pen, Eraser, Highlighter, 3D Shapes, Shorts Mode, Move Tool,
 * Chemistry Mode, Math Mode, Grid, RTL Mode, Multi-Page Support
 * Developed for @SaraswatiEdu-Tech
 */
public class SaraswatiEduStudio extends JFrame {
    private BoardCanvas board;
    private boolean isRandomColor = false;
    private String activeMode = "PEN";
    private BufferedImage slideImage = null;
    private Color currentColor = Color.WHITE;
    private int fontSize = 26;
    private Object selectedElement = null;
    private boolean isShortsMode = false;
    private boolean isMathMode = false;
    private boolean showGrid = false;
    private boolean isChemistryMode = false;
    private boolean isRTLMode = false;

    // ===== MULTI-PAGE VARIABLES =====
    private List<PageData> pages = new ArrayList<>();
    private int currentPageIndex = 0;
    private JLabel pageLabel;

    private ToolButton btnPen, btnEraser, btnHighlighter, btnType, btnMove, btnShorts, btnShape, btnChem, btnMath, btnRTL;
    private String selectedShape = "RECTANGLE";

    public SaraswatiEduStudio() {
        setTitle("Saraswati Edu-Tech (@SaraswatiEdu-Tech) - Advanced Board");
        setExtendedState(JFrame.MAXIMIZED_BOTH);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        // Initialize first page
        pages.add(new PageData());

        board = new BoardCanvas();
        add(board, BorderLayout.CENTER);

        // --- BOTTOM TOOLBAR ---
        JPanel toolbar = new JPanel(new FlowLayout(FlowLayout.CENTER, 10, 10));
        toolbar.setOpaque(true);
        toolbar.setBackground(new Color(8, 10, 20));
        toolbar.setPreferredSize(new Dimension(0, 110));

        // ===== PAGE NAVIGATION =====
        JButton btnPrevPage = new JButton("◀");
        btnPrevPage.setFont(new Font("Arial", Font.BOLD, 14));
        btnPrevPage.setPreferredSize(new Dimension(45, 40));
        btnPrevPage.setBackground(new Color(50, 100, 150));
        btnPrevPage.setForeground(Color.WHITE);
        btnPrevPage.setBorder(BorderFactory.createEmptyBorder());
        btnPrevPage.setFocusPainted(false);
        btnPrevPage.setCursor(new Cursor(Cursor.HAND_CURSOR));
        btnPrevPage.addActionListener(e -> previousPage());
        toolbar.add(btnPrevPage);

        pageLabel = new JLabel("Page: 1 / 1");
        pageLabel.setFont(new Font("Arial", Font.BOLD, 12));
        pageLabel.setForeground(new Color(200, 200, 200));
        pageLabel.setPreferredSize(new Dimension(100, 40));
        toolbar.add(pageLabel);

        JButton btnNextPage = new JButton("▶");
        btnNextPage.setFont(new Font("Arial", Font.BOLD, 14));
        btnNextPage.setPreferredSize(new Dimension(45, 40));
        btnNextPage.setBackground(new Color(50, 100, 150));
        btnNextPage.setForeground(Color.WHITE);
        btnNextPage.setBorder(BorderFactory.createEmptyBorder());
        btnNextPage.setFocusPainted(false);
        btnNextPage.setCursor(new Cursor(Cursor.HAND_CURSOR));
        btnNextPage.addActionListener(e -> nextPage());
        toolbar.add(btnNextPage);

        JButton btnAddPage = new JButton("+");
        btnAddPage.setFont(new Font("Arial", Font.BOLD, 14));
        btnAddPage.setPreferredSize(new Dimension(45, 40));
        btnAddPage.setBackground(new Color(100, 150, 100));
        btnAddPage.setForeground(Color.WHITE);
        btnAddPage.setBorder(BorderFactory.createEmptyBorder());
        btnAddPage.setFocusPainted(false);
        btnAddPage.setCursor(new Cursor(Cursor.HAND_CURSOR));
        btnAddPage.addActionListener(e -> addNewPage());
        toolbar.add(btnAddPage);

        JButton btnDeletePage = new JButton("✘");
        btnDeletePage.setFont(new Font("Arial", Font.BOLD, 14));
        btnDeletePage.setPreferredSize(new Dimension(45, 40));
        btnDeletePage.setBackground(new Color(200, 100, 100));
        btnDeletePage.setForeground(Color.WHITE);
        btnDeletePage.setBorder(BorderFactory.createEmptyBorder());
        btnDeletePage.setFocusPainted(false);
        btnDeletePage.setCursor(new Cursor(Cursor.HAND_CURSOR));
        btnDeletePage.addActionListener(e -> deleteCurrentPage());
        toolbar.add(btnDeletePage);

        toolbar.add(new JSeparator(JSeparator.VERTICAL));

        // Creating Drawing Buttons
        btnPen = createToolBtn("✎", new Color(255, 255, 200), e -> {
            activeMode = "PEN";
            updateButtonStyles();
        });

        btnEraser = createToolBtn("🧹", new Color(200, 200, 200), e -> {
            activeMode = "ERASER";
            updateButtonStyles();
        });

        btnHighlighter = createToolBtn("🖍️", new Color(255, 255, 100), e -> {
            activeMode = "HIGHLIGHTER";
            updateButtonStyles();
        });

        btnType = createToolBtn("T", new Color(220, 200, 255), e -> {
            activeMode = "TEXT";
            updateButtonStyles();
        });

        btnMove = createToolBtn("✥", new Color(200, 255, 200), e -> {
            activeMode = "MOVE";
            updateButtonStyles();
        });

        btnShape = createToolBtn("📐", new Color(200, 230, 255), e -> {
            activeMode = "SHAPE";
            String[] options = { "Line", "Rectangle", "Circle", "Cube", "Cylinder", "Cone", "Arrow" };
            String choice = (String) JOptionPane.showInputDialog(this, "Select Shape:", "Saraswati Shapes",
                    JOptionPane.PLAIN_MESSAGE, null, options, options[0]);
            if (choice != null) {
                selectedShape = choice.toUpperCase();
                updateButtonStyles();
            }
        });

        btnShorts = createToolBtn("📱", new Color(255, 120, 255), e -> {
            isShortsMode = !isShortsMode;
            updateButtonStyles();
            board.repaint();
        });

        btnChem = createToolBtn("⚗️", new Color(100, 255, 150), e -> {
            isChemistryMode = !isChemistryMode;
            updateButtonStyles();
            board.repaint();
        });

        btnMath = createToolBtn("∑", new Color(255, 220, 120), e -> {
            isMathMode = !isMathMode;
            updateButtonStyles();
            board.repaint();
        });

        btnRTL = createToolBtn("א", new Color(255, 150, 200), e -> {
            isRTLMode = !isRTLMode;
            updateButtonStyles();
            board.repaint();
        });

        // Adding to Toolbar
        toolbar.add(wrapInPanel(btnPen, "Pen"));
        toolbar.add(wrapInPanel(btnEraser, "Eraser"));
        toolbar.add(wrapInPanel(btnHighlighter, "High"));
        toolbar.add(wrapInPanel(btnType, "Type"));
        toolbar.add(wrapInPanel(btnMove, "Move"));
        toolbar.add(wrapInPanel(btnShape, "Shapes"));
        toolbar.add(wrapInPanel(btnShorts, "Shorts"));
        toolbar.add(wrapInPanel(btnChem, "Chem"));
        toolbar.add(wrapInPanel(btnMath, "Math"));
        toolbar.add(wrapInPanel(btnRTL, "RTL"));

        toolbar.add(wrapInPanel(createToolBtn("🎨", new Color(255, 200, 200), e -> {
            Color c = JColorChooser.showDialog(this, "Color", currentColor);
            if (c != null) {
                currentColor = c;
                isRandomColor = false;
            }
        }), "Color"));

        toolbar.add(wrapInPanel(createToolBtn("⬛", new Color(180, 180, 180), e -> {
            Color c = JColorChooser.showDialog(this, "Board", board.getBackground());
            if (c != null) {
                board.setBackground(c);
                board.repaint();
            }
        }), "Board"));

        toolbar.add(wrapInPanel(createToolBtn("+", new Color(200, 230, 255), e -> fontSize += 4), "S+"));
        toolbar.add(wrapInPanel(createToolBtn("-", new Color(200, 230, 255), e -> {
            if (fontSize > 10)
                fontSize -= 4;
        }), "S-"));

        toolbar.add(wrapInPanel(createToolBtn("📽️", new Color(255, 230, 180), e -> {
            JFileChooser fc = new JFileChooser();
            if (fc.showOpenDialog(this) == JFileChooser.APPROVE_OPTION) {
                try {
                    slideImage = ImageIO.read(fc.getSelectedFile());
                    board.repaint();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }), "Slide"));

        toolbar.add(wrapInPanel(createToolBtn("🔴", new Color(255, 0, 0), e -> {
            try {
                Desktop.getDesktop().browse(new URI("https://www.youtube.com/upload"));
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }), "YT"));

        toolbar.add(wrapInPanel(createToolBtn("✨", new Color(255, 180, 255), e -> isRandomColor = !isRandomColor),
                "Magic"));

        toolbar.add(wrapInPanel(createToolBtn("🗑️", new Color(255, 100, 100), e -> {
            clearCurrentPage();
            slideImage = null;
            board.repaint();
        }), "Clear"));

        toolbar.add(wrapInPanel(createToolBtn("⊞", new Color(150, 150, 255), e -> {
            showGrid = !showGrid;
            board.repaint();
        }), "Grid"));

        add(toolbar, BorderLayout.SOUTH);
        updateButtonStyles();
        updatePageLabel();
        setVisible(true);
    }

    // ===== MULTI-PAGE METHODS =====
    private void addNewPage() {
        pages.add(new PageData());
        currentPageIndex = pages.size() - 1;
        updatePageLabel();
        board.repaint();
        JOptionPane.showMessageDialog(this, "New page added! Page " + (currentPageIndex + 1));
    }

    private void deleteCurrentPage() {
        if (pages.size() == 1) {
            JOptionPane.showMessageDialog(this, "Cannot delete the last page!");
            return;
        }
        
        pages.remove(currentPageIndex);
        if (currentPageIndex >= pages.size()) {
            currentPageIndex = pages.size() - 1;
        }
        updatePageLabel();
        board.repaint();
        JOptionPane.showMessageDialog(this, "Page deleted!");
    }

    private void previousPage() {
        if (currentPageIndex > 0) {
            currentPageIndex--;
            updatePageLabel();
            board.repaint();
        }
    }

    private void nextPage() {
        if (currentPageIndex < pages.size() - 1) {
            currentPageIndex++;
            updatePageLabel();
            board.repaint();
        }
    }

    private void updatePageLabel() {
        pageLabel.setText("Page: " + (currentPageIndex + 1) + " / " + pages.size());
    }

    private void clearCurrentPage() {
        pages.get(currentPageIndex).clear();
    }

    private PageData getCurrentPage() {
        return pages.get(currentPageIndex);
    }

    private void updateButtonStyles() {
        btnPen.setSelection(activeMode.equals("PEN"));
        btnEraser.setSelection(activeMode.equals("ERASER"));
        btnHighlighter.setSelection(activeMode.equals("HIGHLIGHTER"));
        btnType.setSelection(activeMode.equals("TEXT"));
        btnMove.setSelection(activeMode.equals("MOVE"));
        btnShape.setSelection(activeMode.equals("SHAPE"));
        btnShorts.setSelection(isShortsMode);
        btnChem.setSelection(isChemistryMode);
        btnMath.setSelection(isMathMode);
        btnRTL.setSelection(isRTLMode);
    }

    private ToolButton createToolBtn(String icon, Color bg, ActionListener action) {
        ToolButton button = new ToolButton(icon);

        button.setPreferredSize(new Dimension(40, 40));
        button.setMinimumSize(new Dimension(40, 40));
        button.setMaximumSize(new Dimension(40, 40));

        button.setFont(new Font("Segoe UI Symbol", Font.PLAIN, 20));
        button.setForeground(bg);
        button.setBackground(bg);

        button.setOpaque(false);
        button.setContentAreaFilled(false);
        button.setBorderPainted(false);
        button.setFocusPainted(false);

        button.setCursor(new Cursor(Cursor.HAND_CURSOR));
        button.addActionListener(action);

        return button;
    }

    private JPanel wrapInPanel(JButton b, String label) {
        JPanel p = new JPanel(new BorderLayout(0, 3));
        p.setOpaque(false);
        p.setPreferredSize(new Dimension(58, 70));

        p.add(b, BorderLayout.CENTER);

        JLabel lbl = new JLabel(label, SwingConstants.CENTER);
        lbl.setFont(new Font("Segoe UI", Font.BOLD, 9));
        lbl.setForeground(new Color(230, 230, 230));

        p.add(lbl, BorderLayout.SOUTH);

        return p;
    }

    // ========== MATH FORMATTING ==========
    private String toMathFormat(String text) {
        StringBuilder result = new StringBuilder();
        
        for (int i = 0; i < text.length(); i++) {
            char current = text.charAt(i);
            
            if (Character.isDigit(current)) {
                if (i > 0 && Character.isDigit(text.charAt(i - 1))) {
                    result.append(digitToSuperscript(current));
                } else {
                    result.append(current);
                }
            } else {
                result.append(current);
            }
        }
        
        String formatted = result.toString();
        formatted = formatted.replace("pi", "π");
        formatted = formatted.replace("alpha", "α");
        formatted = formatted.replace("beta", "β");
        formatted = formatted.replace("gamma", "γ");
        formatted = formatted.replace("delta", "δ");
        formatted = formatted.replace("epsilon", "ε");
        formatted = formatted.replace("lambda", "λ");
        formatted = formatted.replace("theta", "θ");
        
        return formatted;
    }
    
    private char digitToSuperscript(char digit) {
        switch (digit) {
            case '0': return '⁰';
            case '1': return '¹';
            case '2': return '²';
            case '3': return '³';
            case '4': return '⁴';
            case '5': return '⁵';
            case '6': return '⁶';
            case '7': return '⁷';
            case '8': return '⁸';
            case '9': return '⁹';
            default: return digit;
        }
    }

    private String toChemicalFormat(String text) {
        return text
            .replace("0", "₀")
            .replace("1", "₁")
            .replace("2", "₂")
            .replace("3", "₃")
            .replace("4", "₄")
            .replace("5", "₅")
            .replace("6", "₆")
            .replace("7", "₇")
            .replace("8", "₈")
            .replace("9", "₉");
    }

    // ================= ToolButton Class =================
    class ToolButton extends JButton {
        private boolean selected = false;
        private boolean hover = false;

        public ToolButton(String text) {
            super(text);

            setOpaque(false);
            setContentAreaFilled(false);
            setBorderPainted(false);
            setFocusPainted(false);
            setHorizontalAlignment(SwingConstants.CENTER);
            setVerticalAlignment(SwingConstants.CENTER);
            setCursor(new Cursor(Cursor.HAND_CURSOR));

            addMouseListener(new MouseAdapter() {
                @Override
                public void mouseEntered(MouseEvent e) {
                    hover = true;
                    repaint();
                }

                @Override
                public void mouseExited(MouseEvent e) {
                    hover = false;
                    repaint();
                }
            });
        }

        public void setSelection(boolean value) {
            selected = value;
            repaint();
        }

        @Override
        protected void paintComponent(Graphics g) {
            Graphics2D g2 = (Graphics2D) g.create();

            g2.setRenderingHint(
                    RenderingHints.KEY_ANTIALIASING,
                    RenderingHints.VALUE_ANTIALIAS_ON);

            int w = getWidth();
            int h = getHeight();

            Color originalColor = getForeground();

            if (selected) {
                g2.setColor(new Color(255, 255, 255, 230));
            } else if (hover) {
                g2.setColor(new Color(255, 255, 255, 35));
            }

            if (selected || hover) {
                g2.fillRoundRect(0, 0, w - 1, h - 1, 12, 12);
            }

            if (selected) {
                g2.setColor(new Color(0, 200, 255));
                g2.setStroke(new BasicStroke(2f));
                g2.drawRoundRect(1, 1, w - 3, h - 3, 12, 12);
            }

            g2.dispose();

            setForeground(originalColor);
            super.paintComponent(g);
        }
    }

    // ====================== BoardCanvas CLASS ======================
    class BoardCanvas extends JPanel {

        public BoardCanvas() {
            setBackground(new Color(15, 15, 20));
            setFocusable(true);

            addMouseListener(new MouseAdapter() {
                @Override
                public void mousePressed(MouseEvent e) {
                    requestFocusInWindow();
                    PageData currentPage = getCurrentPage();
                    currentPage.lastMouse = e.getPoint();

                    if (activeMode.equals("MOVE")) {
                        selectedElement = currentPage.findElement(e.getPoint());

                    } else if (activeMode.equals("TEXT")) {
                        Color c = isRandomColor
                                ? new Color(
                                        new Random().nextInt(256),
                                        new Random().nextInt(256),
                                        new Random().nextInt(256))
                                : currentColor;

                        currentPage.texts.add(new TextData(e.getPoint(), c, fontSize));

                    } else if (activeMode.equals("SHAPE")) {
                        Color c = isRandomColor
                                ? new Color(
                                        new Random().nextInt(256),
                                        new Random().nextInt(256),
                                        new Random().nextInt(256))
                                : currentColor;

                        currentPage.shapes.add(new ShapeData(selectedShape, e.getPoint(), c));

                    } else if (activeMode.equals("PEN")
                            || activeMode.equals("HIGHLIGHTER")) {

                        Color c = activeMode.equals("HIGHLIGHTER")
                                ? new Color(
                                        currentColor.getRed(),
                                        currentColor.getGreen(),
                                        currentColor.getBlue(),
                                        100)
                                : currentColor;

                        int thick = activeMode.equals("HIGHLIGHTER") ? 15 : 3;

                        currentPage.allLines.add(new LineData(c, thick));
                        currentPage.allLines.get(currentPage.allLines.size() - 1).pts.add(e.getPoint());

                    } else if (activeMode.equals("ERASER")) {
                        currentPage.removeElementAt(e.getPoint());
                    }

                    repaint();
                }

                @Override
                public void mouseReleased(MouseEvent e) {
                    selectedElement = null;
                }
            });

            addMouseMotionListener(new MouseMotionAdapter() {
                @Override
                public void mouseDragged(MouseEvent e) {
                    PageData currentPage = getCurrentPage();

                    if ((activeMode.equals("PEN")
                            || activeMode.equals("HIGHLIGHTER"))
                            && !currentPage.allLines.isEmpty()) {

                        currentPage.allLines.get(currentPage.allLines.size() - 1).pts.add(e.getPoint());

                    } else if (activeMode.equals("SHAPE")
                            && !currentPage.shapes.isEmpty()) {

                        currentPage.shapes.get(currentPage.shapes.size() - 1).end = e.getPoint();

                    } else if (activeMode.equals("ERASER")) {

                        currentPage.removeElementAt(e.getPoint());

                    } else if (activeMode.equals("MOVE")
                            && selectedElement != null) {

                        int dx = e.getX() - currentPage.lastMouse.x;
                        int dy = e.getY() - currentPage.lastMouse.y;

                        if (selectedElement instanceof TextData) {
                            ((TextData) selectedElement).p.translate(dx, dy);

                        } else if (selectedElement instanceof LineData) {
                            for (Point p : ((LineData) selectedElement).pts) {
                                p.translate(dx, dy);
                            }

                        } else if (selectedElement instanceof ShapeData) {
                            ((ShapeData) selectedElement).start.translate(dx, dy);
                            ((ShapeData) selectedElement).end.translate(dx, dy);
                        }

                        currentPage.lastMouse = e.getPoint();
                    }

                    repaint();
                }
            });

            addKeyListener(new KeyAdapter() {
                @Override
                public void keyTyped(KeyEvent e) {
                    if (activeMode.equals("TEXT")) {
                        PageData currentPage = getCurrentPage();
                        if (!currentPage.texts.isEmpty()) {
                            TextData last = currentPage.texts.get(currentPage.texts.size() - 1);

                            if (e.getKeyChar() == '\b') {
                                if (last.str.length() > 0) {
                                    last.str.deleteCharAt(last.str.length() - 1);
                                }
                            } else if (e.getKeyChar() == '\n') {
                                last.str.append("\n");
                            } else {
                                last.str.append(e.getKeyChar());
                            }

                            repaint();
                        }
                    }
                }
            });
        }

        @Override
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);

            Graphics2D g2 = (Graphics2D) g;
            g2.setRenderingHint(
                    RenderingHints.KEY_ANTIALIASING,
                    RenderingHints.VALUE_ANTIALIAS_ON);

            PageData currentPage = getCurrentPage();

            // Draw Grid if enabled
            if (showGrid) {
                drawGrid(g2);
            }

            // Shorts Mode Border
            int shortsWidth = getHeight() * 9 / 16;
            int startX = (getWidth() - shortsWidth) / 2;

            if (isShortsMode) {
                g2.setColor(new Color(255, 50, 150, 60));
                g2.setStroke(new BasicStroke(
                        3,
                        BasicStroke.CAP_BUTT,
                        BasicStroke.JOIN_MITER,
                        10,
                        new float[]{15, 10},
                        0));
                g2.drawRect(startX, 0, shortsWidth, getHeight());
            }

            // Slide Image
            if (slideImage != null) {
                g2.drawImage(
                        slideImage,
                        50,
                        20,
                        getWidth() - 100,
                        getHeight() - 150,
                        null);
            }

            // Branding
            int logoX = isShortsMode ? startX + 20 : 20;

            g2.setColor(new Color(255, 255, 255, 40));
            g2.fillRoundRect(logoX, 20, 230, 45, 12, 12);

            g2.setColor(Color.WHITE);
            g2.setFont(new Font("Arial", Font.BOLD, 16));
            g2.drawString("SARASWATI EDU-TECH", logoX + 15, 48);

            // Draw Shapes
            for (ShapeData sd : currentPage.shapes) {
                drawShape(g2, sd);
            }

            // Draw Lines
            for (LineData ld : currentPage.allLines) {
                g2.setColor(ld.c);
                g2.setStroke(new BasicStroke(
                        ld.thickness,
                        BasicStroke.CAP_ROUND,
                        BasicStroke.JOIN_ROUND));

                for (int i = 0; i < ld.pts.size() - 1; i++) {
                    g2.drawLine(
                            ld.pts.get(i).x,
                            ld.pts.get(i).y,
                            ld.pts.get(i + 1).x,
                            ld.pts.get(i + 1).y);
                }
            }

            // Draw Text
            for (TextData td : currentPage.texts) {
                drawText(g2, td);
            }
        }

        private void drawGrid(Graphics2D g2) {
            g2.setColor(new Color(100, 100, 120, 50));
            g2.setStroke(new BasicStroke(1));

            int gridSize = 50;
            for (int x = 0; x < getWidth(); x += gridSize) {
                g2.drawLine(x, 0, x, getHeight());
            }
            for (int y = 0; y < getHeight(); y += gridSize) {
                g2.drawLine(0, y, getWidth(), y);
            }
        }

        private void drawShape(Graphics2D g2, ShapeData sd) {
            g2.setColor(sd.c);
            g2.setStroke(new BasicStroke(3));

            int x1 = sd.start.x;
            int y1 = sd.start.y;
            int x2 = sd.end.x;
            int y2 = sd.end.y;

            int x = Math.min(x1, x2);
            int y = Math.min(y1, y2);
            int w = Math.abs(x1 - x2);
            int h = Math.abs(y1 - y2);

            switch (sd.type) {
                case "LINE":
                    g2.drawLine(x1, y1, x2, y2);
                    break;
                case "RECTANGLE":
                    g2.drawRect(x, y, w, h);
                    break;
                case "CIRCLE":
                    g2.drawOval(x, y, w, h);
                    break;
                case "ARROW":
                    drawArrow(g2, x1, y1, x2, y2);
                    break;
                case "CUBE":
                    drawCube(g2, x, y, w, h);
                    break;
                case "CYLINDER":
                    drawCylinder(g2, x, y, w, h);
                    break;
                case "CONE":
                    drawCone(g2, x, y, w, h);
                    break;
            }
        }

        private void drawArrow(Graphics2D g2, int x1, int y1, int x2, int y2) {
            g2.drawLine(x1, y1, x2, y2);

            double angle = Math.atan2(y2 - y1, x2 - x1);

            g2.drawLine(
                    x2, y2,
                    (int)(x2 - 15 * Math.cos(angle - 0.4)),
                    (int)(y2 - 15 * Math.sin(angle - 0.4)));

            g2.drawLine(
                    x2, y2,
                    (int)(x2 - 15 * Math.cos(angle + 0.4)),
                    (int)(y2 - 15 * Math.sin(angle + 0.4)));
        }

        private void drawCube(Graphics2D g2, int x, int y, int w, int h) {
            int offset = w / 3;

            g2.drawRect(x, y, w - offset, h - offset);
            g2.drawRect(x + offset, y + offset, w - offset, h - offset);

            g2.drawLine(x, y, x + offset, y + offset);
            g2.drawLine(x + w - offset, y, x + w, y + offset);
            g2.drawLine(x, y + h - offset, x + offset, y + h);
            g2.drawLine(x + w - offset, y + h - offset, x + w, y + h);
        }

        private void drawCylinder(Graphics2D g2, int x, int y, int w, int h) {
            int eh = h / 4;

            g2.drawOval(x, y, w, eh);
            g2.drawOval(x, y + h - eh, w, eh);

            g2.drawLine(x, y + eh / 2, x, y + h - eh / 2);
            g2.drawLine(x + w, y + eh / 2, x + w, y + h - eh / 2);
        }

        private void drawCone(Graphics2D g2, int x, int y, int w, int h) {
            int eh = h / 4;

            g2.drawOval(x, y + h - eh, w, eh);
            g2.drawLine(x + w / 2, y, x, y + h - eh / 2);
            g2.drawLine(x + w / 2, y, x + w, y + h - eh / 2);
        }

        private void drawText(Graphics2D g2, TextData td) {
            int y = td.p.y;

            for (String line : td.str.toString().split("\\n")) {
                String displayText = line;
                Color textColor = td.c;

                // Chemistry Mode: H2O -> H₂O
                if (isChemistryMode) {
                    displayText = toChemicalFormat(displayText);
                }

                // Math Mode: Apply formatting (no blue highlight)
                if (isMathMode) {
                    displayText = toMathFormat(displayText);
                }

                // Set font
                g2.setFont(new Font("Arial", Font.BOLD, td.size));
                g2.setColor(textColor);

                // RTL Mode: Draw from right to left
                if (isRTLMode) {
                    FontMetrics fm = g2.getFontMetrics();
                    int textWidth = fm.stringWidth(displayText);
                    
                    // Draw text from right side
                    AffineTransform original = g2.getTransform();
                    g2.scale(-1, 1);
                    g2.drawString(displayText, -td.p.x - textWidth, y);
                    g2.setTransform(original);
                } else {
                    // Normal LTR mode
                    g2.drawString(displayText, td.p.x, y);
                }

                y += td.size + 5;
            }
        }
    }

    // ====================== PAGE DATA CLASS ======================
    class PageData {
        List<LineData> allLines = new ArrayList<>();
        List<TextData> texts = new ArrayList<>();
        List<ShapeData> shapes = new ArrayList<>();
        Point lastMouse;

        public void clear() {
            allLines.clear();
            texts.clear();
            shapes.clear();
        }

        private void removeElementAt(Point p) {
            allLines.removeIf(line -> line.pts.stream().anyMatch(pt -> pt.distance(p) < 25));
            texts.removeIf(text -> p.distance(text.p) < 40);
            shapes.removeIf(shape -> p.distance(shape.start) < 50
                    || p.distance(shape.end) < 50);
        }

        private Object findElement(Point p) {
            for (TextData td : texts) {
                if (p.distance(td.p) < 60)
                    return td;
            }

            for (ShapeData sd : shapes) {
                if (p.distance(sd.start) < 60)
                    return sd;
            }

            for (LineData ld : allLines) {
                for (Point pt : ld.pts) {
                    if (p.distance(pt) < 30)
                        return ld;
                }
            }

            return null;
        }
    }

    // ====================== LineData ======================
    class LineData {
        List<Point> pts = new ArrayList<>();
        Color c;
        int thickness;

        LineData(Color color, int t) {
            c = color;
            thickness = t;
        }
    }

    // ====================== TextData ======================
    class TextData {
        Point p;
        StringBuilder str = new StringBuilder();
        Color c;
        int size;

        TextData(Point p, Color c, int s) {
            this.p = new Point(p);
            this.c = c;
            this.size = s;
        }
    }

    // ====================== ShapeData ======================
    class ShapeData {
        String type;
        Point start, end;
        Color c;

        ShapeData(String t, Point p, Color color) {
            type = t;
            start = new Point(p);
            end = new Point(p);
            c = color;
        }
    }

    // ====================== Main Method ======================
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new SaraswatiEduStudio().setVisible(true);
        });
    }
}
