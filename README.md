# MineSweeper
the game of mine sweeper
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Runtime.InteropServices;
using System.Security.Cryptography.X509Certificates;
using System.Text;
using System.Threading.Tasks;

namespace MineSweeperLibrary

{

    public class MinesweeperModel
    {
        public static int ALIVE = 1;
        public static int DEAD = 0;
        public static int WINNER = 3;
        public int state = ALIVE;
        public Random random = new Random();
        public HashSet<int> bombs = new HashSet<int>();
        public int[,] board;
        public int[,] revealedBoard;
        public int[,] newRevealedSquares;// used for a gui implementation to update only the newly revealed squares
        public HashSet<Point> squaresToDisplay = new HashSet<Point>(); 
        private  int bombCount;
        private  int rows;
        private  int columns;


        public MinesweeperModel(int bombCount, int rows, int columns)
        {
            this.bombCount = bombCount;
            this.rows = rows;
            this.columns = columns;
            board = new int[rows, columns];
            newRevealedSquares = new int[rows, columns];
            revealedBoard = new int[rows, columns];
            state = ALIVE;
            if (bombCount > rows * columns) // there are only rows*columns squares on the board
            {
                this.bombCount = rows * columns;
            }


        }

        public void DropBombs(int row, int column)
        {
            int totalDropped = 0;
            while (totalDropped < bombCount)
            {
                for (int i = 0; i < board.GetLength(0); i++)
                {
                    for (int j = 0; j < board.GetLength(1); j++)
                    {
                        if (i == row && j == column) // doesn't let there be a bomb in the first square you pick
                        {
                            continue;
                        }
                        if (board[i, j] >= 0)
                        {
                            if (random.Next(100) < 10) // randomly drop bombs
                            {
                                board[i, j] = -1;
                                MarkNeighbors(i, j);
                                totalDropped++;
                                if (totalDropped >= this.bombCount)
                                {
                                    goto End;
                                }
                            }
                        }
                    }
                }
                
            }   End: ;
        }
            
       
        public void MarkNeighbors(int row, int column)
        {
            for (int i = Math.Max(0, row - 1); i <= Math.Min(board.GetLength(0) - 1, row + 1); i++)
            {
                for (int j = Math.Max(0, column - 1); j <= Math.Min(board.GetLength(1) - 1, column + 1); j++)
                {
                    if (row == i && column == j || board[i, j] <= -1
                    ) // Don't mark yourself or another bomb as a neighbor of a bomb
                    {
                        continue;
                    }

                    int value = (board[i, j] += 1);
                    board[i, j] = value;
                }
            }
        }

        public int Inspect(int row, int column)
        {
            return board[row, column];
        }

        public void MakeAMove(Point p)
        {
            if (board[p.X, p.Y] > 0)
            {
                revealedBoard[p.X, p.Y] = 1;
                newRevealedSquares[p.X, p.Y] = 1;
                squaresToDisplay.Add(p);
            }
            else if (board[p.X, p.Y] < 0)
            {
                Console.WriteLine("BOMB!!!!!!!!!!!! GAME OVER");
                this.ToString();
                newRevealedSquares[p.X, p.Y] = 1;
                state = DEAD;
            }
            else
            { 
                for (int i = Math.Max(0, p.X - 1); i <= Math.Min(board.GetLength(0) - 1, p.X + 1); i++)
                {
                    for (int j = Math.Max(0, p.Y - 1); j <= Math.Min(board.GetLength(1) - 1, p.Y + 1); j++)
                    {
                        Point point = new Point(i, j);
                        if (board[i, j] == 0)
                        {
                            revealedBoard[i, j] = 1;
                            newRevealedSquares[i, j] = 1;
                            if (squaresToDisplay.Add(point))
                            {
                                MakeAMove(point);
                            }
                        }
                        else
                        {
                             revealedBoard[i, j] = 1;
                             newRevealedSquares[i, j] = 1;
                             squaresToDisplay.Add(point);
                        }
                    }
                }
            }
            

            if (squaresToDisplay.Count >= (this.rows * this.columns) - bombCount)
            {
                state = WINNER;
            }

           
        }

        public override string ToString()
        {
            String toReturn = "The bombs are located on the squares: ";
            foreach (var square in bombs)
            {
                toReturn += square + ", ";
            }

            for (int i = 0; i < board.GetLength(0); i++)
            {

                for (int j = 0; j < board.GetLength(1); j++)
                {
                    int numInSquare = board[i, j];
                    String toPrint = numInSquare > -1 ? numInSquare + "" : "B";
                    Console.Write($"| {toPrint} ");
                }

                Console.WriteLine("|");

            }

            return toReturn;
        }

        public void RevealBoard()
        {
            for (int i = 0; i < board.GetLength(0); i++)
            {

                for (int j = 0; j < board.GetLength(1); j++)
                {
                    if (revealedBoard[i, j] == 0)
                    {
                        Console.Write($"| _ ");
                    }
                    else
                    {
                        int numInSquare = board[i, j];
                        String toPrint = numInSquare > -1 ? numInSquare + "" : "B";
                        Console.Write($"| {toPrint} ");
                    }
                    
                }
                Console.WriteLine("|");
                }
            }
    }
}
