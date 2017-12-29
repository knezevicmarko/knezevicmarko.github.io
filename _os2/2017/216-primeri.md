---
layout: lekcija
title: Zadatak
main_category: Materijali za vežbe
sub_category: Apache Spark
image: k.png
active: true
comment: true
archive: false
---
[Dati .csv fajlovi](/assets/os2/employees.zip) sadrže podatke koji odgovaraju šemi sa slike.
<div style="width: 100%; text-align: center;">
  <figure>
    <img style="width: 100%" src="/assets/os2/employees-schema.png" alt="Employess baza podataka" />
    <figcaption style="font-size: 0.8em">Employess baza podataka</figcaption>
  </figure>
</div>

Napisati jednu pySpark skriptu koja:

* Za svakog zaposlenog računa ukupnu sumu novca koju je primao po godinama. Razultat treba da bude u obliku

 (ime, prezime, godina, suma novca),

* Pronalazi radnike kojima se plata nikada nije smanjivala. Rezultat je oblika:

(ime, prezime)

* Za svakog radnika ispisati kom departmanu pripada i ko mu je menadžer.

(imeRadnika, prezimeRadnika, imeDepartmana, imeMenadžera, prezimeMenadžera)

* Pronaći departman koji je isplatio najviše novca.

(imeDepartmana, kolicinaNovca)


Primer učitavanja jednog .csv fajla u odgovarajući RDD
{% highlight python %}
>>> import csv
>>> from datetime import datetime
>>> csvRDD = sc.textFile("dept_emp.csv").mapPartitions(lambda x: csv.reader(x))
>>> deptEmp = csvRDD.map(lambda (empt_no, dept_no, from_date, to_date): (empt_no + " " + dept_no, (datetime.strptime(from_date, "%Y-%M-%d").date(), datetime.strptime(to_date, "%Y-%M-%d").date()))).partitionBy(4).cache()
>>> print deptEmp.toDebugString()
>>> deptEmp.first()
{% endhighlight %}
