---
layout: lekcija
title: Signali
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: lmw.png
active: false
comment: true
---
# Hardverske inovacije

Najznačajnija hardverska inovacija koja je omogućila multiprogramske sisteme je I/O (Input/Output) procesor (kontroleri, kanali). I/O procesor može da izvrši specijalizovani I/O program bez intervencije CPU-a. CPU mora samo da definiše niz aktivnosti za I/O procesor. Nakon toga I/O procesor izvršava I/O instrukcije i prekida CPU samo kada je čitava sekvenca I/O instrukcija izvršena. Za posledicu ovakvog načina upravljanja I/O uređajima imamo rasterećenje CPU-a pri sporoj interakciji poslova sa I/O uređajima.

**Slika** io procesor.

Uopšteni oblik instrukcije za I/O:

**Slika** Opšti oblik instrukcije

gde je:
* **did** - identifikacija I/O uređaja (traka, disk, štampač, ...),
* **op** - kod operacije (ulaz, izlaz, kontrola),
* **sts** - izlazni status (prenos_uspešno_završen, potrebna_intervencija, otkaz, kraj_datoteke, kraj_medijuma ...),
* **char *B** - I/O bafer.

Naredba STARTIO signalizira procesoru da može da započne I/O operaciju i blokira proces (Running -> Waiting) koji ju je izdao. Sada CPU može da se dodeli nekom drugom procesu, a I/O procesor asinhrono (nezavisno od CPU) radi na izvršenju I/O operacije.

Kada se I/O operacija (uspešno ili neuspešno) završi, I/O procesor "šalje" prekid u CPU. Operativni sistem tada zna da može ponovo da aktivira proces koji je tražio I/O (Waiting -> Ready).

# Prekidi

Prekid (interrupt) je signal, koji šalje hardver, procesoru da postoji događaj koga je potrebno odmah obraditi. Prekid obaveštava procesor da se pojavio neki uslov visokog prioriteta koji zahteva prekid izvršenja trenutnog posla. Procesor odgovara na prekid tako što premešta aktivan proces u spremne (Running -> Ready) i izvršava funkciju koja se zove **interrupt handler** da bi obradio prekid. Ovaj prekid je privremen i nakon njegove obrade procesor nastavlja sa svojim normalnim aktivnostima.

Algoritam rada procesora bez prekida:
**Slika** rada procesora bez prekida.

Algoritam rada procesora sa prekidom (uprošćena verzija):
**Slika** rad procesora sa prekidom.

Procedura prekida (interrupt handler):
**Slika** interrupt handler.

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
| SIGSTOP | 23 | Stop | Stop (cannot be caught or ignored) |
| SIGTSTP | 24 | Stop | Stop (job control, e.g., CTRL-z)) |
| SIGCONT | 25 | Ignore | Continued |
| SIGTTIN | 26 | Stop | Stopped--tty input (ref termio(7I)) |
| SIGTTOU | 27 | Stop | Stopped--tty output (ref termio(7I)) |
| SIGXFSZ | 31 | Core | File size limit exceeded (ref getrlimit(2)) |
| SIGRTMIN | 38 | Exit | Highest priority real-time signal |
| SIGRTMAX | 45 | Exit | Lowest priority real-time signal|
