---
layout: lekcija
title: Linkovi
main_category: Materijali za vežbe
sub_category: Linux
image: link.png
active: true
comment: true
archive: false
---

Na UNIX sistemima postoje dve vrste linkova, i to hard link i simbolički link (symbolic link).

### Hard linkovi

Kada korisnik pozove datoteku po imenu (na primer: cat tekst1), UNIX prevodi simboličko ime datoteke koje je naveo korisnik u interno ime, koje koristi operativni sistem. Zbog posebne interne reprezentacije, korisnici mogu datotekama dodeliti veći broj imena. Hard link je jedno od tih imena, odnosno alternativno ime datoteke.
{% highlight bash %}
$ ln file1 file2
$ ls file*
file1 file2
{% endhighlight %}

Ukoliko korisnik obriše datoteku file1, **file2 se ne briše**. Osobine:

* link i original imaju isti i-node, tako da se moraju nalaziti na fizički istom sistemu datoteka (hard link se ne sme nalaziti na drugoj particiji ili na drugom disku). Ne mogu se linkovati datoteke sa mrežnog sistema datoteka (NFS),
* ne može se linkovati direktorijum niti nepostojeća datoteka,
* vlasnik, grupa i prava pristupa su isti za link i za original,
* slobodan prostor na disku neznatno se umanjuje (jedna dir-info struktura više za alternativno ime datoteke),
* broj linkova originalne datoteke uvećava se za jedan nakon linkovanja.


### Simbolički linkovi

Simbolički linkovi se mogu kreirati na dva načina:
{% highlight bash %}
$ ln -s original linkname
$ cp -s original linkname
{% endhighlight %}

Osobine:

* svaki simbolički link koristi poseban i-node i jedan blok podataka u sistemu datoteka; mogu se kreirati nalaziti na fizički istom ili različitom sistemu datoteka, odnosno na istoj ili drugoj particiji (disku). Takođe, mogu se linkovati datoteke sa mrežnog sistema datoteka (NFS);
* može se linkovati direktorijum, kao i nepostojeća datoteka,
* u odnosu na original, link može imati različitog vlasnika, grupu i prava pristupa. Na korisnika koji datoteci ili direktorijumu pristupa putem simboličkog linka primenjuje se unija restrikcija (presek dozvola) linka i datoteke. Na primer, neka je korisnik **user2** vlasnik linka **link1** koji ukazuje na datoteku **file1**, i nek **pripada grupi** kojoj je ta datoteka formalno dodeljena. Ukoliko su pristupna prava za link i datoteku 777 i 640 respektivno, korisnik će imati samo pravo čitanja te datoteke, dakle, za grupu (rwx) and (r--) = (r--)
* slobodan prostor na disku se umanjuje (za jedan blok podataka). Takođe, simbolički link troši jedan i-node iz i-node tabele,
* broj linkova originalne datoteke se ne uvećava za jedan nakon linkovanja, već ostaje isti kao pre linkovanja,
* s obzirom da simbolički link može ukazivati na nepostojeći objekat, originalna datoteka se može obrisati sa diska bez obzira na broj simboličkih linkova koji upućuju na nju.
{% highlight bash %}
$ ln -s /etc dir_etc
$ ls -l dir_etc
lrwxrwxrwx 1 root root 4 Sep 5 14:40 dir_etc -> /etc
$ ls -l unexist
ls: unexis: No such file or directory
$ ln -s unexist junk
$ ls -l junk
lrwxrwxrwx 1 root root 15 Sep 5 14:40 junk -> unexist
{% endhighlight %}
![Linkovi.](/assets/os1/linkoviInode.jpg "Linkovi.")

## Vlasnička prava i kopiranje

* Vlasnik kopije je korisnik koji je pokrenuo komandu cp,
* Datoteka se dodeljuje primarnoj grupi korisnika koji je pokrenuo komandu cp,
* Pristupna prava kopije se dobijaju se logičkim množenjem bitova pristupnih prava originala i vrednosti promenljive **umask**. Na primer: ako su pristupna prava originalne datoteke 666, a vrednost promenljive umask 002, pristupna prava kopije biće 664, tj, za  other permisije menja se pravo na sledeći način
6<sub>8</sub> and not (2<sub>8</sub>) = (0110)<sub>2</sub> and not (0010)<sub>2</sub> = (0100)<sub>2</sub> = 4<sub>8</sub>
* maska se takođe može setovati komandom **umask maska**
* Sva tri vremena kopije (vreme kreiranja, poslednjeg pristupa i poslednje modifikacije) jednaka su vremenu pokretanja komande **cp**. Vreme poslednjeg pristupa originalne datoteke se takoće menja i jednako je vremenu pokretanja komande cp.
