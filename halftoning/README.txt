- Benchmark QUBO problems for digital halftoning -

Problems are generated from a gray scale images flower.png and flower-k.png (256x256 pixels).
The original grayscale image was downloaded from unsplash.com, which offers free photos for commercial and non-commercial purposes.

Two QUBO problems:
(1) flower: 65536-bit QUBO problems with unknown optimal solution.
    Problem file: flower.json.gz

(2) flower-k: 65536-bit QUBO problems with a known optimal solution.
    Problem file: flower.json.gz
    Optimal solution: -225466781

These QUBO problems are designed to obtain binary images that reporduce flower.png/flower-k.png.
By arranging 65536-bit solution in a 256x256 matrix such that 0/1 are black/white, we can obtain such binary images.

