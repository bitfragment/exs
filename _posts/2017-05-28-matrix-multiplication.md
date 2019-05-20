---
title: "Matrix multiplication"
date: 2017-05-28
tags: java
---

```java
import static java.lang.Math.abs;

public class MyMatrix {
    private final int size;
    private int[][] matrix;

    public MyMatrix(int size) {
        this.size = size;
        this.matrix = init(size);
    }

    public int getSize() {
        return size;
    }

    public int[][] getMatrix() {
        return matrix;
    }
```

## init()

Initialize matrix with numeric values.

```java
java> init(3)
int[][] res20 = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

---

```java
        private int[][] init(int n) {
            int[][] sq = new int[n][n];
            for (int i = 1, row = 0; row < n; row++) {
                for (int col = 0; col < n; col++) {
                    sq[row][col] = i;
                    i++;
                }
            }
            return sq;
        }
```

## copyMatrix()

Return copy of matrix.

```java
    private int[][] copyMatrix() {
        int[][] copy = new int[size][size];
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                copy[row][col] = matrix[row][col];
            }
        }
        return copy;
    }
```

## printMatrix()

Print matrix.

```java
java> MyMatrix s = new MyMatrix(3)
java> s.printMatrix()
   1   2   3
   4   5   6
   7   8   9
```

---

```java
    public void printMatrix() {
        for (int row = 0; row < size; row++) {
            for (int col = 0; col < size; col++) {
                System.out.printf("%4d", matrix[row][col]);
            }
            System.out.println();
        }
    }
```

## upsideDownFlip()

Flip `ring` of matrix horizontally.  
Rings of matrix are numbered from the outermost ring (1).  
Return a new `MyMatrix`.

```java
java> MyMatrix sFlipped = new MyMatrix(3).upsideDownFlip(1)
java> sFlipped.printMatrix()
   7   8   9
   4   5   6
   1   2   3
```

---

```java
    public MyMatrix upsideDownFlip(int ring) {
        MyMatrix flipped = new MyMatrix(size);
        if (size / ring < 2) return flipped;

        int[][] flippedS = flipped.matrix;
        int[][] thisS = this.matrix;

        // Flip top and bottom rows of ring.
        for (int col = 0; col < size; col++) {
            if (col >= ring - 1 && col <= size - ring) {
                flippedS[ring - 1][col] = thisS[size - ring][col];
                flippedS[size - ring][col] = thisS[ring - 1][col];
            }
        }

        // Flip inner elements of left and right columns of ring.
        for (int row = ring; row <= size - 2; row++) {
            flippedS[row][ring - 1] = thisS[size - row - 1][ring - 1];
            flippedS[row][size - ring] = thisS[size - row - 1][size - ring];
        }

        return flipped;
    }
```

## mainDiagonalFlip()

Flip `ring` of matrix around its main diagonal (top left to bottom right).  
Rings of matrix are numbered from the outermost ring (1).  
Return a new `MyMatrix`.  

```java
java> MyMatrix sFlipped = new MyMatrix(4).mainDiagonalFlip(1)
java> sFlipped.printMatrix()
  1   5   9  13
  2   6   7  14
  3  10  11  15
  4   8  12  16
```

---

```java
    public MyMatrix mainDiagonalFlip(int ring) {
        MyMatrix flipped = new MyMatrix(size);
        if (size / ring < 2) return flipped;

        int[][] flippedS = flipped.matrix;
        int[][] thisS = this.matrix;

        // Flip top row and left column of ring.
        for (int rowcol = ring; rowcol < size - ring + 1; rowcol++) {
            flippedS[ring - 1][rowcol] = thisS[rowcol][ring - 1];
            flippedS[rowcol][ring - 1] = thisS[ring - 1][rowcol];
        }

        // Flip bottom row and right column of ring.
        for (int rowcol = ring; rowcol < size - ring; rowcol++) {
            flippedS[size - ring][rowcol] = thisS[rowcol][size - ring];
            flippedS[rowcol][size - ring] = thisS[size - ring][rowcol];
        }

        return flipped;
    }
```

## rotateRight()

Rotate the current matrix.

```java
java> MyMatrix s = new MyMatrix(4)
java> s.rotateRight(1)
java> s.printMatrix()
 13   9   5   1
 14  10   6   2
 15  11   7   3
 16  12   8   4
java> s.rotateRight(-2)
java> s.printMatrix()
  4   8  12  16
  3   7  11  15
  2   6  10  14
  1   5   9  13
```

---

```java
    public void rotateRight(int numberOfTurns) {
        int[][] copy = copyMatrix();
        int copyRow, copyCol;
        for (int turn = 0; turn < abs(numberOfTurns); turn++) {
            for (int row = 0; row < size; row++) {
                for (int col = 0; col < size; col++) {
                    copyRow = numberOfTurns > 0 ? size - col - 1 : col;
                    copyCol = numberOfTurns > 0 ? row : size - row - 1;
                    matrix[row][col] = copy[copyRow][copyCol];
                }
            }
        copy = copyMatrix();
        }
    }
```

---
```java
} // class MyMatrix
```
