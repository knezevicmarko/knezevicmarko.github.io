---
layout: lekcija
title: SHELL - rad iz komandne linije
main_category: Materijali za vežbe
sub_category: Linux
image: terminal.png
active: false
comment: true
archive: false
---

Grafičko radno okruženje opterećuje procesor i povećava rizik u smislu sigurnosti sistema, tako da se, po pravilu, ne instalira na serverima. Tada sistem administratorima na raspolaganju ostaje komandni interpreter (shell) i prateći skup alata za rad sa fajlovima.

# KOMANDNI INTERPRETER (shell)

Shell je interfejs između korisnika i kernela, odnosno jezgra OS-a. Shell prihvata komande koje korisnik zadaje, zatim ih interpretira i potom ih izvršava, pri čemu po potrebi pokreće odgovarajuće programe. Na UNIX sistemima postoji više različitih komandnih interpretera, a korisnici u toku rada po potrebi mogu preći iz jednog u drugi.

Komandni interpreter je proces koji obavlja sledeće funkcije:

* interpretaciju komandne linije,
* pokretanje programa,
* redirekciju ulaza i izlaza,
* povezivanje komandi u pipeline,
* rad sa fajlovima iz komandne linije
* zamenu imena datoteka,
* rukovanje promenljivama i kontrolu okoline (environment),
* shell programiranje.

## Interpretiranje komandne linije

Kad se korisnik prijavi na sistem u kontekstu tekućeg login procesa izvršava se proces shell, odnosno komandni interpreter. Na ekranu se prikazuje komandni prompt (shell prompt), a to je najčešće znak **$, ukoliko se na sistem prijavi običan korisnik, odnosno #, ukoliko se na sistem prijavi root**. Kada korisnik zada neku komandu (odnosno otkuca neki tekst i pritisne Enter), shell to pokušava da interpretira. Tekst unet u shell prompt naziva se komandna linija (command line), čiji je opšti oblik:
{% highlight bash %}
$ command [opcije] [argumenti]
{% endhighlight %}

Znak $ je odzivni znak komandnog interpretera (shell prompt). Komanda može biti **interna (ugrađena u shell) ili eksterna (realizovana kao poseban program koji se nalazi u sistemskoj putanji)**. Opcije i argumenti su parametri koje shell prenosi komandi, pri čemu su argumenti najčešće obavezni i predstavljaju ime nekog fajla, direktorijuma, korisnika ili, na primer, identifikator procesa.

Ime komande, opcije i argumenti razdvajaju se razmakom. Shell interpretira razmak kao graničnik i na osnovu toga razdvaja argumente i opcije od imena komande. U jednu komandnu liniju može se uneti **najviše 256 karaktera**. Imena većine UNIX komandi po pravilu se formiraju od malih slova. Više UNIX komandi mogu se navesti u istoj komandnoj liniji ukoliko su razdvojene znakom tačka-zarez.

{% highlight bash %}
$   cal                 #  samo komanda
$   df /dev/sda         #  komanda (fd) i argument (/dev/sda)
$   cp 1.txt 2.txt      #  komanda (cp) i dva argumenta (1.txt i 2.txt)
$   date –u             #  komanda (date) i opcija (-u)
$   ls –l /etc          #  komanda (ls), opcija (-l) i argument (/etc)
$   clear ; date	#  dve komande koje se izvršavaju jedna za drugom
{% endhighlight %}

Opcije su osetljive na velika i mala slova (case-sensitive) i mogu se navesti na dva načina:

* -x		znak minus (-) praćen jednim slovom,
* --option	dva znaka minus (--) praćena punim imenom opcije.

### echo

Jedna od često koričćenih komandi je **echo** koja prikazuje tekst ili vrednost promenljive na ekranu. Sintaksa komande echo je:
{% highlight bash %}
$ echo [opcije] [string, promenljive...]
{% endhighlight %}
**Opcije**:

* -n ova opcija ne prebacuje kursor u novi red,nakon izvršenja echo komande
* -e omogućava interpretaciju sledećih karaktera u kombinaciji sa obrnutom kosom crtom:
  * \a upozorenje (alert bell)
  * \b povratak unazad (backspace)
  * \c ne prelaziti u novi red (suppress trailing new line)
  * \n novi red (new line)
  * \r povratak na početak reda (carriage return)
  * \t horizontalni tabulator (horizontal tab)
  * \\ obrnuta kosa crta (backslash)

**Primer.**
{% highlight bash %}
$ echo -e "Petar\n\t\t Petrovic"
Petar
		 Petrovic
{% endhighlight %}

### pwd (Print Working Directory)

Sve u Linux-u je fajl i ova konstatacija će postajati sve jasnija kako se budemo detaljnije upoznavali sa Linux-om. Za sada je bitno da se svi fajlovi nalaze unutar direktorijuma (foldera) koji su organizovani u hijerarhiji stabla. Prvi direktorijum (koren stabla) se naziva root direktorijum. Root direktorijum ima fajlove i poddirektorijume koji mogu da sadrže fajlove i direktorijume...
{% highlight bash %}
/

|-- bin

|   |-- file1

|   |-- file2

|-- etc

|   |-- file3

|   `-- directory1

|       |-- file4

|       `-- file5

|-- home

|-- var
{% endhighlight %}

Lokacija fajla ili direktorijuma se naziva **putanja**. Ako imamo direktorijum koji se zove *home* i koji sadrži poddirektorijum *marko* koji u sebi ima folder *Movies*, onda je putanja do tog direktorijuma **/home/marko/Movies**.

Pri kretanju po sistemu fajlova (fajl sistemu), kao i u realnom životu, korisno je znati gde se trenutno nalazimo i gde hoćemo da idemo. Da bi videli gde se trenutno nalazimo možemo koristiti komandu **pwd**, skraćeno od Print Working Directory (Štampa Radnog Direktorijuma), koja prikazuje u kom direktorijumu se trenutno nalazimo. Putanja je prikazana od root direktorijuma.
{% highlight bash %}
$ pwd
{% endhighlight %}

### cd (Change Directory)

Kada znamo gde se nalazimo (**pwd**) možemo da naučimo i kao da se krećemo. Da bi se kretali moramo da koristimo putanje. Postoje dva načina da se odredi putanja:
*  **Apsolutna putanja** je putanja od root direktorijuma. Root direktorijum se obično prikazuje kao *slash* (/) karakter. Svaki putanja koja počinje / je putanja koja počinje od root direktorijuma. Npr. /home/marko/Desktop.
* **Relativna putanja** je putanja od trenutne lokacije. Ako se nalazimo u direktorijumu /home/marko/Documents i želimo da stignemo u direktorijum porez koji se nalazi unutar direktorijuma Documents nije potrebno navoditi celu putanju /home/marko/Documents/porez, možemo da koristimo samo porez/.

Komanda za kretanje po direktorijumima je **cd** (change directory).
{% highlight bash %}
$ cd /home/marko/Pictures
{% endhighlight %}

Sada smo promenili lokaciju na /home/marko/Pictures.

Ako u direktorijum Pictures imamo direktorijum Zurke možemo do tog direktorijuma stići komandom
{% highlight bash %}
$ cd Zurke
{% endhighlight %}

Pored navigacije apsolutnim i relativnim putanjama komanda **cd** ima i skraćenice

*  . - trenutni direktorijum,
*  .. - direktorijum roditelj,
*  ~ - home direktorijum,
*  - - prethodni direktorijum.
{% highlight bash %}
$ cd .
$ cd ..
$ cd ~
$ cd -
{% endhighlight %}

### ls (List Directory)

Kada znamo da se krećemo po sistemu potrebno je da znamo i šta nam je dostupno. Za to možemo da koristimo komandu **ls** koja lista sadržaj direktorijuma. Komanda ls, bez argumenata, lista sadržaj direktorijuma u kome se trenutno nalazimo, ali je moguće je i uneti putanju do direktorijuma za koji hoćemo da dobijemo sadržaj.
{% highlight bash %}
$ ls
$ ls /home/marko
{% endhighlight %}

Komanda ls neće prikazati sve fajlove. Ako ime fajla počinje sa . taj fajl je skriven. Ako prosledimo opciju -a onda komanda ls prikazuje i skrivene fajlove.
{% highlight bash %}
$ ls -a
{% endhighlight %}

Još jedna korisna opcija je -l, koja prikazuje detaljnu listu fajlova. Detaljna lista sadrži dozvole za fajlove, broj linkova, ime vlasnika, vlasničku grupu, veličinu fajla, vreme kreiranja ili poslednje modifikacije i ime fajla/direktorijuma.
{% highlight bash %}
$ ls -l
{% endhighlight %}

Moguće je proslediti jednoj komandi više opcija odjednom. Za komadnu ls moguće je proslediti -la. Redosled navođenja opcija određuje i redosled izvršavanja opcija. U većini slučajeva redosled nije bitan tako da je moguće komandi ls proslediti opciju -la i -al.
{% highlight bash %}
$ ls -la
{% endhighlight %}

### touch

Jedan od načina za kreiranje fajlova je komanda touch.  Touch kreira novi prazan fajl.
{% highlight bash %}
$ touch prazanfajl
{% endhighlight %}

Touch takođe menja vreme poslednje modifikacije postojećeg fajla.

### file

Na linux sistemima ime fajla ne mora da reprezentuje sadržaj fajla (kao u Windows sistemima). Moguće je kreirati fajl funny.gif koji u stvari nije GIF.

Da bi saznali koji je tip fajla, možemo da koristimo komandu **file**.
{% highlight bash %}
$ file banana.jpg
{% endhighlight %}

### cat

Osnovna komanda za prikaz sadržaja fajla je komanda **cat**, skraćeno od concatenate. Ona može i da kombinuje više fajlova i prikže njihov sadržaj.
{% highlight bash %}
$ cat dogfile birdfile
{% endhighlight %}

Komanda cat nije pogodna za prikaz velikih fajlova.

### less

Ako je potrebno prikazati sadržaj velikog tekstualnog fajla možemo koristiti komandu less.
{% highlight bash %}
$ less /home/marko/Documents/text1
{% endhighlight %}

Za navigaciju se koriste sledeće komande:

*  q - Izlazak iz less i povratak u shell,
*  Page up, Page down, Up i Down - Navigacija po dokumentu,
*  g - Pozicionira se na početak fajla,
*  G - Pozicionira se na kraj fajla,
*  /reč - Pronalazi zadatu reč u dokumentu.
*  h - poziva pomoć za less.
