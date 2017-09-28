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

## Predlog rešenja za problem 1.

{% highlight c %}
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define JA 10
#define SA 10

pthread_mutex_t mutexMost;

void *juzniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   printf("Automobil %ld sa juga dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   printf("Automobil %ld sa juga prelazi most.\n", tid);
   sleep(1);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

 void *severniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   printf("Automobil %ld sa severa dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   printf("Automobil %ld sa severa prelazi most.\n", tid);
   sleep(1);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

int main (int argc, char *argv[])
{
   pthread_t threads[JA + SA];
   pthread_mutex_init(&mutexMost, NULL);
   long t;
   srand(time(NULL));

   for (t = 0; t < JA; t++)
      pthread_create(&threads[t], NULL, juzniAutomobili, (void *)t);

   for (t = 0; t < SA; t++)
       pthread_create(&threads[JA + t], NULL, severniAutomobili, (void *)t);

   pthread_exit(NULL);
}
{% endhighlight %}

## Predlog rešenja za problem 2.

{% highlight c %}
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define JA 10
#define SA 10

pthread_mutex_t mutexMost;
pthread_cond_t condMost;

long bj = 0, bs = 0;

void *juzniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   //printf("Automobil %ld sa juga dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   while (bs > 0)
       pthread_cond_wait(&condMost, &mutexMost);
   bj++;
   pthread_mutex_unlock(&mutexMost);

   printf("Automobil %ld sa juga prelazi most.\n", tid);
   sleep(1);

   pthread_mutex_lock (&mutexMost);
   bj--;
   pthread_cond_broadcast(&condMost);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

 void *severniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   //printf("Automobil %ld sa severa dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   while (bj > 0)
       pthread_cond_wait(&condMost, &mutexMost);
   bs--;
   pthread_mutex_unlock(&mutexMost);

   printf("Automobil %ld sa severa prelazi most.\n", tid);
   sleep(1);

   pthread_mutex_lock(&mutexMost);
   bs--;
   pthread_cond_broadcast(&condMost);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

int main (int argc, char *argv[])
{
   pthread_t threads[JA + SA];

   pthread_mutex_init(&mutexMost, NULL);
   pthread_cond_init (&condMost, NULL);

   long t;
   srand(time(NULL));

   for (t = 0; t < JA; t++)
      pthread_create(&threads[t], NULL, juzniAutomobili, (void *)t);

   for (t = 0; t < SA; t++)
       pthread_create(&threads[JA + t], NULL, severniAutomobili, (void *)t);

   pthread_exit(NULL);
}
{% endhighlight %}

## Predlog rešenja za problem 3.

{% highlight c %}
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define JA 50
#define SA 60
#define N 10

typedef struct _Pravac
{
    int ceka;
    int prelazi;
    int preslo;
} PRAVAC;

pthread_mutex_t mutexMost;
pthread_cond_t condMost;

PRAVAC jug = {0, 0, 0}, sever = {0, 0, 0};


void *juzniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   //printf("Automobil %ld sa juga dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   jug.ceka++;
   while ((sever.prelazi > 0) || (jug.preslo >= N))
       pthread_cond_wait(&condMost, &mutexMost);
   jug.ceka--;
   jug.prelazi++;
   if (sever.ceka > 0)
       jug.preslo++;
   printf("JUG:\tSever ceka %d\n", sever.ceka);
   printf("JUG:\tAutomobil %ld sa juga prelazi most.\n\n", tid);
   pthread_mutex_unlock(&mutexMost);

   sleep(1);

   pthread_mutex_lock (&mutexMost);
   jug.prelazi--;
   if (jug.prelazi == 0)
       sever.preslo = 0;
   pthread_cond_broadcast(&condMost);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

 void *severniAutomobili(void *threadid)
{
   long tid;
   tid = (long)threadid;

   int sTime = rand() % 3;
   sTime++;
   sleep(sTime);
   //printf("Automobil %ld sa severa dolazi na most posle vremena %d.\n", tid, sTime);

   pthread_mutex_lock (&mutexMost);
   sever.ceka++;
   while ((jug.prelazi > 0) || (sever.preslo >= N))
       pthread_cond_wait(&condMost, &mutexMost);
   sever.ceka--;
   sever.prelazi++;
   if (jug.ceka > 0)
       sever.preslo++;
   printf("SEVER:\tJug ceka %d\n", jug.ceka);
   printf("SEVER:\tAutomobil %ld sa severa prelazi most.\n\n", tid);
   pthread_mutex_unlock(&mutexMost);

   sleep(1);

   pthread_mutex_lock(&mutexMost);
   sever.prelazi--;
   if (sever.prelazi == 0)
       jug.preslo = 0;
   pthread_cond_broadcast(&condMost);
   pthread_mutex_unlock (&mutexMost);

   pthread_exit(NULL);
}

int main (int argc, char *argv[])
{
   pthread_t threads[JA + SA];

   pthread_mutex_init(&mutexMost, NULL);
   pthread_cond_init (&condMost, NULL);


   long t;
   srand(time(NULL));

   for (t = 0; t < JA; t++)
      pthread_create(&threads[t], NULL, juzniAutomobili, (void *)t);

   for (t = 0; t < SA; t++)
       pthread_create(&threads[JA + t], NULL, severniAutomobili, (void *)t);

   pthread_exit(NULL);
}
{% endhighlight %}
