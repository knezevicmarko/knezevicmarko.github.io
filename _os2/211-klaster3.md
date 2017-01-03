---
layout: lekcija
title: Kategorizacija kompanija
main_category: Materijali za vežbe
sub_category: Parametarski posao
image: company.png
active: true
comment: true
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

## Predlog rešenja

program.c
{% highlight bash %}
#include <stdio.h>
#include <stdlib.h>
#include <time.h>


int main(int argc, char *argv[]) {

	int posao = atoi(argv[1]);
	FILE *kat;
	int i = 0;
	double val, valNext;
	int pib[1000];

	kat = fopen("kategorije.txt","r");

	while(1) {
		fscanf(kat, "%lf", &val);
		if(i == posao) {
			if(fscanf(kat, "%lf", &valNext) == 0) {
				valNext = 0;
			}
			break;		
		}
		i++;
	}

	fclose(kat);

	///////////////////

	for(i = 0; i < 1000; i++) {
		pib[i] = 0;
	}

	///////////////////

	FILE *kvartal1, *kvartal2, *kvartal3, *kvartal4;
	double iznos;
	int pib1, pib2;
	kvartal1 = fopen("kvartal1.txt", "r");

	while(fscanf(kvartal1, "%d%d%lf", &pib1, &pib2, &iznos) == 3) {
		if(iznos > val) {
			if(iznos < valNext || valNext == 0) {
				pib[pib1-1]++;
				pib[pib2-1]++;
			}
		}

	}

	kvartal2 = fopen("kvartal2.txt", "r");

	while(fscanf(kvartal1, "%d%d%lf", &pib1, &pib2, &iznos) == 3) {
		if(iznos > val) {
			if(iznos < valNext || valNext == 0) {
				pib[pib1-1]++;
				pib[pib2-1]++;
			}
		}

	}

	kvartal3 = fopen("kvartal3.txt", "r");

	while(fscanf(kvartal1, "%d%d%lf", &pib1, &pib2, &iznos) == 3) {
		if(iznos > val) {
			if(iznos < valNext || valNext == 0) {
				pib[pib1-1]++;
				pib[pib2-1]++;
			}
		}

	}

	kvartal4 = fopen("kvartal4.txt", "r");

	while(fscanf(kvartal1, "%d%d%lf", &pib1, &pib2, &iznos) == 3) {
		if(iznos > val) {
			if(iznos < valNext || valNext == 0) {
				pib[pib1-1]++;
				pib[pib2-1]++;
			}
		}

	}

	fclose(kvartal1);
	fclose(kvartal2);
	fclose(kvartal3);
	fclose(kvartal4);

	///////////////////

	int maxCountPib = pib[0];

	for(i = 1; i < 1000; i++) {
		if(pib[i] > maxCountPib) {
			maxCountPib = pib[i];
		}
	}

	for(i = 0; i < 1000; i++) {
		if(pib[i] == maxCountPib) {
			printf("%d\n", i+1);
		}
	}

	return 0;
}
{% endhighlight %}

posao.sub

{% highlight bash %}
#!/bin/bash
#PBS -N kategorizacija
#PBS -q batch
#PBS -r n
#PBS -l nodes=1:ppn=1

cd $PBS_O_WORKDIR

./program $PBS_ARRAYID > kat$PBS_ARRAYID.txt
{% endhighlight %}

pokreni.sh

{% highlight bash %}
#!/bin/bash

gcc program.c -o program

qsub -t 0-7 posao.sub
{% endhighlight %}
