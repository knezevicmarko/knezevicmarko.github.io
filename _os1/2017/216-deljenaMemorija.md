---
layout: lekcija
title: Deljena memorija
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: memory.png
active: true
comment: true
archive: false
---

Deljenom memorijom se definiše oblast (segment) memorije koja se mapira i deli između više procesa. Ovo je najbrži oblik IPC zato što nema posrednika. Informacije se mapiraju direktno iz memorijskog segmenta u adresni prostor pozivajućeg procesa. Segment kreira jedan proces, a više procesa mogu da čitaju i pišu iz njega. Jezgro OS održava specijalnu internu strukturu za svaki segment deljene memorije koji postoji u njegovom adresnom prostoru (linux/shm.h)

{% highlight c %}
struct shmid_ds
{
  struct ipc_perm shm_perm; /* operation perms */
  int shm_segsz; /* size of segment (bytes) */
  time_t shm_atime; /* last attach time */
  time_t shm_dtime; /* last detach time */
  time_t shm_ctime; /* last change time */
  unsigned short shm_cpid; /* pid of creator */
  unsigned short shm_lpid; /* pid of last operator */
  short shm_nattch; /* no. of current attaches */
  unsigned short shm_npages; /* size of segment (pages) */
  unsigned long *shm_pages; /* array of ptrs to frames  ‐> SHMMAX */
  struct vm_area_struct *attaches; /* descriptors for attaches */
};
{% endhighlight %}

## Sistemski poziv shmget()

Alocira segment deljene memorije.

{% highlight c %}
#include <sys/ipc.h>
#include <sys/shm.h>

int shmget(key_t key, size_t size, int shmflg);
{% endhighlight %}
Ako je uspešno izvedeno alociranje povratna vrednost je identifikator ka segmentu memorije, a ako nije -1 i postavlja errno na vrednosti:

* EINVAL (Invalid segment size specified)
* EEXIST (Segment exists, cannot create)
* EIDRM (Segment is marked for deletion, or was removed)
* ENOENT (Segment does not exist)
* EACCES (Permission denied)
* ENOMEM (Not enough memory to create segment)

Primer:
{% highlight c %}
int open_segment(key_t keyval, int segsize)
{
  int shmid;

  if((shmid=shmget(keyval,segsize,IPC_CREAT|0660)) == ‐1)
    return ‐1;

    return shmid;
}
{% endhighlight %}

## Sistemski poziv shmctl()

Izvršava operaciju kontrole definisanu sa cmd parametrom nad segmentom deljene memorije čiji je identifikator definisan sa shmid parametrom.
{% highlight c %}
#include <sys/ipc.h>
#include <sys/shm.h>

int shmctl(int shmid, int cmd, struct shmid_ds *buf);
{% endhighlight %}
Povratna vrednost je 0 ako je operacija uspešno izvedena ili -1 ako nije i errno prima vrednosti:

* EACCES (No read permission and cmd is IPC_STAT)
* EFAULT (Address pointed to by buf is invalid with IPC_SET and IPC_STAT commands)
* EIDRM (Segment was removed during retrieval)
* EINVAL (shmqid invalid)
* EPERM (IPC_SET or IPC_RMID command was issued, but calling process does not have write (alter) access to the segment)

Vrednosti parametra cmd mogu biti:

* **IPC_STAT** - Uzima vrednost shmid_ds strukture segmenta i skladišti u adresu buf argumenta
* **IPC_SET** - Postavlja vrednost ipc_perm člana ili shmid_ds strukture segmenta. Uzima vrednosti iz buf argumenta.
* **IPC_RMID** - Obeležava segment za uklanjanje. Stvarno uklanjanje se vrši kada se poslednji proces koji je prikačen na segment otkači. Ako nijedan proces nije nakačen na segment uklanjanje je trenutno.

## Sistemski poziv shmat()

Kači memorijski segment identifikovan sa shmid u adresni prostor pozivajućeg procesa.
{% highlight c %}
#include <sys/ipc.h>
#include <sys/shm.h>

void *shmat(int shmid, const void *shmaddr, int shmflg);
{% endhighlight %}
Ako je argument shmaddr 0 (NULL), jezgro OS pokušava da nađe nemapirani region. Ovakav način je preporučljiv. Adresa može biti definisana, ali se takav način uglavnom koristi samo za vlasnički hardver ili da bi se razrešili konflikti sa drugim aplikacijama.

Povratna vrednost je adresa na kojoj je segment prikačen za proces ili -1 i errno:

* EINVAL (Invalid IPC ID value or attach address passed)
* ENOMEM (Not enough memory to attach segment)
* EACCES (Permission denied)

Primer:
{% highlight c %}
char *attach_segment( int shmid)
{
  return (shmat(shmid,0,0));
}
{% endhighlight %}

## Sistemski poziv shmdt()

Odvaja segment deljenje memorije lociran na adresi definisanoj sa shmaddr od adresnog prostora pozivajućeg procesa. Segment koji treba da se odvaja mora prethodno biti prikačen pozivom shmat(). Parametar shmaddr je povratna vrednost sistemskog poziva shmat().
{% highlight c %}
#include <sys/ipc.h>
#include <sys/shm.h>

int shmdt(const void *shmaddr);
{% endhighlight %}
Ovim pozivom se ne uklanjanje segment iz jezgra OS. Nakon uspešnog odvajanja shm_nattch član strukture shmid_ds se smanjuje za jedan. Kada vrednost postane 0 jezgro fizički uklanja segment.

Ako je uspešno izvedeno povratna vrednost je 0, a ako nije -1 i errno:

* EINVAL (Invalid attach address passed)

### Primer

Program radi jednu od dve stari:

1. Ako nije prosledjen nijedan parametar komandne linije štampa se sadržaj segmenta deljene memorije
2. Ako je prosleđen jedan parametar komandne linije onda se taj parametar skladišti u segmentu deljene memorije.
{% highlight c %}
/* shmdemo.c */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHM_SIZE 1024

int main(int argc, char *argv[])
{
    key_t key;
    int shmid;
    char *data;
    int mode;

    if (argc > 2)
    {
        fprintf(stderr, "usage: shmdemo [data_to_write]\n");
        exit(1);
    }

    /* pravi kljuc */
    if ((key = ftok("shmdemo.c", 'R')) == -1)
    {
        perror("ftok");
        exit(1);
    }

    /* povezuje se (i ako je moguce kreira) segment */
    if ((shmid = shmget(key, SHM_SIZE, 0644 | IPC_CREAT)) == -1)
    {
        perror("shmget");
        exit(1);
    }

    /* prikaci segment da bi dobio pokazivac na njega */
    data = shmat(shmid, (void *)0, 0);
    if (data == (char *)(-1))
    {
        perror("shmat");
        exit(1);
    }

    /* citaj ili modifikuj segment, na osnovu argumenta komandne linije */
    if (argc == 2)
    {
        printf("writing to segment: \"%s\"\n", argv[1]);
        strncpy(data, argv[1], SHM_SIZE);
    }
    else
        printf("segment contains: \"%s\"\n", data);

    /* odvaja se od segmenta */
    if (shmdt(data) == -1)
    {
        perror("shmdt");
        exit(1);
    }

    return 0;
}
{% endhighlight %}
Program iz primera pristupa deljenoj memoriji bez upotrebe semafora. Zašto je to opasno?
