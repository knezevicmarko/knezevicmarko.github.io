---
layout: lekcija
title: Korisnici i grupe
main_category: Materijali za vežbe
sub_category: Linux
image: linux1.png
active: false
comment: true
---

Svakom korisniku Linux sistema dodeljen je **jedinstven celobrojni identifikator – UID** (user ID) na osnovu kog kernel identifikuje korisnike. Ovaj metod predstavljanja kernelu karakterističan je za većinu operativnih sistema, uzevši u obzir da procesor brže radi sa brojnim vrednostima. Posebna baza podataka, koja radi u korisničkom režimu rada, dodeljuje tekstualna imena ovim numeričkim vrednostima, odnosno uparuje UID sa konkretnim korisničkim imenom. Dodatno, u bazi se nalaze i informacije o korisniku, kao što su opis, lokacija ličnog direktorijuma (home) i podrazumevani komandni interpreter (shell).

Na UNIX sistemima postoje dve vrste korisnika:

* **sistemski korisnici**, koji nastaju prilikom instalacije operativnog sistema i služe za specijalne namene, a ne za prijavljivanje na sistem. Jedini sistemski korisnik koji se može prijaviti na sistem je superuser **root**. Root ima sve privilegije i služi isključivo za administraciju sistema;
* **regularni korisnici**, koji služe za prijavljivanje na sistem. Regularne korisnike kreira superuser root.

# Osnovno o nalozima

Na Unix-like sistemima postoje tri primarne vrste naloga:

* root nalog (superuser),
* system nalog i
* user nalog.

Skoro svi nalozi raspoređuju se po ovim kategorijama.

* **Root nalog** - Root korisnik ima potpunu kontrolu nad sistemom, do te mere da može pokrenuti komande za uništenje sistema. Root može uraditi bilo šta bez ikakvih ograničenja, bez obzira na osobine fajlova ili direktorijuma. Princip po kome Unix-like sistemi funkcionišu je takav da se za root-a podrazumeva da zna šta radi, tako da on ako u bilo kom trenutku pokrene komande za uništavanje sistema, sistem će mu to i dozvoliti.
* **Sistemski nalog** - Sistemski nalozi su potrebni za operacije koje izvršavaju specifične komponente sistema. One uključuju, na primer mail account i ssh account (za ssh komunikaciju). Sistemski nalozi se prave u toku instalacije sistema i asistiraju u održavanju servisa ili programa koje korisnici zahtevaju. Postoje različiti sistemski nalozi, pri čemu se samo neki nalaze na određenom sistemu. Spisak sistemskih naloga na određenom sistemu se može naći u fajlu **/etc/passwd**. Neki od njih  su:  alias, apache, bind, ftp, halt, mail, mysql, root i sys. Ovi nalozi kao što je rečeno služe u obavljanju određenih operacija i ne treba ih menjati.
* **User nalog** - User nalozi omogućuju korisniku ili grupi korisnika, pristup sistemu. Generalno gledano, user nalozi imaju određena ograničenja tj. nemaju pristupa kritičnim fajlovima. Ime naloga je isto kao i ime usera.
* **Group nalog** - Grupni nalozi daju mogućnost da se više naloga logički povežu u smislu datih ograničenja. Ograničenja  su podeljena u tri vrste i to: vlasnik tj. onaj ko je napravio fajl, grupa  i drugi. Postojanje grupe daje mogućnost vlasniku da na svom fajlu da ograničenje za neku grupu korisnika. Dobra strana grupa je što jedan nalog može pripadati većem broju grupa, pri čemu se veoma precizno mogu odrediti dozvole i ograničenja bilo kog naloga. Svaki korisnik sistema mora pripadati najmanje jednoj grupi – tzv. primarna grupa. Primarna grupa korisnika je obavezan atribut svakog korisnika - njen GID (Group ID) je naveden u datoteci /etc/passwd.

# Administracija korisnika i grupa

U administraciji korisnicima i grupama važna su sledeća tri fajla:

* **/etc/passwd** - izlistava sve naloge
* **/etc/shadow** - za svaki nalog sadrzi kriptovanu lozinku.
* **/etc/group** - sadrži podatke o grupama.

### /etc/passwd

U fajlu **/etc/passwd** se nalaze podaci slični ovima:
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
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
{% endhighlight %}

Za svaki nalog je vezano nekoliko atributa.

**Slika** Izgled reda u /etc/passwd

* **Login ID** atribut je ime naloga tj. korisnicko ime koje korisnik unosi pri prijavljivanju na sistem.
* **Encrypted Password** predstavljen sa “x” je mesto gde se u ranijim verzijama sistema ovde se nalazila kriptovana lozinka ali se danas zbog sigurnosti nalazi u posebnom fajlu /etc/shadow.
* **UID** je način na koji sistem prepoznaje korisnika. Korisničko ime postoji da bi se korisniku olakšao rad, dok sistem koristi ovaj broj. Na sistemu ovaj broj bi trebao da bude jedinstven, jer bi preklapanjem ovog broja za više korisnika bilo konfuzno odrediti ovlašćenja.
* **GID** je broj kojim se identifikuje grupa kojoj korisnik pripada. Ovaj broj ne mora biti jedinstven, jer više korisnika može pripadati istoj grupi. Manje vrednosti GID-a pripadaju grupama sistemskih naloga.
* **GCOS** je neki opis kao što je puno ime korisnika, kontakt ili neka informacija.
* **Home directory** opisuje putanju na kojoj se nalazi home direktorijum.
* **Login shell** predstavlja ime shella koji korisnik koristi.

### /etc/shadow

Ovaj fajl sadrži kriptovane lozinke svih korisnika (korisničkih naloga), kao i vreme posle kojeg nisu validni. Npr:
{% highlight bash %}
man:*:13991:0:99999:7:::
lp:*:13991:0:99999:7:::
mail:*:13991:0:99999:7:::
…
marko:tc2kk31xv1PxQ:12735::::::
{% endhighlight %}
Podaci o svakom nalogu imaju devet mogućih mesta za podatke.

**Slika** Izgled linije fajla /etc/shadow

* **Login ID** predstavlja naziv korisničkog naloga.
* **Encrypted Password** je niz karaktera koji predstavlja kriptovan password. Ovo polje može sadržati 13 ili više karaktera, a ako je ovo polje prazno korisnik se na ovaj nalog može prijaviti bez lozinke.
* **Last Changed** polje predstavlja koliko je dana prošlo od poslednje promene lozinke.
* **Warning** je broj dana posle kojih će nalog biti blokiran. Uglavnom će korisnik naloga biti obavešten o datumu isteka njegovog naloga tako da se može obratiti administratoru za produženje naloga. Polja od šestog do devetog su prazna u skoro svim distribucijama Unix sistema.

### /etc/group

Ovaj fajl sadrži podatke o svim grupama. Npr:
{% highlight bash %}
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:root
...
{% endhighlight %}

* Prvo polje predstavlja **ime grupe**.
* Sledi polje koje sadrži **lozinku** ali je ona obično kriptovana i nalazi se u fajlu /etc/gshadow.
* Broj koji zatim sledi je **jedinstveni numerički identifikator grupe**.
* Poslednje polje govori **koji korisnički nalozi pripadaju datoj grupi**.

Komande za kreiranje, menjanje i brisanje naloga i grupa je uglavnom standardizovano na svim Unix i Unix-like sistemima. Sledeće komande su dostupne na većini sistema:

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| useradd | Dodaje nalog. |
| usermod | Menja opcije naloga. |
| userdel | Briše nalog sa sistema. |
| groupadd | Dodaje grupu. |
| groupmod | Menja opcije grupe. |
| groupdel | Briše grupu sa sistema. |

# Administracija korisničkih naloga

**Manuelno** dodavanje naloga editovanjem odgovarajućih fajlova:

* Promena **/etc/passwd** fajla tako što će se dodati ili izbrisati nalog. Ova datoteka se ne sme otvoriti standardnim editorom (kao što su vi, emacs ili joe), već isključivo pomoću vipw editora. vipw datoteku zatvara za upis, tako da druge komande ne mogu istovremeno da promene njen sadržaj. Komanda se navodi bez argumenata.
* Promena **/etc/shadow** fajla.
* Promena **/etc/group** fajla pomoću vigr editora.
* I na kraju sledi kreiranje direktorijuma željenog naloga u **/home direktorijumu**.

Naravno ovi koraci se mogu izbeći koristeći sledeću komandu, podrazumevajući da ste registrovani kao administrator tj. root.
