<center>
<a href="https://michaeldahlquist.github.io/clas299">
<img border="0" alt="Home" src="https://raw.githubusercontent.com/michaeldahlquist/clas299/master/images/home.png" width="100" height="100">
</a>
</center>

# Jupyter notebook for Ptolemny's Spatial Analysis scala script for csv

This notebook contains scala scripts that analyze ptolemys spatial data, and corrects it to modern day standard longitude and latitutde coordinates. This data is then formatted into a csv for export.

There’s a [Jupyter notebook](https://mybinder.org/v2/gh/michaeldahlquist/clas299/master?filepath=ptolemy%2Fptolemy.ipynb) to run the following blocks of scala code. 

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/michaeldahlquist/clas299/master?filepath=ptolemy%2Fptolemy.ipynb)

The resulting data is [available here.](https://github.com/michaeldahlquist/clas299/blob/master/ptolemy/ptolemy.csv)

***

# Analyzing Ptolemy's geographic data

## Overview

This notebook will help you adjust Ptolemy's values for longitude and latitude to account for:

- his mistakenly small dimension of the earth's circumference
- his origin of longitude (ca. 12.8 degrees west of Greenwich)
- his use of the "parallel through Rhodes" (36 degrees north latitude in Ptolemy's data) as the baseline for computing latitude values.

## Using your adjusted data in a GIS

You could use the contents of this notebook in several ways:

1. run as a Jupyter notebook directly (either on mybinder.org, or downloaded and run with software like nteract)
2. download the notebook as Scala, open the Scala content in Atom, and run the code directly there.

In either case, you will want to write your adjusted data to a `.csv` file you can use directly in a GIS.

A Jupyter notebook on mybinder won't have access to your host computer's file system, so you'll have to print out the values in your notebook, and copy and paste them in to a file on your computer.  If you're running the code in a local environment like Atom, you can write the output directly from your Scala code.  The instructions at the end of this notebook will show you how to do both of these things.

## Load data for Ptolemy

You can use an existing code library to read an XML edition of Ptolemy's *Geography* and extract the 6,000 geographic points into a class of object that will make them straightforward to work with.


```scala
// 1. Add maven repository where we can find our libraries
val myBT = coursierapi.MavenRepository.of("https://dl.bintray.com/neelsmith/maven")
interp.repositories() ++= Seq(myBT)

```


```scala
// 2. Make libraries available with `$ivy` imports:
import $ivy.`edu.holycross.shot::ptolemy:1.2.1`
import scala.xml._
```


```scala
// read and parse XML file of Ptolemy:
val url = "https://raw.githubusercontent.com/neelsmith/ptolemy/master/tei/tlg0363.tlg009.epist03-p5-u8.xml"
val root = XML.load(url)
```


```scala
// parse XML text into objects
import edu.holycross.shot.ptolemy._
val delimited = TeiParser.parseTEI(root, false)
val ptolemyPoints = delimited.map(ln => PtolemyString(ln))
```

Each of the `ptolemyPoints` objects has a `lon` and a `lat` member.

Look at the example of a single point in following cell to figure out what class the `lon` and `lat` members belong to.


```scala
val firstPoint = ptolemyPoints(0)
firstPoint.id
firstPoint.lon
firstPoint.lat
firstPoint.continent
firstPoint.province
```

## 1. Scale the data

As you know from your background reading, we will use the ratio of Eratosthenes' figure for the circumference of the earth to Ptolemy's figure to scale the longitude and latitude values down by about 72%.


```scala
val ratio = 18.0 / 25.0
// We'll take one arbitrary point as an example
// Here's an example:
firstPoint.lon
firstPoint.lon * ratio
firstPoint.lat
firstPoint.lat * ratio

```



To simplify your work, you could work just with the longitude and latitude values for each point.  Scala's case class is a natural way to accomplish that.

The following cell defines a class named `GeoPoint` that has three members, plus one function to format the contents as a comma-separated String. It shows how you can create instances of that class.


```scala
case class GeoPoint (id: String, lon: Double, lat: Double) {
    def csv : String = {
        id + "," + lon + "," + lat
    }
}


val firstGeo = GeoPoint(firstPoint.id, firstPoint.lon, firstPoint.lat)
firstGeo.csv
```

This makes it very straightforward to map the Vector of ptolemy points to a new Vector of `GeoPoint` objects.


```scala
val ptolemyGeo = ptolemyPoints.map(pt => GeoPoint(pt.id, pt.lon, pt.lat))
```

### Verify the size is the same


```scala
ptolemyGeo.size
ptolemyPoints.size
```

If we wanted to create a rescaled version of the first longitude, latitude pair, we could easily do that: 


```scala
val firstRescaled = GeoPoint(firstPoint.id, 
                             firstPoint.lon * ratio, 
                             firstPoint.lat * ratio)
firstPoint
```

### Task: create a Vector of rescaled points

Now create a Vector of `GeoPoint` objects.  Verify that you have the same number of them as the size of your original Vector of Ptolemy points.


```scala
val ptolemyRescaled = ptolemyGeo.map( pt => 
                                     GeoPoint(
                                       pt.id, 
                                       pt.lon * ratio, 
                                       pt.lat * ratio))
```

## 2. Adjust origin of longitude

Empirical comparison suggests that Ptolemy's origin of longitude was about 12.8 degrees west of Greenwich.

The following cell creates a `GeoPoint` adjusting Ptolemy's origin of longitude to align with our origin of longitude.



```scala
// negative because Ptolemy's 0 is *west* of Greenwich:
val originLongitude = -12.8
firstRescaled
val firstLonAdjusted = GeoPoint(firstRescaled.id, 
                                firstRescaled.lat, 
                                firstRescaled.lon + 
                                       originLongitude)
```

### Task: create a Vector of points with adjusted longitude



```scala
// Map your existing ptolemyRescaled Vector:

val ptolemyLonAdjusted = ptolemyRescaled.map( pt => 
                                             GeoPoint(
                                               pt.id, 
                                               pt.lat, 
                                               pt.lon + 
                                                originLongitude))
```

## 3. Adjust base of latitude

When Ptolemy converted ground distances to spherical coordinates, he did not use the equator (0 degrees of latitude) as his baseline to compute from.  Instead, he used "the parallel through Rhodes," which he gives as 36 degrees north of the equator.  But if we scale his raw value of 36 degrees by the ratio of 18/25, then the baseline he thought was 36 degrees north of the equator was actually less than 26 degrees north of the equator.  We need to *add* to each latitude value this difference (roughly 10 degrees) between the raw value of 36 degrees and the scaled-down value.

The following cell shows how to compute that offset value, and apply it to one point.


```scala
val rhodesRaw = 36.0
val rhodesAdjusted = ratio * rhodesRaw
val offset = rhodesRaw - rhodesAdjusted

```


```scala
val firstLonLatAdjusted = GeoPoint(firstLonAdjusted.id, 
                                   firstLonAdjusted.lat + 
                                        offset, 
                                   firstLonAdjusted.lon)
```

### Task: create a Vector of points with all three adjustments


```scala
// Map the existing ptolemyLonAdjusted Vector:

val ptolemyAdjusted = 
    ptolemyLonAdjusted.map(pt => GeoPoint(pt.id, 
                                          pt.lat + offset, 
                                          pt.lon))
```

### Verify the size is the same


```scala
ptolemyAdjusted.size
```

## Get your data into a GIS

We'd like to write a file with our data in `.csv` format that a GIS can read.

This requires two steps:

1. format the Vector of `GeoPoint` objects as csv Strings.
2. write the formatted Strings to a file

The `csv` method of the `GeoPoint` class will simplify this: we can simply map every `GeoPoint` to the String output of its `csv` method.


```scala
val csvVector = ptolemyAdjusted.map(pt => pt.csv)
```

Vectors have a handy `mkString` method to make a String out of a Vector.  It takes one parameter:  a String value used to join each element.  The following cell turns the Vector of Strings into a single String with new lines separating the components of the source Vector.


```scala
val csv = csvVector.mkString("\n")
```

We should define a header line to include in our `csv` file:


```scala
val header = "id,lon,lat\n"
```

### If running locally (e.g., in Atom)

`PrintWriter` is a clunky old Java class but if you just clone the code in the following cell, it's easy enough to write your output to a file in your local file system.


```scala
//import java.io.PrintWriter
//new PrintWriter("ptolemy-output.csv"){ write(header + csv); close; }
```

### If running Jupyter notebook on `mybinder.org`

If you're running the Jupyter notebook on `mybinder.org`, use `println` to display all values, that you can then (tediously) copy and paste into a text file.


```scala
println(header + csv)
```
### You can find the `csv` data [here.](https://github.com/michaeldahlquist/clas299/blob/master/ptolemy/ptolemy.csv)

## Load your CSV file into QGIS and visualize!

