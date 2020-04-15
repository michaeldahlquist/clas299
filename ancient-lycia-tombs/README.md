# Ancient Lycia Necropoleis (Southwestern Turkey Cemetery Site)

There’s a [Jupyter notebook](https://mybinder.org/v2/gh/michaeldahlquist/clas299/master?filepath=ancient-lycia-tombs%2Fancient-lycia-tombs.ipynb) to run the following blocks of scala code. 

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/michaeldahlquist/clas299/master?filepath=ancient-lycia-tombs%2Fancient-lycia-tombs.ipynb)

# Ancient Lycia Necropoleis 

## Southwestern Turkey Cemetery Site

In ancient Lycia, rock-cut tombs often clustered together in cemetery sites, or necropoleis, like ancient Myra (modern Demre), illustrated above. This data set contains data about Lycian necropoleis including the number of tombs at each site. You will figure out how many total tombs are represented in the data set. The records describe for each site its name, a typological classification by the Danish scholar Jan Zahle, the number of tombs, an English-language summary, and an identifier in a geographic data set.

### The Data Set

The dataset is available as a delimited-text file [here](https://raw.githubusercontent.com/michaeldahlquist/clas299/master/ancient-lycia-tombs/lycianNecropoleis.cex). The format is one record per row, and columns are delimited by the pound sign (hash tag) `#`. The file includes a header row:

`sitename#ztype#tombcount#comments#ztypetext#rageid`

The records describe for each site its name, a typological classification by the Danish scholar Jan Zahle, the number of tombs, an English-language summary, and an identifier in a geographic data set.

### Retrieve the data set

To download the data set, you can use the Scala `Source` object. We need to import its class:

`import scala.io.Source`


```scala
import scala.io.Source
val lycianNecropoleis = "https://raw.githubusercontent.com/michaeldahlquist/clas299/master/ancient-lycia-tombs/lycianNecropoleis.cex"
```

We then extract a sequence of lines from the URL source, and convert them to a vector.


```scala
val lines = Source.fromURL(lycianNecropoleis).getLines.toVector
```

### Extract the numeric count of tombs

You should now have a Vector of Strings. You want to split up each String on the `#` character, to create a new Vector – this time, a Vector of Vectors. You’ll be mapping each line of the source data to a Vector of strings, one per column.

We also will take the `tail` of the vector as the header is not part of the data.


```scala
val data = lines.tail.map(ln => ln.split("#").toVector)
```

The `tombcount` is in the third column (index number 2) of each record. Now we need to create a new Vector that contains only the tomb count.


```scala
val tombData = data.map(columns => columns(2))
```

But the result is a Vector of Strings and we need a Vector of integer values. We can use the String class’s `toInt` method to create an `Int` from a `String`.


```scala
val tombCounts = tombData.map(s => s.toInt)
```

Now we can use the `sum` method that will handily sum up a Vector of numeric values.


```scala
tombInts.sum
```

## There are 1085 tombs