---
title: Algorithm学习笔记
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 广阔天地，大有所为。好少年光芒万丈。
authorDesc: 广阔天地，大有所为。好少年光芒万丈。
categories: 技术
comments: true
mathjax: true
date: 2019-09-14 13:23:48
tags:
    - cousera
    - 技术
    - 学习笔记
keywords: cousera学习笔记
description: Cousera Algorithm Part I 学习笔记
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/algorithm_I.png
---
<style>
table, th, td{
  border: 1px solid #1e1b26;
}

th{
  background: #CCEEFF;
  color: #FBFBEF;
}
</style>

本文系Cousera Algorithm I学习笔记， 课程网址为：https://www.coursera.org/learn/algorithms-part1/home/welcome
# Union-Find
Introduce the union–find data type and consider several implementations (quick find, quick union, weighted quick union, and weighted quick union with path compression).Finally, apply the union–find data type to the percolation problem from physical chemistry.

## Quick-Find
```java
public class QuickFindUF{
    private int[] id;
    public QuickFindUF(int N){
        id = new int[N];
        for(int i = 0; i < N; i++){
        id[i] = i;
        }
    }
    
    public boolean connected(int p, int q){
        return id[p] == id[q];
    }
    
    public void union(int p, int q){
        int pid = id[p];
        int qid = id[q];
        for(int i = 0; i < id.length; i++){
            if(id[i] == pid){
                id[i] = qid;
            }
        }
    }
}
```
| Algorithm | initialize | union | find |
| :----: | :----: | :----: | :----: |
| quick-find | N | N | 1|

Union is too expensive. It takes $N^2$ array access to process a sequence of N union commands on $N$ objects. 
Quadratic algorithm do not scale.
 
## Quick Union
```java
public class QuickUnionUF{
    private int[] id;
    public QuickUnionFindUF(int N){
        id = new int [N];
        for(int i = 0; i < id.length; i++){
            id[i] = i;
        }
    }
    
    private int root(int i){
        while(i!= id[i]){
            i = id[i];
        }
        return i;
    }
    
    public boolean connected(int p, int q){
        return root(p) == root(q);
    }
    
    public void union(int p, int q){
        int i = root(p);
        int j = root(q);
        id[i] = j;
    }
}
```
connect the root each time.

| Algorithm | initialize | union | find |
| :----: | :----: | :----: | :----: |
| quick-find | N | N | 1|
| quick-union | N | N | N |

Quick-find defect.

 - Union too expensive (N array accesses).
 - Trees are flat, but too expensive to keep them flat.
Quick-union defect. 

 - Trees can get tall.
 - Find too expensive (could be N array accesses).
## Improvement
### weighted quick-union
The root of the smaller tree goes below the root of the bigger tree. Additional array to store the size of each tree.
 
 ```java
    # only change the union function from the quick-union
    public void union(int p, int q){
        int i = root(p);
        int j = root(q);
        if(i == j) return;
        if(sz[i] < sz[j]){
            id[i] = j;
            sz[j] += sz[i];
        }else{
            id[j] = i;
            sz[i] += sz[j];
        }
    }
 ```
 
 Running time : 
 
  - Find : takes proportional to depth pf p and q.
  - Union : takes constant time, given root.
The size of the tree containing x at least doubles and can double at most $lgN$ times.

| Algorithm | initialize | union | find |
| :----: | :----: | :----: | :----: |
| quick-find | N | N | 1|
| quick-union | N | N | N |
| weighted-QU | N | $lgN$ | $lgN$ |

### path compression (WQUPC)
Just after computing the root of p, set the id of each examined node to point to that root. 
Implementation:

```java
private int root(int i){
    while (i != id[i]){
        id[i] = id[id[i]];
        i = id[i];
    }
    return i;
}
```
Halving the length of the path.
In theory, WQUPC is not linear. In practice, WQUPC is linear.

## Summary
*M union-find operations on a set of N objects*


| Algorithm | worst-case time |
| :----: | :----: |
| quick-find | $MN$ |
| quick-union | $MN$ |
| weighted QU | $N + M log N$ |
| QU + path compression | $N + M log N$ |
| weighted QU + path compression | $N + M lg* N$ |

## Practice Interview Question
### Question 1
 Social network connectivity : 
Given a social network containing $n$ members and a log file containing $m$ timestamps at which times pairs of members formed friendships, design an algorithm to determine the earliest time at which all members are connected (i.e., every member is a friend of a friend of a friend ... of a friend). Assume that the log file is sorted by timestamp and that friendship is an equivalence relation. The running time of your algorithm should be $mlogn$ or better and use extra space proportional to $n$.

```java
    class solution{
        //The input is 2d array, which contains [timestamp, friend1, friend2], there are N entries in total.
        int[] id;
        public int earliestTime(int[][]logs, int N){
            id = new int[N];
            for(int i= 0; i < N; i++){
                id[i] = i;
            }
            for(int[] log : logs){
                if(!connected(log[1],log[2])){
                    union(log[1],log[2]);
                    N--;
                }
                if(N == 1){
                    return log[0];
                }
            }
            return -1;
        }
        private boolean connected(int p, int q){
            return root(p) == root(q);
        }
        private int root(int i){
            while(i != id[i]){
                id[i] = id[id[i]];
                i = id[i];
            }
            return i;
        }
        private void union(int p, int q){
            int p_id = root(p);
            int q_id = root(q);
            id[p_id] = q_id;
        }
    }
```
### Question 2
Union-find with specific canonical element. Add a method 𝚏𝚒𝚗𝚍() to the union-find data type so that 𝚏𝚒𝚗𝚍(𝚒) returns the largest element in the connected component containing ii. The operations, 𝚞𝚗𝚒𝚘𝚗(), 𝚌𝚘𝚗𝚗𝚎𝚌𝚝𝚎𝚍(), and 𝚏𝚒𝚗𝚍() should all take logarithmic time or better. 
For example, if one of the connected components is $\{1, 2, 6, 9\}$, then the 𝚏𝚒𝚗𝚍() method should return $9$ for each of the four elements in the connected components.

### Question 3
Successor with delete. Given a set of $n$ integers $S = \{ 0, 1, ... , n-1 \}$ and a sequence of requests of the following form:
Remove $x$ from $S$
Find the successor of xx: the smallest $y$ in $S$ such that $y \ge x$.
design a data type so that all operations (except construction) take logarithmic time or better in the worst case.

