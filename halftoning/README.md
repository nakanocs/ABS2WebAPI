# Benchmark QUBO problems for digital halftoning #
Problems are generated from gray scale images flower.png and flower-k.png (256x256 pixels).
The original grayscale image was downloaded from unsplash.com, which offers free photos for commercial and non-commercial purposes.

## Two QUBO problems:
*  **flower.json.gz**: 65536-bit QUBO problems with unknown optimal solution.
    Problem files: flower.json.gz/flower.mm.gz   (two files store the same QUBO problem)

* **flower-k.json.gz**: 65536-bit QUBO problems with a known optimal solution. 
Optimal solution is **flower-k.optimal.json** with energy -225466781.


These QUBO problems are designed to obtain binary images that reporduce gray scale images **flower.png** and **flower-k.png**.
By arranging 65536-bit solution in a 256x256 matrix such that 0/1 are black/white pixels, we can obtain such binary images.

