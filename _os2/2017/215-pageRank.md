---
layout: lekcija
title: Page Rank
main_category: Materijali za vežbe
sub_category: Apache Spark
image: r.png
active: true
comment: true
archive: false
---

PageRank algoritam meri značajnost (rang) svakog dokumenta na osnovu toga koliko dokumenata ima link ka njemu. Može da se koristi da rangira web strane, ali i naučna dokumenta, uticajnost korisnika isl.

Algoritam ima dva skupa podataka:

* links (pageId, linkList)  - sadrži listu suseda svakog dokumenta i
* ranks (pageID, rank) – sadrži trenutni rang svakog dokumenta.

Algoritam se izvršava na sledeći način:

1. Inicijalizuje rang svakog dokumenta na 1.0,
2. U svakoj iteraciji, svaka stranica šalje svoj doprinos u rangu svim susedima (stranica p ima doprinos(p) = rang(p)/brojSuseda(p) ).
3. Novi rang svake stranice se računa po formuli 0.15 + 0.85 * sumaDoprinosaSvihSuseda
4. Broj iteracija je proizvoljan

Podaci u ulaznom fajlu su oblika:

{% highlight bash %}
idStranice1 idStraniceLinka11 idStraniceLinka12 idStraniceLinka13 ...
idStranice2 idStraniceLinka21 idStraniceLinka22 idStraniceLinka23 ...
.
.
.
{% endhighlight %}

Predlog rešenja

{% highlight python %}
from pyspark import SparkConf, SparkContext
import sys

def mapa(x):
	lst = x.split(" ")
	return (lst[0], lst[1:])

def computeContribs(urls, rank):
	n = len(urls)
	for url in urls:
		yield (url, rank / n)

if __name__ == "__main__":
	conf = SparkConf().setMaster("local").setAppName("My App")
	sc = SparkContext(conf = conf)
	if len(sys.argv) != 3:
		print >> sys.stderr, "Args <file> <iterations>"
		exit(1)

	lines = sc.textFile(sys.argv[1])
	links = lines.map(mapa).partitionBy(4).persist()

	ranks = links.mapValues(lambda x: 1.0)

	for iteration in xrange(int(sys.argv[2])):
		contribs = links.join(ranks).flatMap( lambda (url, (urls, rank)): computeContribs(urls, rank))
		ranks = contribs.reduceByKey(lambda x, y: x + y).mapValues(lambda rank: rank * 0.85 + 0.15)

	ranks.saveAsTextFile("izlazPage")
{% endhighlight %}
