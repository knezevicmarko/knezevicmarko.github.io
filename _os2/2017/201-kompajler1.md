---
layout: lekcija
title: Prevođenje izvornog koda
main_category: Materijali za vežbe
sub_category: Prevođenje
image: c.png
active: true
comment: true
---
# C programski jezik

C je razvio Dennis Ritchie u periodu od 1969. do 1973. godine u Bell Labs, za potrebe reimplementacije Unix operativnog sistema. Danas je jedan od najčešće korišćenih programskih jezika. Poseduje C kompajlere različitih proizvođača i dostupan je na svim glavnim platformama i operativnim sistemima. Za njegovu standardizaciju se brine Američki Nacionalni Institut za Standarde (ANSI) i kasnije Internacionalna Organizacija za Standardizaciju (ISO).

# Kompajler

Kompajler je kompjuterski program (ili skup programa) koji transformiše izvorni kod napisan u nekom programskom jeziku (izvorni jezik) u drugi programski jezik, najčešće u binarni format (objektni kod). Razlog ovakve konverzije je kreiranje izvršnog programa. Ime "kompajler" se koristi za programe koji prevode izvorni kod iz programskih jezika visokog nivoa (c, c++, ...) u jezike niskog nivoa (asembler, mašinski kod).

Prevođenje programa napisanog u programskom jeziku C u izvršni kod se izvodi u 4 koraka:

1. Preprocesuiranje,
2. Prevođenje,
3. Montaža,
4. Povezivanje.

Za bilo koji ulazni fajl, sufiks imena fajla (ekstenzija fajla) određuje koja vrsta prevođenja je urađena. Primer za GCC kompajler je dat u tabeli ispod.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Ekstenzija fajla   |      Opis      |
|----------|-------------|
| ime_fajla.c | C izvorni kod koji mora da bude preprocesuiran |
| ime_fajla.i | C izvorni kod koji ne treba da bude preprosesuiran |
| ime_fajla.h | C heder fajl (ne treba da bude kompajliran i povezan) |
| ime_fajla.s | Kod asemblera. |
| ime_fajla.o | Objektni fajl.|

Na UNIX/Linux sistemima izvršni ili binarni fajl ne sadrži ekstenziju, dok na Windows sistemima ekstenzija može biti .exe, .com ili .dll.

![Slika koraka.](/assets/os2/koraci.jpg "Slika koraka."){:width="100%"}

# Primer 1.

Fajl **primer1.c**.

{% highlight c %}
/* Ovo su makroi */
#define BROJ 3
#define STRING "Ovo je poruka."

/* Ovo je definicija neinicijalizovane globalne varijable */
int x_global_uninit;

/* Ovo je definicija inicijalizovane globalne varijable */
int x_global_init = 1;

/* Ovo je definicija neinicijalizovane globalne varijable
kojoj moze da se pristupi samo u ovom C fajlu */
static int y_global_uninit;

/* Ovo je definicija inicijalizovane globalne varijable
kojoj moze da se pristupi samo u ovom C fajlu */
static int y_global_init = 2;

/* Ovo je deklaracija globalne promenljive koja postoji
negde drugde u programu */
extern int z_global;

/* Ovo je deklaracija funkcije koja postoji negde drugde u programu
(moze se dodati "extern" kljucna rec ali nije neophodno) */
int fn_a(int x, int y);

/* Ovo je definicija funkcije kojoj moze da se pristupi,
po imenu samo iz ovog C fajla */
static int fn_b(int x)
{
  return x+1;
}

/* Ovo je definicija funkcije.
Parametri funkcije se vode kao lokalne promenljive */
int fn_c(int x_local)
{
  /* Ovo je definicija neinicijalizovane lokalne promenljive */
  int y_local_uninit;

  /* Ovo je definicija inicijalizovane lokalne promenljive */
  int y_local_init = BROJ;

  /* Linije koda kojima se referenciraju lokalne i globalne
  promenljive i funkcije */
  x_global_uninit = fn_a(x_local, x_global_init);
  y_local_uninit = fn_a(x_local, y_local_init);
  y_local_uninit += fn_b(z_global);
  return (y_global_uninit + y_local_uninit);
}
{% endhighlight %}

## 1. Preprocesuiranje

Proces prevodjenja C programa počinje sa preprocesuiranjem direktiva (npr. #include i #define). Preprocesor je odvojen program koji se automatski poziva tokom prevođenja. Na primer, komanda {% highlight c %}#define BROJ 3{% endhighlight %} na liniji 2 fajla primer1.c govori preprocesu da svako pojavljivanje makroa BROJ zameni sa brojem 3 (vrednost makroa). Rezultat je novi fajl (obicno sa ekstenzijom .i). U praksi, preprocesuiran fajl se ne pamti na hdd osim ako nije uključena opcija -save-temps.
Ovo je prva faza procesa prevođenja gde preprocesor proširuje fajl. Da bi izvršio ovaj korak gcc kompajler interno pokreće komadnu {% highlight bash %}$ cpp primer1.c > primer1.i{% endhighlight %} Rezultat je fajl primer1.i koji sadrži izvorni kod sa proširenim svim makroima. Ovako izvršena komanda pamti fajl primer1.i na hard disk.

## 2. Prevođenje

U ovoj fazi kompajler prevodi fajl primer1.i u primer1.s izvršenjem komande {% highlight bash %}$ gcc -S primer1.i{% endhighlight %}Fajl primer1.s sadrži asemblerski kod. Opcija komandne linije -S govori kompajleru da preprocesuirani fajl konvertuje u kod asemblerskog jezika bez kreiranja objektnog fajla. Dodatno objašnjenje komandi asemblerskog koda mogu se pronaći u [dokumentaciji programa **as**](https://sourceware.org/binutils/docs-2.27/as/index.html){:target="_blank"}.

Prevodilac dozvoljava da se, u kodu, referencira samo na promenljivu ili funkciju koja je prethodno deklarisana. Deklarisanje je obećanje da definicija (promenljive ili funkcije) postoji negde drugde u celom programu.

![Prevođenje](/assets/os2/prevodjenje.jpg "Prevođenje"){:width="100%"}

## 3. Montaža

U ovom koraku montažer (assembler - as) prevodi primer1.s u jezik mašinskih instrukcija i genereiše objektni fajl primer1.o. Izvršenjem komande{% highlight bash %}$ as primer1.s -o primer1.o{% endhighlight %} se pokreće asembler. Rezultujući fajl primer1.o sadrži mašinske instrukcije C programa primer1.c.
Zavisno od platforme na kojoj se kod prevodi objektni fajl može da bude kreiran u više formata. U tabli ispod su prikazani formati objektnog fajla.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Format objektnog fajla   |      Opis      |
|:----------:|-------------|
| a.out | a.out je format fajla za Unix. Sastoji se od tri sekcije: text, data i bss u kojima se nalazi programski kod, inicijalizovani podaci i neinicijalizovani podaci, respektivno. Ovaj format je jednostavan. Ne sadrži mesto rezervisano za informacije debagera.|
| COFF | COFF (Common Object File Format) je uveden na System V Release 3 (SVR3) Unix-u. Može da sadrži više sekcija, koje imaju kao prefiks ime zaglavlja. Broj sekcija je ograničen. Sadrži podršku za debagovanje ali je količina informacija ograničena.|
| ECOFF | Varijanta COFF-a. ECOFF je proširen (extended) COFF originalno koriščen na Mips i Alpha radnim stanicama. |
| PE | Windows 9x i NT koriste PE (Portable Executable) format. PE je u suštini COFF sa dodatnim zaglavljima. |
| ELF | ELF (Executable and Linking Format) je uveden na System V Release 4 (SVR4) Unix-u. ELF je sličan COFF formatu tako što je organizovan u sekcijama, ali ne postoje brojna COFF ograničenja. ELF se koristi u većini modernih Unix-olikih sistema, uključujući GNU/Linux, Solaris i Irix. |

Objektni fajl sadrži oblasti koje se nazivaju sekcije. U sekcijama može da se nalazi kod koji se izvršava, podaci, informacije za dinamičko povezivanje, informacije debagera, tabelu simbola, informacije za realokaciju, komentare, tabelu stringova i beleške.
Neke sekcije se učitavaju u sliku procesa, neke sadrže informacije potrebne za kreiranje slike procesa, a neke se koriste samo za povezivanje objektnih fajlova.
Određene sekcije se pojavljuju bez obzira na tip formata (moguće je da su drugačijeg imena, zavisno od kompajlera/linkera):

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Sekcija   |      Opis      |
|----------|-------------|
| .text | Sadrži kod instrukcija koje se izvršavaju i svaki proces koji pokreće isti izvršni fajl koristi istu sekciju. Ova sekcija najčešće ima samo Read i Execute dozvolu. Mogućnost optimizacije je najveća u ovoj sekciji.|
| .data | Sadrži inicijalizovane globalne i static promenljive i njihove vrednosti. Najčešće zauzima najveći deo memorije izvršnog procesa. Poseduje Read / Write dozvolu. |
| .bss | BSS (Block Started by Symbol) je sekcija koja sadrži neinicijalizovane globalne i static promenljive. Pošto .bss sadrži samo promenljive koje još uvek nemaju vrednost nije potrebno , pre kreiranja procesa, rezervacija prostora za vrednosti promenljivih. Veličina koja je potrebna za ovu sekciju pri izvršavanju je upisana u fajlu objekta, ali .bss (za razliku od .data) ne zauzima nikakav prostor u objektnom fajlu. |
| .rdata | Obeležava se i kao .rodata (read-only data) sekcija. Sadrži konstante i stringove. |
| .reloc | Sadrži informacije potrebne za realociranje slike procesa tokom učitavanja. |
| Symbol table | Simbol je u stvari ime i adresa. Tabela simbola sadrži informacije potrebne za lociranje i realociranje programskih simboličkih definicija i referenci.  |
| Relocation records | Relokacija je proces povezivanja simboličkih referenci sa simboličkom definicijom. Npr. kada program pozove funkciju, neophodno je pronaći odgovarajuću adresu gde se nalazi definicija te funkcije. Prostije rečeno slogovi realokacije su informacije koje linker koristi da prilagodi sadržaj sekcije.  |

Sadržaj objektnog fajla moguće je prikazati pomoću **readelf** programa komandom {% highlight bash %}$ readelf -a primer1.o{% endhighlight %}

## 4. Povezivanje

Ranije smo napomenuli da je deklaracija funkcije ili promenljive obećanje C prevodiocu da negde drugde u programu je definicija te funkcije ili promenljive i posao povezivača (linker) je da to obećanje ostvari. Da bi smo ostvarili obećanja o postojanju funkcije int fn_a(int x, int y) i promenljive z_global kreiraćemo fajl primer2.c sledećeg sadržaja:

{% highlight c %}
/* inicijalizovana globalna promenljiva */
int z_global = 11;
/* Jos jedna globalna promenljiva sa imenom y_global_int, ali su obe static */
static int y_global_init = 2;
/* Deklaracija jos jedne globalne promenljive */
extern int x_global_init;

int fn_a(int x, int y)
{
  return(x+y);
}

int main(int argc, char *argv[])
{
  return fn_a(11,12);
}
{% endhighlight %}

Pokretanjem komande {% highlight bash %}$ gcc -c primer2.c{% endhighlight %}kreira se objektni fajl primer2.o.

![Prevođenje](/assets/os2/prevodjenje2.jpg "Prevođenje 2"){:width="100%"}

Analizom objektnih fajlova primer1.o i primer2.o možemo zaključiti da je moguće sve povezati u jednu celinu (svaka deklaracija ima svoju definiciju). Posao linkera je između ostalog da osigura da svaka stvar ima svoje mesto i da svako mesto ima svoju stvar.

![Prevođenje](/assets/os2/prevodjenje_sve.jpg "Prevođenje sve"){:width="100%"}

Linker u stvari omogućava odvojeno prevođenje. Izvršni fajl može da bude sastavljen od većeg broja izvornih fajlova (prevedenih i sastavljenih u nezavisne objektne fajlove).

![Linker](/assets/os2/linker.jpg "Linker"){:width="100%"}

GCC kompajler povezuje fajlove pomocu alata **ld**. U procesu povezivanja objektnih fajlova neophodno je uključiti i potrebne sistemske fajlove. Spisak potrebnih fajlova se razlikuje od platforme do platforme, takođe i lokacija tih fajlova može da se razlikuje. Pozivanjem komande {% highlight bash %}$ gcc -o primer primer1.o primer2.o -v{% endhighlight %} se implicitno poziva alat ld i ispisuje koje se biblioteke koriste pri procesu povezivanja. Na primer na x64 Ubuntu 16.04 Linux-u pozivanjem komande {% highlight bash %}$ ld -dynamic-linker /lib64/ld-linux-x86-64.so.2 /usr/lib/x86_64-linux-gnu/crt1.o /usr/lib/x86_64-linux-gnu/crti.o /usr/lib/x86_64-linux-gnu/crtn.o primer1.o primer2.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbegin.o -L /usr/lib/gcc/x86_64-linux-gnu/5/ -lgcc -lgcc_eh -lc -lgcc -lgcc_eh /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o -o primer{% endhighlight %} se kreira izvršni fajl **primer** koji povezuje primer1.o i primer2.o.

Pri procesu prevođenja na mašinski kod asembler uklanja sve labele iz koda. Objektni fajl mora da zadrži te informacije na drugoj lokaciji. Ta druga lokacija je **tabela simbola** koja sadrži listu imena i odgovarajuće adrese ka text ili data segmentima.

Objektni fajlovi uključuju reference ka kodovima i/ili podacima drugih objektnih fajlova, tj. ka različitim lokacijama, pa se sve mora ukomponovati u jednu celinu tokom procesa povezivanja. Na primer, objektni fajl koji sadrži funkciju main() može da uključuje pozive ka funkcijama funct() i printf(). Nakon povezivanja svih objektnih fajlova, linker koristi podatke iz sekcije **relocation records** da pronađe sve potrebne adrese.

![Relocation record.](/assets/os2/relocation.png "Relocation record.")

# Deljeni objekti

U tipičnom sistemu biće pokrenut veći broj programa. Svaki program koristi veći broj funkcija od kojih su neke standardne C funkcije (printf(), malloc(), strcpy(), ...), a neke su nestandardne ili korisnički definisane funkcije. Ako svaki program koristi standardne C biblioteke, to znači da bi svaki program trebalo da ima jedinstvenu kopiju te biblioteke u izvršnom fajlu. Takav pristup kao rezultat ima rasipanje resursa i pad efikasnosti i performansi. Zato što su C biblioteke uobičajne, bolje je da se programi referenciraju na jednu instancu biblioteke umesto da imaju svako svoju kopiju. Ovo može da se postigne tako što će neki objekti da se povezuju tokom povezivanja (statically linked), a neki tokom izvršenja (dynamic/deferred linking).

## Statičko povezivanje

Statičko povezivanje je vrsta povezivanja u kome se program i određena biblioteka povezuju tako što izvršni fajl postaje unija tih objektnih fajlova. Njihovo povezivanje je fiksno i poznato pri procesu povezivanja (pre pokretanja programa). Takođe, nemoguće je promeniti ovu vezu osim ako ne pokrenemo ponovno povezivanje sa novom verzijom biblioteke.
Program koji je statički povezan se povezuje sa arhivom objekata (biblioteka) koja obično ima ekstenziju **.a**. Primer ovakve kolekcije objekata je standardna C biblioteka, libc.a. Ovakav način povezivanja se obično koristi kada je potrebno povezati program sa tačno određenom verzijom biblioteke, a kada ne možemo biti sigurni da je ta verzija dostupna pri pokretanju programa. Loša strana ovakvog pristupa je ta da izvršni fajl je značajno veće veličine. Objektni fajlovi mogu da kreiraju statičku biblioteku korišćenjem alata **ar**.
{% highlight bash %}
$ ar cr static.a objetni_fajl1.o objektni_fajl2.o
$ gcc -Wall main.c static.a -o izvrsni_fajl
{% endhighlight %}

## Dinamičko povezivanje

Dinamičkim povezivanjem se program i određena biblioteka koju koristi ne kombinuju zajedno već povezuju pri pokretanju. Linker postavlja informacije u izvršni fajl koji govori loader-u u kom objektnom modulu i koji linker treba da koristi da bi se pronašle i povazale reference. Ovo znači da se povezivanje programa i deljenih objekta vrši ili pre nego što se program startuje (load-time dynamic linking) ili kada se referencira na na neki simbol iz biblioteke (run-time dynamic linking). U suštini, pri povezivanju linker samo proverava da li simboli deljenog objekta postoje u naznačenoj biblioteci i u izvršni fajl unosi informaciju na kojoj lokaciji se nalazi biblioteka. Na ovaj način se povezivanje odlaže sve do pokretanja programa ili kasnije.

Dinamičke biblioteke obično imaju prefiks **lib** i ekstenziju **.so**. Primer takvog objekta je deljena verzija standardne C biblioteke libc.so. Prednosti odloženog povezivanja nekih objekata/modula do trenutka kada su oni stvarno potrebni su sledeće:

1. Programski fajlovi (na hdd-u) postaju značajno manji. Ne moraju da sadrže sve neophodne text i data segmentne informacije.
2. Standardne biblioteke mogu da se menjaju (nove verzije) bez potrebe da se svaki program ponovo linkuje.
3. U kombinaciji sa virtuelnom memorijom, dinamičko povezivanje omogućava da dva ili više procesa dele read-only izvršni modul (npr. C biblioteku). Korišćenjem ove tehnike samo jedna kopija modula je potrebna u memoriji u bilo kom trenutku. Rezultat je značajna ušteda memorije, ali zahteva efikasnu politiku zamene...

{% highlight bash %}
$ gcc -c -Wall -Werror -fpic ime_fajla.c
$ gcc -shared -o ime_biblioteke.so objetni_fajl.o
$ gcc -Wall -o izvršni main.c ime_biblioteke.so
{% endhighlight %}
GCC flag -fpic govori da se kod prevede u PIC (position-independent code). Takav kod radi bez obzira gde se u memoriji nalazi. Ne sadrži fiksne memorijske adrese, da bi takvu biblioteku mogli da koriste više procesa.

## Učitavanje procesa

U Linux sistemima izvršni programi se učitavaju u ELF formatu. Pre izvršenja potrebno je učitati procese u RAM. Proces učitavanja se pokreće sistemskim pozivom execve() ili spawn(). Posao učitavanja obavlja **loader** koji je deo operativnog sistema. Loader, izveđu ostalog, radi sledeće:

1. Validira memoriju i prava pristupa: Jezgro OS čita zaglavlje fajla i validira tipove, prava pristupa i zahteve za memorijom, kako i sposobnost da izvršava instrukcije programa. On potrvrđuje da je fajl izvršni i preračunava potrebnu memoriju.
2. Podešavanje procesa uključuje:
  - Alokaciju primarne memorije potrebne za izvršavanje programa. Pri alokaciji se ostavlja prostor za .text i .data i .bss sekcije, kao i za stek i heap. Ovakva organizacija memorije dozvoljava da se napravi "razmak" između dinamički alocirane memorije heap-a i stack-a.
  - Tipičan proces sadrži 5 različitih memorijskih oblasti (za text, bss, data, stack i heap) koji se nazivaju segmenti.
  - Kopira .text i .data sekcije iz sekundarne u primarnu memoriju.
  - Kopira argumente komandne linije na stek.
  - Inicijalizuje registre (postavlja esp (stack pointer) na vrh steka i čisti ostale registre)
  - Skače na početnu rutinu, koja kopira argumente main() funkcije sa steka i skače na main().

![Izgled procesa u memoriji.](/assets/os2/proces.png "Izgled procesa u memoriji.")
