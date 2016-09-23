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
