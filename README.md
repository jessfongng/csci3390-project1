# Large Scale Data Processing: Project 1
## Author: Jessica Fong Ng & Qingwei Meng
Project 1 report of Large Scale Data Processing class at Boston College. The code had been modify from [project 1 assignment description](https://github.com/CSCI3390/project_1). 

## Bitcoin mining - SHA
The program takes in: Bitcoin heading string `S`, difficulty of the puzzle `k` and number of trials `n`. It will attempt to find a positive integer `x` ("nonce") such that the concatenation `xS` is hashed to a hexadecimal number with `k` leading zeros.<br/>
First validate your environment by:<br/>   
```sbt clean package
   spark-submit --class project_1.main --master local[*] target/scala-2.12/project_1_2.12-1.0.jar
```
The following message should appear<br/>   
`Usage: project_1 string difficulty #trials`<br/>   
After that, you can run the code by appending the `S`, `k` and `n` variable.<br/>  
```
// Linux  
spark-submit --class project_1.main --master local[*] target/scala-2.12/project_1_2.12-1.0.jar this_is_a_bitcoin_block_of_12345678 2 100    
   
// Unix  
spark-submit --class "project_1.main" --master "local[*]" target/scala-2.12/project_1_2.12-1.0.jar this_is_a_bitcoin_block_of_12345678 2 100
```
Please reference the specific project description and instruction [here](https://github.com/CSCI3390/project_1).
## Report Finding
### 1. Run the program on local machine with `k = 2,3,4,5,6`, the result is listed in a table below.

k | total time of elapsed (s) | number of trials | xS | hash value 
------------| ------------|------------|------------|------------
2 | 2 | 100 | `1209735597this_is_a_bitcoin_block_of_62425405_11160090` | `001142511ea768b76db39b89813e53f72c2d262af548edd20d768cdfae8294fe`
3 | 2 | 1000 |`92477141this_is_a_bitcoin_block_of_62425405_11160090` | `000a6b6c7c318d154b2b0a533416b5fcdd1157848910f453ac7268751250eb46`
4 | 2 | 50000 | `566719889this_is_a_bitcoin_block_of_62425405_11160090` | `0000376df8a03266e2f09f99d1b1dca6dc5bc0b2d162fb34c52c22afd433432d`
5 | 3 | 1000000 | `1919540112this_is_a_bitcoin_block_of_62425405_11160090` | `00000458b7edf6753a8fbf41f0eadfbe0bd31e9f9c9dea18f39b3d33c981f0b8`
6 | 27 | 20000000 | `1051875242this_is_a_bitcoin_block_of_62425405_11160090` | `000000124f5fd2f46a862eff988e48eacdcddcd2126ea4f22d4477f790532860`


### 2. Run the program on GCP to solve the case k = 7.
k | total time of elapsed (s) | number of trials | xS | hash value 
------------| ------------|------------|------------|------------
7 | 369|0.1 billion|`227809291this_is_a_bitcoin_block_of_62425405_11160090`|`0000000ca1f8ccdbd8b83b6701b5139c28ded54bd917b1abbd08a2c9e41af967`

*Cluster Configuration*: 3 machines and 12 cpus.   
Based on the result above, we discover that we need between <img src="https://render.githubusercontent.com/render/math?math=10^{k}"> to <img src="https://render.githubusercontent.com/render/math?math=10^{k %2B2}"> trials in order to find the solution, so we tried <img src="https://render.githubusercontent.com/render/math?math=10^{8}"> trials to see if the solution can be found, and it worked.

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
The table below show the total time of elapsed (s) between random nonce and ranged nonce. 
k | #trial |  random nonce total time of elapsed (s) | ranged nonce total time of elapsed (s)
------------|------------|------------|------------
2 | 20000 | 3| 4
2 | 50000 | 3| 6
2 | <img src="https://render.githubusercontent.com/render/math?math=10^{5}"> | 4| 7
2 | <img src="https://render.githubusercontent.com/render/math?math=10^{7}"> | 27| 25

The table below shows the numbers of found hash found over 10000 iterations of the code. 
k | #trial |  #random nonce | time for random nonce (s) | #ranged nonce |  time for ranged nonce (s)
------------|------------|------------|------------|------------|------------
2 | 10 | 372| 162 | 0 | 156
3 | 100 |24| 201|  0 | 178 

As the first table shows that the elapsed time over large number of trial. The randomize nonce has lower computing time overall. The second table shows that with low numbers of trial, the randomize nonce is more likely to find the `xS` than ranged nonce. Thus, the approach to generate the nonce using the range number is less effective than randomly. Random nonce covers a bigger range than using range number. If the range number is small, then it is challenging to find `xS` that finds the criteria.  
