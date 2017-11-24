---
layout: lekcija
title: Emiteri
main_category: Materijali za vežbe
sub_category: Parametarski posao
image: emiteri.png
active: true
comment: true
archive: false
---

# Emiteri i mrežni uređaji

Neka je dat fajl sa 100.000 linija, korisnici.txt, u kome je u svakoj liniji upisana x i y koordinata korisnika koji poseduju mobilni telefon. Pored pomenutog fajla data su i tri fajla sa koordinatama emitera različitih klasa (A, B, C): emiteriA.txt, emiteriB.txt, emiteriC.txt i fajl opsezi.txt u kome se u tri reda nalaze dometi emitera za svaku klasu A, B i C respektivno.

Za date fajlove, potrebno je napisati programsko rešenje koje će se izvršavati na klasteru i upisati u fajl "nemajuDomet.txt" koordinate onih korisnika koji nemaju domet i mora sadržati sledeće fajlove:

* obrada.c - za date fajlove štampa koordinate onih korisnika koji nemaju domet,
* posao.sub - PBS komandni fajl za pokretanje posla na klasteru, treba imati u vidu da se radi o parametarskom poslu,
* pokreni.sh - zadužen je za pokretanje posla i skupljanje rezultata

Programsko rešenje mora podeliti posao na 4 procesora i rešenje upisati u fajl "nemajuDomet.txt".

**Napomena**: Za potrebe testiranja napraviti programski kod u C-u koji će generisai slučajne vrednosti za koordinate u opsegu [-1000, 1000]. Korisnika ima 100 000, emitera klase A ima 10, klase B 20 i klase C 30. U fajl opseg.txt upisati redom vrednosti 130, 100, 70.
