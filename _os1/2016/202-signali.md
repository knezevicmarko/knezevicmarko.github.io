---
layout: lekcija
title: Signali
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: signal.png
active: true
comment: true
archive: true
---
# Hardverske inovacije

Najznačajnija hardverska inovacija koja je omogućila multiprogramske sisteme je I/O (Input/Output) procesor (kontroleri, kanali). I/O procesor može da izvrši specijalizovani I/O program bez intervencije CPU-a. CPU mora samo da definiše niz aktivnosti za I/O procesor. Nakon toga I/O procesor izvršava I/O instrukcije i prekida CPU samo kada je čitava sekvenca I/O instrukcija izvršena. Za posledicu ovakvog načina upravljanja I/O uređajima imamo rasterećenje CPU-a pri sporoj interakciji poslova sa I/O uređajima.

![I/O procesor.](/assets/os1/iop.jpg "I/O procesor.")

Uopšteni oblik instrukcije za I/O:

![Opšti oblik instrukcije](/assets/os1/startio.jpg "Opšti oblik instrukcije"){:width="50%"}

gde je:
* **did** - identifikacija I/O uređaja (traka, disk, štampač, ...),
* **op** - kod operacije (ulaz, izlaz, kontrola),
* **sts** - izlazni status (prenos_uspešno_završen, potrebna_intervencija, otkaz, kraj_datoteke, kraj_medijuma ...),
* **char \*B** - I/O bafer.

Naredba STARTIO signalizira procesoru da može da započne I/O operaciju i blokira proces (Running -> Waiting) koji ju je izdao. Sada CPU može da se dodeli nekom drugom procesu, a I/O procesor asinhrono (nezavisno od CPU) radi na izvršenju I/O operacije.

Kada se I/O operacija (uspešno ili neuspešno) završi, I/O procesor "šalje" prekid u CPU. Operativni sistem tada zna da može ponovo da aktivira proces koji je tražio I/O (Waiting -> Ready).

# Prekidi

Prekid (interrupt) je signal, koji šalje hardver, procesoru da postoji događaj koga je potrebno odmah obraditi. Prekid obaveštava procesor da se pojavio neki uslov visokog prioriteta koji zahteva prekid izvršenja trenutnog posla. Procesor odgovara na prekid tako što premešta aktivan proces u spremne (Running -> Ready) i izvršava funkciju koja se zove **interrupt handler** da bi obradio prekid. Ovaj prekid je privremen i nakon njegove obrade procesor nastavlja sa svojim normalnim aktivnostima.

Algoritam rada procesora bez prekida:

![Rada procesora bez prekida.](/assets/os1/bezInter.jpg "Rada procesora bez prekida.")

Algoritam rada procesora sa prekidom (uprošćena verzija):

![Rad procesora sa prekidom.](/assets/os1/saInter.jpg "Rad procesora sa prekidom.")

* **PSW** (Processor Status Word) je poseban registar (registar stanja) unutar CPU-a koji čuva stanja (statuse), tj. informacije o stanjima određenih registara po poslednjoj izvršenoj operaciji. Svaki bit registra postavlja se nezavisno i registruje određenu situaciju nastalu posle izvršenja naredbe. Na primer PSW sadrži sledeće bitove:
  - sadržaj akumulatora nula (Z-bit)
  - sadržaj akumulatora negativan (N-bit)
  - sadržaj akumulatora pozitivan (P-bit)
  - postoji prenos (P-bit)
  - postoji prekoraćenje (V-bit)
* **PC** (Program Counter) ili brojač instrukcija je takođe poseban registar koji sadrži adresu sledeće naredbe koja treba da se izvrši.
* **w** - registar instrukcija
* **M** - niz adresa u radnoj memoriji na kojoj se nalaze podaci
* **ic** - kod vrste prekida (Interapt Condition)
* **ie** - Indikator dozvole prekida
* **ivec** - Vektor prekida (između ostalog se nalazi psw i pc funkcije koja obrađuje prekid)

**Procedura prekida** (interrupt handler):
Pre početka procedure prekida se prepisuju registri koji se menjaju unutar procedure. Odrađuju se instrukcije funkcije prekida i nakon toga se ic podešava na 0 (kao i ie). Psw i pc dobijaju vrednosti koje su imali pre prekida.

# Kontrola signala

Signali mogu da se opišu kao softverski prekidi. Kada se signal pošalje procesu, tada je moguće ući u funkciju signal handler. Ovaj način je sličan sistemskom ulasku u interrupt handler kada se primi prekid. Program se izvršava sekvencijalno ako se svaka instrukcija izvršava kako treba. U slučaju greške ili neke anomalije tokom izvršavanja programa, jezgro može da koristi signale da bi obavestilio proces. Sagnali se koriste i za komunikaciju i sinhronizaciju procesa i jednostavnu međuprocesnu komunikaciju (IPC).

Signal se generiše kada se pojavi neki događaj i tada jezgro prosleđuje taj događaj procesu. Pored komunikacije između procesa, postoje razne situacije kada je jezgro generator signala. Npr. kada veličina fajla prekorači dozvoljenu vrednost, kada je I/O uređaj spreman, kada se naleti na nedozvoljenu instrukciju ili kada korisnik pošalje signal prekida (terminal interrupt) (Ctrl-C ili Ctrl-Z).

Svaki signal ima ime koje počinje sa SIG i definiše se kao pozitivan jedinstven broj:
* 1 - 31: standardni signali,
* 32 - 63: nova clasa signala (signali u realnom vremenu)

Pokretanjem komande {% highlight c %}kill -l{% endhighlight %}sa Linux terminala dobija spisak svih signala sa brojem i odgovarajućim imenom.

Kada proces primi signal, jedna od tri stvari mogu da se dogode:

1. proces može da **ignoriše** signal,
2. može da uhvati signal i izvrši specijalizovanu funkciju **signal handler**,
3. da izvrši **predefinisanu akciju** za taj signal. Postoje četri moguće predefinisane akcije:
    * **Exit** - proce se prekida,
    * **Core** - proces se prekida i kreira se fajl jezgra,
    * **Stop** - proces se zaustavlja
    * **Ignore** ignoriše signal (ne preduzima nikakvu akciju).
    * **Continue** nastavlja proces ako je zaustavljen, ako nije ignoriše.

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Ime | Broj | Predefinisana akcija | Opis |
|----|----|----|-------|
| SIGHUP | 1 | Exit | Hangup (ref termio(7I)) |
| SIGINT | 2 | Exit | Interrupt (ref termio(7I)) |
| SIGQUIT | 3 | Core | Quit (ref termio(7I)) |
| SIGILL | 4 | Core | Illegal Instruction |
| SIGFPE | 8 | Core | Arithmetic exception |
| SIGKILL | 9 | Exit | Kill |
| SIGBUS | 10 | Core | Bus error--a misaligned address error |
| SIGSEGV | 11 | Core | Segmentation fault, an address reference boundary error |
| SIGSYS | 12 | Core | Bad system call |
| SIGCHLD | 18 | Ignore | Child process status changed |
| SIGPWR | 19 | Ignore | Power failor restart |
| SIGSTOP | 24 | Stop | Stop (cannot be caught or ignored) |
| SIGTSTP | 25 | Stop | Stop (job control, e.g., CTRL-z)) |
| SIGCONT | 26 | Ignore | Continued |
| SIGTTIN | 27 | Stop | Stopped--tty input (ref termio(7I)) |
| SIGTTOU | 28 | Stop | Stopped--tty output (ref termio(7I)) |
| SIGXFSZ | 34 | Core | File size limit exceeded (ref getrlimit(2)) |
| SIGRTMIN | 38 | Exit | Highest priority real-time signal |
| SIGRTMAX | 45 | Exit | Lowest priority real-time signal|

POSIX standardi obezbeđuju skup interfejsa za korišćenje signala u kodu. Implementacija signala na savremenim Linux-ima je potpuno kompatibilna sa POSIX-om.

Da bi mogli da koristimo signale moramo da koristimo funkciju **signal()** biblioteke signal.
{% highlight c %}
#include <signal.h>
void (*signal(int sig, void (*func)(int)))(int);
{% endhighlight %}
**Objašnjenje**: Funkciji signal() se, kao parametri, prosleđuju koji signal želimo da obrađujemo i adresu funkcije koja obrađuje taj signal. Funkcija koja obrađuje signal ima parametar tipa int i vraća void. Povratna vrednost funkcije signal() može da bude ili greška ili pokazivač ka prethodnoj funkciji koja obrađuje signal.

## Primer 1

{% highlight c %}
/* signal1.c */
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <signal.h>

int main(void)
{
	void sigint_handler(int sig);/* prototype */

	char s[200];

	if (signal(SIGINT, sigint_handler) == SIG_ERR)
	{
		perror("signal");
		exit(1);
	}

	printf("Enter a string:\n");
	if (!fgets(s, sizeof s, stdin))
		perror("fgets");
	else
		printf("You entered: %s\n", s);

	return 0;
}

void sigint_handler(int sig)
{
	printf("Not this time!\n");
}
{% endhighlight %}

## Primer 2

{% highlight c %}
/* signal2.c */
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void my_handler(int sig);/* function prototype */

int main(void)
{
	/* Part I: Catch SIGINT */

	signal(SIGINT, my_handler);
	printf ("Catching SIGINT\n");
	sleep(3);
	printf("Catch SIGINT ends\n");

	/*Part II: Ignore SIGINT */
	signal(SIGINT, SIG_IGN);
	printf("Ignoring SIGINT\n");
	sleep(3);
	printf("Ignoring SIGINT ends\n");

	/* Part III: Default action for SIGINT */
	signal(SIGINT, SIG_DFL);
	printf("Default action for SIGINT\n");
	sleep(3);
	printf("No SIGINT within 3 seconds\n");
	return 0;
}

/* User‐defined signal handler function */
void my_handler(int sig)
{
	printf("I got SIGINT, number %d\n", sig);
}
{% endhighlight %}

## Primer 3

{% highlight c %}
/* salji.c */
#include <signal.h>
#include <stdio.h>
#include <errno.h>

int main()
{
	int process_id;
	printf("Enter process_id which you want to send a signal:");
	scanf("%d", &process_id);
	if (!(kill(process_id, SIGINT)))
		printf("SIGINT sent to %d\n", process_id);
	else
		if (errno == EPERM)
			printf("Operation not permitted.\n");
		else
			printf("%d doesn't exist\n", process_id);

	return 0;
}
{% endhighlight %}

{% highlight c %}
/* primi.c */
#include <signal.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main()
{
	printf("This process id is %d. "
		"Waiting for SIGINT.\n", getpid());
	for (;;);
}
{% endhighlight %}
