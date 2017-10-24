---
layout: lekcija
title: Kolokvijum IV
main_category: Materijali za vežbe
sub_category: Relaciona algebra
image: join_left.png
active: false
comment: true
archive: false
---

## DML - Data Manipulation Language

{% highlight sql %}
	SELECT [{ALL | DISTINCT}] select_item [AS alias] [,...]
	FROM { table_name [[AS] alias] | view_name [[AS] alias]} [,...]
	[ [join_type] JOIN join_condition ]
	[WHERE search_condition] [ {AND | OR | NOT} search_condition [...] ]
	[GROUP BY group_by_expression{group_by_columns}
	[HAVING search_condition] ]
	[ORDER BY {order_expression [ASC | DESC]} [,...] ]
{% endhighlight %}

### SELECT

{% highlight sql %}
SELECT column_name,column_name as 'alias'
FROM table_name;

use STUDIJE
{% endhighlight %}

* Prikazati spisak svih studenata

{% highlight sql %}
select *
from studenti
{% endhighlight %}

* Prikazati Imes, indeks, upisan, mesto iz tabele STUDENTI

{% highlight sql %}
select Imes, indeks, upisan, mesto
from studenti
{% endhighlight %}

* Prikazati Imes, Indeks, Upisan, Datr iz tabele studenti, gde je Datr naslovljena kao 'Datum rodjenja'

{% highlight sql %}
select Imes, indeks, upisan, mesto, Datr as 'Datum rodjenja'
from studenti
{% endhighlight %}

### DISTINCT

{% highlight sql %}
SELECT DISTINCT column_name,column_name
FROM table_name;
{% endhighlight %}

* Selektovati različita mesta iz tabele Studenti, i dopisati kolonu sa NULL vrednostima

{% highlight sql %}
select distinct mesto
from studenti
{% endhighlight %}

### WHERE

{% highlight sql %}
SELECT column_name,column_name
FROM table_name
WHERE predikat;
{% endhighlight %}

* Prikazati podatke o studentima koji su upisani 2000 godine

{% highlight sql %}
select *
from studenti
where upisan = 2000
{% endhighlight %}

### BETWEEN

{% highlight sql %}
SELECT column_name(s)
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
{% endhighlight %}

* Prikazati sve studente koji su upisani izmedju 2000 i 2005

{% highlight sql %}
select *
from studenti
where upisan between 2000 and 2005
{% endhighlight %}

### IN

{% highlight sql %}
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...);
{% endhighlight %}

* Prikazati Studente koji su iz Kragujevca ili Kraljeva

{% highlight sql %}
select *
from studenti
where mesto in ('Kragujevac', 'Kraljevo')
{% endhighlight %}

* Prikazati Studente koji NISU iz Kragujevca ili Kraljeva

{% highlight sql %}
select *
from studenti
where mesto not in ('Kragujevac', 'Kraljevo')
{% endhighlight %}

### LIKE

{% highlight sql %}
SELECT column_name(s)
FROM table_name
WHERE column_name LIKE pattern;
{% endhighlight %}

* Prikazati sve studente koji dolaze iz grada čije ime počinje na "K"

{% highlight sql %}
select *
from studenti
where mesto like 'K%'
{% endhighlight %}

* Prikazati studente za koje je nepoznat datum rodjenja

{% highlight sql %}
select *
from studenti
where datr is null
{% endhighlight %}

### ORDER BY

{% highlight sql %}
SELECT column_name, column_name
FROM table_name
ORDER BY column_name ASC|DESC, column_name ASC|DESC;
{% endhighlight %}

* Prikazati Studente koji NISU iz Kragujevca ili Kraljeva sortirane po imenu (od najmanjeg ka najvećem)

{% highlight sql %}
select *
from studenti
where mesto not in ('Kragujevac', 'Kraljevo')
order by Imes desc
{% endhighlight %}

### CONCAT && CAST

{% highlight sql %}
SELECT CAST(column_name as TYPE)
FROM table_name

SELECT CONCAT(column_name, value, column_name, ...)
FROM table_name
{% endhighlight %}

* Za svako ime studenta kreirati kolonu u kojoj će pisati njegov Indeks kao "br_indeksa/godina" (CONCAT i CAST)

**I način**

{% highlight sql %}
select Imes, CONCAT(Indeks, '/', Upisan) as Indeks
from Studenti
{% endhighlight %}

**II način**

{% highlight sql %}
select Imes, CAST(Indeks as varchar(5)) + '/' + CAST(Upisan as varchar(5))
from Studenti
{% endhighlight %}

### CASE

I) Simple CASE expression:   

{% highlight sql %}
CASE input_expression   
  WHEN when_expression THEN result_expression [ ...n ]   
  [ ELSE else_result_expression ]   
END
{% endhighlight %}

II)	Searched CASE expression:  

{% highlight sql %}
CASE  
  WHEN Boolean_expression THEN result_expression [ ...n ]   
  [ ELSE else_result_expression ]   
END
{% endhighlight %}

* Pronađi smeštaj - Prikazati sve studente i dodati kolonu u kojoj za studente koji nisu iz kragujevca piše vrednost "Potraban smestaj", u suprotnom piše "Lokalno"

{% highlight sql %}
select Imes,
case Mesto
  when 'Kragujevac' then 'LOKALNO'
  else 'POTREBAN SMESTAJ'
end as 'Smestaj'
from Studenti
{% endhighlight %}

* Selektovati sve ocene iz prijava, i dodati kolonu koja za svaku ocenu ispisuje da li je ocena
  * "Ispod proseka" (5.6),
  * "Prosek" (7,8),
  * "Odlicna" (9)
  * "Izuzetna" (10)

{% highlight sql %}
select Indeks, Upisan, Spred,
case
  when Ocena > 5 and Ocena < 7 then 'Ispod proseka'
  when Ocena >= 7 and Ocena < 9 then 'Proseck'
  when Ocena = 9 then 'Odlicana'
  else 'Izuzetana'
end as Status
from Prijave
{% endhighlight %}

### DATE FUNCTIONS

{% highlight sql %}
DATEDIFF ( datepart , startdate , enddate )
DATENAME ( datepart , date )
DATEPART ( datepart , date )
GETDATE  ( )
{% endhighlight %}

* Prikazati godišta i starost, svih studenata koji imaju više od 25 godina

{% highlight sql %}
select Imes, Indeks, Upisan, DATEPART(year, datr) as 'Godina', DATEDIFF(year, datr, GETDATE()) as Starost
from Studenti
where DATEDIFF(year, datr, GETDATE()) > 25
{% endhighlight %}

### AGREGATNE FUNKCIJE

{% highlight sql %}
COUNT , MIN, MAX, SUM, AVG
{% endhighlight %}

* Prikazati broj studenata koji su upisani na PMF-u

{% highlight sql %}
select COUNT(*)
from Studenti
{% endhighlight %}

* Prikazati broj studenata koji su upisani na PMF-u, a imaju poznat datum rodjenja

{% highlight sql %}
select COUNT(datr)
from Studenti
{% endhighlight %}

* Prikazati prosecnu, minimalnu i maksimalnu ocenu na predmetu sa sifrom 23

{% highlight sql %}
select MAX(ocena) as minimalna, MIN(ocena) as maksimalna, AVG(cast(ocena as real)) as prosecna
from Prijave
where Spred = 23

select MAX(ocena) as minimalna, MIN(ocena) as maksimalna, AVG(ocena * 1.0) as prosecna
from Prijave
where Spred = 23
{% endhighlight %}

### GROUP BY

{% highlight sql %}
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
{% endhighlight %}

* Za svako mesto ispisati koliko studenata dolazi iz njega.

{% highlight sql %}
select Mesto,count(*) as 'broj studenata'
from Studenti
group by Mesto
{% endhighlight %}

* Za svako godište studenata koje se pojavljuje u tabeli Studenti, ispisati broj studenata koji su rođeni te godine.

{% highlight sql %}
select datepart(year, Datr) godiste, COUNT(*) as 'broj studenata'
from studenti
group by datepart(year, Datr)
{% endhighlight %}

* Sa svakog studenta (indeks, upisan) ispisati koliko ispita je polozio i njegovu prosečnu ocenu

{% highlight sql %}
select Indeks, Upisan, count(*) 'broj polozenih', Avg(cast(Ocena as real)) as Prosek
from Prijave
group by Indeks, Upisan
{% endhighlight %}

### HAVING

{% highlight sql %}
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
{% endhighlight %}

* Prikazati sve studente koji imaju prosek veći od 6

{% highlight sql %}
select Indeks, Upisan, Avg(cast(Ocena as real)) as Prosek
from Prijave
group by Indeks, Upisan
having Avg(cast(Ocena as real)) > 6.0
{% endhighlight %}

* Koji predmeti se drže na više različitih smerova (group by + having)

{% highlight sql %}
select Spred, count(*)
from Planst
group by Spred
having count(Ssmer) > 1
{% endhighlight %}

* Koliko ima predmeta na svakoj godini?

{% highlight sql %}
select (semestar + 1) / 2 as Godina, count(*) as Br
from Planst
group by (semestar + 1) / 2
{% endhighlight %}

* Za svaki predmet ispisati kada je poslednji put polagan?

{% highlight sql %}
select spred, max(datump)
from Prijave
group by spred
{% endhighlight %}

* Poslednji polozen predmet za svakog studenta?

{% highlight sql %}
select indeks, upisan, max(datump)
from Prijave
where ocena > 5
group by Indeks, Upisan
{% endhighlight %}

* Za svaki ispitni rok ispisati koliko se u njemu položilo ispita do sada?

{% highlight sql %}
/*  
  rbr_meseca, rok
  1, Jan
  2, Feb
  6, Jun
  8, Avg
  9, Sept
  10 Okt
*/

select DATEPART(m, datump) as mesec, count(*) as Broj,
case datepart(m, datump)
  when 1 then 'Januarski'
  when 2 then 'Feb'
  when 6 then 'Jun'
  when 8 then 'Avg'
  when 9 then 'Sept'
  when 10 then 'Okt'
end as Rok
from Prijave
where ocena > 5
group by datepart(m, datump)
{% endhighlight %}

* Prikazati uspeh (prosecnu ocenu) svake generacije

{% highlight sql %}
select upisan generacija, avg(ocena * 1.0)
from prijave
where ocena > 5
group by upisan
{% endhighlight %}
