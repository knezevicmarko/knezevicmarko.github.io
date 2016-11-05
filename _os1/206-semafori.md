---
layout: lekcija
title: Semafori
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: semaphore1.png
active: true
comment: true
---

Semafor predstavlja generički mehanizam kontrole pristupa. Može da se koristi za kontrolu pristupa fajlovima, deljenoj memoriji isl. Osnovne funkcionalnosti semafora su: njegovo postavljanje, provera ili čekanje dok se ne ispune uslovi i postavljanje ("test-and-set").

Skup semafora je predstavljen strukturom semid_ds iz linux/sem.h (jedna semid_ds struktura za svaki skup semafora na sistemu).
{% highlight c %}
struct semid_ds
{
  struct ipc_perm sem_perm; /* permissions.. */
  time_t sem_otime; /* last semop time */
  time_t sem_ctime; /* last change time */
  struct sem *sem_base; /* ptr to first semaphore in array */
  struct wait_queue *eventn;
  struct wait_queue *eventz;
  struct sem_undo *undo; /* undo requests on this array */
  ushort sem_nsems; /* no. of semaphores in array */
};
{% endhighlight %}

U semid_ds strukturi postoji pokazivač na niz semafora (sem_base). Svaki član niza je struktura struct sem (linux/sem.h):
{% highlight c %}
/* One semaphore structure for each semaphore in the system. */
struct sem
{
  short sempid; /* pid of last operation */
  ushort semval; /* current value */
  ushort semncnt; /* num procs awaiting increase in semval */
  ushort semzcnt; /* num procs awaiting semval = 0 */
};
{% endhighlight %}

## Sistemski poziv semget()

Pozivom funkcije semget() dobija se identifikator skupa semafora.
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);
{% endhighlight %}
Povratna vrednost je identifikator skupa semafora ako je uspešno ili -1 i errno:

* EACCESS (permission denied)
* EEXIST (set exists, cannot create (IPC_EXCL))
* EIDRM (set is marked for deletion)
* ENOENT (set does not exist, no IPC_CREAT was used)
* ENOMEM (Not enough memory to create new set)
* ENOSPC (Maximum set limit exceeded)

Primer
{% highlight c %}
int open_semaphore_set (key_t keyval, int numsems)
{
  int sid;

  if (!numsems)
    return ‐1;

  if((sid = semget (mykey, numsems, IPC_CREAT | 0660)) == ‐1)
    return ‐1;

  return sid;
}
{% endhighlight %}

## Sistemski poziv semop()

Vrši operacije nad semaforima.

{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semop(int semid, struct sembuf *sops, size_t nsops);
{% endhighlight %}
Povratna vrednost je 0 ako je uspešno izvedeno (sve operacije izvršene) ili -1 i errno:

* E2BIG (The argument nsops is greater than SEMOPM, the maximum number of operations allowed per system call.)
* EACCES (The calling process does not have the  permissions  required  to perform  the  specified  semaphore operations, and does not have the CAP_IPC_OWNER capability.)
* EAGAIN (An operation could not proceed immediately and either IPC_NOWAIT was  specified in sem_flg or the time limit specified in timeout expired.)
* EFAULT (An address specified in either the sops or the timeout  argument isn't accessible.)
* EFBIG (For  some  operation  the  value  of  sem_num  is less than 0 or greater than or equal to the number of semaphores in the set.)
* EIDRM (The semaphore set was removed.)
* EINTR (While blocked in this system call, the thread caught  a  signal.)
* EINVAL (The semaphore set doesn't exist, or semid is less than zero, or nsops has a nonpositive value.)
* ENOMEM (The sem_flg of some operation specified SEM_UNDO and the  system does not have enough memory to allocate the undo structure.)
* ERANGE (For some operation sem_op+semval is greater than SEMVMX, the implementation dependent maximum value for semval.)

Argument sops je pokazivač ka strukturi tipa sembuf (linux/sem.h).
{% highlight c %}
struct sembuf
{
  ushort sem_num; /* semaphore index in array */
  short sem_op; /* semaphore operation */
  short sem_flg; /* operation flags */
};
{% endhighlight %}
Promenljiva sem_num je broj semafora u skupu kojim se manipuliše, sem_op je operacija koja se vrši nad semaforom. Ponašanje sistemskog poziva semop() zavisi od toga da li je sem_op nula, negativan ili pozitivan broj.

* **Negativan** - Alocira resurse. Blokira pozivajući proces dok je vrednost semafora manja od apsolutne vrednosti sem_op (Sve dok se dovoljan broj resursa ne oslobodi da bi proces mogao da ih alocira). Ako je vrednost semafora veća ili jednaka vrednosti sem_op vrednost semafora se smanjuje za sem_op.
* **Pozitivan** - Oslobađa resurse. Vrednost sem_op se dodaje na vrednost semafora.
* **Nula (0)** - Ovaj proces se blokira dok semafor ne dostigne vrednost 0.

Promenljiva sem_flg se koristi da bi se dodatno modifikovali efekti poziva semop(). Najčešće korišćene opcije su:

* IPC_NOWAIT - kojom poziv semop() vraća grešku EAGAIN ako se pojavi situacija koja bi normalno blokirala proces. Najčešće se koristi i slučajevima kada se samo proverava da li može da se alocira resurs.
* SEM_UNDO - semop() pamti promene na semaforu. Kada program završi sa radom jezgro OS automatski vraća sve promene koje su markirane sa SEM_UNDO. Koristi se ako se očekuje da program može da se neočekivano završi (npr. SIGKILL signalom) a potrebno je osloboditi resurse.

Primer
{% highlight c %}
struct sembuf sem_lock = {0, -1, IPC_NOWAIT};
if ((semop(sid, &sem_lock, 1) == -1)
  perror("semop");
{% endhighlight %}

## Sistemski poziv semctl()

Vrši kontrolne operacije nad skupom semafora definisanim sa semid ili nad nekim semaforom skupa definisanim sa semnum.

{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semctl(int semid, int semnum, int cmd, ... /* arg */);
{% endhighlight %}
Povratna vrednost je pozitivan ceo broj ili -1 i errno:

* EACCESS (permission denied)
* EFAULT (invalid address pointed to by arg argument)
* EIDRM (semaphore set was removed)
* EINVAL (set doesn't exist, or semid is invalid)
* EPERM (EUID has no privileges for cmd in arg)
* ERANGE (semaphore value out of range)

Poslednji argument arg, ako je potreban, mora biti definisan na sledeći način.
{% highlight c %}
union semun {
    int val;               /* used for SETVAL only */
    struct semid_ds *buf;  /* used for IPC_STAT and IPC_SET */
    ushort *array;         /* used for GETALL and SETALL */
};
{% endhighlight %}

Različita polja union semun se koriste zavisno od vrednosti cmd parametra poziva setctl(). Najznačajnije vrednosti cmd parametra mogu biti (za ostale man strane):

* **SETVAL** - Postavlja vrednost na definisanom semaforu na vrednost val člana unije semun.
* **GETVAL** - Vraća vrednost za zadati semafor.
* **SETALL** - Postavlja vrednost svih semafora u skupu na vrednost iz niza na koji pokazuje pokazivac array unije semun. Ne koristi se parametar semnum poziva semctl().
* **GETALL** - Vraća vrednost svih semafora u skupu i skladišti ih u niz na koji pokazuje pokazivač array unije semun. Vrednost semnum parametra se ne definiše.
* **IPC_RMID** - Uklanja skup iz jezgra.
* **IPC_STAT** - Preuzima semid_ds strukturu skupa i skladišti je na adresu buf argumenta semun unije.
* **IPC_SET** - Postavlja vrednost ipc_perm člana semid_ds strukture za skup. Uzima vrednost iz buf argumenta semun unije.

Primer 1.
{% highlight c %}
int get_sem_val (int sid, int semnum)
{
  return (semctl(sid, semnum, GETVAL, 0));
}
{% endhighlight %}

Primer 2.
{% highlight c %}
#define MAX_PRINTERS 5
printer_usage()
{
  int x;
  for (x = 0; x < MAX_PRINTERS; x++)
    printf("Printer %d: %d\n", x, get_sem_val(sid, x));
}
{% endhighlight %}

Primer 3.
{% highlight c %}
void init_semaphore(int sid, int sumnum, int initval)
{
  union semun semopts;
  semopts.val = initval;
  semctl(sid, semnum, SETVAL, semopts);
}
{% endhighlight %}

## Primer

{% highlight c %}
/* semdemo.c */
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

#define MAX_RETRIES 10

union semun
{
    int val;
    struct semid_ds *buf;
    ushort *array;
};

/*
** initsem() -- more-than-inspired by W. Richard Stevens' UNIX Network
** Programming 2nd edition, volume 2, lockvsem.c, page 295.
*/

int initsem(key_t key, int nsems)  /* key from ftok() */
{
  int i;
  union semun arg;
  struct semid_ds buf;
  struct sembuf sb;
  int semid;

  semid = semget(key, nsems, IPC_CREAT | IPC_EXCL | 0666);
  if (semid >= 0)
  { /* we got it first */
    sb.sem_op = 1;
    sb.sem_flg = 0;
    arg.val = 1;

    printf("press return\n");
    getchar();

    for(sb.sem_num = 0; sb.sem_num < nsems; sb.sem_num++)
    {
      /* do a semop() to "free" the semaphores. */
      /* this sets the sem_otime field, as needed below. */

      if (semop(semid, &sb, 1) == -1)
      {
        int e = errno;
        semctl(semid, 0, IPC_RMID); /* clean up */
        errno = e;
        return -1; /* error, check errno */
      }
    }
  }
  else if (errno == EEXIST)
  { /* someone else got it first */
    int ready = 0;
    semid = semget(key, nsems, 0); /* get the id */
    if (semid < 0)
      return semid; /* error, check errno */

    /* wait for other process to initialize the semaphore: */
    arg.buf = &buf;

    for(i = 0; i < MAX_RETRIES && !ready; i++)
    {
      semctl(semid, nsems-1, IPC_STAT, arg);
      if (arg.buf->sem_otime != 0)
        ready = 1;
      else
        sleep(1);
    }

    if (!ready)
    {
      errno = ETIME;
      return -1;
    }
  }
  else
    return semid; /* error, check errno */
  return semid;
}

int main(void)
{
  key_t key;
  int semid;
  struct sembuf sb;
  sb.sem_num = 0;
  sb.sem_op = -1;  /* set to allocate resource */
  sb.sem_flg = SEM_UNDO;

  if ((key = ftok("semdemo.c", 'J')) == -1)
  {
    perror("ftok");
    exit(1);
  }

  /* grab the semaphore set created by seminit.c: */
  if ((semid = initsem(key, 1)) == -1)
  {
    perror("initsem");
    exit(1);
  }

  printf("Press return to lock: ");
  getchar();

  printf("Trying to lock...\n");
  if (semop(semid, &sb, 1) == -1)
  {
    perror("semop");
    exit(1);
  }

  printf("Locked.\n");
  printf("Press return to unlock: ");
  getchar();

  sb.sem_op = 1; /* free resource */
  if (semop(semid, &sb, 1) == -1)
  {
    perror("semop");
    exit(1);
  }
  printf("Unlocked\n");

  return 0;
}
{% endhighlight %}

{% highlight c %}
/* semrm.c */
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

union semun
{
    int val;
    struct semid_ds *buf;
    ushort *array;
};

int main(void)
{
  key_t key;
  int semid;
  union semun arg;

  if ((key = ftok("semdemo.c", 'J')) == -1)
  {
    perror("ftok");
    exit(1);
  }

  /* grab the semaphore set created by seminit.c: */
  if ((semid = semget(key, 1, 0)) == -1)
  {
    perror("semget");
    exit(1);
  }

  /* remove it: */
  if (semctl(semid, 0, IPC_RMID, arg) == -1)
  {
    perror("semctl");
    exit(1);
  }

  return 0;
}
{% endhighlight %}
