---
layout: page
title: Flip's Blog
tagline: Flip's Blog voor de Big Data course
---

Dit is mijn blog voor Big Data

**Assignment 2**

For this assignment I've managed to setup a HDFS and run a couple of different Map-Reduce algorithms. Here I will briefly explain the process and the results.

**Running HDFS as a standalone:**
During this part of the assignment I ran a simple algorithm on all of Hadoops .xml files. The algorithm lists all the strings that matched the submitted regular expression 'dfs[a-z.]+' (=all words beginning with 'dfs'). The only word that matched this regex was dfsadmin.

**Running HDFS as a pseudo-distributed operation:** Unfortunately I was unable to successfully establish a connection with the separate Hadoop daemon, and thus was unable to successfully complete this part.

**Using MapReduce on the Complete Shakespeare:** During the last part of this assignment I've run a WordCount MapReduce algorithms on a text file containing all of Shakespeare's work. This algorithm maps every word it encounters, the mapped words are then tallied together to compile a list of all of the words in the text and their respective occurrences. This mapping yielded 156 occurrences of Romeo, 153 occurrences of Julia and most importantly 335 occurrences of Othello. These values were attained by searching for strings matching the regex'es of Romeo, Julia and Othello. This was necessary because the WordCount algorithm distinguishes between strings such as 'Romeo' and 'Romeo.'.

# Assignment 3: Parsing Magic The Gathering Card Data

For this assignment we will be taking a look at magic the gathering cards. Magic is a big hobby of mine, and I thought that would be interesting to combine it with this assignment.

The carddata has been pulled from https://github.com/mtgjson/mtgcsv.git.
However the dataset is not up to date and thus does not contain the cards from last year.

The dataset contains different .csv files, each listing the cards contained in their respective set. We can pull this data from the git and to make sure that we have de .csv files installed, we execute the following command, which should return a long list of .csv files.
```scala
:sh ls /opt/docker/notebooks/BigData/mtgcsv/csv
```
Now that we have the .csv files installed, we have to do some preparations before we can get on to the fun part.
```
import org.apache.spark.sql.types._ 

val spark = SparkSession
   .builder()
   .appName("A3b-spark-df")
   .getOrCreate()
```
Now we want to cache all those beautifully formated .csv files so that we get one single dataset containing all the cards.
```
val cardData = spark.read.format("csv").option("header", true).load("/opt/docker/notebooks/BigData/mtgcsv/csv/*.csv").cache()
cardData.show(3, false)
```
We now have a list containing all the magic cards up to 2016. However Magic cards get reprinted, this means that the same card might show up in different sets, thus we will remove all duplicate cards.
```
val cd = cardData.dropDuplicates("name").cache
```
Now we have a dataset that is a perfect starting ground for experimentation.

## Experiment 1: Distribution of evergreen keywords across color identity
In Magic, *evergreen* keywords are keywords that will return (almost) every set.
The evergreen abilities that we will be looking at are: Deathtouch, Defender, Double Strike, First Strike, Flash, Flying, Haste, Hexproof, Indestructible, Lifelink, Menace, Prowess, Reach, Trample and Vigilance.
What I am interested in is the distribution of these keywords among colors, and whether some colors have an *affinity* for certain abilities.
```
//We want a list of all the keywords
val keyWords = Seq("Deathtouch", "Defender", "Double strike", "First strike", "Flash", "Flying", "Haste", "Hexproof", "Indestructible", "Lifelink"
                   , "Menace", "Prowess", "Reach", "Trample", "Vigilance")

var distributions : List[Tuple6[String, Integer, Integer, Integer, Integer, Integer]] = List()
var keyWord = ""
for(keyWord <- keyWords){
  val distribution = cd.filter(col("text").like("%_"+keyWord.substring(1)+"%")).cache()
  
  val totW : Integer = distribution.filter(col("colorIdentity").contains("W")).count().toInt
  val totU : Integer = distribution.filter(col("colorIdentity").contains("U")).count().toInt
  val totB : Integer = distribution.filter(col("colorIdentity").contains("B")).count().toInt
  val totR : Integer = distribution.filter(col("colorIdentity").contains("R")).count().toInt
  val totG : Integer = distribution.filter(col("colorIdentity").contains("G")).count().toInt
  val keywordDist = (keyWord, totW, totU, totB, totR, totG)
  distributions = distributions :+(keywordDist)
  println(keyWord)
}

```
Now that we have the distributions of the keywords, the only thing that remains for this experiment is showing the data in a readable fashion.
```
val distribution = null
for (distribution <- distributions){
  val total = distribution._2 + distribution._3 + distribution._4 + distribution._5 + distribution._6
  
  println(distribution._1+ ": " + (distribution._2 * 100 / total) + "%, " + (distribution._3 * 100 / total) + "%, "
         + (distribution._4 * 100 / total) + "%, " + (distribution._5 * 100 / total) + "%, " + (distribution._6 * 100 / total) + "%")
}
```
## Experiment 2: Finding the highest scry value
Another keyword in Magic is the keyword 'scry N'; a keyword that allows you to look at the top N cards of your deck, and decide which ones you want to keep at the top of your deck and which ones you want to put at the bottem of your deck. The higher the amount of cards you can scry, the more control you have over which cards you will draw.
What I want to find out is what the highest amount is that you can *scry*.
```
val cdScry = cd.filter(col("text").like("%_cry%")).cache()
```
Now that we have the list of cards with the *scry* keyword on them, we need to find a way to get the highest amount a card allows you to scry.
```
var continue = true
var scryN :DataFrame = null
var n = 0
while(continue==true){
  n += 1
  scryN = cdScry.filter(col("text").like("%_cry "+n+"%"))
  continue = (scryN.count() > 0)
  print("I")
}
print(n)
```
According to our approach the highest value you can scry is 6.
