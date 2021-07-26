---
layout: post
title:  "[Java] First Posting"
subtitle:   "Java 첫 번째 포스팅"
date: 2021-07-26 22:17:14 +0900
categories: study
tags: java
comments: true
---

# [JAVA] Slinding Puzzle
2학년 1학기 자바프로그래밍의 기말 프로젝트로 Slinding Puzzle을 GUI로 구현하기가 주어졌다. 처음에는 막막했지만 결과적으로 열심히 공부하여 만들어보았다.

# [Model]
package java_finalterm01_sliding_puzzle_GUI;

    import javax.swing.*;
    import java.awt.*;
    import java.awt.image.BufferedImage;

    public class Model extends JFrame implements Runnable{

    boolean reverse = false;
    boolean modechange_lock = true;
    boolean movepiece_lock = true;
    int mode = 1;
    int[][] puzzle = new int[mode][mode];
    int cnt = 0;
    int[] register = new int[10000];

    int smoothmove = 0;
    int piecesize = 480 / mode;
    int centerlocation_x = (480 - piecesize) / 2;
    int centerlocation_y = (480 - piecesize) / 2;
    Container c = getContentPane();
    JPanel panel_puzzle01 = new JPanel();
    JPanel panel_puzzle01_background = new JPanel();
    JPanel panel_puzzle02 = new JPanel();
    JPanel succese_alert01 = new JPanel();
    JPanel succese_alert02 = new JPanel();

    ImageIcon image = new ImageIcon("images/pupple.jpg");
    Image img = image.getImage();

    JLabel title = new JLabel("Sliding Puzzle");
    JLabel modelabel = new JLabel("모드");
    JLabel count = new JLabel("시도 횟수 : " + cnt);
    JLabel label_picture = new JLabel(image);
    JLabel pushedlabel = new JLabel();
    JLabel[][] piece = new JLabel[mode][mode];
    JLabel blank = new JLabel();
    JLabel last_piece = new JLabel();
    JLabel tmp = new JLabel();
    JLabel succeselabel = new JLabel();

    JButton[] choosemode = new JButton[3];
    JButton shuffle = new JButton("Shuffle");
    JButton reversebutton = new JButton("되돌리기");
    JButton succese_check = new JButton("확인");

    public Model() {
        for (int i = 0; i < 3; i++) {
            choosemode[i] = new JButton((i + 3) + "X" + (i + 3) + " 모드");
        }
    }

    public void pic_Crop() {
        BufferedImage piecepicture = new BufferedImage(piecesize, piecesize, BufferedImage.TYPE_INT_RGB);
        Graphics g = piecepicture.getGraphics();
        g.drawImage(img, 53, 53, 375, 375, this);
        label_picture = new JLabel(new ImageIcon(piecepicture));
    }

    public void modeConvert(int mode) {
        this.mode = mode;
        puzzle = new int[mode][mode];
        piece = new JLabel[mode][mode];
        panel_puzzle02.setLayout(new GridLayout(mode, mode));
        piecesize = 480 / mode;
        centerlocation_x = (480 - piecesize) / 2;
        centerlocation_y = (480 - piecesize) / 2;
    }

    public void setPuzzleSequence() {
        for (int i = 0; i < mode; i++) {
            for (int j = 0; j < mode; j++) {
                if (i == mode - 1 && j == mode - 1) {
                    puzzle[i][j] = 0;
                    piece[i][j] = new JLabel();
                    piece[i][j].setBounds(piecesize * j, piecesize * i, piecesize, piecesize);
                    blank.setBounds(piecesize * (mode - 1), piecesize * (mode - 1), piecesize, piecesize);

                    BufferedImage piecepicture = new BufferedImage(piecesize, piecesize, BufferedImage.TYPE_INT_RGB);
                    Graphics g = piecepicture.getGraphics();
                    g.drawImage(img, 0, 0, piecesize, piecesize, piecesize * j, piecesize * i, piecesize * (j + 1), piecesize * (i + 1), this);
                    last_piece = new JLabel(new ImageIcon(piecepicture));
                    last_piece.setBounds(piecesize * (mode - 1), piecesize * mode, piecesize, piecesize);
                } else {
                    puzzle[i][j] = i * mode + j + 1;
                    BufferedImage piecepicture = new BufferedImage(piecesize, piecesize, BufferedImage.TYPE_INT_RGB);
                    Graphics g = piecepicture.getGraphics();
                    g.drawImage(img, 0, 0, piecesize, piecesize, piecesize * j, piecesize * i, piecesize * (j + 1), piecesize * (i + 1), this);
                    piece[i][j] = new JLabel(new ImageIcon(piecepicture));
                    piece[i][j].setBounds(piecesize * j, piecesize * i, piecesize, piecesize);
                }
            }
        }
    }

    public int findIndex(int piece) {
        int index = 0, i = 0, j = 0;
        breakpoint:
        for (i = 0; i < mode; i++)
            for (j = 0; j < mode; j++)
                if (puzzle[i][j] == piece)
                    break breakpoint;

        index = mode * i + j;
        return index;
    }

    public void reverseMove() {
        if (cnt > 0) {
            movePiece(findIndex(register[cnt]), findIndex(0));
            cnt --;
        }
    }

    public boolean moveCondition(int pieceindex, int zeroindex) {
        int piecerow = pieceindex / mode;
        int piececol = pieceindex % mode;
        int zerorow = zeroindex / mode;
        int zerocol = zeroindex % mode;

        if (((zerorow == piecerow + 1 || zerorow == piecerow - 1) && zerocol == piececol)
                || ((zerocol == piececol + 1 || zerocol == piececol - 1) && zerorow == piecerow))
            return true;
        else
            return false;
    }

    public String movePiece(int pieceindex, int zeroindex) {
        int piecerow = pieceindex / mode;
        int piececol = pieceindex % mode;
        int zerorow = zeroindex / mode;
        int zerocol = zeroindex % mode;

        try {
            if (!reverse) {
                register[cnt + 1] = puzzle[piecerow][piececol];
                cnt++;
            }
            reverse = false;
            puzzle[zerorow][zerocol] = puzzle[piecerow][piececol];
            puzzle[piecerow][piececol] = 0;

            tmp.setIcon(piece[zerorow][zerocol].getIcon());
            piece[zerorow][zerocol].setIcon(piece[piecerow][piececol].getIcon());
            piece[piecerow][piececol].setIcon(tmp.getIcon());

            return "noerror";

        } catch (Exception e) {
            return "exception";
        }
    }

    public void mixPuzzle() {
        int zeroindex;
        int _case;
        boolean[] mixflag = new boolean[4];

        for (int i = 0; i < 200; i++) {
            zeroindex = findIndex(0);
            do {
                _case = (int) (Math.random() * 4);
            } while (mixflag[_case]);
            for (int j = 0; j < 4; j++) mixflag[j] = false;

            switch (_case) {
                case 0:
                    if (!movePiece(zeroindex - 1, zeroindex).equals("noerror"))
                        i--;
                    mixflag[1] = true;
                    break;
                case 1:
                    if (!movePiece(zeroindex + 1, zeroindex).equals("noerror"))
                        i--;
                    mixflag[0] = true;
                    break;
                case 2:
                    if (!movePiece(zeroindex - mode, zeroindex).equals("noerror"))
                        i--;
                    mixflag[3] = true;
                    break;
                case 3:
                    if (!movePiece(zeroindex + mode, zeroindex).equals("noerror"))
                        i--;
                    mixflag[2] = true;
                    break;
            }
        }

        zeroindex = findIndex(0);
        for (int i = (zeroindex / mode); i < mode - 1; i++) {
            movePiece(zeroindex + mode, zeroindex);
            zeroindex = zeroindex + mode;
        }

        for (int i = (zeroindex % mode); i < mode - 1; i++) {
            movePiece(zeroindex + 1, zeroindex);
            zeroindex = zeroindex + 1;
        }
        cnt = 0;
        for(int i = 0; i < 1000; i++)
            register[i] = 0;

        register[0] = mode * mode - 1;
    }

    public void addPieces() {
        for (int i = 0; i < mode; i++) {
            for (int j = 0; j < mode; j++) {
                panel_puzzle02.add(piece[i][j]);
            }
        }
    }

    public void removePieces(){
        try {
            for (int i = 0; i < mode; i++)
                for (int j = 0; j < mode; j++)
                    panel_puzzle02.remove(piece[i][j]);
            System.out.println("퍼즐 삭제");
        }
        catch (Exception e) {
            return;
        }
    }

    public void changeMode(int mode){
        ModeChangeThread.start();
        modeConvert(mode);
        setPuzzleSequence();
        mixPuzzle();
        addPieces();
        System.out.printf("%dX%d 모드\n", mode, mode);
        count.setText("시도 횟수 : " + cnt);
    }

    public boolean answerFlag(){
        int answercount = 0;
        for (int i = 0; i < mode; i++)
            for (int j = 0; j < mode; j++)
                if (puzzle[i][j] == mode * i + j + 1)
                    answercount += 1;

        if (answercount == (mode * mode - 1))
            return true;
        else
            return false;
    }
    // 퍼즐 움직일 때 애니메이션 효과
    public Thread MovePuzzleThread = new Thread(this) {

        public void run() {
            MovePuzzleThread = new Thread(this);
            try {
                int i, j = 0;
                breakout:
                for (i = 0; i < mode; i++)
                    for (j = 0; j < mode; j++)
                        if (piece[i][j] == pushedlabel)
                            break breakout;

                int piece_x = j * piecesize;
                int piece_y = i * piecesize;

                int l = 1;
                for (int k = 1; k <= piecesize / smoothmove; k++) {
                    if (k % 10 == 9)
                        l += 4;
                    sleep(l);
                    piece[i][j].setLocation(piece_x + (blank.getLocation().x - piece_x) / (piecesize / smoothmove) * k, piece_y + (blank.getLocation().y - piece_y) / (piecesize / smoothmove) * k);
                }
                if (!reverse) {
                    movePiece(mode * i + j, findIndex(0));
                    count.setText("시도 횟수 : " + cnt);
                    if (answerFlag()) {
                        movepiece_lock = false;
                        System.out.println("성공!");
                        succeselabel.setText("<html>Sliding Puzzle [" + mode + " X " + mode  + "] 성공!!! <br> 시도 횟수 : " + cnt);
                        modechange_lock = false;
                        SucceseThread01.start();
                    }
                }
                else {
                    reverseMove();
                    count.setText("시도 횟수 : " + cnt);
                }
                piece[i][j].setLocation(piece_x, piece_y);
                blank.setLocation(piece_x, piece_y);


            } catch (Exception e) {
                return;
            }
        }
    };

    // 셔플 애니메이션 효과
    public Thread ShuffleThread = new Thread(this) {

        public void run() {
            ShuffleThread = new Thread(this);
            try {
                int[][] piecelocation_x = new int[mode][mode];
                int[][] piecelocation_y = new int[mode][mode];
                for (int i = 0; i < mode; i++) {
                    for (int j = 0; j < mode; j++) {
                        piecelocation_x[i][j] = j * piecesize;
                        piecelocation_y[i][j] = i * piecesize;
                    }
                }
                int l = 1;
                for (int i = 1; i <= piecesize / smoothmove; i++) {
                    if (i % 10 == 9)
                        l += 5;
                    sleep(l);
                    for (int j = 0; j < mode; j++) {
                        for (int k = 0; k < mode; k++) {
                            piece[j][k].setLocation(piecelocation_x[j][k] + (centerlocation_x - piecelocation_x[j][k]) / (piecesize / smoothmove) * i, piecelocation_y[j][k] + (centerlocation_y - piecelocation_y[j][k]) / (piecesize / smoothmove) * i);
                        }
                    }
                }
                sleep(200);
                removePieces();
                setPuzzleSequence();
                mixPuzzle();
                addPieces();

                for (int i = 0; i < mode; i++) {
                    for (int j = 0; j < mode; j++) {
                        piecelocation_x[i][j] = j * piecesize;
                        piecelocation_y[i][j] = i * piecesize;
                    }
                }
                l = 1;
                for (int i = 1; i <= piecesize / smoothmove; i++) {
                    if (i % 10 == 9)
                        l += 5;
                    sleep(l);
                    for (int j = 0; j < mode; j++) {
                        for (int k = 0; k < mode; k++) {
                            piece[j][k].setLocation(centerlocation_x + (piecelocation_x[j][k] - centerlocation_x) / (piecesize / smoothmove) * i, centerlocation_y + (piecelocation_y[j][k] - centerlocation_y) / (piecesize / smoothmove) * i);
                        }
                    }
                }
                count.setText("시도 횟수 : " + cnt);
                modechange_lock = true;
            } catch (Exception e) {
                modechange_lock = true;
                return;
            }
        }
    };

    // 모드 변경 애니메이션
    public Thread ModeChangeThread = new Thread(this) {

        public void run() {
            ModeChangeThread = new Thread(this);
            try {
                int l = 1;
                for (int i = 0; i <= 100; i++) {
                    if (i % 25 == 24)
                        l *= 2;
                    panel_puzzle02.setLocation(10, - 490 + 5 * i);
                    sleep(l);
                }
                modechange_lock = true;
            } catch (Exception e) {
                return;
            }
        }
    };

    // 성공 시 모션 애니메이션 01
    public Thread SucceseThread01 = new Thread(this){

        public void run() {
            SucceseThread01 = new Thread(this);
            panel_puzzle02.add(last_piece);
            try {
                int lastpiece_x = last_piece.getLocation().x;
                int lastpiece_y = last_piece.getLocation().y;
                int l = 3;
                sleep(400);
                for (int k = 1; k <= piecesize / smoothmove; k++) {
                    if (k % 10 == 9)
                        l += 4;
                    sleep(l);
                    last_piece.setLocation(lastpiece_x + (blank.getLocation().x - lastpiece_x) / (piecesize / smoothmove) * k, lastpiece_y + (blank.getLocation().y - lastpiece_y) / (piecesize / smoothmove) * k);
                }
                sleep(600);
                l = 1;
                for (int i = 0; i <= 100; i++) {
                    if (i % 25 == 24)
                        l *= 2;
                    succese_alert01.setLocation(100, -250 + 4 * i);
                    sleep(l);
                }
            }
            catch (Exception e){
                return;
            }
        }
    };

    // 성공 시 모션 애니메이션 02
    public Thread SucceseThread02 = new Thread(this){

        public void run() {
            SucceseThread02 = new Thread(this);
            try {
                sleep(100);
                int l = 1;
                for (int i = 0; i <= 100; i++) {
                    if (i % 25 == 24)
                        l *= 2;
                    panel_puzzle02.setLocation(10, 5 * i);
                    succese_alert01.setLocation(100, 150 + 5 * i);
                    sleep(l);
                }
                panel_puzzle02.remove(last_piece);
                removePieces();
                modechange_lock = true;
                movepiece_lock = true;
                mode = 0;
                cnt = 0;
                count.setText("시도 횟수 : " + cnt);
            }
            catch (Exception e){
                return;
            }
        }
    };

    @Override
    public void run() { }
}

# [Controller]

package java_finalterm01_sliding_puzzle_GUI;

    public class Controller extends EventListener{

        public void start() {
            new View();
        }
        public Controller() {
            panel_puzzle02.addMouseListener(new MovePieceListener());
            succese_check.addActionListener(new ButtonListener());
            shuffle.addActionListener(new ButtonListener());
            reversebutton.addActionListener(new ButtonListener());
            for (int i = 0; i < 3; i++) {
                choosemode[i].addActionListener(new ButtonListener());
            }
        }
    }

# [View]
package java_finalterm01_sliding_puzzle_GUI;

    import javax.swing.*;
    import java.awt.*;

    public class View extends Controller{

        public View() {
            setSize(1350, 800);
            setVisible(true);
            setResizable(false);
            setTitle("Sliding Puzzle");
            setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

            pic_Crop();

            c.setBackground(new Color(200, 184, 223));
            c.setLayout(null);
            panel_puzzle01.setBounds(110, 160, 500, 500);
            panel_puzzle01.setBackground(new Color(181, 159, 213));
            panel_puzzle01.setLayout(null);
            panel_puzzle01_background.setBounds(10, 10, 480, 480);
            panel_puzzle01_background.setBackground(new Color(177, 151, 215));
            panel_puzzle02.setBounds(10, 10, 480, 480);
            panel_puzzle02.setBackground(new Color(177, 151, 215));
            panel_puzzle02.setLayout(null);
            panel_puzzle02.setBounds(10, 10, 480, 480);
            panel_puzzle02.setBackground(new Color(177, 151, 215));
            panel_puzzle02.setLayout(null);
            succese_alert01.setBounds(100, -350, 300, 200);
            succese_alert01.setBackground(new Color(202, 172, 229));
            succese_alert01.setLayout(null);
            succese_alert02.setBounds(10, 10, 280, 180);
            succese_alert02.setBackground(new Color(212, 194, 229));
            succese_alert02.setLayout(null);

            title.setFont(new Font("배달의민족 한나", Font.BOLD, 40));
            title.setBounds(225, 50, 500, 100);
            modelabel.setFont(new Font("배달의민족 한나", Font.BOLD, 25));
            modelabel.setBounds(850, 25, 50, 75);
            count.setFont(new Font("배달의민족 한나", Font.BOLD, 20));
            count.setBounds(750, 600, 200, 50);
            label_picture.setBounds(750, 200, 375, 375);
            succeselabel.setFont(new Font("배달의민족 한나", Font.BOLD, 15));
            succeselabel.setBounds(50, 10, 200, 100);

            shuffle.setFont(new Font("배달의민족 한나", Font.BOLD, 15));
            shuffle.setBounds(1125, 100, 100, 50);
            reversebutton.setFont(new Font("배달의민족 한나", Font.BOLD, 15));
            reversebutton.setBounds(750, 650, 100, 50);
            succese_check.setFont(new Font("배달의민족 한나", Font.BOLD, 10));
            succese_check.setBounds(100, 125, 75, 30);

            for (int i = 0; i < 3; i++) {
                choosemode[i].setFont(new Font("배달의민족 한나", Font.BOLD, 15));
                choosemode[i].setBounds(700 + 125 * i, 100, 100, 50);
                c.add(choosemode[i]);
            }

            c.add(title);
            c.add(modelabel);
            c.add(shuffle);
            c.add(count);
            c.add(reversebutton);
            c.add(panel_puzzle01);
            c.add(label_picture);
            panel_puzzle01.add(succese_alert01);
            succese_alert01.add(succese_alert02);
            succese_alert02.add(succeselabel);
            succese_alert02.add(succese_check);
            panel_puzzle01.add(panel_puzzle02);
            panel_puzzle01.add(panel_puzzle01_background);
        }
    }

# [EventListener]//

package java_finalterm01_sliding_puzzle_GUI;

    import javax.swing.*;
    import java.awt.*;
    import java.awt.event.ActionEvent;
    import java.awt.event.MouseAdapter;
    import java.awt.event.MouseEvent;

    public class EventListener extends Model{
        // 이벤트 클래스 (퍼즐 클릭)
        class MovePieceListener extends MouseAdapter {
            public void mouseReleased(MouseEvent e) {
                if (movepiece_lock) {
                    int col = e.getX();
                    int row = e.getY();
                    int index_row = row / piecesize;
                    int index_col = col / piecesize;
                    System.out.println("push [" + index_row + ", " + index_col + "]");
                    pushedlabel = piece[index_row][index_col];
                    if (moveCondition(mode * index_row + index_col, findIndex(0))) {
                        System.out.println("퍼즐 이동");
                        MovePuzzleThread.start();
                    } else
                        System.out.println("근접한 퍼즐을 눌러주세요");
                }
            }
        }

        // 이벤트 클래스 (모드 변경 + 기능)
        class ButtonListener implements java.awt.event.ActionListener {
            public void actionPerformed(ActionEvent e) {
                JButton b = (JButton) e.getSource();
                switch (b.getText()) {
                    case "3X3 모드":
                        if (mode != 3 && modechange_lock) {
                            modechange_lock = false;
                            removePieces();
                            smoothmove = 8;
                            changeMode(3);
                        }
                        break;
                    case "4X4 모드":
                        if (mode != 4 && modechange_lock) {
                            modechange_lock = false;
                            removePieces();
                            smoothmove = 6;
                            changeMode(4);
                        }
                        break;
                    case "5X5 모드":
                        if (mode != 5 && modechange_lock) {
                            modechange_lock = false;
                            removePieces();
                            smoothmove = 4;
                            changeMode(5);
                        }
                        break;
                    case "되돌리기":
                        if (cnt > 0 && modechange_lock) {
                            breakpoint:
                            for (int i = 0; i < mode; i++)
                                for (int j = 0; j < mode; j++)
                                    if (puzzle[i][j] == register[cnt]) {
                                        pushedlabel = piece[i][j];
                                        break breakpoint;
                                    }
                            reverse = true;
                            MovePuzzleThread.start();
                            count.setText("시도 횟수 : " + cnt);
                            System.out.println("되돌리기 완료");
                        } else {
                            System.out.println("되돌리기 실패");
                        }
                        break;
                    case "Shuffle":
                        if (modechange_lock) {
                            modechange_lock = false;
                            ShuffleThread.start();
                            System.out.println("Shuffle");
                        }
                        break;
                    case "확인":
                        SucceseThread02.start();
                        break;
                }
            }
        }
    }

# [Main]
package java_finalterm01_sliding_puzzle_GUI;

        public class _Sliding_puzzle {
            public static void main (String[] args) {
                Controller _controller = new Controller();
                _controller.start();
            }
        }

