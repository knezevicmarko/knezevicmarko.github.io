---
layout: lekcija
title: Upiti nad više tabela
main_category: Materijali za vežbe
sub_category: SQL Upiti
image: sql1.png
active: true
comment: true
archive: false
---

### Više tabela kao izvori podataka (Dekartov, INNER JOIN, LEFT JOIN, RIGHT JOIN)

* Izlistati imena nastavnika i šifre predmeta koje predaju

**I nacin**

{% highlight sql %}
/*
	Dekartov filtrirani
*/

select Nastavnici.Imen, Angazovanje.Spred
from Nastavnici, Angazovanje
where Nastavnici.Snast = Angazovanje.Snast
{% endhighlight %}


**II nacin**
{% highlight sql %}
/*
	INNER JOIN
*/

select n.Imen, a.Spred
from Nastavnici n
join Angazovanje a on n.Snast = a.Snast
{% endhighlight %}
* Izlistati imena nastavnika i šifre predmeta koje predaju (u skupu trebaju da se nadju i nastavnici koji nisu angažovani)

{% highlight sql %}
select n.Imen, a.Spred
from Nastavnici n
LEFT join Angazovanje a on n.Snast = a.Snast
{% endhighlight %}

* Spisak nastavnika koji nisu angažovani.

{% highlight sql %}
select n.Snast
from Nastavnici n
left join Angazovanje a on n.Snast = a.Snast
where a.Snast is null
{% endhighlight %}

* Spisak nastavnika i predmeta (samo šifre) koji dele predmet sa još nekim

{% highlight sql %}
select a1.Snast, a1.Spred
from Angazovanje a1
join Angazovanje a2 on a1.Spred = a2.Spred and a1.Snast <> a2.Snast
{% endhighlight %}

* Izlistati imena nastavnika i NAZIVE predmeta koje predaju

{% highlight sql %}		
select n.Imen, p.NAZIVP
from Nastavnici n
join Angazovanje a on n.Snast = a.Snast
    join PREDMETI p on a.Spred = p.SPRED
{% endhighlight %}

* Spisak brucoša koji imaju druga na fakultetu

{% highlight sql %}
select s1.Imes, s2.Imes
from Studenti s1
join Studenti s2 on (s1.Upisan = s2.Upisan and s1.Mesto = s2.Mesto and s1.Indeks != s2.Indeks)
{% endhighlight %}

* Spisak studenata (indeks, upisan, ime studenta) koji imaju bar jedan položen ispit

{% highlight sql %}
select distinct s.Indeks, s.Upisan, s.Imes
from Studenti s
join Prijave p on s.Indeks = p.Indeks and s.Upisan = p.Upisan
where p.Ocena > 5
{% endhighlight %}

* Spisak studenata koji imaju prosek veći od 7.5

{% highlight sql %}
select s.Indeks, s.Upisan, avg(p.Ocena * 1.0)
from Studenti s
join Prijave p on s.Indeks = p.Indeks and s.Upisan = p.Upisan
where p.Ocena > 5
group by s.Indeks, s.Upisan
having avg(p.Ocena * 1.0) > 7.5
{% endhighlight %}

* Maksimalna ocena za svaki predmet

{% highlight sql %}
select p2.Spred, p2.NAZIVP, max(ocena)
from prijave p1
join PREDMETI p2 on p1.Spred = p2.SPRED
group by p2.NAZIVP, p2.Spred
{% endhighlight %}
