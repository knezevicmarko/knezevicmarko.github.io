---
layout: lekcija
title: SHELL - alati za shell programiranje
main_category: Materijali za vežbe
sub_category: Linux
image: shell1.png
active: true
comment: true
archive: false
---

### wc
Brojanje reči karaktera i linija
{% highlight bash %}
$ wc [-cwl] filename
$ wc -l /etc/protocols
{% endhighlight %}

### Redirekcija ulaza i izlaza

UNIX komande primaju podatke sa **standardnog ulaza (stdin)**, rezultate izvršenja šalju na **standardni izlaz (stdout)**, a poruke o greškama na **standardni uređaj za greške (stderr)**. Većina UNIX komandi koristi tastaturu kao standardni ulaz, a monitor kao standardni izlaz i uređaj za greške.

**Ulaz komande preusmerava se pomoću znaka <** (manje od) na sledeći način:
{% highlight bash %}
$ command < inputdevice
{% endhighlight %}
Primer:
{% highlight bash %}
$ wc -l < /tmp/jsnmith.dat
{% endhighlight %}
**Za redirekciju izlaza se koristi znak >**. Ukoliko se redirekcija vrši u postojeću datoteku, datoteka se briše, a zatim se kreira nova u koju se smešta rezultat izvršenja komande. Za dodavanje izlaza na postojeću datoteku koristi se **znak >>**.
{% highlight bash %}
$  sort kyuss.txt > /dev/lp0
$  ls -l /home/jsmith > myfile
$  ls -l /tmp/jsmith >> myfile
$  >emptyfile
{% endhighlight %}

### Povezivanje komandi u pipeline

Pipeline funkcioniše na sledeći način: standardni izlaz komande sa leve strane znaka pipe (\|) postaje standardni ulaz komande sa desne strane znaka. Znak pipe zahteva komande i sa leve i sa desne strane, a razmaci između znaka i komande su proizvoljni.

Primer:

{% highlight bash %}
$ ls -l /etc/ > /tmp/files_in_etc
$ wc -l < /tmp/files_in_etc
145
{% endhighlight %}
{% highlight bash %}
$ ls -l /etc | wc -l
145
{% endhighlight %}

### tee komanda

Komanda tee omogućava da se izlaz komande šalje na više lokacija odjednom. Na primer, ako se želi redirekcija u neki fajl, i istovremeni ispis na ekran, može se uraditi sledeće:
{% highlight bash %}
ps -ef | tee /tmp/troubleshooting_file
{% endhighlight %}
Da bi se, umesto da se fajl obriše, tekući izlaz na njega dodao, treba dodati opciju -a:
{% highlight bash %}
ps -ef | tee -a /tmp/troubleshooting_file
{% endhighlight %}
Mogu se specificirati i više fajlova kao argumenti **tee** komande, *output* procesa će ići u svaki od tih fajlova.

### head
Prikaz početka fajla
{% highlight bash %}
$ head -n 3 /var/log/syslog
{% endhighlight %}

### tail
Prikaz kraja fajla
{% highlight bash %}
$ tail -n 5 /var/log/syslog
{% endhighlight %}
Za nadgledanje promena u fajlu koristi se opcija -f.

### join i split

Komanda `join` dozvoljava spajanje više fajlova po zajedničkom polju.

**Primer**:

{% highlight bash %}
$ cat file1.txt
1 Pera
2 Mika
3 Laza
$ cat file2.txt
1 Peric
2 Lazic
3 Mikic
$ join file1.txt file2.txt
1 Pera Peric
2 Mika Lazic
3 Laza Mikic
{% endhighlight %}
U ovom slučaju fajlovi su spojeni preko prvog polja i ona moraju da budu identična. Ako nisu onda mogu da se sortiraju tako da se spajanje, u ovom slučaju, vrši preko vrednosti 1, 2 i 3.

Ako su dati sledeći fajlovi:
{% highlight bash %}
$ cat file1.txt
Pera 1
Mika 2
Laza 3
$ cat file2.txt
1 Peric
2 Lazic
3 Mikic
{% endhighlight %}
Da bi izvršili njihovo spajanje potrebno je pozvati sledeću komandu:
{% highlight bash %}
$ join -1 2 -2 1 file1.txt file2.txt
1 Pera Peric
2 Mika Lazic
3 Laza Mikic
{% endhighlight %}
-1 se odnosi na file1.txt, a -2 na file2.txt.

Komandom `split` fajl se deli u odgovarajući broj različitih fajlova.
{% highlight bash %}
$ split somefile
{% endhighlight %}
Ovako se u novonastalim fajlovima nalazi po 1000 linija iz originalnog fajla i oni se zovu x\*\*.

### sort komanda

Korisna komanda za sortiranje izlaza neke komande ili fajla u specificiranom redosledu. Njene opcije su sledeće:

* -d 	Sorts via dictionary order, ignoring non-alphanumerics or blanks.
* -f 	Ignores case when sorting.
* -g 	Sorts by numerical value.
* -M 	Sorts by month (i.e., January before December).
* -r 	Provides the results in reverse order.
* -m 	Merges sorted files.
* -u 	Sorts, considering unique values only.

**Primer**:

* Kreirati fajl /tmp/outoforder sa sledećim sadržajem:
{% highlight bash %}
Zebra
Quebec
hosts
Alpha
Romeo
juliet
unix
XRay
xray
Sierra
Charlie
horse
horse
horse
Bravo
1
11
2
23
{% endhighlight %}
* Sortirati fajl u rečničkom redosledu:
{% highlight bash %}
sort -d /tmp/outoforder
{% endhighlight %}
Rezultat je:
{% highlight bash %}
1
11
2
23
Alpha
Bravo
Charlie
Quebec
Rome
Sierra
XRay
Zebra
horse
horse
horse
hosts
juliet
unix
xtra
{% endhighlight %}
* Treba primetiti da se string horse pojavljuje više puta. Da bi se uklonile te dodatne pojave, koristiti opciju -u:
{% highlight bash %}
sort -du /tmp/outoforder
{% endhighlight %}

## Korišćenje kontrolinih karaktera

Kontrolni karakteri se zadaju: <Ctrl> + karakter (<Ctrl> se na ekranu prikazuje kao simbol ^ (carret)). Kontrolni karakteri Bourne-again shella koji se najčešće koriste su:

* **\<Ctrl-c\>** 	prekida izvršenje procesa koji radi u prvom planu;
* **\<Ctrl-d\>** 	označava kraj fajla; napuštanje programa koji podatke čitaju sa standardnog ulaza
* **\<Ctrl-u\>** 	briše celu komandnu liniju;
* **\<Ctrl-w\>** 	briše zadnju reč u komandnoj liniji;
* **\<Ctrl-s\>** 	privremeno zaustavlja izvršenje procesa u prvom planu. Može se koristiti prilikom pregledanja sadržine nekog velikog direktorijuma komandom ls ili ukoliko se neka datoteka prikazuje na ekranu programom cat;
* **\<Ctrl-g\>** 	nastavlja se izvršenje procesa u prvom planu.

**Primer**: bc (basic calculator)
{% highlight bash %}
$ bc      # pokreće bc
100/5 		# inicira operaciju deljenja
20 			  # program bc prikazuje rezultat prethodne operacije
<Ctrl-d> 	# napuštanje programa i povratak u shell
$
{% endhighlight %}

# Promenljive

Na Linux sistemima postoje dva tipa promenljivih:

* **sistemske promenljive**, koje kreira i održava sam operativni sistem. Ne preporučuje se promena njihovog sadržaja (zašto?). Ovaj tip promenljivih definiše se strogo velikim slovima,
* **korisnički definisane promenljive** (User defined variables - UDV), koje kreiraju i održavaju korisnici. Ovaj tip promenljivih se obično definiše malim slovima.

**U shell programiranju promenljive se ne deklarišu za specifični tip podataka** - dovoljno je dodeliti vrednost promenljivoj i ona će biti alocirana prema toj vrednosti. U Bourne Again Shell-u, promenljive mogu sadržavati brojeve, karaktere ili nizove karaktera.

## Važnije sistemske promenljive

Sistemske promenljive mogu se videti pozivom komande **env**:
{% highlight bash %}
$ env
LC_PAPER=sr_RS
XDG_VTNR=7
MATE_DESKTOP_SESSION_ID=this-is-deprecated
LC_ADDRESS=sr_RS
SSH_AGENT_PID=2681
XDG_SESSION_ID=c2
LC_MONETARY=sr_RS
...
{% endhighlight %}
Neke od njih su:

* SHELL lokacija komandnog interpretera
* HOME home directorijum korisnika
* PATH putanja u kojoj se traže izvršne datoteke
* PWD tekući direktorijum

Pojedinačno, sadržaj promenljive može se videti pozivom:
{% highlight bash %}
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/bin/X11:/usr/games
{% endhighlight %}

## Definisanje korisničkih promenljivih

Svaka promenljiva je univerzalna i nema nikakvu deklaraciju tipa (integer, float, string) i definiše se na sledeći način:
{% highlight bash %}
variablename=value
{% endhighlight %}
Primer.
{% highlight bash %}
$ br=10
{% endhighlight %}
Prilikom definisanja, odnosno dodele vrednosti, potrebno je primeniti sledeće konvencije o imenima promenljivih:

* **Ime promenljive mora početi alfanumeričkim karakterom ili donjom crtom ‘_'** (underscore character) praćenim jednim ili više alfanumeričkih karaktera.

Primer.
Korektne promenljive su: HOME, SYSTEM_VERSION, br, \_ime;

* **Prazne karaktere ne treba stavljati ni sa jedne strane znaka jednakosti** prilikom dodele vrednosti promenljivim.

Primer.
{% highlight bash %}
$ br =10 # neispravno - prazni karakteri
$ br= 10 # neispravno - prazni karakteri
$ br = 10 # neispravno - prazni karakteri
$ br=10 # ispravno
{% endhighlight %}
case sensitive
{% highlight bash %}
$ bR=20
$ Br=30
$ echo $bR
20
{% endhighlight %}
Može se definisati **promenljiva nulte dužine (NULL variable)**, odnosno promenljiva koja nema vrednost u trenutku definisanja. Vrednosti ovih promenljivih se ne mogu prikazati komandom echo, sve dok im se ne dodeli vrednost;
{% highlight bash %}
$ br=
$ ime=""
{% endhighlight %}
* **Imena promenljivih ne smeju sadržati specijalne znake** (poput ? i \*).

Primer 1.
{% highlight bash %}
$ echo $ime # prikazuje vrednost promenljive ime
johnny
$ echo ime # prikazuje string ime
ime
{% endhighlight %}
Primer 2.
{% highlight bash %}
$ x=10
$ xn=abc
$ echo $x $abc
10 abc
{% endhighlight %}

### Navodnici

Bash shell prepoznaje tri tipa navodnika, i to:

* **Dvostruki navodnici** - "Double Quotes". Sve što se nalazi u ovim navodnicima gubi originalno značenje (osim \ i $).
* **Jednostruki navodnici** - 'Single quotes'. Sve što je zatvoreno jednostrukim navodnicima ostaje nepromenjeno.
* **Obrnuti navodnici** - \`Back quote\`. Izraz zatvoren obrnutim navodnicima tretira se kao komanda koju treba izvršavati.

**Primer.**
{% highlight bash %}
$ echo "Danasnji datum : date" # tretira date kao string
Danasnji datum : date
$ echo "Danasnji datum : `date`" # tretira date kao komandu
Danasnji datum : Fri Apr 2 16:30:35 CEST 2004
{% endhighlight %}

# Regularni izrazi i metakarakteri

Regulrni izrazi su sintaksički skup fraza koje reprezentuju šablone unutar teksta ili stringova. Regularni izrazi omogućavaju reprezentaciju različitih nizova karaktera vrlo malim skupom predefinisanih karaktera. Često sadrže i **metakaraktere** - karaktere koji reprezentuju drugu grupu karaktera i komandi.

Fajl koji će se koristiti u svrhu testiranja je recimo **/tmp/testfile** (obratiti pažnju na interpunkciju i mala i velika slova):
{% highlight bash %}
Juliet Capulet
The model identifier is DEn5c89zt.
Sarcastic was what he was.
No, he was just sarcastic.
Simplicity
The quick brown fox jumps over the lazy dog
It’s a Cello? Not a Violin?
This character is (*) is the splat in Unix.
activity
apricot
capulet
cat
celebration
corporation
cot
cut
cutting
dc9tg4
eclectic
housecat
persnickety
The punctuation and capitalization is important in this example.
simplicity
undiscriminating
Two made up words below:
c?t
C?*.t
cccot
cccccot
{% endhighlight %}

### Metakarakteri

Metakarakteri su korisni u redukciji količine teksta koji se koristi sa komandama i za reprezentaciju tekstualnih grupa minimalnim skupom karaktera. Sledeći metakarakteri su u široj upotrebi:

* **. - Tačka. Reprezentuje jedan karakter.**
Primer: Pronaći bilo koju pojavu slova c i slova t sa tačno jednim karakterom između.
{% highlight bash %}
$ grep c.t /temp/testfile
Simplicity
apricot
cat
cot
cut
cutting
dc9tg4
housecat
simplicity
c?t
cccot
cccccot
{% endhighlight %}

* **[] - uglaste zagrade. Rezultat odgovara bilo kojem karakteru unutar zagrada.**

Primer: Pronaći bilo koju instancu slova c i slova t sa tačno jednim samoglasnikom između.
{% highlight bash %}
$ grep c[aeiou]t /temp/testfile
Simplicity
apricot
cat
cot
cut
cutting
housecat
simplicity
cccot
cccccot
{% endhighlight %}

* **\* - zvezda. Reprezentuje nula ili više pojava bilo kojih karaktera.**

Primer: Pronaći sve instance slova c i slova t sa nula ili više karaktera između njih.
{% highlight bash %}
$ grep c*t /temp/testfile
Juliet Capulet
The model identifier is DEn5c89zt.
Sarcastic was what he was.
No, he was just sarcastic.
Simplicity
The quick brown fox jumps over the lazy dog
It’s a Cello? Not a Violin?
This character is (*) is the splat in Unix.
activity
apricot
capulet
cat
celebration
corporation
cot
cut
cutting
dc9tg4
eclectic
housecat
persnickety
The punctuation and capitalization is important in this example.
simplicity
undiscriminating
c?t
C?*.t
cccot
cccccot
{% endhighlight %}

* **[^karakteri] - uglaste zagrade sa kapom između. Nijedan od navedenih karaktera se NE pojavljuje.**

Primer: Pronaći sve pojave karaktera c i karaktera 5, a da između njih ne stoji nikakav samoglasnik.

{% highlight bash %}
$ grep c[^aeiou]t /temp/testfile
dc9tg4
c?t
{% endhighlight %}

* **^karakter - Odgovara sekveci samo ako je na početku linije.**

Primer: Pronaći sve pojave stringa ca na početku linije.
{% highlight bash %}
$ grep ^ca /temp/testfile
capulet
cat
{% endhighlight %}
Bez karaktera ^, sekvenca ca bi mogla da se nađe bilo gde u stringu:
{% highlight bash %}
Sarcastic was what he was.
No, he was just sarcastic.
capulet
cat
housecat
The punctuation and capitalization is important in this example.
{% endhighlight %}

* **^[karakter(i)] - Kapa ispred sekvence u uglastim zagradama. Odgovara bilo kom karakteru u uglastim zagradama, ali na početku linije.**

Primer: Pronaći sve pojave slova c praćenog samoglasnikom i slovom t na početku linije.
{% highlight bash %}
$ grep ^c[aoiue]t /temp/testfile
cat
cot
cut
cutting
{% endhighlight %}
Da je izostavljen karakter ^, sekvenca bi mogla biti bilo gde u liniji:
{% highlight bash %}
Simplicity
apricot
cat
cot
cut
cutting
housecat
simplicity
cccot
cccccot
{% endhighlight %}
* **$ - Znak dolara. Odgovara pojavi sekvence karaktera na kraju linije.**

Primer: Pronaći linije koje se završavaju slovima c i t između kojih može biti bilo šta.
{% highlight bash %}
$ grep c*t$ /temp/testfile
Juliet Capulet
apricot
capulet
cat
cot
cut
housecat
c?t
C?*.t
cccot
cccccot
{% endhighlight %}
* **\ - Backslash. Anulira specijalno značenje karaktera koji ga neposredno sledi.**

Primer: Pronaći sve pojave sekvence karaktera c?t.

* **? - Upitnik. Reprezentuje nula ili jednu pojavu karaktera (ne treba ga mešati sa *, koji odgovara nula, jednom, ili više karaktera). Ne podržavaju ga svi UNIX programi.**

Primer: Pronaći sve pojave karaktera c i karaktera t sa jednim ili nijednim karakterom između njih.

* **[a-z] - Potpuna engleska abeceda malim slovima. Odgovara bilo kojem slovu abecede.**

Primer: Pronaći sve pojave karaktera c i t sa bilo kojim slovom između njih.

* **[0-9] - Odgovara bilo kojoj cifri.**

Primer: Pronaći sve pojave karaktera c i t sa bilo kojom cifrom između njih.

* **[d-m7-9] - Odgovara jednom pojavljivanju bilo kog karaktera iz opsega d-m ili 7-9. Primer ilustuje grupisanje komandi.**

Primer: Pronaći sve pojave slova c i slova t, sa jednim karakterom između koji može biti bilo koje slovo u opsegu od c do t ili cifra između 0 i 4.

### Prošireni regularni izrazi (extended regular expressions)

Kod proširenih regularnih izraza koji se aktiviraju npr. navođenjem opcije -E grep komandi, a dostupni su i u programima, awk, emacs, vi, itd. Karakteristični su sledeći metakarakteri koji služe za označavanje ponavljanja karaktera ili podizraza (blokova):

* **(karakteri)** - označeni podizraz (blok).
* **+** - Izraz od jednog karaktera nakon kojeg sledi "+" sparuje jednu ili više kopija izraza. Na primer, "ab+c" sparuje "abc", "abbbc" itd. "[xyz]+" sparuje "x", "y", "zx", "zyx", i tako dalje
* **{x,y}** -	Sparuje poslednji blok barem "x" i ne više od "y" puta. Na primjer, "a{3,5}" sparuje "aaa", "aaaa" ili "aaaaa"
* **{x}** - Prethodni karakter se pojavljuje tačno x puta
* **{x,}** - Prethodni karakter se pojavljuje najmanje x puta
* **{,y}** - Prethodni karakter se pojavljuje najviše y puta
* **?** - prethodni karakter je opcioni i pojavljuje se najviše jednom.

**Složen primer:**

Fajl proba.txt:
{% highlight bash %}
www8.dobarsajt1.edu
www678.amu.ac.zu5l
www6.ailmit.net
www.euler.ni.ac.yu
mitcl.edu
www.core.amu.edu
www.znanje.edu
{% endhighlight %}
Izlaz komande:
{% highlight bash %}
$ grep -E "^(w{3}[0-9]*\.)?[a-z]{1}[a-z0-9]{1,7}((\.ac\.[a-z]{2})|(\.edu))$" proba.txt
mitcl.edu
www.znanje.edu
{% endhighlight %}
**Objašnjenje**:
Navedeni regularni izraz znači da string može, a ne mora (zbog “?”) početi sekvencom “www”+bilo koje cifre nakon čega sledi tačka. Iza tačke mora biti slovo, a zatim sekvenca od najmanje 1, a najviše 7 alfanumerika (slova ili brojeva), dok na kraju mora biti domen “.ac.(dva bilo koja slova)” ili “.edu”.

## Domaći zadaci

### Prvi zadatak

Napisati regularni izraz koji pronalazi decimalne brojeve.

**Primer**

{: .w3-table .w3-margin .w3-card-4}
|-----|------------------|
| decimalni broj | 3.14529 |
| decimalni broj | -255.34 |
| decimalni broj | 128 |
| decimalni broj | 1.9e10 |
| decimalni broj | 123,340.00 |
| nije decimalni broj | 720p |

### Drugi zadatak

Napisati regularni izraz koji pronalazi brojeve telefona.

**Primer ispravnih brojeva telefona**
{% highlight bash %}
415-555-1234
650-555-2345
(416)555-3456
202 555 4567
4035555678
1 416 555 9292
{% endhighlight %}
