---
layout: lekcija
title: SHELL - osnove programiranja 1
main_category: Materijali za vežbe
sub_category: Linux
image: shell.gif
active: true
comment: true
archive: false
---

### Inicijalizacija programa

Nakon interpretacije komandne linije shell inicira izvršenje zadate komande. Ukoliko komanda nije interna (ugrađena u shell, poput komande cd) shell traži izvršni fajl koji odgovara imenu komande u direktorijumima navedenim u sistemskoj putanji (koja se može dobiti komandom **echo $PATH**). Nakon toga shell pokreće program i prosleđuje mu argumente i opcije navedene u komandnoj liniji.
Napomena: Ukoliko se izvršni fajl nalazi u tekućem direktorijumu ili u nekom direktorijumu koji nije u sistemskoj putanji ($PATH), ime komande se mora zadati sa putanjom. Slede i primeri koji ilustruju pokretanje programa koji se nalaze u tekućem direktorijumu i direktorijumu /usr/sbin:
{% highlight bash %}
$ ./myscript
$ /usr/sbin/useradd
{% endhighlight %}


### Primer najjednostavnijeg skripta

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

### awk komanda

Awk je, u stvari, jednostavan programski jezik namenjen procesiranju teksta, tj. transformisanju teksta u formatirani output. Awk uzima sva ulaza:

* komandu, skup komandi ili komandni fajl koji sadrže instrukcije za poklapanje tekstualnih šablona i smernice za procesiranje i generisanje izlaza.
* podatke sa kojima se radi ili fajl sa podacima.

Prvi primer je:
{% highlight bash %}
$ awk ‘{ print $0 }’ /etc/passwd
{% endhighlight %}
Rezultat izvršenja je nešto kao:
{% highlight bash %}
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
...
{% endhighlight %}
U gornjem primeru AWK ne procesira nikakve podatke, već prosto čita sadržaj **/etc/passwd** fajla i štampa nefilterisan izlaz na *stdout*, isto kao **cat** komanda. Kada se pozove AWK, opskrbljen je sa dve informacije, a to je komanda za editovanje i podaci za editovanje. Dakle, u primeru je **/etc/passwd** reprezent ulaznih podataka, a komanda za uređivanje prosto štampa (**$0 je oznaka za celu liniju teksta**) fajl u istom redosledu.

Prava primena AWK-a leži u odvajanju traženih delova iz većeg skupa podataka. Na primer, iz fajla /etc/passwd može se dobiti daleko čitljiviji izlaz:
{% highlight bash %}
awk -F”:” ‘{ print “username: “ $1 “\t\t\t user id:” $3 }’ /etc/passwd
{% endhighlight %}
Razultat bi bio nešto kao:
{% highlight bash %}
username: root			 user id:0
username: daemon			 user id:1
username: bin			 user id:2
username: sys			 user id:3
username: sync			 user id:4
username: games			 user id:5
username: man			 user id:6
{% endhighlight %}
Podrazumevano, AWK koristi blanko za separator ulaznih podataka, ali se ovo ponašanje može promeniti opcijom **-F**. Ovde je kao separator iskorišćen karakter “:”. **$1** sadrži tekst do prvog separatora, **$2** tekst do drugog separatora itd. **$0** je uvek cela linija.

AWK komanda za editovanje se uvek sastoji iz dva dela:

* **šabloni**
* **komande**

Šabloni se upoređuju sa linijama u fajlu, a ako šablon nije naveden, kao u gornjem primeru, sve linije dolaze u obzir.

#### Rad sa šablonima

Šabloni u AWK-u sastoje se od teksta i jednog ili više regularnih izraza između karaktera “/” (slash). Na primer:
{% highlight bash %}
# String example
/text pattern/
# Reg Ex example match any lowercase chars
/[a-z]/
{% endhighlight %}
Sledeća komanda:
{% highlight bash %}
$ awk -F":" '/^m/ { print "username: " $1 "\t\t\t user id:" $3 }' /etc/passwd
username: man			 user id:6
username: mail			 user id:8
username: messagebus			 user id:106
username: marko			 user id:1000
{% endhighlight %}
kao što se vidi, u obzir uzima samo linije sa početnim karakterom “m”.

#### Komande

Uobičajene komande AWK-a su **=, print, printf, if, while, i for**. Ove instrukcije se ponašaju kao odgovarajuće instukcije bilo kog programskog jezika, omogućavajući dodelu vrednosti varijablama, štampanje izlaza i kontrolu toka.

#### Programiranje pomoću AWK-a

AWK komande koje se navode u komandnoj liniji mogu se snimiti i u fajl, na primer:

* Tekst editorom kreirati fajl print.awk sa sledećim sadržajem:
{% highlight bash %}
BEGIN {
FS=”:”
}
{ printf “username: “ $1 “\t\t\t user id: “ $3 }
{% endhighlight %}
* Izvršiti komandu na sledeći način:
{% highlight bash %}
$ awk -f print.awk /etc/passwd
username: root user id:0
username: bin user id:1
…
{% endhighlight %}
Pošto je AWK strukturirani jezik, fajl mora da sadrži određene blokove, i to:

1. **Komande koje se izvršavaju samo jednom na početku, blok počinje ključnom rečju BEGIN**. U gornjem primeru u tom delu se postavlja separator,
2. **Komande za poklapanje šablona koje se izvršavaju po jedanput za svaku liniju ulaza**. U gornjem primeru to je blok koji sadrži print komandu.
3. **Komande koje se izvršavaju samo jednom na kraju, blok počinje ključnom rečju END**. U gornjem primeru taj deo je izostavljen, a mogao bi da glasi:
{% highlight bash %}
END {
printf “Zavrseno procesiranje fajla /etc/passwd”
}
{% endhighlight %}

**FS (field separator)** je jedna od nekoliko specijalnih varijabli AWK-a. Još neke su:

* **NF** - Variable for providing a count as to the number of words on a specific line.
* **NR** - Variable for the record being processed. That is, the value in NR is the current line in a file awk is working on.
* **FILENAME** - Variable for providing the name of the input file.
* **RS** - Variable for denoting what the separator for each line in a file is.

**Primer 1.**
{% highlight bash %}
$ cat /tmp/data
123abc
Wile E.
aabcc
Coyote
$ awk '/abc/ {print}' /tmp/data
123abc
aabcc
{% endhighlight %}
U ovom slučaj awk traži uzorak ‘abc’ u datoteci **/tmp/data**, a akcija koja se pri tom obavlja nad nađenim uzorcima je prikazivanje teksta na ekranu (**print**).

**Primer 2.**

Drugi primer je štampanje broja linija koje sadrže string “abc”.
{% highlight bash %}
$ awk '/abc/ {i=i+1} END {print i}' /tmp/data
2
{% endhighlight %}
Ukoliko se u datoteci traži više uzoraka i ukoliko se vrši više obrada, potrebno je prvo napraviti datoteku u kojoj su opisane akcije (na primer actionfile.awk). Prilikom zadavanja komande awk potrebno je zameniti tekst između navodnika, kojim su opisani uzorak i akcija, imenom datoteke: '-f actionfile.awk'.


### Komanda expr
{% highlight bash %}
$ expr 6 + 3   # expr posmatra 6 + 3 kao matematički izraz
9
{% endhighlight %}
Komanda expr određuje rezultat neke matematičke operacije. Sintaksa komande expr je:
{% highlight bash %}
expr op1 operacija op2
{% endhighlight %}
gde su op1 i op2 celi brojevi, a operator +, -, \*, /, &. Rezultat operacije je ceo broj. Argumenti op1, op2 i operator se moraju razdvojiti praznim karakterom.
{% highlight bash %}
$ expr 6 + 3   # ispravno
9
{% endhighlight %}

### Specijalne promenljive

Ove promenljive su rezervisane za specifične funkcije. Na primer, karakter $ reprezentuje proces ID (PID) tekućeg shell-a.
Ako se ukuca
{% highlight bash %}
$ echo $?
{% endhighlight %}
u shell promptu, dobiće se izlazni status poslednje komande.

Specijalne promenljive koje mogu da se koriste u bash skriptama.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Promenljiva | Funkcija |
|-------------|----------|
| ? | Izlazni status prethodne komande |
| $ | PID tekućeg shell procesa |
| - | Opcije sa kojima je pozvan aktuelni shell |
| ! |	PID poslednje komande koja radi u pozadini |
| 0 |	ime fajla tekućeg skripta |
| 1-9 |	argumenti komandne linije dati tekućem skriptu $1 je prvi argument $9 deveti |
| _ |	poslednji argument dat prethodno pozvanoj komandi |

### Čitanje podataka sa ulaza - read

Komanda read se koristi za čitanje ulaznih podataka sa tastature i memorisanje unete vrednosti u promenljivu. Sintaksa komande je:
{% highlight bash %}
$ read varible1, varible2,...varibleN
{% endhighlight %}
Primer.
{% highlight bash %}
#
# ss2.sh: upotreba komande read
#
echo "Unesite podatak:"
read var1
echo "Uneli ste: $var1"
{% endhighlight %}
Pokretanje:
{% highlight bash %}
$ bash ss2.sh
Unesite podatak: 123
Uneli ste: 123
{% endhighlight %}

### Izlazni status komandi

Nakon izvršenja Linux komande vraćaju vrednost na osnovu koje se može odrediti da li je komanda izvršena uspešno ili ne. Ako je povratna vrednost 0, komanda je izvršena uspešno. Ako je povratna vrednost različita od 0 (veća od 0), komanda se nije uspešno završila, a taj broj predstavlja neku vrstu dijagnostičkog statusa koja se naziva izlazni status.
{% highlight bash %}
$ rm plumph
rm: cannot remove `plumph`: No such file or directory
$ echo $?
1 # izlazni status 1 -> komanda izvršena s greškom
$ date
$ echo $?
0 # izlazni status 0 -> komanda izvršena bez greške
{% endhighlight %}

### Argumenti

Komanda može biti zadata bez parametara (npr. **date, clear, who**), kao i sa jednim ili više parametara (**ls -l, ls -l /etc, mount -t ntfs /dev/hda1 /mnt/winc**).

Promenljiva **$#** memoriše broj argumenata specifirane komandne linije, a **$\*** ili **$@** upućuju na sve argumente koji se prosleđuju shell programu.

Komandni argumenti se na isti način mogu zadati i shell script-u.

Na primer:
{% highlight bash %}
$ ss3.sh arg1 arg2 arg3
{% endhighlight %}
Argumente ovako pozvanog script-a se može pamte sledeće promenljive:

* $0 je ime programa - ss3.sh
* $1 je prvi komandni argument - arg1
* $2 je drugi komandni argument - arg2
* $3 je treći komandni argument - arg3
* $# je broj komandnih argumenta - 3
* $\* su svi komandni argumenti. $\* se proširuje u `$1,$2...$9` - arg1 arg2 arg3.

Primer.
{% highlight bash %}
$ df
$ less /etc/passwd
$ ls -l /etc
$ mount -r /dev/hda2 /mnt/winc
{% endhighlight %}

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| ime programa $0 | broj argumenata $# | $1 | $2 | $3 |
|-----------------|--------------------|----|----|----|
| df | 0 | | | |
| less | 1 | /etc/passwd | | |
| ls | 2 | -l | /etc | |
| mount | 3 | -r | /dev/hda2 | /mnt/winc |

Primer.
{% highlight bash %}
#
# ss3.sh: korišcenje argumenata komandne linije
#
echo "Ukupan broj argumenata komandne linije: $#"
echo "$0 je ime programa, a $1 je prvi argument."
echo "Svi argumenti su redom: $*"
{% endhighlight %}
Pokretanje:
{% highlight bash %}
$ bash ss3.sh arg1 arg2 arg3
{% endhighlight %}

### Kontrola toka

Rad sa promenljivama je koristan, ali mora se proširiti odgovarajućom kontrolom toka u vidu petlji i uslova kako bi se dobio zaista koristan skript koji nešto radi. Uobičajene su dve vrste kontrole toka:

* kondicione i
* iterativne.

#### Kondiciona kontrola toka pomoću if-then izraza

U opštem slučaju, if-then konstrukcija izgleda ovako:
{% highlight bash %}
if [uslov]
then
	naredbe
fi
{% endhighlight %}
**OBRATITI PAŽNJU NA RAZMAKE I NOVE REDOVE!** If-then blok se završava naredbom **fi** (obrnuto od **if**).

Primer.
{% highlight bash %}
#!/bin/bash
echo 'Guess the secret color'
read COLOR
if [ $COLOR = 'purple' ]
then
	echo 'You are correct.'
fi
{% endhighlight %}
Više uslova može se dodati korišćenjem klauzule **else**:

Primer.
{% highlight bash %}
#!/bin/bash
echo 'Guess the secret color'
read COLOR
if [ $COLOR = 'purple' ]
then
echo 'You are correct.'
else
echo 'Your guess was incorrect.'
fi
{% endhighlight %}
Takođe, prisutna je i klauzula **elif**:
{% highlight bash %}
#!/bin/bash
echo 'Guess the secret color'
read COLOR
if [ $COLOR = 'purple' ]
then
	echo 'You are correct.'
elif [ $COLOR = 'blue' ]
then  echo 'You’re close.'
else
	echo 'Your guess was incorrect.'
fi
{% endhighlight %}
Instrukcija **elif** može biti proizvoljno mnogo, što je posebno zgodno kod npr. ispitivanja poslatih argumenata komandne linije. Evo još nekih primera korišćenja **if** klauzule u ugnežđenom obliku i složenih uslova dobijenih logičkim operatorima:

Multiple Conditions:
{% highlight bash %}
if [ condition1 ]
then
	if [ condition2 ]
	then
		some action
	fi
fi
{% endhighlight %}
Ili isto to, ali jednostavnije, korišćenjem logičkih operatora:
{% highlight bash %}
if [ condition1 && condition2 ]
then
	some action
fi
{% endhighlight %}
Isto kao u C-u, postoji i logički operator  “\|\|”:
{% highlight bash %}
if [ condition1 || condition2 ]
then
	some action
fi
{% endhighlight %}
što je ekvivalentno sledećem:
{% highlight bash %}
if [ condition1 ]
then
	some action
elif [ condition2 ]
then
	the same action
fi
{% endhighlight %}

#### Komanda test

Komanda test se koristi za evaluaciju uslova. Naime, primećuje se da svi navedeni primeri uslova uključuju uglaste zagrade oko uslova koji se izračunava. **Uglaste zagrade su praktično ekvivalentne komandi test**. Na primer, uslov iz prethodnog skripta bi mogao da glasi i ovako:
{% highlight bash %}
if ( test $COLOR = 'purple' )
{% endhighlight %}
Ovaj koncept je važan iz razloga što se veliki broj opcija komande test može iskoristiti za testiranje različitih uslova. Na primer, provera da li neki fajl postoji na fajl sistemu:
{% highlight bash %}
if ( test -e filename )
{% endhighlight %}
što je isto što i
{% highlight bash %}
if [ -e filename ]
{% endhighlight %}
U slučaju da fajl postoji , vraća se true, tj. izlazni status 0, a ako ne postoji vraća se false, tj. 1. Sledeća tabela daje pregled najkorišćenijih opcija **test** komande:

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Opcija | Uslov |
|--------|-------|
| -d | Specificirani fajl postoji i u pitanju je direktorijum |
| -e | Specificirani fajl postoji |
| -f | Fajl postoji i u pitanju je regularni fajl (a ne direktorijum ili drugi specijalni fajl) |
| -G | Fajl postoji i vlasništvo je efektivnog GID-a |
| -nt | Fajl je noviji od drugog navedenog fajla (sintaksa je file1 -nt file2). |
| -ot | Fajl je stariji od drugog navedenog fajla (sintaksa je file1 -nt file2). |
| -O | Korisnik koji izvršava komandu je vlasnik fajla |
| -r | Korisnik koji izvršava komandu ima dozvolu čitanja nad fajlom |
| -s | Fajl postoji i nije prazan |
| -w | Korisnik koji izvršava komandu ima dozvolu pisanja |
| -x | Korisnik koji izvršava komandu ima dozvolu izvršavanja |

Sa svim ovim opcijama, komanda će vratiti ili **true** ili **false**.

#### Operatori poređenja
Operatori poređenja rade isto kao i u bazičnoj aritmetici. Stringovi se takođe mogu porediti, standardno u abecednom redosledu.

*	= jednako
*	!= različito
*	\> veće od
*	< manje od

#### Case izraz

Opšti format case komande je:
{% highlight bash %}
case expression in
pattern1)
	action1
	;;
pattern2)
	action2
	;;
pattern3)
	action3
	;;
esac
{% endhighlight %}
**Treba primetiti da se svaka sekcija zaključuje duplim znakom “;”**. Case izrazi su odlični za razvrstavanje argumenata komandne linije. Sledeći skrip je uzet samo kao primer, a služi za kontrolu NFS (Network File System) servisa.
{% highlight bash %}
# See how we were called.
case “$1” in
start)
	# Start daemons.
	action $”Starting NFS services: “ /usr/sbin/exportfs -r
	echo -n $”Starting NFS quotas: “
	daemon rpc.rquotad
	echo
	echo -n $”Starting NFS mountd: “
	daemon rpc.mountd $RPCMOUNTDOPTS
	echo
	echo -n $”Starting NFS daemon: “
	daemon rpc.nfsd $RPCNFSDCOUNT
	echo
	touch /var/lock/subsys/nfs
	;;
stop)
	# Stop daemons.
	echo -n $”Shutting down NFS mountd: “
	killproc rpc.mountd
	echo
	echo -n $”Shutting down NFS daemon: “
	killproc nfsd
	echo
	action $”Shutting down NFS services: “ /usr/sbin/exportfs -au
	echo -n $”Shutting down NFS quotas: “
	killproc rpc.rquotad
	echo
	rm -f /var/lock/subsys/nfs
	;;
status)
	status rpc.mountd
	status nfsd
	status rpc.rquotad
	;;
restart)
	echo -n $”Restarting NFS services: “
	echo -n $”rpc.mountd “
	killproc rpc.mountd
	daemon rpc.mountd $RPCMOUNTDOPTS
	/usr/sbin/exportfs -r
	touch /var/lock/subsys/nfs
	echo
	;;
reload)
	/usr/sbin/exportfs -r
	touch /var/lock/subsys/nfs
	;;
probe)
	if [ ! -f /var/lock/subsys/nfs ] ; then
	echo start; exit 0
	fi
	/sbin/pidof rpc.mountd >/dev/null 2>&1; MOUNTD=”$?”
	/sbin/pidof nfsd >/dev/null 2>&1; NFSD=”$?”
	if [ $MOUNTD = 1 -o $NFSD = 1 ] ; then
	echo restart; exit 0
	fi
	if [ /etc/exports -nt /var/lock/subsys/nfs ] ; then
	echo reload; exit 0
	fi
	;;
*)
	echo $”Usage: $0 {start|stop|status|restart|reload}”
	exit 1
esac
{% endhighlight %}

#### Iterativna kontrola - while petlja
Ponaša se isto kao u jeziku C:
{% highlight bash %}
#!/bin/bash
echo 'Guess the secret color: red, blue, yellow, purple, or orange \n'
read COLOR
while [ $COLOR != 'purple' ]
do
	echo 'Incorrect. Guess again. \n'
	read COLOR
done
echo 'Correct.'
{% endhighlight %}

#### Iterativna kontrola - until izraz

Razlikuje se od **while** petlje samo po uslovu. Prethodni primer bi mogao da se napiše i ovako:
{% highlight bash %}
#!/bin/bash
echo 'Guess the secret color: red, blue, yellow, purple, or orange \n'
read COLOR
until [ $COLOR = 'purple' ]
do
echo 'Incorrect. Guess again. \n'
read COLOR
done
echo 'Correct.'
{% endhighlight %}

#### Iterativna kontrola - for petlja

Sintaksa for petlje ne liči mnogo na sintaksu C jezika:
{% highlight bash %}
for name [in words ...];
do
	commands;
done
{% endhighlight %}
Promenljiva name dobija vrednost tekućeg člana liste **word**. Ako se “**in words**” izostavi u naredbi select, ili ako se specificira 'in "$@"', tada će name uzimati vrednost pozicionih parametara. **Izlazni status for petlje jednak je izlaznom statusu zadnje izvršene komande u grupi commands**. Ako je lista words prazna nijedna komanda se neće izvršiti i tada će izlazni status biti 0.
{% highlight bash %}
#
# ss11.sh: upotreba for petlje
#
if [ $# -eq 0 ]
then
	echo "Greška - numericki argument nije naveden"
	echo "Sintaksa : $0 broj"
	echo "Program prikazuje tablicu množenja za dati broj"
	exit 1
fi
n=$1
for i in 1 2 3 4 5 6 7 8 9 10
do
	echo "$n * $i = `expr $i \* $n`"
done
Sledeći primer ilustruje upotrebu alternativne for petlje:
for i in `seq 1 10`
do
	echo $i
done
{% endhighlight %}
For petlja najpre kreira promenljivu **i**, a zatim joj redom dodeljuje vrednosti iz liste (u ovom slučaju numeričke vrednosti od 1 do 10). Shell izvršava echo naredbu za svaku vrednost promenljive i.

#### Naredba select

Select naredba služi za prikaz menija čije su stavke definisane u **words** i prihvatanje izbora od korisnika.
{% highlight bash %}
select name [in words ...];
do
	commands;
done
{% endhighlight %}
Lista reči se proširuje generišući listu stavki (item). Skup proširenih reči prikazuje se na standardnom izlazu za greške, pri čemu svakoj prethodi redni broj. Ako se 'in words' izostavi u naredbi select, ili ako se specificira 'in "$@"', tada se prikazuju pozicioni parametri. U slučaju 'in "$@"' PS3 prompt se prikazuje i linije se čitaju sa standardnog ulaza. Ako se linija sastoji od broja koji odgovara jednoj od prikazanih reči tada se vrednost promenljive **name** postavlja u tu reč. Ukoliko je linija prazna reč i prompt se prikazuju ponovo. Ako se pročita EOF select komanda završava rad. Svaka druga pročitana vrednost uzrokuje da promenljiva **name** bude postavljena na nulu. Pročitana linija se čuva u promenljivoj **REPLY**.

Komande se izvršavaju posle svake selekcije sve dok se ne izvrši **break** komanda, čime se komanda select završava.

**Primer ilustruje upotrebu naredbe select**: program dozvoljava korisniku da sa tekućeg direktorijuma izabere datoteku čije će ime i indeks nakon toga biti prikazani.
{% highlight bash %}
select fname in *;
do
	echo Datoteka: $fname \($REPLY\)
	break;
done
{% endhighlight %}
Sledeći primer ilustruje kreiranje prostog menija:
{% highlight bash %}
opcije="Pozdrav Kraj"
select op in $opcije;
do
	if [ "$op" = "Kraj" ];
	then
		echo OK.
		exit
	elif [ "$op" = "Pozdrav" ];
	then
		echo Linux Rulez !
	else
		clear
		echo Opcija ne postoji.
	fi
done
{% endhighlight %}

### STDIN, STDOUT, i STDERR

Svaki put kada se otvori shell, Unix otvara tri fajla koja program koristi:

* STDIN (standard in) - uglavnom tastatura terminala
* STDOUT (standard out) - uglavnom monitor terminala
* STERR (standard error) - obično je i ovo monitor terminala

Bažno je zapamtiti da se podrazumevano ulaz uzima sa tastature i štampa na ekran. Evo ponovljenog skupa operatora za redirekciju:

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Operator | Akcija |
|----------|--------|
| \> | Redirektuje STDOUT u fajl |
| < |	Redirektuje STDIN iz fajla |
| >> | Dodaje STDOUT fajlu |
| \| | Uzima izlaz iz jednog programa i šalje kao ulaz drugom |
| << graničnik |	Pridružuje tekući ulazni tok STDIN-u dok se ne dostigne odgovarajući graničnik |

Primeri.
{% highlight bash %}
$ ls > fileList
{% endhighlight %}
STDOUT se upisuje u fajl fileList.
{% highlight bash %}
$ ls /home/student >> fileList
{% endhighlight %}
STDOUT se dodaje u fajl fileList.
{% highlight bash %}
$ ls | wc
{% endhighlight %}
Pipeline koji broji reči u izlazu komande **ls**.
{% highlight bash %}
cat <<KRAJ
The cat
Sat on the
Mat.
KRAJ
{% endhighlight %}
Gornji primer koristi poslednji navedeni operator redirekcije ulaza sa graničnikom. Sve dok se ne unese reč KRAJ, vrši se redirekcija iz STDIN. Komanda **cat** zatim štampa uneti sadržaj.
{% highlight bash %}
$ ls >& fileList
{% endhighlight %}
**Korišćenje >& znači da se se i STDOUT i STDERR usmeravaju u navedeni fajl**.

Standardni ulaz (STDIN), standardni izlaz (STDOUT) i standardni izlaz za greške (STDERR) su deskriptori datoteke kojima su dodeljeni brojevi po sledećim pravilima:

* 0 predstavlja STDIN,
* 1 predstavlja STDOUT i
* 2 predstavlja STDERR.

**Još primera redirekcije**:

Sledeći primer demonstrira kreiranje datoteke greperr.txt i upis poruka o greškama koje proizvodi komanda grep u datoteku:
{% highlight bash %}
$ grep kyuss * 2> greperr.txt
{% endhighlight %}
Redirekcija STDERR u STDOUT je demonstrirana sledećim primerom. Rezultat izvršenja komande **grep** smešta se u fajl i može se naknadno videti, a poruke o greškama koje komanda grep proizvodi prikazuju se na standardnom izlazu, a to je u podrazumevanom stanju ekran;
{% highlight bash %}
$ grep kyuss * > greperr.txt 2>&1
{% endhighlight %}
Ova vrsta redirekcije je korisna za programe koji rade u pozadini, tako da se od njih očekuje da poruke ne upisuju na ekran, već u neku datoteku. Dodatno, ukoliko korisnik ne želi da vidi "feedback"komande, izlaz i poruke o greškama mogu se preusmeriti na uređaj /dev/null, kao u sledećem primerom:
{% highlight bash %}
$ rm -f $(find / -name core) &> /dev/null
{% endhighlight %}
