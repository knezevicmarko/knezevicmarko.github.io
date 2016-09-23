---
layout: lekcija
title: Prevođenje izvornog koda
main_category: Materijali za vežbe
sub_category: Prevođenje
image: linux.png
active: true
comment: true
---
# C programski jezik

C je razvio Dennis Ritchie u periodu od 1969. do 1973. godine u Bell Labs, za potrebe reimplementacije Unix operativnog sistema. Danas je jedan od najčešće korišćenih programskih jezika. Poseduje C kompajlere različitih proizvođača i dostupan je na svim glavnim platformama i operativnim sistemima. Za njegovu standardizaciju se brine Američki Nacionalni Institut za Standarde (ANSI) i kasnije Internacionalna Organizacija za Standardizaciju (ISO).

# Kompajler

Kompajler je kompjuterski program (ili skup programa) koji transformiše izvorni kod napisan u nekom programskom jeziku (izvorni jezik) u drugi programski jezik, najčešće u binarni format (objektni kod). Najčešći razlog ovakve konverzije je kreiranje izvršnog programa. Ime "kompajler" se najčešće koristi za programe koji prevode izvorni kod iz programskih jezika visokog nivoa (c, c++, ...) u jezike niskog nivoa (asembler, mašinski kod).

Prevođenje programa napisanog u programskom jeziku C u izvrsni kod se izvodi u 4 koraka:
1. Preprocesuiranje,
2. Kompajliranje,
3. Sastavljanje,
4. Povezivanje.

Za bilo koji ulazni fajl, sufiks imena fajla (ekstenzija fajla) odriđuje koja vrsta prevođenja je urađena. Primer za GCC kompajler je dat u tabeli ispod.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Ekstenzija fajla   |      Opis      |
|----------|-------------|
| ime_fajla.c | C izvorni kod koji mora da bude preprocesuiran |
| ime_fajla.i | C izvorni kod koji ne treba da bude preprosesuiran |
| ime_fajla.h | C heder fajl (ne treba da bude kompajliran i povezan) |
| ime_fajla.s | Kod asemblera. |
| ime_fajla.o | Objektni fajl. Kreira se tako sto se ekstenzije .c, .i, .s zamene sa .o |

Na UNIX/Linux sistemima izvršni ili binarni fajl ne sadrži ekstenziju, dok na Windows sistemima ekstenzija može biti .exe, .com ili .dll.

Slika koraka.

# Primer 1.

Fajl **primer1.c**.

{% highlight c %}
/* poziv ka standardnoj biblioteci stdio */
#include <stdio.h>

/* Ovo su makroi */
#define BROJ 3
#define STRING Ovo je poruka.

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
  printf(STRING);
  return (y_global_uninit + y_local_uninit);
}
{% endhighlight %}

## 1. Preprocesuiranje

Proces prevodjenja C programa počinje sa preprocesuiranjem direktiva (npr. #include i #define). Preprocesor je odvojen program koji se automatski poziva tokom prevođenja. Na primer, komanda {% highlight c %}#include <stdio.h>{% endhighlight %} na liniji 2 fajla primer1.c govori preprocesu da procita sadržaj sistemskog zaglavlja fajla stdio.h i da ga ubaci direktno u tekst programa. Rezultat je novi fajl (obicno sa ekstenzijom .i). U praksi, preprocesuiran fajl se ne pamti na disk osim ako nije uključena opcija -save-temps.
Ovo je prva faza procesa prevođenja gde preprocesor proširuje fajl (obično makroima i zaglavljima uključenih fajlova). Da bi izvršio ovaj korak gcc kompajler izvršava interno komadnu {% highlight bash %} cpp primer1.c > primer1.i{% endhighlight %} Rezultat je fajl primer1.i koji sadrži izvorni kod sa proširenim svim makroima. Ovako izvršena komanda pamti fajl primer1.i na hard disk.

## 2. Kompajliranje

U ovoj fazi kompajler prevodi fajl primer1.i u primer1.s. Fajl primer1.s sadrži asemblerski kod. Izvršenjem komande {% highlight bash %}gcc -S primer1.i{% endhighlight %} gcc kompajler prevodi fajla primer1.i u primer1.s. Opcija komandne linije -S govori kompajleru da preprocesuirani fajl konvertuje u kod asemblerskog jezika bez kreiranja objektnog fajla.
