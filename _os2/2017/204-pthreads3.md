---
layout: lekcija
title: Flaše
main_category: Materijali za vežbe
sub_category: Pthreads
image: bottle.png
active: true
comment: true
archive: false
---

# Flaše

Neka je dat unapred poznat broj predmeta(flaša) N. Napisati C program koji pomoću pthread biblioteke simulira rad 3 niti. Prve 2 niti uzimaju jedan po jedan predmet(flašu) i svaka obrađuje preuzeti predmet nezavisno, u dužini t1, odnosno t2. Nakon što završe jedan predmet, preuzimaju sledeći. Predmeti su numerisani rednim brojevima. Kada na red dođe predmet (flaša) čiji je broj deljiv sa 5 (5, 10, 15,...) tada obradu tog predmeta preuzima 3-ća nit. Obrada predmeta od strane 3-će niti traje t3. Ako su i ostale 2 niti završile svoje predmete dok treća obađuje svoj, ne smeju da uzimaju dalje predmete na obradu. Tek kad 3-ća nit završi svoj posao, ostale 2 niti mogu da nastave. 3-ća nit miruje, dok ostale 2 niti obrađuju predmete do sledećg čiji je redni broj deljiv sa 5. Program dok radi treba da ispisuje poruke koja je nit u kom trenutku preuzela koji predmet na obradu. Program prestaje sa radom kada se potroše svi predmeti.

{% highlight c %}
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define N 51

pthread_mutex_t flasaLock;
pthread_cond_t condRegular;
pthread_cond_t condSpecial;

int brojFlasa = 1;

void *regularWork(void *t)
{
	long time = (long)t;
	int obrada;

	while (1)
	{
		pthread_mutex_lock(&flasaLock);
		while ((brojFlasa % 5 == 0) && (brojFlasa <= N))
			pthread_cond_wait(&condRegular, &flasaLock);
		if (brojFlasa > N)
		{
			pthread_cond_signal(&condSpecial);
			pthread_mutex_unlock(&flasaLock);
			break;
		}
		obrada = brojFlasa++;
		if (brojFlasa % 5 == 0)
			pthread_cond_signal(&condSpecial);
		pthread_mutex_unlock(&flasaLock);
		sleep(time + 1);
		printf("Nit %ld je obradila flasu %d\n", (long)t, obrada);
	}

	pthread_exit(NULL);
}

void *specialWork(void *t)
{

	while(1)
	{
		pthread_mutex_lock(&flasaLock);
		while ((brojFlasa % 5 != 0) && (brojFlasa <= N))
			pthread_cond_wait(&condSpecial, &flasaLock);
		if (brojFlasa > N)
		{
			pthread_cond_broadcast(&condRegular);
			pthread_mutex_unlock(&flasaLock);
			break;
		}
		sleep(1);
		printf("Kontrolna nit je obradila flasu %d\n", brojFlasa++);
		pthread_cond_broadcast(&condRegular);
		pthread_mutex_unlock(&flasaLock);
	}

	pthread_exit(NULL);
}


int main (int argc, char *argv[])
 {
   long i;
   pthread_t threads[3];
   pthread_attr_t attr;

   /* Initialize mutex and condition variable objects */
   pthread_mutex_init(&flasaLock, NULL);
   pthread_cond_init (&condRegular, NULL);
   pthread_cond_init (&condSpecial, NULL);

   /* For portability, explicitly create threads in a joinable state */
   pthread_attr_init(&attr);
   pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

   for (i = 0; i < 2; i++)
   		pthread_create(&threads[i], &attr, regularWork, (void *)i);

   pthread_create(&threads[i], &attr, specialWork, NULL);

   /* Wait for all threads to complete */
   for (i=0; i < 3; i++)
     pthread_join(threads[i], NULL);


   printf ("Main(): Waited on %ld  threads. Done.\n", i);

   /* Clean up and exit */
   pthread_attr_destroy(&attr);
   pthread_mutex_destroy(&flasaLock);
   pthread_cond_destroy(&condSpecial);
   pthread_cond_destroy(&condRegular);
   pthread_exit(NULL);

 }
{% endhighlight %}

# Domaći zadatak

Potrebno je napisati C program koji korišćenjem pthread niti simulira procesiranje prometa filijala. Program startuje 2 niti. Svaka od niti učitava promet iz pridruženih datoteka f0.txt i f1.txt. Datoteke su tekstualne i svaka linija je formata:

id iznos

gde je id 0 ili 1 i označava identitet klijenta, a iznos je ceo broj i označava promet za datog klijenta u datoj filijali. Početno stanje svakog klijenta je 0.
