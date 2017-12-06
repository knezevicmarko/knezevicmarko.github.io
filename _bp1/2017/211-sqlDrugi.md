---
layout: lekcija
title: Ugnježdeni upiti
main_category: Materijali za vežbe
sub_category: SQL Upiti
image: sql1.png
active: true
comment: true
archive: false
---

{% highlight sql %}
use STUDIJE
go
{% endhighlight %}

## UGNJEZDENI UPITI (BEZ KORELISANIH)

* Spisak nastavnika koji nisu angažovani.

{% highlight sql %}
select *
from nastavnici
where Snast not in (select snast from Angazovanje)
{% endhighlight %}

* Svi studenti koji dolaze iz gradova iz kojih je došlo više od 2 studenta

{% highlight sql %}
select *
from nastavnici
where Snast not in (select snast from Angazovanje)
{% endhighlight %}

* Broj indeksa, godina upisa, ocena studenata čije su ocene manje od svih ocena koje je dao nastavnik sa "s" u imenu. (ALL, ANY)

**I način**

{% highlight sql %}
select indeks, upisan, ocena
from prijave
where ocena < all (
    select ocena
    from prijave where
    Snast = ANY (
        select snast
        from Nastavnici
        where Imen like '%s%'
        )
    )
{% endhighlight %}

**II način (ALL, IN)**

{% highlight sql %}
select indeks, upisan, ocena
from prijave
where ocena < all (
    select ocena
    from prijave where
    Snast in (
        select snast
        from Nastavnici
        where Imen like '%s%'
        )
    )
{% endhighlight %}

## KORELISANI UGNJEZDENI UPITI

* Spisak nastavnika i predmeta (samo šifre) koji dele predmet sa još nekim

**I način (IN)**

{% highlight sql %}
select Snast, Spred
from Angazovanje a1
where Spred in (
    select Spred
    from Angazovanje a2
    where a1.Snast <> a2.Snast
    )
{% endhighlight %}

**II način (EXISTS)**

## EXISTS

Sintaksa:
{% highlight sql %}
	WHERE [NOT] EXISTS ( pod-upit );
{% endhighlight %}
pod-upit - predstavlja SELECT upit koji ako vrati najmanje jednu n-torku u svom rezultujućem setu klauzula EXISTS će se oceniti kao TRUE, u suprotnom, ako ne vrati ni jednu n-torku EXISTS klauzula će se oceniti kao FALSE.

Napomena: SQL upiti koji koriste EXISTS uslov su veoma neefikasni jer pod-upit se izvrsava za svaki red u spoljašnjem upitu.
{% highlight sql %}
select a1.Snast, a1.Spred from Angazovanje a1
where exists (
    select *
    from Angazovanje a2
    where a1.Snast <> a2.Snast and a1.Spred = a2.Spred
    )
{% endhighlight %}

* Spisak nastavnika koji nisu angažovani, drugi nacin - exists
{% highlight sql %}
select * from Nastavnici n
where not exists (
    select * f
    from Angazovanje a
    where a.Snast = n.Snast
    )
{% endhighlight %}

* Spisak studenata koji imaju bar jedan polozen ispit

{% highlight sql %}
select * from Studenti s
where exists (
    select *
    from Prijave p
    where ocena > 5 and s.Indeks = p.Indeks and p.Upisan = s.Upisan
    )
{% endhighlight %}

* Spisak studenata koji imaju prosek veci od 7.5
{% highlight sql %}
select * from Studenti s
where exists (
    select Indeks, Upisan
    from prijave
    where ocena > 5 and s.Indeks = Indeks and s.Upisan = Upisan
    group by Indeks, Upisan -- da li grupisanje moze da se izbaci ?!
    having avg(ocena * 1.0) > 7.5
    )
{% endhighlight %}
