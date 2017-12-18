---
layout: lekcija
title: Apache Spark pair RDDs
main_category: Materijali za vežbe
sub_category: Apache Spark
image: p.png
active: true
comment: true
archive: false
---

### Prosleđivanje funkcija Sparku

Većina Spark transformacija i neke od akcija očekuju da se kao argument prosledi f-ja koju Spark koristi za preračunavanje.
Postoje tri načina na koja mogu da se proslede f-je Sparku.
* Preko lambda izraza:
{% highlight python %}
PythonLines = lines.filter(lambda x: "Python" in x)
{% endhighlight %}

* Korisnički definisana f-ja:
{% highlight python %}
def hasPython(line):
    return "Python" in line

pythonLines = lines.filter(hasPython)
{% endhighlight %}
* Unutar klase. Kada se prosledjuju parametri Spark funkciji korišćenjem klase treba voditi računa da se ne prosledi funkcija te klase ili referenca na neki član objekta. Ako se prosledi referenca ka članu objekta Spark šalje ceo objekat na radni čvor. To može dovesti do toga da se umesto male količine informacija pošalje velika količina informacija koje nisu potrebne. Ponekad, ako je objekat takav da Python ne može da ga serijalizuje (Python pickle), može dovesti do toga da program ne može da se izvrši.
Primer neispravnog prosleđivanja Sparku
{% highlight python %}
class SearchFunctions(object):
    def __init__(self, query):
        self.query = query

    def isMatch(self, s):
        return self.query in s

    def getMatchesFunctionReference(self, rdd):
         # Problem: references all of "self" in "self.isMatch"
        return rdd.filter(self.isMatch)

    def getMatchesMemberReference(self, rdd):
        # Problem: references all of "self" in "self.query"
        return rdd.filter(lambda x: self.query in x)
{% endhighlight %}
Umesto prosledjivanja argumenata sa referencom na polje potrebno je proslediti lokalnu varijablu sa potrebnim objektom
{% highlight python %}
class WordFunctions(object):
...
def getMatchesNoReference(self, rdd):
# Safe: extract only the field we need into a local variable
query = self.query
return rdd.filter(lambda x: query in x)
{% endhighlight %}

### Persistence (Keširanje)

Spark RDD koristi lenju evaluaciju i nekada ima potrebe da se koristi isti RDD više puta. Ako koristimo isti RDD više puta, predefinisano ponašanje je takvo da Spark za svaku akciju preračunava RDD. Ovo može biti veoma skupo, naručito za iterativne algoritme.
Trivijalni primer:
{% highlight python %}
lines = sc.textFile("README.md")
sparkLines = lines.filter(lambda x: "Python" in x)
sparkLines.count()
sparkLines.collect()
{% endhighlight %}
Da bi izbegli preračunavanje RDD-a više puta možemo da zamolimo Spark da kešira podatke. U tom slučaju čvorovi koji preračunavaju RDD skladište njegove delove. Ako čvor koji ima keširane podatke otakže, Spark će te delove podataka ponovo preračunati kada budu bili potrebni. Takođe, moguće je skladištenje podataka sa redudansom, na više čvorova, ako je potrebno da se upravlja otkazom čvorova bez usporavanja.
Postoje više nivoa keširanja podataka. Podatke je moguće čuvati na heap-u JVM kao serijalizovane ili neserijalizovane objekte. Takođe, moguće je čuvati i podatke na disku. Podaci na disku se uvek čuvaju serijalizovano.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Nivo | Upotrebljnost prostora | CPU vreme | U memoriji | Na disku |
|-----|-----|-----|-----|------|
| MEMORY_ONLY | Visoka | Niska | Da | Ne |
| MEMORY_ONLY_SER | Niska | Visoka | Da | Ne |
| MEMORY_AND_DISK | Visoka | Srednja | Neki | Neki |
| MEMORY_AND_DISK_SER | Niska | Visoka | Neki | Neki |
| DISK_ONLY | Niska | Visoka | Ne | Da |

#### Primer
{% highlight python %}
>>> lines = sc.textFile("README.md")
>>> sparkLines = lines.filter(lambda x: "Python" in x)
>>> from pyspark import StorageLevel
>>> sparkLines.persist(StorageLevel.DISK_ONLY)
>>> sparkLines.count()
>>> sparkLines.collect()
{% endhighlight %}
Potrebno je pozvati persist() pre prve akcije. Ako probamo da keširamo više podataka nego što može da stane u memoriju, Spark će automatski izbaciti najstarije particije Least Recently Used (LRU) algoritmom. Za memory_only nivoe izbačene particije će preračunati kada one budu bile potrebne, a za ostale nivoe izbačene particije će biti zapamćene na disk.

RDD ima i metod `unpersist()`` kojim se ručno uklanja RDD iz memorije.

## Ključ / Vrednost Parovi

Spark ima specijalne operacije nad RDD-ovima koji sadrže ključ vredsnot parove. Ovi RDD-ovi se zovu pair RDD. Pair RDD su korisni zato što mogu da se koriste operacije koje se izvode nad ključevima paralelno ili regrupišu podatke preko mreže.

Postoji više načina za kreiranje pair RDD. Mnogi formati fajlova direktno vraćaju pair RDD. U ostalim slučajevima možemo da pretvorima regularni u pair RDD. To se postiže korišćenjem f-je map() koja kao rezultat vraća key/values parove.
{% highlight python %}
pairs = lines.map(lambda x: (x.split(" ")[0], x))
{% endhighlight %}

### Pair RDD transformacije

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Transformacije nad jednim RDD-om |
|------|---------|
| reduceByKey(func) | Kombinuje vrednosti sa istim ključem |
| groupByKey() | Grupiše vrednosti sa istim ključem |
| combineByKey(createCombiner, mergeValues, mergeCombiners, partitioner) | Kombinuje vrednosti sa istim ključem koristeći različite tipove rezultata |
| mapValues(func) | Primenjuje func nad svakom vrednošću unutar RDD-a. |
| flatMapValues(func) | Slično kao i mapValues, ali mapira jedan original u 0 ili više vrednosti |
| keys() | Vraća RDD koji se sastoji samo od ključeva |
| values() | Vraća RDD koji ima samo vrednosti |
| sortByKey() | Sortira RDD po ključu. |

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Transformacije nad dva RDD-a |
|-------|---------|
| subtractByKey(drugiRDD) | Uklanja elemente sa ključem koji postoji u drugom RDD-u |
| join(drugiRDD) | Unutrašnje spajanje dva RDD-a |
| rightOuterJoin(drugiRdd) | Desno spajanje dva RDD-a |
| leftOuterJoin(drugiRdd) | Levo spajanje dva RDD-a |
| cogroup(drugiRdd) | Grupiše podatke koj dele isti ključ iz oba RDD-a |

Pair RDD je izvedena klasa iz RDD tako da podržava iste f-je kao i RDD.

{% highlight python %}
>>> pair = sc.parallelize([(1,2), (3,4), (3,6)])
>>> pair.filter(lambda keyValue: keyValue[1] > 3).collect()
[(3, 4), (3, 6)]  
{% endhighlight %}

Kada je skup podataka opisan kao ključ/vrednost parovi, najčešće se vrši spajanje svih elemenata sa istim ključem.  Akcije reduce(), fold() i aggregate() običnog RDD imaju slične transformacije nad pair RDD-om. Ove f-je kao povratnu vrednost imaju RDD pa se zato zovu transformacije, a ne akcije.
F-ja reduceByKey() je slična f-ji reduce() obe uzimaju funkciju i koriste je da bi kombinovale vrednosti. reduceByKey() izvršava više paralelnih operacija redukcije, jednu za svaki ključ u skupu podataka, gde svaka operacija kombinuje vrednosti sa istim ključem. Skup podataka može da ima veliki broj ključeva, pa reduceByKey nije implementirana kao akcija koja vraća rezultat u driver program. Umesto toga, reduceByKey() je transformacija koja kreira novi RDD koji se sastoji od ključa i redukovanih vrednosti za taj ključ.

#### Primer reduceByKey()

{% highlight python %}
>>> u =sc.parallelize([1, 2, 3, 3, 3, 2]).map(lambda x: (x, 1))
>>> a=u.reduceByKey(lambda x, y: x + y)
>>> a.collect()
[(1, 1), (2, 2), (3, 3)]
{% endhighlight %}

#### Primer mapValues() i reduceByKey() za prosek po ključu

{% highlight python %}
>>> rdd1 = sc.parallelize([("panda", 0), ("pink", 3), ("pirate", 3), ("panda", 1), ("pink", 4)])
>>> rdd1.mapValues(lambda x: (x, 1)).reduceByKey(lambda x, y: (x[0] + y[0], x[1] + y[1])).collect()
[('pink', (7, 2)), ('panda', (1, 2)), ('pirate', (3, 1))]
{% endhighlight %}

combineByKey() je najopštija ključ/vrednost agregacija. Kao i aggregate(), combineByKey() dozvoljava povratne vrednosti koje nisu istog tipa kao i ulazni podaci.

#### Primer prosečna vrednost po ključu koristeći combineByKey()

{% highlight python %}
>>> sumCount = rdd1.combineByKey((lambda x: (x, 1)),
... (lambda x,y: (x[0] + y, x[1] + 1)),
... (lambda x,y: (x[0] + y[0], x[1] + y[1])))
>>> sumCount.map(lambda x: (x[0], x[1][0]/float(x[1][1]))).collect()
[('pink', 3.5), ('panda', 0.5), ('pirate', 3.0)]
{% endhighlight %}

Ako su podaci već grupisani na način koji je potreban groupByKey() će grupisati podatke po ključu u RDD-u. Ako se RDD sastojati od ključeva tipa K i vrednosti tipa V nakon groupByKey() RDD će se biti sledećeg tipa [K, Iterable[V]]
{% highlight python %}
>>> sc.parallelize([(1,2),(3,4),(3,6)]).groupByKey().collect()
[(1, <pyspark.resultiterable.ResultIterable object at 0x7fd9f57b47d0>), (3, <pyspark.resultiterable.ResultIterable object at 0x7fd9f5818050>)]
>>> for i in sc.parallelize([(1,2),(3,4),(3,6)]).groupByKey().collect():
...   print i[0]
...   for a in i[1]:
...     print a
...
1
2
3
4
6
{% endhighlight %}

#### Primer transformacija nad dva RDD-a
{% highlight python %}
>>> rdd1.subtractByKey(rdd2).collect()
[(1, 2)]
>>> rdd1.join(rdd2).collect()
[(3, (4, 9)), (3, (6, 9))]
>>> rdd1.rightOuterJoin(rdd2).collect()
[(3, (4, 9)), (3, (6, 9))]                                                      
>>> rdd1.leftOuterJoin(rdd2).collect()
[(1, (2, None)), (3, (4, 9)), (3, (6, 9))]
{% endhighlight %}

### Akcije nad pair RDD

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
|-------|---------|
| countByKey() | Koliko elemeneta ima sa istim ključem |
| collectAsMap() | Skuplja podatke kao map-u radi lakšeg pristupa |
| lookup(key) | Sve vrednosti koji imaju ključ key |

### Zadatak 1.

Napraviti Spark skriptu koji broji koliko puta se svaka reč pojavljuje u fajlu README.md.

#### Predlog rešenja
{% highlight python %}
from pyspark import SparkConf, SparkContext

conf = SparkConf().setMaster("local").setAppName("My App")
sc = SparkContext(conf = conf)

lines = sc.textFile("README.md")   
words = lines.flatMap(lambda x: x.split(" "))
rez = words.map(lambda x: (x, 1)).reduceByKey(lambda x,y: x + y)
rez.saveAsTextFile("izlaz")
{% endhighlight %}

### Domaći zadatak

Na osnovu podataka finansijskog prometa između kompanija (transfer novca među njima) potrebno je izvršiti kategorizaciju kompanija. U 4 tekstualne datoteke **kvartal1.txt, … , kvartal4.txt** se u svakoj liniji nalazi transfer novca. Transfer je dat u formatu:

pib<sub>uplatilac</sub>	pib<sub>primalac</sub>	iznos<sub>transfer</sub>

gde pib-ovi identifikuju kompanije (poreski identifikacioni broj).  U datoteci **kategorizacija.txt** nalaze se podaci prema kojima se kompanija kategorizuje u formatu:

iznos<sub>kat1</sub>
. . .
iznos<sub>kat8</sub>

gde je broj kategorizacionih granica 8, definisanih u vrednostima iznos<sub>kati</sub>, i =1 .. 8.

Ukoliko je iznos transfera 12345.67 , a kategorizacione granice iznos<sub>kat3</sub> i iznos<sub>kat4</sub> iznose redom 10240.12 i 14567.89 , tada takav transfer za obe kompanije govori da imaju transfer u kategoriji 3 . Ukoliko je iznos transfera manji od iznos<sub>kat1</sub>, transfer se ne uzima u obzir u kategorizaciji. Za transfer veći od iznos<sub>kat8</sub> , uzimamo da obe kompanije imaju transfer u kategoriji 8.

Potrebno je za svaku kompaniju utvrditi koliko je imala transakcija u svakoj kategoriji.

**Napomena : Svi iznosi transfera su brojevi u decimalnom zapisu, a pib -ovi su prirodni brojevi.**

Generator fajlova kvartal\*.txt
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <string.h>

#define BROJ_TRANSAKCIJA 1000000
#define BROJ_KOMPANIJA 1000
#define MAX_VREDNOST 10000.0
#define MIN_VREDNOST 1.0

int main()
{
	FILE *f;
	srand(time(NULL));
	long i;
	int j;
	double range = (MAX_VREDNOST - MIN_VREDNOST);
	double div = RAND_MAX / range;
	char file[13];

	for (j = 1; j < 5; j++)
	{
		sprintf(file, "kvartal%d.txt", j);
		f = fopen(file, "w");
		for (i = 0; i < BROJ_TRANSAKCIJA; i++)
		{
			long kid1 = rand() % BROJ_KOMPANIJA;
			long kid2 = rand() % BROJ_KOMPANIJA;
			while (kid1 == kid2)
				kid2 = rand() % BROJ_KOMPANIJA;
			double vrednost = MIN_VREDNOST + (rand() / div);
			fprintf(f, "%ld %ld %.2lf\n", kid1, kid2, vrednost);
		}
		fclose(f);
	}
}
{% endhighlight %}

Fajl kategorizacija.txt
{% highlight bash %}
100.0
2000.0
3000.0
4000.0
5000.0
6230.0
7800.0
9000.0
{% endhighlight %}
