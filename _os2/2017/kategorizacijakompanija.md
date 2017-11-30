---
layout: lekcija
title: Kategorizacija kompanija
main_category: Materijali za vežbe
sub_category: Parametarski posao
image: company.png
active: true
comment: true
archive: false
---

Na osnovu podataka finansijskog prometa između kompanija (transfer novca među njima) potrebno je izvršiti kategorizaciju kompanija. U 4 tekstualne datoteke **kvartal1.txt, … , kvartal4.txt** se u svakoj liniji nalazi transfer novca. Transfer je dat u formatu:

pib<sub>uplatilac</sub>	pib<sub>primalac</sub>	iznos<sub>transfer</sub>

gde pib-ovi identifikuju kompanije (poreski identifikacioni broj).  U datoteci **kategorizacija.txt** nalaze se podaci prema kojima se kompanija kategorizuje u formatu:

iznos<sub>kat1</sub>
. . .
iznos<sub>kat8</sub>

gde je broj kategorizacionih granica 8, definisanih u vrednostima iznos<sub>kati</sub>, i =1 .. 8.

Ukoliko je iznos transfera 12345.67 , a kategorizacione granice iznos<sub>kat3</sub> i iznos<sub>kat4</sub> iznose redom 10240.12 i 14567.89 , tada takav transfer za obe kompanije govori da imaju transfer u kategoriji 3 . Ukoliko je iznos transfera manji od iznos<sub>kat1</sub>, transfer se ne uzima u obzir u kategorizaciji. Za transfer veći od iznos<sub>kat8</sub> , uzimamo da obe kompanije imaju transfer u kategoriji 8 . Potrebno je za svaku kategoriju utvrditi koja kompanija (kompanije, ako ih ima više) ima najviše transfera u datoj kategoriji. Rešenje problema implementirati za KRAGUJ klaster platformu.

Rezultat job -a su 8 datoteka, kat1.txt ,..., kat8.txt , gde svaka od njih sadrži pib (ili više pib -ova po linijama) kompanije(a) koji imaju najviše transfer-a u datoj kategoriji.

**Napomena : Svi iznosi transfera su brojevi u decimalnom zapisu, a pib -ovi su prirodni brojevi.**
