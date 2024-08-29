# HW2_ASC

### Organization
The assignment involves implementing a matrix operation using 3 distinct methods, in order to compare execution time and cache memory usage, both actions based on input datasets of the form *input_*. The implementation time was approximately 1 day and 5 hours, with the optimization part found in the *solver_opt.c* file taking longer. Moreover, for better performance measurement, the *custom_extended_input* file containing 14 tests was used.

### Implementation
Notation: A' -> A transposed.
As general optimizations, all iteration indices *k* have different start and end values to avoid calculations that represent simple multiplications by 0, in other words, the aim was to highlight the fact that matrix A is *upper triangular*. These observations occur in the implementations *solver_neopt.c* and *solver_opt.c*.

For the calculation of *BA_t = B * A'*, traversing A by rows was chosen to increase the number of cache hits due to spatial locality of data. Also, to determine the value of BA_t[i][j], it is sufficient to sum the products of elements on row i of matrix B with elements on row j of matrix A, obviously starting from column j, as we want to avoid unnecessary computational calculation.

For the calculation of *ABA_t = A * B * A'*, similar to the BA_t operation, avoiding traversing the 0's in A was chosen, therefore the iteration index k starts from i, and not from 0 (avoiding the overhead of processing null elements).

For the calculation of *B_tB_t = B' * B'*, no modification was chosen to produce a certain optimization since B is an arbitrary square matrix.

*Blas* Version

For this version, it was decided to use specialized functions for matrix calculation from the BLAS library. Initially, memory was allocated for the matrices used in processing so that there are memory areas to store intermediate results. The implementation is based on optimizations made by BLAS functions, such as cblas_dtrmm() which multiplies an upper triangular matrix with an arbitrary matrix.

These are the running times:
```
N = 400: Time = 0.039051
N = 520: Time = 0.082263
N = 640: Time = 0.150279
N = 760: Time = 0.248679
N = 880: Time = 0.383806
N = 1000: Time = 0.557041
N = 1120: Time = 0.736165
N = 1240: Time = 1.007427
N = 1360: Time = 1.331663
N = 1480: Time = 1.715105
N = 1520: Time = 1.835016
N = 1580: Time = 2.062585
N = 1590: Time = 2.114121
N = 1600: Time = 2.169437
```

*Neopt* Version

For this version, the classical approach was chosen for matrix multiplication, obviously not ignoring the fact that A is an upper triangular matrix. There is a function for each operation necessary in calculating matrix C.

These are the running times:
```
N = 400: Time = 1.104875
N = 520: Time = 2.427099
N = 640: Time = 5.561833
N = 760: Time = 7.442277
N = 880: Time = 11.474910
N = 1000: Time = 16.781019
N = 1120: Time = 25.995712
N = 1240: Time = 32.177250
N = 1360: Time = 43.591892
N = 1480: Time = 56.415199
N = 1520: Time = 62.512680
N = 1580: Time = 70.806465
N = 1590: Time = 76.373138
N = 1600: Time = 80.531036
```

*Opt_m* Version

For this version, certain optimizations were chosen so that operations are executed faster by the processor. Well, according to the observations provided at the beginning of the document, the variant of traversing A by rows is faster to avoid as many cache misses as possible. Also, the calculation of indices of the form j * N + i was eliminated with the help of references that are incremented for each term in the sum / position in the case of the respective matrix. Finally, a significant reduction in the number of operations performed by the CPU is obtained, in other words, it no longer needs to calculate multiplications involving N, of the form i * N. This technique was applied to all functions that return partial results, such as B * A', A * B * A', B' * B'. Last but not least, all information necessary for multiplications is stored in CPU registers with the aim of eliminating the overhead of accessing memory areas for variables used in this process. Given that the nehalem partition is 64-bit, there are enough registers to avoid any latencies due to reg <-> reg switches.

These are the running times:
```
N = 400: Time = 0.276390
N = 520: Time = 0.576796
N = 640: Time = 1.524378
N = 760: Time = 1.806661
N = 880: Time = 2.721285
N = 1000: Time = 3.941125
N = 1120: Time = 6.827338
N = 1240: Time = 7.688742
N = 1360: Time = 11.320905
N = 1480: Time = 13.950172
N = 1520: Time = 16.152508
N = 1580: Time = 18.188416
N = 1590: Time = 20.713333
N = 1600: Time = 23.733250
```

*Cachegrind Analysis*

It is observed that the BLAS implementation has the lowest number of *I_refs* among all methods, suggesting high performance, through the high rate of cache hits, for that library. Given that the Neopt method has the highest *I_refs*, it can be said that it is also the most inefficient. Regarding *D_refs*, it can be observed that the BLAS program performs the fewest memory accesses, which would lead to more efficient use of the cache. The optimized Opt_m version manages to make fewer accesses compared to Neopt, in other words, the effect of optimization is visible. Obviously, the previous observations remain valid for *LL_refs* as well, given that it indicates the number of 64-bit data writes or reads from memory. Last but not least, it is found that, again, the BLAS version has the lowest number of Branches, suggesting simpler flow control that can lead to better branch prediction and reduction of pipeline stalls. In contrast, Neopt and Opt_m have approximately the same high number of Branches indicating poorer prediction and more stalls leading to performance degradation.

*Graph Study*

Analyzing the graphs, it is easily observed that the BLAS version is the most performant of all, efficiently using the cache in the case of traversing matrices in blocks. In the case of the Neopt version, the shape of the graph can be visualized, which admits, approximately, the shape of an exponential for sizes exceeding the value of 900.

### Resources Used

3: https://netlib.org/blas/

4: https://valgrind.org/docs/manual/cg-manual.html

5: https://stackoverflow.com/questions/1907557/optimized-matrix-multiplication-in-c
