---
layout: lekcija
title: Most
main_category: Materijali za vežbe
sub_category: Pthreads
image: bridge.png
active: true
comment: true
archive: true
---

# Most koji ima samo jednu kolovoznu traku

Automobili koji dolaze sa severa i juga dolaze do mosta koji ima jednu kolovoznu traku. Automobili iz suprotnog smera ne mogu istovremeno da budu na mostu. Napisati C program koji koršćenjem pthread biblioteke:

1. Simulira ponašanje autmobila ako pretpostavimo da se zbog slabe konstrukcije mosta na njemu može naći najviše jedan automobil.
2. Dati rešenje problema ako pretpostavimo da automobili koji dolaze iz istog smera mogu bez ikakvih ograničenja istovremeno da prelaze most.
3. Usavršiti prethodno rešenje tako da bude pravedno, odnosno da automobili ne mogu da čekaju neograničeno dugo kako bi prešli most. Da bi se to postiglo smer saobraćaja treba da se menja svaki put nakon što most pređe određen broj automobila iz jednog istog smera N (ovo važi pod uslovom da je bar jedan automobil čekao da pređe most sa suprotne strane. Dok god se ne pojavi automobil sa suprotne strane automobili iz istog smera mogu slobodno da prelaze most.)
4. **Za domaći:** Modifikovati rešenje iz tačke 2. ako pretpostavimo da zbog ograničenog kapaciteta najviše K automobila iz istog smera može istovremeno da se nađe na mostu.
