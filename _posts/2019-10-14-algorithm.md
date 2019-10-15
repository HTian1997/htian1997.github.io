---
title: 'Algorithm - Coursera'
date: 2019-10-14
---


Course website [here](https://www.coursera.org/learn/algorithms-part1)

### Week 1 assignment

Question [here](https://coursera.cs.princeton.edu/algs4/assignments/percolation/specification.php)

1. Percolation.java

```java
import edu.princeton.cs.algs4.WeightedQuickUnionUF;

public class Percolation {
    private int topRoot = 0;
    private int bottomRoot = 0;
    private int openSites = 0;
    private int dimension;
    private WeightedQuickUnionUF unionFind;
    private WeightedQuickUnionUF fullsites;
    private boolean[][] sites;

    // creates n-by-n grid, with all sites initially blocked
    public Percolation(int n){
        if(n < 1)
            throw new IllegalArgumentException();
        dimension = n;
        unionFind = new WeightedQuickUnionUF(n * n + 2);
        fullsites = new WeightedQuickUnionUF(n * n + 1);
        topRoot = n * n;
        bottomRoot = n * n + 1;
        sites = new boolean[n][n];
    }

    // opens the site (row, col) if it is not open already
    public void open(int row, int col){
        inRange(row, col);

        if(isOpen(row, col))
            return;

        int index = getIndex(row, col);

        if(row == 1){
            unionFind.union(index, topRoot);
            fullsites.union(index, topRoot);
        } else{
            unionIfOpen(row - 1, col, index);
        }

        if (row == dimension){
            unionFind.union(index, bottomRoot);
        } else{
            unionIfOpen(row + 1, col, index);
        }

        if (col < dimension){
            unionIfOpen(row, col + 1, index);
        }

        if (col > 1){
            unionIfOpen(row, col - 1, index);
        }

        sites[row - 1][col - 1] = true;
        openSites++;
    }

    // is the site (row, col) open?
    public boolean isOpen(int row, int col){
        inRange(row, col);
        return sites[row - 1][col - 1] == true;
    }

    // is the site (row, col) full?
    public boolean isFull(int row, int col){
        inRange(row, col);
        int index = getIndex(row, col);
        return fullsites.connected(index, topRoot);
    }

    // returns the number of open sites
    public int numberOfOpenSites(){
        return openSites;
    }

    // does the system percolate?
    public boolean percolates(){
        return unionFind.connected(topRoot, bottomRoot);
    }

    private void inRange(int row, int col){
        if (row < 1 || row > dimension || col < 1 || col > dimension)
            throw new IllegalArgumentException();
    }

    private int getIndex(int row, int col){
        return dimension * (row - 1) + col - 1;
    }

    private void unionIfOpen(int row, int col, int indexUnion){
        if(isOpen(row, col)){
            int index = getIndex(row, col);
            unionFind.union(index, indexUnion);
            fullsites.union(index, indexUnion);
        }
    }

}
```

2. PercolationStats.java
```java
import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;
import edu.princeton.cs.algs4.StdOut;

public class PercolationStats {

    private static final double confidence = 1.96;
    private double[] threshold;


    // perform independent trials on an n-by-n grid
    public PercolationStats(int n, int trials){
        if (n < 1 || trials < 1)
            throw new IllegalArgumentException();

        threshold = new double[trials];

        for(int i = 0; i < trials; i++){
            Percolation percolation = new Percolation(n);
            while(!percolation.percolates()){
                int row = StdRandom.uniform(n) + 1;
                int col = StdRandom.uniform(n) + 1;
                percolation.open(row, col);
            }

            threshold[i] = percolation.numberOfOpenSites() / (double) (n * n);
        }
    }

    // sample mean of percolation threshold
    public double mean(){
        return StdStats.mean(threshold);
    }

    // sample standard deviation of percolation threshold
    public double stddev(){
        return StdStats.stddev(threshold);
    }

    // low endpoint of 95% confidence interval
    public double confidenceLo(){
        return mean() - (confidence * stddev() / Math.sqrt(threshold.length));
    }

    // high endpoint of 95% confidence interval
    public double confidenceHi(){
        return mean() + (confidence * stddev() / Math.sqrt(threshold.length));
    }

    // test client (see below)
    public static void main(String[] args){

        if (args.length != 2){
            throw new IllegalArgumentException();
        }

        int dimension = Integer.parseInt(args[0]);
        int trials = Integer.parseInt(args[1]);
        
        PercolationStats stats = new PercolationStats(dimension, trials);

        StdOut.println("mean                    = " + stats.mean());
        StdOut.println("stddev                  = " + stats.stddev());
        StdOut.println("95% confidence interval = [" + stats.confidenceLo() + ", " + stats.confidenceHi() + "]");
    }

}
```

