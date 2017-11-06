---
layout: lekcija
title: Filozofi
main_category: Materijali za vežbe
sub_category: Pthreads
image: philosophy.png
active: true
comment: true
archive: false
---

Pet filozofa sedi za okruglim stolom. Svaki filozof provodi svoj život tako što naizmenično razmišlja i jede. Filozofi nisu veoma spretni, moraju da koriste dva štapića kada jedu. Nažalost, postoji samo pet štapića, između svaka dva susedna filozofa po jedna. Filozofi mogu da dohvate samo štapić koja je neposredno levo i neposredno desno od njih. Posude sa pirinčem se neprestano pune i filozofi su uvek gladni posle razmišljanja. Napisati C program koristeći pthread koji simulira ponašanje filozofa.

![Filozofi](/assets/os2/philtable.png "Filozofi"){:style="width: auto;"}

**Deadlock** nastaje ako su zadovoljena sledeća četri uslova:

1. Procesi imaju pravo ekskluzivnog pristupa resursima (Mutual exclusion).
2. Proces drže resurs sve vreme dok čeka pristup drugom resursu (Wait and hold).
3. Samo proces koji drži resurs može da ga otpusti (No preemption).
4. Postoji zatvoren krug procesa koji čekaju jedan na drugog (Circular wait).

#### Predlog rešenja **sa deadlockom**

{% highlight c %}
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutexi[BROJ_FILOZOFA];

void *doWork(void *t)
{
	long i = (long) t;

	while (1)
	{
		printf("Filozof %ld misli.\n", i);
		sleep(2); //mislim

		pthread_mutex_lock(&mutexi[i]);
		pthread_mutex_lock(&mutexi[(i + 1)%BROJ_FILOZOFA]);
		printf("Filozof %ld jede.\n", i);
		sleep(1); //jedem
		pthread_mutex_unlock(&mutexi[i]);
		pthread_mutex_unlock(&mutexi[(i + 1)%BROJ_FILOZOFA]);
	}

	pthread_exit(NULL);
}

int main(int argc, char **argv)
{
	long i;
	pthread_t niti[BROJ_FILOZOFA];
	pthread_attr_t attr;

	pthread_attr_init(&attr);
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_init(&mutexi[i], NULL);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_create(&niti[i], &attr, doWork, (void *)i);


	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_join(niti[i], NULL);


	pthread_attr_destroy(&attr);
	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_destroy(&mutexi[i]);

	pthread_exit(NULL);

	return 0;
}
{% endhighlight %}

#### Predlog rešenja koje poništava uslov **circular wait** za stvaranje *deadlocka*

{% highlight c %}
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutexi[BROJ_FILOZOFA];

void *doWork(void *t)
{
	long i = (long) t;

	while (1)
	{
		printf("Filozof %ld misli.\n", i);
		sleep(2); //mislim

		pthread_mutex_lock(&mutexi[i]);
		pthread_mutex_lock(&mutexi[i + 1]);
		printf("Filozof %ld jede.\n", i);
		sleep(1); //jedem
		pthread_mutex_unlock(&mutexi[i]);
		pthread_mutex_unlock(&mutexi[i + 1]);
	}

	pthread_exit(NULL);
}

void *doLast(void *t)
{
	while (1)
	{
		printf("Poslednji misli.\n");
		sleep(2);

		pthread_mutex_lock(&mutexi[0]);
		pthread_mutex_lock(&mutexi[BROJ_FILOZOFA - 1]);
		printf("Poslednji jede.\n");
		pthread_mutex_unlock(&mutexi[0]);
		pthread_mutex_unlock(&mutexi[BROJ_FILOZOFA - 1]);
	}
}


int main(int argc, char **argv)
{
	long i;
	pthread_t niti[BROJ_FILOZOFA];
	pthread_attr_t attr;

	pthread_attr_init(&attr);
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_init(&mutexi[i], NULL);

	for (i = 0; i < (BROJ_FILOZOFA -1); i++)
		pthread_create(&niti[i], &attr, doWork, (void *)i);

	pthread_create(&niti[i], &attr, doLast, (void *)i);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_join(niti[i], NULL);


	pthread_attr_destroy(&attr);
	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_destroy(&mutexi[i]);

	pthread_exit(NULL);

	return 0;
}
{% endhighlight %}


#### Predlog rešenja koje poništava uslov **wait and hold** za stvaranje *deadlocka*

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <time.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutexi[BROJ_FILOZOFA];

void *doWork(void *t)
{
	long i = (long) t;

	while (1)
	{
		printf("Filozof %ld misli.\n", i);
		sleep(2); //mislim

		pthread_mutex_lock(&mutexi[i]);
		printf("%ld thread ceka na trylock\n",i);
		sleep(rand()%10);
		if(!pthread_mutex_trylock(&mutexi[(i + 1)%BROJ_FILOZOFA])){
			printf("Filozof %ld jede.\n", i);
			sleep(1); //jedem
			pthread_mutex_unlock(&mutexi[i]);
			pthread_mutex_unlock(&mutexi[(i + 1)%BROJ_FILOZOFA]);
		}else{
		  	pthread_mutex_unlock(&mutexi[i]);
		}

	}

	pthread_exit(NULL);
}

int main(int argc, char **argv)
{
	long i;
	pthread_t niti[BROJ_FILOZOFA];
	pthread_attr_t attr;
	srand(time(NULL));
	pthread_attr_init(&attr);
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_init(&mutexi[i], NULL);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_create(&niti[i], &attr, doWork, (void *)i);


	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_join(niti[i], NULL);


	pthread_attr_destroy(&attr);
	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_destroy(&mutexi[i]);

	pthread_exit(NULL);

	return 0;
}
{% endhighlight %}

### Domaći 1.

Rešiti problem filozofa automatizacijom. Filozofi ne pristupaju štapićima istovremeno i uzimaju oba ili nijedan štapić.

#### Predlog rešenja
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <time.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutexi;
/******************************
fniz cuva broj dostupnih stapica
za svakog filozofa. Vrednosti mogu
biti 0, 1 i 2.
*******************************/
int fniz[BROJ_FILOZOFA];

int getLeviIndex(int index)
{
    return (index - 1) % BROJ_FILOZOFA;
}

int getDesniIndex(int index)
{
    return (index + 1) % BROJ_FILOZOFA;
}

void *doWork(void *t)
{
    long i = (long) t;
    int index;

    while (1)
    {
        printf("Filozof %ld misli.\n", i);
        sleep(2); //mislim
        while (1)
        {
            pthread_mutex_lock(&mutexi);

            if (fniz[i] == 2)
            {
                fniz[getLeviIndex(i)]--;
                fniz[getDesniIndex(i)]--;

                pthread_mutex_unlock(&mutexi);

                printf("Filozof %ld jede.\n", i);
                sleep(1); // jedem

                pthread_mutex_lock(&mutexi);
                fniz[getLeviIndex(i)]++;
                fniz[getDesniIndex(i)]++;
                pthread_mutex_unlock(&mutexi);
                break;
            }
            else
            {
                pthread_mutex_unlock(&mutexi);
                sleep(rand() % 5);
            }
        }
    }

    pthread_exit(NULL);
}

int main(int argc, char **argv)
{
    long i;
    pthread_t niti[BROJ_FILOZOFA];
    pthread_attr_t attr;
    srand(time(NULL));
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

    for (i = 0; i < BROJ_FILOZOFA; i++)
        fniz[i] = 2;

    pthread_mutex_init(&mutexi, NULL);

    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_create(&niti[i], &attr, doWork, (void *)i);


    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_join(niti[i], NULL);


    pthread_attr_destroy(&attr);
    pthread_mutex_destroy(&mutexi);

    pthread_exit(NULL);

    return 0;
}
{% endhighlight %}

Proširiti problem tako da se prati promena stanja filozofa. Stanja mogu biti: jede, misli i gladan.

#### Predlog rešenja
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <time.h>

#define BROJ_FILOZOFA 5

#define MISLIM 0
#define JEDEM 1
#define GLADAN 2

pthread_mutex_t mutexi;
/***********************************
fniz cuva stanje za svakog filozofa.
************************************/
int fniz[BROJ_FILOZOFA];
pthread_cond_t conds[BROJ_FILOZOFA];

int getLeviIndex(int index)
{
    return (index - 1) % BROJ_FILOZOFA;
}

int getDesniIndex(int index)
{
    return (index + 1) % BROJ_FILOZOFA;
}

void *doWork(void *t)
{
    long i = (long) t;
    int index;

    while (1)
    {
        pthread_mutex_lock(&mutexi);
        fniz[i] = MISLIM;
        pthread_mutex_unlock(&mutexi);
        printf("Filozof %ld misli.\n", i);
        sleep(2); //mislim

        pthread_mutex_lock(&mutexi);
        fniz[i] = GLADAN;
        // gladan sam
        pthread_mutex_unlock(&mutexi);
        printf("Filozof %ld je gladan.\n", i);

        pthread_mutex_lock(&mutexi);
        // ako je filozof i gladan i susedi ne jedu -> on moze da jede
        while (fniz[getLeviIndex(i)] == JEDEM || fniz[getDesniIndex(i)] == JEDEM)
            pthread_cond_wait(&conds[i], &mutexi);

        fniz[i] = JEDEM;
        pthread_mutex_unlock(&mutexi);
        printf("Filozof %ld jede.\n", i);
        sleep(1); //jede

        pthread_mutex_lock(&mutexi);

        index = getDesniIndex(i);
        // ako su ispunjeni uslovi obavestava desnog da moze da jede
        if (fniz[index] == GLADAN && fniz[getDesniIndex(index)] != JEDEM)
            pthread_cond_signal(&conds[index]);

        index = getLeviIndex(i);
        // ako su ispunjeni uslovi obavestava levog da moze da jede
        if (fniz[index] == GLADAN && fniz[getLeviIndex(index)] != JEDEM)
            pthread_cond_signal(&conds[index]);

        pthread_mutex_unlock(&mutexi);  
    }

    pthread_exit(NULL);
}

int main(int argc, char **argv)
{
    long i;
    pthread_t niti[BROJ_FILOZOFA];
    pthread_attr_t attr;
    srand(time(NULL));
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

    for (i = 0; i < BROJ_FILOZOFA; i++)
        fniz[i] = 2;

    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_cond_init(&conds[i], NULL);

    pthread_mutex_init(&mutexi, NULL);

    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_create(&niti[i], &attr, doWork, (void *)i);


    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_join(niti[i], NULL);


    pthread_attr_destroy(&attr);
    pthread_mutex_destroy(&mutexi);

    for (i = 0; i < BROJ_FILOZOFA; i++)
        pthread_cond_destroy(&conds[i]);

    pthread_exit(NULL);

    return 0;
}
{% endhighlight %}

### Domaći 2.

Neka su data fajlovi - klijenti.txt i fn.txt {n = 1, 2, 3 ...}

Fajl klijenti.txt sadrzi koliko korisnika ima banka, i u svakom redu trenutno stanje racuna za svakog korisnika.

{% highlight bash %}
brojKorisnika
id1 iznos1
id2 iznos2
...
{% endhighlight %}

Fajlovi fn.txt {n = 1,2,...} sadrze informacije o njihovim transakcijama.

{% highlight bash %}
idUplatioca idPrimaoca iznos
{% endhighlight %}
Potrebno je napisati C program koriscenjem pthread niti koji simulira procesiranje prometa filijala.
