---
layout: lekcija
title: Redovi za poruke
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: message.png
active: true
comment: true
archive: false
---

# Međuprocesna komunikacija

Međuprocesna komunikacija (interprocess communication - IPC) predstavlja mehanizme operativnog sistema koji omogućavaju procesima da komuniciraju i dele podatke. Obično, procesi mogu da koriste IPC kao klijenti ili serveri. **Klijent** je proces koji zahteva uslugu od neke drugog procesa, a **server** je proces koji obezbeđuje zahtevanu uslugu. Jedan proces može da se ponaša i kao server i kao klijent, zavisno od situacije. Metodi za postizanje IPC su podeljeni u nekoliko kategorija:

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Metod | Kratak opis |
|----|----|
| File | Podaci uskladišteni na disku ili grupisani na zahtev fajl servera, kojim mogu da pristupaju više procesa |
| Signal | Sistemske poruke koje jedan proces šalje drugom. Obično se ne koriste za transfer podataka već za daljinsko upravljanje partnerskim procesima |
| Socket | Tok podataka poslat ka drugom procesu na istom računaru ili na nekom drugom računaru. Podaci upisani preko socket-a zahtevaju formatiranje da bi sačuvali granice poruka. |
| Message queue | Tok podataka sličan socket-ima koji obično čuvaju granice poruka. Obično se implementiraju preko OS i omogućavaju procesima da čitaju i pišu poruke, a da nisu direktno povezani |
| Pipe | Jednosmerni kanal podataka. Podaci upisani na kraju cevi u kojoj se upisuju podaci operativni sistem čuva dok ih ne pročita proces sa kraja cevi za čitanje podataka. |
| Semaphore | Jednostavna struktura koja sinhronizuje više procesa koji koriste deljene resurse. |
| Shared memory | Više procesa dobijaju pristup istom bloku memorije koji se ponaša kao deljeni bafer. |
| Message passing | Omogućava da više procesa komuniciraju redovima za poruke i/ili kanalima kojima ne upravlja OS |
| Memory-mapped file | Fajl mapiran u RAM koji može direktno da menja memorijske adrese. |

# Redovi za poruke

Redovi za poruke se mogu opisati kao interno povezana lista u adresnom prostoru jezgra OS. Proces može da kreira nov red za poruke ili da se poveže na neki postojeći. Povezivanjem na postojeći red procesi mogu da razmenjuju informacije preko zajedničkog reda za poruke. Jednom kreiran red za poruke ne može da nestane dok se ne uništi. Svi procesi koji su koristili red mogu da završe sa radom, ali će red još uvek da postoji. Korišćenjem komande **ipcs** može da se proveri da li postoji neki red za poruke koji nije uništen. Red za poruke može da se uništi komandom **ipcrm**.

Strukture podataka koje se kreiraju unutar adresnog prostora jezgra OS i obezbeđuju realizaciju redova za poruke su:

* Bafer za poruke
* Struktura jezgra **msg**
* Struktura jezgra **msqid_ds**
* Struktura jezgra **ipc_perm**

## Strukture podataka

Može da se posmatra kao šablon podataka za poruke. Iako programer može da definiše svoju strukturu za podatke, veoma je važno da se razume da postoji struktura podataka koja se naziva **msgbuf**. Ona je deklarisana u **linux/msg.h**:
{% highlight c %}
struct msgbuf
{
  long mtype; // tip poruke
  char mtext[1]; // tekst poruke
};
{% endhighlight %}
mtype - pozitivan broj, mtext - podaci poruke. Maksimalna veličina poruke na Linux sistemima definisana je u linux/msg.h
{% highlight c %}
#define MSGMAX 4056 // <= 4056 bajtova
{% endhighlight %}
Sve poruke unutar jednog reda za poruke se povezuju strukturom msg (linux/msg.h)
{% highlight c %}
// Jedna msg struktura za svaku poruku
struct msg
{
  struct msg *msg_next; //sledeca poruka u redu
  long msg_type;
  char *msg_spot; // adresa teksta poruke
  short msg_ts; // velicina teksta poruke
};
{% endhighlight %}
* **msg_next** - Pokazivač ka sledećoj poruci u redu. Poruke se skladište kao jednostruko povezana lista unutar adresnog prostora jezgra OS.
* **msg_type** - Tip poruke, dodeljen u strukturi msgbuf.
* **msg_spot** - Pokazivač na početak tela poruke.
* **msg_ts** - Dužina tela poruke.

Jezgro OS kreira, skladišti i održava jednu instancu strukture msqid_ds za (linux/msg.h) svaki red za poruke kreiran na sistemu.
{% highlight c %}
// Jedna msg struktura za svaku poruku
struct msqid_ds
{
  struct ipc_perm msg_perm;
  struct msg *msg_first; // prva poruka u redu
  struct msg *msg_last; // poslednja poruka u redu
  time_t msg_stime; // vreme poslednjeg msgsnd zahteva
  time_t msg_rtime; // vreme poslednjeg msgrcv zahteva
  time_t msg_ctime; // vreme poslednje promene
  struct wait_queue *wwait;
  struct wait_queue *rwait;
  ushort msg_cbytes;
  ushort msg_qnum; // Broj poruka trenutno na redu
  ushort msg_qbytes; //Maksimalan broj bajtova na redu
  ushort msg_lspid; // pid poslednjeg msgsend
  ushort msg_lrpid; // pid poslednjeg msgrcv
};
{% endhighlight %}
Jezgro OS skladišti informacije o dozvolama za IPC objekte u strukturi ipc_perm (linux/msg.h)
{% highlight c %}
struct ipc_perm
{
  key_t key;
  ushort uid; // euid i egid vlasnika
  ushort gid;
  ushort cuid; // euid i egid kreatora
  ushort cgid;
  ushort mode; // nacini pristupa
  ushort seq;
};
{% endhighlight %}
* **seq** -(slot usage sequence number). Svaki put kada se IPC objekat zatvori sistemskim pozivom (uništi), ova vrednost se poveća na maksimalni broj IPC objekta koji mogu da se pokrenu na sistemu.

![Strukture jezgra.](/assets/os1/strukture_jezgra.jpg "Strukture jezgra.")

## Sistemski poziv msgget()

Red za poruke može da se kreira ili da se poveže na već postojeći funkcijom:
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgget(key_t key, int msgflg);
{% endhighlight %}
msgget() kao povratnu vrednost ima ID reda ako je uspešno izvedeno ili -1 ako nije (errno se postavlja na odgovarajuću vrednost). Vrednosti za errno mogu biti:

* EACCESS (permission denied)
* EEXIST (Queue exists, cannot create)
* EIDRM (Queue is marked for deletion)
* ENOENT (Queue does not exist)
* ENOMEM (Not enough memory to create queue)
* ENOSPC (Maximum queue limit exceeded)

Prvi argument *key* je jedinstveni identifikator reda za poruke, na nivou sistema, na koji hoćemo da se povežemo (ili kreiramo). Svaki proces koji hoće da se poveže na ovaj red mora da koristi isti ključ. Drugi argument *msgflg*, govori funkciji msgget() šta da radi sa tim redom. Ako hoćemo da kreiramo red, polje mora da sadriži IPC_CREAT bitOR uparen (\|) sa dozvolama za taj red.
{% highlight c %}
int open_queue (key_t keyval)
{
  int qid;
  if ((qid = msgget(keyval, IPC_CREAT | 0660)) == -1)
    return -1;
  return qid;
}
{% endhighlight %}

### Ključ

Pošto je key_t u stvari long možemo da koristimo bilo koji broj. Problem nastaje ako hard-kodiramo neki broj i neki drugi nevezani proces hard-kodira isti broj ali želi drugi red za poruke. Rešenje je korišćenje funkcije koja generiše ključ kada joj se proslede dva argumenta:
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>

key_t ftok(const char *path, int id);
{% endhighlight %}

* **path** je putanja ka fajlu koji proces može da pročita,
* **id** je neki proizvoljno izabran karakter, npr. 'A'.

**ftok()** koristi informacije o navedenom fajlu i pomoću argumenta id generiše jedinstveni ključ za msgget(). Proces koji hoće da koristi isti red mora da generiše isti ključ, pa mora da prosledi iste parametre funkciji ftok().

## Sistemski poziv msgsnd()

Dodaje poruku na red za poruke.
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgsnd(int msqid, struct msgbuf *msgp, int msgsz, int msgflg);
{% endhighlight %}
vraća 0 ako je uspešno izvedeno ili -1 ako nije. U tom slučaju errno ima vrednosti:

* AGAIN (queue is full, and IPC_NOWAIT was asserted)
* EACCES (permission denied, no write permission)
* EFAULT (msgp address isn't accessable ‐ invalid)
* EIDRM (The message queue has been removed)
* EINTR (Received a signal while waiting to write)
* EINVAL (Invalid message queue identifier, nonpositive message type, or invalid message size)
* ENOMEM (Not enough memory to copy message buffer)
{% highlight c %}
int send_message (int qid, struct mymsgbuf *qbuf)
{
  int result, length;
  length = sizeof(struct mymesgbuf) - sizeof(long);
  if ((result = msgsnd(qid, qbuf, length, 0)) == -1)
    return -1;
  return result;
}
{% endhighlight %}

## Sistemski poziv msgrcv()

Čita poruku sa reda za poruke
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgrcv(int msqid, struct msgbuf *msgp, int msgsz, long mtype, int msgflg);
{% endhighlight %}
Povratna vrednost je broj bajtova koji su kopirani u bafer poruke ili -1 i errno:

* E2BIG (Message length is greater than msgsz, no MSG_NOERROR)
* EACCES (No read permission)
* EFAULT (Address pointed to by msgp is invalid)
* EIDRM (Queue was removed during retrieval)
* EINTR (Interrupted by arriving signal)
* EINVAL (msgqid invalid, or msgsz less than 0)
* ENOMSG (IPC_NOWAIT asserted, and no message exists in the queue to satisfy the request)
{% highlight c %}
int read_message (int qid, long type, struct mymesgbuf *qbuf)
{
  int result, length;
  length = sizeof (struct mymesgbuf) - sizeof(long);
  if ((result = msgrcv(qid, qbuf, length, type, 0)) == -1)
    return -1;
  return result;
}
{% endhighlight %}

## Sistemski poziv msgctl()

Vrši operacije kontrole na određenom redu za poruke.
{% highlight c %}
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

int msgctl(int msqid, int cmd, struct msqid_ds *buf);
{% endhighlight %}
Povratna vrednost je 0 ako je operacija uspešno izvedena ili ako nije -1 i errno:

* EACCES (No read permission and cmd is IPC_STAT)
* EFAULT (Address pointed to by buf is invalid with IPC_SET and IPC_STAT commands)
* EIDRM (Queue was removed during retrieval)
* EINVAL (msgqid invalid, or msgsz less than 0)
* EPERM (IPC_SET or IPC_RMID command was issued, but calling process does not have write (alter access to the queue))

Najčešće korišćene vrednosti parametra cmd su:

* IPC_STAT - Adresu msqid_ds strukture odgovarajućeg reda za poruke skladišti u bafer (buf).
* IPC_SET - Postavlja vrednost ipc_perm člana msqid_ds strukture odgovarajućeg reda za poruke. Uzima vrednosti iz buf argumenta.
* IPC_RMID - Briše odgovarajući red za poruke iz jezgra.

Primer 1.
{% highlight c %}
int get_queue_ds(int qid, struct msqid_ds *qbuf)
{
  if (msgctl(qid, IPC_STAT, qbuf) == -1)
    return -1;
  return 0;
}
{% endhighlight %}
Primer 2.
{% highlight c %}
int remove_queue(int qid)
{
  if (msgctl(qid, IPC_RMID, 0) == -1)
    return -1;
  return 0;
}
{% endhighlight %}
Primer 3.
{% highlight c %}
int change_queue_mode(int qid, char *mode)
{
  struct msqid_ds tmpbuf;
  get_queue_ds(qid, &tmpbuf);
  sscanf(mode, "%ho", %tmpbuf.msg_perm.mode);
  if (msgctl(qid, IPC_SET, &tmpbuf) == -1)
    return -1;
  return 0;
}
{% endhighlight %}

# Primer
{% highlight c %}
/* kirk.c */
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct my_msgbuf
{
    long mtype;
    char mtext[200];
};

int main(void)
{
    struct my_msgbuf buf;
    int msqid;
    key_t key;

    if ((key = ftok("kirk.c", 'B')) == -1)
    {
        perror("ftok");
        exit(1);
    }

    if ((msqid = msgget(key, 0644 | IPC_CREAT)) == -1)
    {
        perror("msgget");
        exit(1);
    }

    printf("Enter lines of text, ^D to quit:\n");
    buf.mtype = 1; /* we don't really care in this case */

    while(fgets(buf.mtext, sizeof buf.mtext, stdin) != NULL)
    {
        int len = strlen(buf.mtext);
        /* ditch newline at end, if it exists */
        if (buf.mtext[len-1] == '\n') buf.mtext[len-1] = '\0';
        if (msgsnd(msqid, &buf, len+1, 0) == -1) /* +1 for '\0' */
            perror("msgsnd");
    }

    if (msgctl(msqid, IPC_RMID, NULL) == -1)
    {
        perror("msgctl");
        exit(1);
    }
    return 0;
}
{% endhighlight %}
{% highlight c %}
/* spock.c */
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct my_msgbuf
{
    long mtype;
    char mtext[200];
};

int main(void)
{
    struct my_msgbuf buf;
    int msqid;
    key_t key;

    if ((key = ftok("kirk.c", 'B')) == -1)
    {  /* same key as kirk.c */
        perror("ftok");
        exit(1);
    }

    if ((msqid = msgget(key, 0644)) == -1)
    { /* connect to the queue */
        perror("msgget");
        exit(1);
    }

    printf("spock: ready to receive messages, captain.\n");
    for(;;)
    { /* Spock never quits! */
        if (msgrcv(msqid, &buf, sizeof(buf.mtext), 0, 0) == -1)
        {
            perror("msgrcv");
            exit(1);
        }
        printf("spock: \"%s\"\n", buf.mtext);
    }
    return 0;
}
{% endhighlight %}
