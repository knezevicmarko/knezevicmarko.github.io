---
layout: lekcija
title: Primeri
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: homework.png
active: true
comment: true
archive: true
---

Primer realizacije višestrukog bafera korišćenjem deljene memorije i semafora.

{% highlight c %}
/* comm.h */
#ifndef COMM_H
#define COMM_H

#define N 10

#define MOD(a,b) ((((a)%(b))+(b))%(b))

typedef struct _DelMem
{
  int last;
  int count;
  int B[N];
} DELMEM;

#define SHMSIZE sizeof(DELMEM)

#endif
{% endhighlight %}

{% highlight c %}
/* init.c */
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/shm.h>
#include <signal.h>
#include <stdlib.h>

#include "comm.h"

#define P semop(sid, &mutexP, 1)
#define V semop(sid, &mutexV, 1)

void sighandler();

int sid;
int mid;
void *shmptr;

int main()
{
  int i;
  DELMEM *delmem;

  struct sembuf mutexP, mutexV;

  mutexP.sem_num = 0;
  mutexP.sem_op = -1;
  mutexP.sem_flg = SEM_UNDO;

  mutexV.sem_num = 0;
  mutexV.sem_op = 1;
  mutexV.sem_flg = SEM_UNDO;

  if ((sid = semget(ftok("init.c", 'S'), 3, 0666 | IPC_CREAT)) == -1)
  {
    perror("Greska u semget");
    exit(1);
  }

  printf("Uspesno kreirana grupa semafora %d\n", sid);
  ushort initS[] = {1, N, 0}; //mutex, space, count

  if((semctl(sid, 0, SETALL, initS)) == -1)
  {
    perror("Greska u semctl");
    exit(1);
  }

  if((mid = shmget(ftok("init.c", 'M'), SHMSIZE, 0666 | IPC_CREAT)) == -1)
  {
    perror("Greska u shmget");                  
    exit(1);
  }

  if((shmptr = shmat(mid, 0, 0)) == (void *)-1)
  {
    perror("Greska u shmat");
    exit(1);
  }

  delmem = (DELMEM *)shmptr;
  delmem->last = 0;
  delmem->count = 0;
  for (i = 0; i < N; i++)
    delmem->B[i] = -1;

  signal(SIGINT,sighandler);

  printf("Dobro dosli u kruzni bafer.\nMozete pokrenuti ostale programe.\n");
  for (;;)
  {
    sleep(20);
    P;
    for (i = 0; i < N; i++)
      printf("%d: %d\t", i, delmem->B[i]);
    printf("\n\n");
    V;
  }
}

void sighandler()
{
  printf("\nSignal primljen, pocinje uklanjanje objekata\n");

  if(shmctl(mid,IPC_RMID,0) == -1)
  {    
    perror("Greska u shmctl(IPC_RMID)");
    exit(1);                          
  }

  if(shmdt(shmptr) == -1)
  {
    perror("Greska u shmdt");
    exit(1);
  }

  if(semctl(sid, 0, IPC_RMID, 0) == -1)
  {
    perror("Greska u semctl(IPC_RMID)");
    exit(1);
  }

  printf("\nSegment oslobodjen\n");
  printf("\nKraj rada!\n");
  exit(0);
}
{% endhighlight %}

{% highlight c %}
/* producer.c */
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/shm.h>
#include <signal.h>
#include <stdlib.h>
#include <time.h>

#include "comm.h"

#define P(_x) semop(sid, _x, 1)
#define V(_x) semop(sid, _x, 1)

void sighandler();

void *shmptr;

int main()
{
  int sid;
  int mid;
  int x;
  DELMEM *delmem;

  struct sembuf mutexP, mutexV, spaceP, countV;

  mutexP.sem_num = 0;
  mutexP.sem_op = -1;
  mutexP.sem_flg = SEM_UNDO;

  spaceP.sem_num = 1;
  spaceP.sem_op = -1;
  spaceP.sem_flg = SEM_UNDO;

  mutexV.sem_num = 0;
  mutexV.sem_op = 1;
  mutexV.sem_flg = SEM_UNDO;

  countV.sem_num = 2;
  countV.sem_op = 1;
  countV.sem_flg = SEM_UNDO;

  srand(time(NULL));

  if ((sid = semget(ftok("init.c", 'S'), 3, 0666)) == -1)
  {
    perror("Greska u semget");
    exit(1);
  }

  if((mid = shmget(ftok("init.c", 'M'), SHMSIZE, 0666)) == -1)
  {
    perror("Greska u shmget");                  
    exit(1);
  }

  if((shmptr = shmat(mid, 0, 0)) == (void *)-1)
  {
    perror("Greska u shmat");
    exit(1);
  }

  delmem = (DELMEM *)shmptr;

  signal(SIGINT,sighandler);

  printf("Uspesno povezan producer sa semaforom i deljenom memorijom\n");

  for (;;)
  {
    x = random() % 10;
    sleep(2); //proizvodi
    printf("Proizveden broj %d\n", x);
    P(&spaceP);
    P(&mutexP);
    /* put(x) */
    delmem->B[delmem->last] = x;
    delmem->last = MOD((delmem->last + 1), N);
    delmem->count = delmem->count + 1;
    /* end */
    V(&mutexV);
    V(&countV);
  }
}

void sighandler()
{
  printf("\nSignal primljen, pocinje uklanjanje\n");

  if(shmdt(shmptr) == -1)
  {
    perror("Greska u shmdt");
    exit(1);
  }

  printf("\nKraj rada!\n");
  exit(0);
}
{% endhighlight %}

{% highlight c %}
/* consumer.c */
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/shm.h>
#include <signal.h>
#include <stdlib.h>

#include "comm.h"

#define P(_x) semop(sid, _x, 1)
#define V(_x) semop(sid, _x, 1)

void sighandler();

void *shmptr;

int main()
{
  int sid;
  int mid;
  int y, c;
  DELMEM *delmem;

  struct sembuf mutexP, mutexV, spaceV, countP;

  mutexP.sem_num = 0;
  mutexP.sem_op = -1;
  mutexP.sem_flg = SEM_UNDO;

  countP.sem_num = 2;
  countP.sem_op = -1;
  countP.sem_flg = SEM_UNDO;

  mutexV.sem_num = 0;
  mutexV.sem_op = 1;
  mutexV.sem_flg = SEM_UNDO;

  spaceV.sem_num = 1;
  spaceV.sem_op = 1;
  spaceV.sem_flg = SEM_UNDO;

  if ((sid = semget(ftok("init.c", 'S'), 3, 0666)) == -1)
  {
    perror("Greska u semget");
    exit(1);
  }

  if((mid = shmget(ftok("init.c", 'M'), SHMSIZE, 0666)) == -1)
  {
    perror("Greska u shmget");                  
    exit(1);
  }

  if((shmptr = shmat(mid, 0, 0)) == (void *)-1)
  {
    perror("Greska u shmat");
    exit(1);
  }

  delmem = (DELMEM *)shmptr;

  signal(SIGINT,sighandler);

  printf("Uspesno povezan consumer sa semaforom i deljenom memorijom\n");

  for (;;)
  {
    P(&countP);
    P(&mutexP);
    /* get(x) */
    c = MOD((delmem->last - delmem->count), N);
    y = delmem->B[c];
    delmem->B[c] = -1;
    delmem->count = delmem->count - 1;
    /* end */
    V(&mutexV);
    V(&spaceV);

    printf("Trosim vrednost %d\n", y);
    sleep(1);
  }
}

void sighandler()
{
  printf("\nSignal primljen, pocinje uklanjanje\n");

  if(shmdt(shmptr) == -1)
  {
    perror("Greska u shmdt");
    exit(1);
  }

  printf("\nKraj rada!\n");
  exit(0);
}
{% endhighlight %}

# Domaći zadatak

Pleme indijanaca jede zajedničku večeru iz kazana koji može da primi M porcija jela. Kada indijanac poželi da ruča onda se on posluži iz zajedničkog kazana, ukoliko kazan nije prazan. Ukoliko je kazan prazan indijanac budi kuvara i sačeka dok kuvar ne napuni kazan. Nije dozvoljeno buditi kuvara ukoliko se nalazi bar malo hrane u kazanu. Koristeći semafore, deljenu memoriju i forkovanje napisati program koji simulira ponašanje indijanaca i kuvara.

Pomoć pri rešavanju: Početni proces kreira potrebne elemente i onda se forkuje. Proces roditelj simulira kuvara dok se dete proces forkuje više puta i simulira indijance.
