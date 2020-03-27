# Archaeological Data Analysis: Coin issuing of the Roman Empire

### Author:  Michael Dahlquist

# Exploring a data set

In this notebook, you'll download a data set derived from the openly licensed content of the [Online Coins of the Roman Empire](http://numismatics.org/ocre/) (OCRE). The original data set is available from <http://nomisma.org/> RDF XML format.  We'l work with a version formatted as a delimited-text file, using `#` as the column delimiter, with a header line labelling each column.

As with any data set, our first task is to figure out what kinds of data it contains, and what the range of values are for each category of data. We'll examine the contents of several columns of data.




## Download delimited-text data

We'll make the standard Scala `Source` object available by `import`ing it, then use it to retrieve the content of a URL.


```scala
import scala.io.Source
val ocreCex = "https://raw.githubusercontent.com/neelsmith/nomisma/master/cex/ocre-cite-ids.cex"
```

We'll extract a sequence of lines from the URL source, and convert them to our favorite type of Scala collection, a `Vector`.

(The following cell downloads the data:  depending on your internet connection, this might take a moment.)


```scala
val lines = Source.fromURL(ocreCex).getLines.toVector
```

## Examine header line

To start with, let's see what the first line looks like, and compare it with the first data line.


```scala
lines.head // same as lines(0)
```


```scala
lines(1)
```

## Split data strings into columns

Every line is a `String`.  If we break it up using the `split` method, we get an `Array` of `String`s, which we'll convert to a `Vector` of `String`s.  The end result will be that from a Vector of Strings, we create a Vector of Vectors of Strings.  Notice that Scala identifies the class of the new `data` expression as  `Vector[Vector[String]]`.
 


```scala
val data = lines.tail.map(ln => ln.split("#").toVector)
```

Mapping each Vector to the first item in the Vector is equivalent to extracting the first column from each Vector.  The header line told us that the first column should contain ID values.


```scala
val ids = data.map(columns => columns(0))
```

We want to be sure that all ID values are unique.  We can verify that by comparing the number of items in the `ids` Vector with the number of *distinct values* in the `ids` Vector.  If they're the same, then every value is unique.


```scala
//println("Records: " + ids.size)
//println("Distinct IDs: " + ids.distinct.size)
if(ids.size == ids.distinct.size) {
    println("All records uniquely identified.")
} else {
    println("Duplicate identifiers in data set.")
}
```

## Distribution of denominations

Let's look at how coin denominations are described.  You can see from the header line that denominations are in the third column, so we'll map each Vector to the thrid column -- and remember that we start indexing with 0, so the third column is indexed as `(2)`.


```scala
val denominations = data.map(columns => columns(2))
```

We'll use a very handy Scala idiom to count how many times each authority appears. If we group the elements in our Vector by their value, the result is a Map from the unique set of values to a list of the matching values.  


```scala
val denominationsGrouped = denominations.groupBy(denom => denom)
```


```scala
// Free puzzle:  notice that the result of this groupBy should be the same size 
//               as the numnber of distinct values in our list:
if (denominationsGrouped.size == denominations.distinct.size) {
    println("Number of groups is same as number of distinct values.")
} else {
    print("Something is terribly wrong.  The number of groups ")
    println("is not the same as the number of distinct values.")
}
```

What we really want to know is *how many times* does each denomination appear?  We can find that out by transforming our mapping of String->Vector[String] to give us a mapping of each denomination to the *size* of the Vector of its occurrences.





```scala
val denominationsCounts = denominationsGrouped.map{ case (d, v) => (d, v.size) }
```

Recall that `Map`s are not ordered in Scala. If we now convert the `Map` to a `Vector`, we will have a Vector pairing a String with an Int.  We can sort the Vector by the second element of the pairing (which will sort from smallest to largest), then reverse the results to have a descedning list of how often each denomination occurs.


```scala
val denominationsVector = 
    denominationsCounts.toVector
val denominationsHisto = 
    denominationsCounts.toVector.sortBy(frequency 
                                        => frequency._2).reverse
```

Now we can easily see the extremes of the counts:


```scala
println("Most frequent denomination: " + denominationsHisto.head)
```


```scala
// Find denominations occurring fewer than some threshhold number of times
val cutOff = 10 
val leastDenominations = 
    denominationsHisto.filter(frequency => frequency._2 < cutOff)
println("Least frequent denominations: \n" + leastDenominations.mkString("\n"))
```

## Assignment


Analyze how many issues are produced by each issuing authority to answer the following questions:

- How many different authorities strike coins in OCRE's data?
- Who strikes the greatest number of issues?  How many?
- What is the smallest number of issues struck by a single authority?

### Gather and organize your data


```scala
// First, to extract the "Authority" column from the data set, uncomment 
// and complete the following line:
val authorities = data.map(columns => columns(4))
```

### Question 1: how many authorities strike coins?




```scala
// Use the distinct method and size method to count 
// how many distinct values you have in `authorities`
authorities.distinct.size
```

### Group records by authority and count them


```scala
// use the groupBy method to group each auhority by the authority value.
// This will give you a Map of Strings to a Vector of Strings
val authoritiesGrouped = authorities.groupBy(authority => authority)
```


```scala
// now convert each pairing of String->Vector[String] to a String->Int counting 
// how many elements are in the original Vector.
// The result is a Map[String->Int].
val authoritiesCounts = authoritiesGrouped.map{ case (auth,v) => (auth, v.size)}
```


```scala
// next convert your Map[String->Int] to a Vector.  The result is a 
// Vector of pairings of (String, Int).
// We'll sort this by the second element of the pairing, namely the Int.  
// Since we sort from smallest to largest
// by default, you can reverse the result so that the 
val authoritiesHistogram = authoritiesCounts.toVector.sortBy(auth => auth._2).reverse
```

### Questions 2 and 3: who strikes the most issues? the fewest?


```scala
// With the authoritiesHistogram you created, you can use the `head` and 
// `last` methods to see the first and last entries in the Vector.
authoritiesHistogram.head
authoritiesHistogram.last
authoritiesHistogram.filter{freq => freq._2 == 1}//all that have 1
```
