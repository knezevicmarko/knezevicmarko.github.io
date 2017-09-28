---
layout: lekcija
title: Štampači
main_category: Materijali za vežbe
sub_category: Pthreads
image: printers.png
active: true
comment: true
archive: true
---

Predlog rešenja za zauzimanje resursa (štampača) korišćenjem pthread_mutex_trylock() funkcije.

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>
#include <unistd.h>

#define BROJ_STAMPACA 4
#define BROJ_NITI 10

pthread_mutex_t fulllock;
pthread_mutex_t locks[BROJ_STAMPACA];
pthread_cond_t condNestoSmisleno;

void *rad(void*d)
{
  int id = (int)d;
  int i = 0, success;

  while(1)
  {
    sleep(1);

    pthread_mutex_lock(&fulllock);

    for (i = 0; i < BROJ_STAMPACA; ++i)
    {
      success = pthread_mutex_trylock(&locks[i]);

      if (success == 0)
      {
        pthread_mutex_unlock(&fulllock);
        printf("Racunar %d stampa na stampacu %d.\n", id, i);
        sleep(3);
        pthread_mutex_unlock(&locks[i]);
        pthread_cond_signal(&condNestoSmisleno);
        printf("Racunar %d je otpustio stampac\n", id);
        break;
      }
    }

    if (success)
    {
      printf("Racunar %d nije zauzeo stampac %d.\n", id, i);
      pthread_cond_wait(&condNestoSmisleno, &fulllock);
      pthread_mutex_unlock(&fulllock);
      printf("Racunar %d nije zauzeo stampac %d posle cond_wait .\n", id, i);
    }

  }

  pthread_exit(NULL);
}

int main()
{
  pthread_t niti[BROJ_NITI];
  pthread_attr_t atribut;
  void *status;
  int i;

  pthread_mutex_init(&fulllock, NULL);

  for (i = 0; i < BROJ_STAMPACA; ++i)
  {
    pthread_mutex_init(&locks[i], NULL);
  }

  pthread_attr_init(&atribut);
  pthread_attr_setdetachstate(&atribut, PTHREAD_CREATE_JOINABLE);
  pthread_cond_init (&condNestoSmisleno, NULL);

  for (i = 0; i < BROJ_NITI; ++i)
  {
    pthread_create(&niti[i], &atribut, rad, (void *)i);
  }

  for (i = 0; i < BROJ_NITI; ++i)
  {
    pthread_join(niti[i], &status);
  }

  pthread_attr_destroy(&atribut);
  pthread_mutex_destroy(&fulllock);

  for (i = 0; i < BROJ_STAMPACA; ++i)
  {
    pthread_mutex_destroy(&locks[i]);
  }

  pthread_exit(NULL);
  return 0;
}
{% endhighlight %}
