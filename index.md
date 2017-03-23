---
layout: page
title: Flip's Blog
tagline: Flip's Blog voor de Big Data course
---

This is my Big Data Blog.

**Assignment 2**

For this assignment I've managed to setup a HDFS and run a couple of different Map-Reduce algorithms. Here I will briefly explain the process and the results.

**Running HDFS as a standalone:**
During this part of the assignment I ran a simple algorithm on all of Hadoops .xml files. The algorithm lists all the strings that matched the submitted regular expression 'dfs[a-z.]+' (=all words beginning with 'dfs'). The only word that matched this regex was dfsadmin.

**Running HDFS as a pseudo-distributed operation:** Unfortunately I was unable to successfully establish a connection with the separate Hadoop daemon, and thus was unable to successfully complete this part.

**Using MapReduce on the Complete Shakespeare:** During the last part of this assignment I've run a WordCount MapReduce algorithms on a text file containing all of Shakespeare's work. This algorithm maps every word it encounters, the mapped words are then tallied together to compile a list of all of the words in the text and their respective occurrences. This mapping yielded 156 occurrences of Romeo, 153 occurrences of Julia and most importantly 335 occurrences of Othello. These values were attained by searching for strings matching the regex'es of Romeo, Julia and Othello. This was necessary because the WordCount algorithm distinguishes between strings such as 'Romeo' and 'Romeo.'.
