# **The Math Of Indexing Arrays**

Arrays are stored as a flat, contiguous block of memory in most programming languages. Even when you think of a structure as two-dimensional (like a grid or matrix), the computer still lays it out as a single line of values in memory. This document explores the mathematical formulas that let us navigate these flat arrays as if they were multi-dimensional structures.

---

### **Converting Indices Of N Dimensional Arrays To 1-Dimension**

When we want to treat a flat 1D array as if it were 2D, 3D, or even 4D, we need a mapping formula — a way to take a set of coordinates (like a row and column) and convert them into a single index that points to the right position in the 1D array.

The key insight is that these formulas are **recursive**: each additional dimension wraps around the formula of the dimension below it.

For a **2D array** with $J$ columns, the flat index for element at row $i$, column $j$ is:

$$idx^{1D}_{2D} = (J \cdot i) + j$$

Think of it this way: to reach row $i$, you must skip over $i$ complete rows, each of which contains $J$ elements. Then you move $j$ steps into that row.

For a **3D array** with dimensions $I \times J \times K$ (indices $i$, $j$, $k$, where $K$ is the size of the last/innermost dimension), we first compute the 2D index and then scale by $K$:

$$idx^{1D}_{3D} = (K \cdot ((J \cdot i) + j)) + k$$

For a **4D array**, we go one level deeper:

$$idx^{1D}_{4D} = (L \cdot ((K \cdot ((J \cdot i) + j)) + k)) + l$$

This recursive pattern is generalized: each new dimension multiplies the accumulated index by its own size and adds the new coordinate. You can extend this pattern to as many dimensions as you need.

---

### Definitions

The matrices here are assumed to be in **Row Major Order**, meaning elements are stored row by row — the last element of row $i$ is immediately followed by the first element of row $i+1$ in memory. This is the default in languages like C, C++, Python (NumPy), and many others. (Fortran, MATLAB, Julia and R use **Column Major Order** instead, where the formula becomes $j \cdot h + i$.)

- $A$ = the underlying 1D array in memory.
- $M$ = the matrix (2D view of the array).
- $h$ = height of the matrix (number of rows).
- $w$ = width of the matrix (number of columns).
- $i$ = row index (0-based).
- $j$ = column index (0-based).

---

### **Creating 1D array as 2D array**

$$M[h,w] = A[h \cdot w]$$

To represent a 2D matrix $M$ with $h$ rows and $w$ columns using a 1D array $A$, we simply allocate $h \times w$ elements in $A$. There is no structural change to the memory — only the way we *interpret* and *index* into it changes. The matrix is just a lens through which we view the flat array.

---

## *NOTE: the examples below obey the following constraints:*

These constraints ensure that every traversal stays within the valid bounds of the array. Violating them would result in accessing memory outside the array (an out-of-bounds error).

- $0 \leq i < h$ — the row index must be within the matrix height.
- $0 \leq j < w$ — the column index must be within the matrix width.
- $i + n < h$ when moving down, and $j + n < w$ when moving right — moving forward by $n$ steps must not exceed the matrix dimensions.
- $i - n \geq 0$ when moving up, and $j - n \geq 0$ when moving left — moving backward by $n$ steps must not go before the start.
- diagonal moves must satisfy both the row and the column condition.

---

### **Indexing 1D arrays as a 2D array / Matrix**

$$M_{i,j} = A_{(i \cdot w + j)}$$

This is the foundational formula. To access element $(i, j)$ of the matrix, we skip $i$ complete rows (each of width $w$) and then move $j$ positions into the current row. For convenience, we call the resulting 1D position `index`:

$$\text{index} = i \cdot w + j$$

All traversal formulas below are expressed as offsets from this base `index`, making them easy to apply in practice.

---

### **Downward Traversal**

Moving downward means increasing the row index $i$. Since each row takes up $w$ cells in memory, going down by one row means adding $w$ to the current index.

- **by 1 cell**

    $$M_{i+1,j} = A_{((i+1) \cdot w + j)} = A_{(i \cdot w + j) + w} = A_{\text{index} + w}$$

    Moving down by 1 row adds exactly one row's worth of elements ($w$) to the index.

- **by n cells**

    $$M_{i+n,j} = A_{((i+n) \cdot w + j)} = A_{(i \cdot w + j) + (n \cdot w)} = A_{\text{index} + (n \cdot w)}$$

    Moving down by $n$ rows means skipping over $n$ entire rows, so we add $n \cdot w$ to the index.

---

### **Upward Traversal**

Moving upward is the mirror of downward — we subtract $w$ per row instead of adding.

- **by 1 cell**

    $$M_{i-1,j} = A_{((i-1) \cdot w + j)} = A_{(i \cdot w + j) - w} = A_{\text{index} - w}$$

- **by n cells**

    $$M_{i-n,j} = A_{((i-n) \cdot w + j)} = A_{(i \cdot w + j) - (n \cdot w)} = A_{\text{index} - (n \cdot w)}$$

    Since rows are contiguous in memory, moving up simply reverses the offset logic of downward traversal.

---

### **Leftward Traversal**

Moving left means decreasing the column index $j$ by 1. Since columns are adjacent in memory (within the same row), this simply decreases the index by 1.

- **by 1 cell**

    $$M_{i,j-1} = A_{(i \cdot w + j - 1)} = A_{(i \cdot w + j) - 1} = A_{\text{index}-1}$$

- **by n cells**

    $$M_{i,j-n} = A_{(i \cdot w + j - n)} = A_{(i \cdot w + j) - n} = A_{\text{index}-n}$$

    Note: Moving left only makes sense as long as $j - n \geq 0$, otherwise you wrap into the previous row, which is typically unintended.

---

### **Rightward Traversal**

Moving right increases the column index $j$, which directly maps to incrementing the flat array index.

- **by 1 cell**

    $$M_{i,j+1} = A_{(i \cdot w + j + 1)} = A_{(i \cdot w + j) + 1} = A_{\text{index}+1}$$

- **by n cells**

    $$M_{i,j+n} = A_{(i \cdot w + j + n)} = A_{(i \cdot w + j) + n} = A_{\text{index}+n}$$

    Similarly, moving right must respect $j + n < w$ to avoid accidentally wrapping into the next row.

---

### **Down-Left Traversal**

Diagonal traversal combines both a row shift and a column shift simultaneously. Moving down-left increases $i$ by $n$ (adding $n \cdot w$) and decreases $j$ by $n$ (subtracting $n$), so the net effect is adding $n \cdot w - n$ to the index.

- **by 1 cell**

    $$M_{i+1, j-1} = A_{((i+1) \cdot w+j-1)} = A_{(i \cdot w+j) + w-1} = A_{\text{index}+w-1}$$

- **by n cells**

    $$M_{i+n,j-n} = A_{((i+n) \cdot w+j-n)} = A_{(i \cdot w+j)+(n \cdot w)-n} = A_{\text{index}+(n \cdot w)-n}$$

---

### **Up-Left Traversal**

Moving up-left decreases both $i$ and $j$, so we subtract $n \cdot w$ for the row shift and subtract $n$ for the column shift. The combined offset is $-(n \cdot w) - n$.

- **by 1 cell**

    $$M_{i-1, j-1} = A_{((i-1) \cdot w+j-1)} = A_{(i \cdot w+j) - w-1} = A_{\text{index}-w-1}$$

- **by n cells**

    $$M_{i-n,j-n} = A_{((i-n) \cdot w+j-n)} = A_{(i \cdot w+j)-(n \cdot w)-n} = A_{\text{index}-(n \cdot w)-n}$$

---

### **Down-Right Traversal**

Moving down-right increases both $i$ and $j$, so both offsets are positive. The combined effect is adding $n \cdot w + n$ to the index.

- **by 1 cell**

    $$M_{i+1,j+1} = A_{((i+1) \cdot w+j+1)} = A_{(i \cdot w+j) + w+1} = A_{\text{index}+w+1}$$

- **by n cells**

    $$M_{i+n,j+n} = A_{((i+n) \cdot w+j+n)} = A_{(i \cdot w+j)+(n \cdot w)+n} = A_{\text{index}+(n \cdot w)+n}$$

---

### **Up-Right Traversal**

Moving up-right decreases $i$ (subtracting $w$ per row, so $n \cdot w$ in total) but increases $j$ (adding $n$), giving a net offset of $-(n \cdot w) + n$.

- **by 1 cell**

    $$M_{i-1,j+1} = A_{((i-1) \cdot w+j+1)} = A_{(i \cdot w+j) - w+1} = A_{\text{index}-w+1}$$

- **by n cells**

    $$M_{i-n,j+n} = A_{((i-n) \cdot w+j+n)} = A_{(i \cdot w+j)-(n \cdot w)+n} = A_{\text{index}-(n \cdot w)+n}$$

---

# Indexing Combined Arrays

Sometimes, we want to compute every combination of elements from two arrays and store the results in a single flat array. This section covers how to reverse-engineer the original indices from any position in the combined array.

## Setup

Say we have an array `A` and array `B`. We create array `C` which contains the result of combining every element of `A` with every element of `B`:

```py
C = []
i = 0
while i < len(A):
    j = 0
    while j < len(B):
        C.append(A[i] + B[j])
        j += 1
    i += 1
```

The total number of elements in `C` is `len(A) * len(B)`. The structure of `C` mirrors a 2D matrix: the outer loop iterates over `A` (like rows), and the inner loop iterates over `B` (like columns). So `C` is essentially `A`-indexed rows laid out in `B`-sized chunks — exactly the row-major layout we've been studying.

## Reverse Indexing

**Q1: Given an index into `C`, which indices in `A` and `B` were used to produce that element?**

Because `C` is built like a row-major 2D array where the "row size" is `len(B)`, we can apply the same logic as converting a 1D index back to 2D coordinates:

```py
def reverse_indexing(C_idx):
    # Integer division tells us how many full "rows" (chunks of B) fit before C_idx,
    # which directly corresponds to the index in A.
    A_idx = (C_idx // len(B))

    # The remainder tells us the position within that row, i.e., the index in B.
    B_idx = (C_idx %  len(B))

    return (A_idx, B_idx)
```

To verify intuitively: the first `len(B)` elements of `C` (indices `0` to `len(B)-1`) all came from `A[0]`, paired with `B[0]`, `B[1]`, ..., `B[len(B)-1]`. The next `len(B)` elements came from `A[1]`, and so on. Integer division (`//`) extracts the "group number" (the `A` index), and modulo (`%`) extracts the position within the group (the `B` index). This is the inverse of the formula $\text{index} = i \cdot w + j$.
