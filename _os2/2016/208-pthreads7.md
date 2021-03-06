---
layout: lekcija
title: SID
main_category: Materijali za vežbe
sub_category: Pthreads
image: list.png
active: true
comment: true
archive: true
---

# Pretraga, dodavanje, brisanje

Postoje tri vrste operacija koje se mogu obavljati nad jednostruko ulančanom listom:

* pretraživanje,
* ubacivanje i
* brisanje.

Pretraživanje samo pregleda listu, tako da se može dozvoliti da više procesa pretražuju u paraleli. Ubacivanje dodaje novi element na kraj liste. Procesi koji ubacuju su međusobno isključivi, ali se ubacivanje može raditi u paraleli sa proizvoljnim brojem pretraživanja. Može se brisati element sa bilo koje pozicije u listi. U jednom trenutku može samo jedan proces koji briše da pristupi listi. Taj proces ima ekskluzivni pristup listi.


## Ne sinhronizovane operacije nad listama

{% highlight c %}
// lista.h
#ifndef LISTA_H
#define LISTA_H

void init_list();
void free_list();
void delete(int id);
void insert(int id);
void search(int id);

#endif
{% endhighlight %}

{% highlight c %}
// lista.c
#include <stdio.h>
#include <stdlib.h>

#include "lista.h"

struct _ELEMENT
{
    int val;
    struct _ELEMENT *next;
};

static struct _ELEMENT *first;

static struct _ELEMENT *_search(int id)
{

   struct _ELEMENT *temp;
   temp = first;

   while (temp->next != NULL)
   {
       if (temp->next->val == id)
       {
           return temp;
       }
       temp = temp->next;
   }

   return temp;
}

void init_list()
{
   first = malloc(sizeof(struct _ELEMENT));
   first->next = NULL;
}

void free_list()
{
   struct _ELEMENT *temp, *t;
   temp = first->next;
   first->next = NULL;

   while(temp != NULL)
   {
       t = temp;
       temp = temp->next;
       free(t);
   }
}

void search(int id)
{
   struct _ELEMENT *temp;
   temp = _search(id);

   if (temp->next == NULL)
       printf("SEARCH:\tLista ne sadrzi element %d\n", id);
   else
       printf("SEARCH:\tLista sadrzi element %d\n", id);
}

void insert(int id)
{
   struct _ELEMENT *temp, *t;
   temp = _search(id);

   if (temp->next != NULL)
   {
        printf("INSERT:\tLista sarzi element %d\n", id);
        return;
   }

   t = (struct _ELEMENT *)malloc(sizeof(struct _ELEMENT));
   t->val = id;
   t->next = NULL;

   temp->next = t;
   printf("INSERT:\tUbacen element %d\n", id);
}

void delete(int id)
{
    struct _ELEMENT *temp, *t;
    temp = _search(id);

    if (temp->next == NULL)
       printf("DELETE:\tLista ne sadrzi element %d\n", id);
   else
   {
       t = temp->next;
       temp->next = t->next;
       free(t);
       printf("DELETE:\tObrisan element %d\n", id);
   }
}
{% endhighlight %}

## Predlog rešenja sinhronizacije

{% highlight c %}
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#include "lista.h"

#define NUM_THREADS 50
#define ELEMENTS 8

void *doInsert(void *tid);
void *doSearch(void *tid);
void *doDelete(void *tid);

pthread_mutex_t mutex;
pthread_cond_t cond;
int nS = 0, nD = 0, nI = 0;

int main()
{
    pthread_t thread[NUM_THREADS];
    pthread_attr_t attr;
    void *status;
    long t;

    srand(time(NULL));

    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);
    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init (&cond, NULL);
    init_list();

    for (t = 0; t <	NUM_THREADS; t++)
    {
        int n = random() % 3;

        switch (n)
        {
            case 0:
                pthread_create(&thread[t], &attr, doInsert, (void *)t);
                break;
            case 1:
                pthread_create(&thread[t], &attr, doSearch, (void *)t);
                break;
            default:
                pthread_create(&thread[t], &attr, doDelete, (void *)t);
                break;
        }
    }

    for (t = 0; t < NUM_THREADS; t++)
    {
        pthread_join(thread[t], &status);
    }

    free_list();
    pthread_attr_destroy(&attr);
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);
    pthread_exit(NULL);
}

void *doSearch(void *tid)
{
    long id = (long) tid;
    int e = random() % ELEMENTS;

    pthread_mutex_lock(&mutex);
    while (nD > 0)
        pthread_cond_wait(&cond, &mutex);
    nS++;
    pthread_mutex_unlock(&mutex);

    search(e);

    pthread_mutex_lock(&mutex);
    nS--;
    pthread_mutex_unlock(&mutex);

    pthread_exit(NULL);
}

void *doInsert(void *tid)
{
    long id = (long) tid;
    int e = random() % ELEMENTS;

    pthread_mutex_lock(&mutex);
    while ((nI > 0) || (nD > 0))
        pthread_cond_wait(&cond, &mutex);
    nI++;
    pthread_mutex_unlock(&mutex);

    insert(e);

    pthread_mutex_lock(&mutex);
    nI--;
    pthread_cond_broadcast(&cond);
    pthread_mutex_unlock(&mutex);

    pthread_exit(NULL);
}

void *doDelete(void *tid)
{
    long id = (long) tid;
    int e = random() % ELEMENTS;

    pthread_mutex_lock(&mutex);
    while ((nS > 0) || (nI > 0) || (nD > 0))
        pthread_cond_wait(&cond, &mutex);
    nD++;
    pthread_mutex_unlock(&mutex);

    delete(e);

    pthread_mutex_lock(&mutex);
    nD--;
    pthread_cond_broadcast(&cond);
    pthread_mutex_unlock(&mutex);

    pthread_exit(NULL);
}
{% endhighlight %}

**Za domaći**:

1. Analizirati zašto ponuđeno rešenje nije pravedno i kako može da se napiše pravednije rešenje.
2. Ponuditi rešenje sa manjim brojem uslova koji se proveravaju.
