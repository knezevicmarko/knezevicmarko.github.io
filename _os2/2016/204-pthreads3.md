---
layout: lekcija
title: Flaše
main_category: Materijali za vežbe
sub_category: Pthreads
image: bottle.png
active: true
comment: true
archive: true
---

# Flaše

Neka je dat unapred poznat broj predmeta(flaša) N. Napisati C program koji pomoću pthread biblioteke simulira rad 3 niti. Prve 2 niti uzimaju jedan po jedan predmet(flašu) i svaka obrađuje preuzeti predmet nezavisno, u dužini t1, odnosno t2. Nakon što završe jedan predmet, preuzimaju sledeći. Predmeti su numerisani rednim brojevima. Kada na red dođe predmet (flaša) čiji je broj deljiv sa 5 (5, 10, 15,...) tada obradu tog predmeta preuzima 3-ća nit. Obrada predmeta od strane 3-će niti traje t3. Ako su i ostale 2 niti završile svoje predmete dok treća obađuje svoj, ne smeju da uzimaju dalje predmete na obradu. Tek kad 3-ća nit završi svoj posao, ostale 2 niti mogu da nastave. 3-ća nit miruje, dok ostale 2 niti obrađuju predmete do sledećg čiji je redni broj deljiv sa 5. Program dok radi treba da ispisuje poruke koja je nit u kom trenutku preuzela koji predmet na obradu. Program prestaje sa radom kada se potroše svi predmeti.

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

#define BROJ_NITI 3
#define BROJ_FLASA 20

pthread_t niti[BROJ_NITI];
pthread_mutex_t brava;
pthread_cond_t kontrolnaUlaz;
pthread_cond_t kontrolnaIzlaz;

int brojacFlasa = 1;

void *kontrola(void *id)
{
	int tid = (int)id;
	int kontrolnaFlasa = 0;
	int ind = 1;

	while (ind)
	{
		pthread_mutex_lock(&brava);

		if(brojacFlasa % 5 != 0)
			pthread_cond_wait(&kontrolnaUlaz,&brava);


		if(brojacFlasa > BROJ_FLASA)
			ind = 0;
		else
		{
			kontrolnaFlasa++;
			brojacFlasa++;
			printf("Kontrolna nit: %d. kontrolna flasa.\n", kontrolnaFlasa);
		}

		pthread_mutex_unlock(&brava);

		pthread_cond_broadcast(&kontrolnaIzlaz);
	}

	printf("Kontrolna nit je obradila %d flasa.\n", kontrolnaFlasa);

	pthread_exit(NULL);
}

void *redovnaObrada(void *id)
{
	int tid = (int)id;
	int ukupno = 0;
	int ind = 1;

	while (ind)
	{
		sleep(tid + 1);

		pthread_mutex_lock(&brava);

		if (brojacFlasa > BROJ_FLASA)
			ind = 0;
		else if (brojacFlasa % 5 == 0)
		{
			pthread_cond_signal(&kontrolnaUlaz);
			printf("Nit %d: %d. flasa je kontrolna!\n", tid, brojacFlasa);
			pthread_cond_wait(&kontrolnaIzlaz, &brava);

			if (brojacFlasa > BROJ_FLASA)
			{
				ind = 0;
			}
		}
		else
		{
			printf("Nit %d: %d. flasa.\n", tid, brojacFlasa);
			ukupno++;
			brojacFlasa++;
		}

		pthread_mutex_unlock(&brava);
	}

	pthread_cond_signal(&kontrolnaUlaz);

	printf("Nit %d, je obradila %d flasa.\n", tid, ukupno);

	pthread_exit(NULL);
}

int main()
{
	void *status;
	pthread_attr_t attr;

	pthread_attr_init(&attr);
	pthread_mutex_init(&brava, NULL);
	pthread_cond_init(&kontrolnaUlaz, NULL);
	pthread_cond_init(&kontrolnaIzlaz, NULL);

	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

	pthread_create(&niti[0], &attr, kontrola, (void*)0);
	pthread_create(&niti[1], &attr, redovnaObrada, (void*)1);
	pthread_create(&niti[2], &attr, redovnaObrada, (void*)2);

	pthread_join(niti[0], &status);
	pthread_join(niti[1], &status);
	pthread_join(niti[2], &status);

	pthread_attr_destroy(&attr);
	pthread_mutex_destroy(&brava);
	pthread_cond_destroy(&kontrolnaUlaz);
	pthread_cond_destroy(&kontrolnaIzlaz);

	pthread_exit(NULL);

	return 0;
}
{% endhighlight %}

# Domaći zadatak

Potrebno je napisati C program koji korišćenjem pthread niti simulira procesiranje prometa filijala. Program startuje 2 niti. Svaka od niti učitava promet iz pridruženih datoteka f0.txt i f1.txt. Datoteke su tekstualne i svaka linija je formata:

id iznos

gde je id 0 ili 1 i označava identitet klijenta, a iznos je ceo broj i označava promet za datog klijenta u datoj filijali. Početno stanje svakog klijenta je 0.
