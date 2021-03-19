# Large Scale Data Processing: Project 1
## Author: Jessica Fong Ng & Qingwei Meng
Project 1 report of Large Scale Data Processing class at Boston College. The code had been modify from [project 1 assignment description](https://github.com/CSCI3390/project_1). 

## Project Description
  
## Report Finding
### 1. Run the program on local machine with `k = 2,3,4,5,6`, the result is listed in a table below.

k | total time of elapsed (s) | number of trials | xS | hash value 
------------| ------------|------------|------------|------------
2 | 2 | 100 | 1209735597this_is_a_bitcoin_block_of_62425405_11160090 | 001142511ea768b76db39b89813e53f72c2d262af548edd20d768cdfae8294fe
3 | 2 | 1000 |92477141this_is_a_bitcoin_block_of_62425405_11160090 | 000a6b6c7c318d154b2b0a533416b5fcdd1157848910f453ac7268751250eb46
4 | 2 | 50000 | 566719889this_is_a_bitcoin_block_of_62425405_11160090 | 0000376df8a03266e2f09f99d1b1dca6dc5bc0b2d162fb34c52c22afd433432d
5 | 3 | 1000000 | 1919540112this_is_a_bitcoin_block_of_62425405_11160090 | 00000458b7edf6753a8fbf41f0eadfbe0bd31e9f9c9dea18f39b3d33c981f0b8
6 | 27 | 20000000 | 1051875242this_is_a_bitcoin_block_of_62425405_11160090 | 000000124f5fd2f46a862eff988e48eacdcddcd2126ea4f22d4477f790532860


### 2. Run the program on GCP to solve the case k = 7.
k | total time of elapsed (s) | number of trials | xS | hash value 
------------| ------------|------------|------------|------------

*Cluster Configuration* is 1 Master, 3 workers. Based on the result above, we discover that we need between 10^k to 10^{k+1} trials in order to find the solution, so we tried 10^7 trials to see if the solution can be found, and it worked.

### 3. Program generates the potential nonce from 1 to `n` (the number of trials) instead of randomly
The original lines of code that generate the potential nonce: 
```
    val nonce = sc.range(0, trials).mapPartitionsWithIndex((indx, iter) => {
      val rand = new scala.util.Random(indx + seed)
      iter.map(x => rand.nextInt(Int.MaxValue - 1) + 1)
    })
```
Our modification to generate potential nonce from 1 to `n`:
```
    val nonce = sc.range(0, trials).mapPartitionsWithIndex((indx, iter) => {
      val rand = new scala.util.Random(indx + seed)
      iter.map(x => rand.nextInt(trials) + 1)
    })
```

