## Gemm types

  - sgemm - for single precision matrix (float)
  - dgemm - for double precision matrix (double)

## Blas GEMM parameters

MatrixA <- left hand side matrix

MatrixB <- right hand side matrix

MatrixC <- output matrix (the matrix product)

NOTE: cblas_?gemm() functions compute ```MatrixC = alpha * (op(MatrixA) * op(MatrixB)) + beta * MatrixC```, where `op()` is the optional transpose. So with `alpha = 1` and `beta = 1` the product is **added** into MatrixC, and with `beta = 0` MatrixC is simply **overwritten** by the product.

**cblas_?gemm(MatrixOrder, TransposeMatrixA?, TransposeMatrixB?, M, N, K, alpha, MatrixA, lda, MatrixB, ldb, beta, MatrixC, ldc);**

  - MatrixOrder - this could either be row major or column major order of accessing elements in the memory
  - TransposeMatrixA? - specify if you want to transpose MatrixA before operation or not
  - TransposeMatrixB? - specify if you want to transpose MatrixB before operation or not
  - M - this is the number of rows of op(MatrixA) (and of MatrixC)
  - N - this is the number of columns of op(MatrixB) (and of MatrixC)
  - K - this is the number of columns of op(MatrixA), and the number of rows of op(MatrixB), (Matrix A's columns & Matrix B's rows should have the same number/counts)
  - alpha - this scales the elements of the matrix product of MatrixA and MatrixB => ```(MatrixA*MatrixB)*alpha```
  - beta - this scales the initial values of the MatrixC before the scaled product is added => ```MatrixC*beta```
  - lda - the **leading dimension** of MatrixA, i.e. the distance (in elements) between the start of two consecutive rows (row major) or columns (column major) **as stored in memory**. If MatrixA is not a **sub matrix** of a larger matrix, then lda is just the number of columns of the stored A (row major) or the number of rows of the stored A (column major). If MatrixA is a **sub matrix** of a larger matrix, lda is the number of columns (row major) / rows (column major) of that larger matrix.
  - ldb - is the same as lda, but for MatrixB
  - ldc - is the same as lda, but for the output MatrixC

## Examples:

### Matrix dot product - Row Major Operation (the more intuitive way)
```c++
cblas_dgemm(CblasRowMajor, CblasTrans, CblasNoTrans,3,3,2,1,A,3,B,3,0,C,3);
```

### Matrix dot product - Column Major Operation
```c++
cblas_dgemm(CblasColMajor, CblasNoTrans, CblasTrans,3,3,2,1,A,3,B,3,0,C,3);
```

note: both examples use the **same arrays** (see the sample program below). The 6 values of `A` can be read as a 2x3 matrix in row major order, or as a 3x2 matrix in column major order (which is exactly the transpose). That's why the row major call needs `CblasTrans` for A (to turn the stored 2x3 into the 3x2 Matrix A), while the column major call doesn't, and the opposite happens for B. In both cases op(A) is 3x2 and op(B) is 2x3, so ```M = 3```, ```N = 3```, ```K = 2```, and the leading dimensions are all ```3```.

### Matrix A
```
1	-3
2	 4
1	-1
```

### Matrix B
```
 1	2	 1
-3	4	-1
```

### Sample Program
```c++
#include <cblas.h>
#include <stdio.h>

int main()
{
    int i=0;

    double A[6] =
    {
         1.0, 2.0,  1.0,
        -3.0, 4.0, -1.0
    };

    double B[6] =
    {
         1.0, 2.0,  1.0,
        -3.0, 4.0, -1.0
    };

    double C[9] =
    {
        0.0, 0.0, 0.0,
        0.0, 0.0, 0.0,
        0.0, 0.0, 0.0
    };

    printf("Column Major Order\n");

    cblas_dgemm(CblasColMajor, CblasNoTrans, CblasTrans,3,3,2,1,A,3,B,3,0,C,3);

    for(i=0; i<9; i++)
    {
            if(i%3==0) printf("\n");
            printf("%lf ", C[i]);
    }

    printf("\n\nRow Major Order\n");

    cblas_dgemm(CblasRowMajor, CblasTrans, CblasNoTrans,3,3,2,1,A,3,B,3,0,C,3);

    for(i=0; i<9; i++)
    {
            if(i%3==0) printf("\n");
            printf("%lf ", C[i]);
    }

    printf("\n");
}
```

output (the result happens to be a symmetric matrix, so it looks the same whether it is read in row or column major order):
```shell
Column Major Order

10.000000 -10.000000 4.000000 
-10.000000 20.000000 -2.000000 
4.000000 -2.000000 2.000000 

Row Major Order

10.000000 -10.000000 4.000000 
-10.000000 20.000000 -2.000000 
4.000000 -2.000000 2.000000 
```
