---
layout: lekcija
title: SHELL - osnove programiranja 1
main_category: Materijali za vežbe
sub_category: Linux
image: shell.gif
active: false
comment: true
archive: true
---

## Inicijalizacija programa

Nakon interpretacije komandne linije shell inicira izvršenje zadate komande. Ukoliko komanda nije interna (ugrađena u shell, poput komande cd) shell traži izvršni fajl koji odgovara imenu komande u direktorijumima navedenim u sistemskoj putanji (koja se može dobiti komandom **echo $PATH**). Nakon toga shell pokreće program i prosleđuje mu argumente i opcije navedene u komandnoj liniji.
Napomena: Ukoliko se izvršni fajl nalazi u tekućem direktorijumu ili u nekom direktorijumu koji nije u sistemskoj putanji ($PATH), ime komande se mora zadati sa putanjom. Slede i primeri koji ilustruju pokretanje programa koji se nalaze u tekućem direktorijumu i direktorijumu /usr/sbin:
{% highlight bash %}
$ ./myscript
$ /usr/sbin/useradd
{% endhighlight %}


# Primer najjednostavnijeg skripta

Rad sa skriptovima se svodi na 3 koraka, i to:

* Formiranje samog skripta pomoću tekst editora ili iz komandne linije redirekcijom standardnog ulaza pomoću komande **cat**. Skript **ss1.sh** (upisan iz *cat*-a u fajl *ss1.sh*)
{% highlight bash %}
$ cat > ss1.sh
#
# ss1.sh: jednostavan shell program
#
clear
echo "Hello, World!"
<CTRL-D>
{% endhighlight %}

* Davanje korisnicima x (execute) dozvole nad datotekom
{% highlight bash %}
$ chmod +x ss1.sh
{% endhighlight %}
**Pitanje**: Kojim korisničkim kategorijama je data dozvola x?

* Pokretanje
{% highlight bash %}
$ ./ss1
{% endhighlight %}
Skript briše ekran (komanda clear), a zatim na ekranu ispisuje poruku Hello, World! Sav tekst u liniji iza znaka # se smatra komentarom.

**Pitanje**: Kako izbeći ./ ?

Prilikom pokretanja skripta može se specificirati komandni interpreter u kome će se program izvršavati. Potrebno je u prvu liniju skripta upisati sledeće:
{% highlight bash %}
#!/bin/bash
{% endhighlight %}
Ukoliko se komandni interpreter ne specificira na ovaj način, program se izvršava u tekućem interpreteru. Skript se može pokrenuti i na drugi način, bez eksplicitne dodele x dozvole - dovoljno je pozvati komandni interpreter da izvrši shell program:
{% highlight bash %}
$ bash ss1.sh
{% endhighlight %}
ili
{% highlight bash %}
$ /bin/sh ss1.sh
{% endhighlight %}
Ukoliko se shell program ne nalazi u tekućem direktorijumu, potrebno je specificirati putanju do programa.
{% highlight bash %}
$ bash /home/jsmith/ss1.sh
{% endhighlight %}
Za razvoj i korišćenje shell skriptova preporučuje se sledeća procedura:

* skript treba razviti na svom direktorijumu,
* zatim ga istestirati:
{% highlight bash %}
$ bash imeskripta
{% endhighlight %}
* na kraju iskopirati u neki direktorijum koji je podrazumevano uključen u sistemsku putanju.

Program se može kopirati u bin poddirektorijum home direktorijuma autora. Ukoliko veći broj korisnika želi da koristi program datoteku treba kopirati u direktorijume **/bin** ili **/usr/bin** ili **/usr/local/bin** kojima mogu pristupati svi korisnici. Dodatno, korisnicima treba dati dozvolu execute da bi mogli da pokreću program pomoću imena datoteke.

Komande koje se mogu zadavati u skriptovima su:

* standardne Linux komande (poput **cp** ili **mv**)
* komande specifične za shell programiranje. Neke od komandi specifičnih za shell programiranje su gotovo kompletni programski jezici (na primer **awk**).
