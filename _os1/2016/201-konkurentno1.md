---
layout: lekcija
title: Operativni Sistem Uvod
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: lmw.png
active: true
comment: true
archive: true
---
# Operativni sistem

Računarski softver se može, na grubo, podeliti u dva tipa:

* Aplikacioni softver - obavlja posao koji je korisnik zadao.
* Sistemski softver - obavlja operacije samog računara.

Najbitniji sistemski program je operativni sistem, čiji je posao da kontroliše sve računarske resurse i obezbedi osnovu na kojoj se može razvijati aplikacioni softver. Operativni sistem se ponaša kao posrednik između korisnika računara i računarskog hardvera.

![Apstraktni pogled na komponente računarskog sistema.](/assets/os1/apstraktni_pogled.jpg "Apstraktni pogled na komponente računarskog sistema.")

Računarski sistem se može podeliti na četri komponente: hardver, operativni sistem, aplikacioni program i koristnik.

Operativni sistem obavlja posao sličan državnoj upravi. Kao i državna uprava, on nema korisnu funkciju sam po sebi, već obezbeđuje okruženje u kome drugi programi mogu da rade korisne stvari.

# Načini posmatranja operativnog sistema

## Operativni sistem kao proširenje hardvera ili virtuelna mašina (ili kao sloj između korisnika i kompjutera)

Operativni sistem sakriva hardverske detalje od korisnika i obezbeđuje prikladan interfejs za korišćenje sistema. Ako ovako posmatramo operativni sistem onda je njegova funkcija da obezbedi korisniku "prihvatljiviji" oblik komunikacije sa hardverskim komponentama tj. da se ponaša kao proširenje hardvera (virtuelna mašina). Mesto operativnog sistema je prikazano na slici ispod.

![Mesto operativnog sistema u računarskom sistemu.](/assets/os1/os_rs.jpg "Mesto operativnog sistema u računarskom sistemu.")

## Operativni sistem kao upravljač resursa

Računarski sistem ima mnoge resurse. Moderan računar se sastoji od procesora, memorije, tajmera, diska, miša, mrežnog interfejsa, štampača i mnogih drugih uređaja. Ako posmatramo OS kao upravljač resursa, onda je njegova funkcija da obezbedi uređeno i kontrolisano dodeljivanje procesora, memorije i ulazno / izlaznih (I/O) uređaja između više programa koji se takmiče za njihovo korišćenje.

![Resursi kojima upravlja operativni sistem.](/assets/os1/os_resursi.jpg "Resursi kojima upravlja operativni sistem.")

Deo operativnog sistema se nalazi u radnoj memoriji. Ovo je jezgro operativnog sistema. Ostatak glavne memorije sadrži korisničke programe i podatke. Dodeljivanje resorsa (glavne memorije) je kontrolisano od strane OS i hardvera za upravljanje memorijom u procesoru.

Moderan računarski sistem, opšte upotrebe, se sastoji od jednog ili više centralnih procesorskih jedinica i većeg broja kontrolora uređaja povezanih preko magistrale koja obezbeđuje i pristup deljenoj memoriji.

# Sistemski pozivi

Računarski poziv je način na koji program zahteva usluge od jezgra operativnog sistema. Ovo uključuje usluge vezane za hardver (npr. pristup hdd-u), kreiranje i startovanje novog procesa i komunikaciju sa servisima jezgra. Sistemski poziv obezbeđuje interfejs između procesa i operativnog sistema.
Savremeni operativni sistemi imaju na stotine sistemskih poziva. Linux ima preko 300 različitih poziva.
Sistemski pozivi se na grubo mogu podeliti u pet glavnih kategorija:

1. Kontrola procesa.
2. Upravljanje fajlovima.
3. Upravljanje uređajima.
4. Održavanje informacija.
5. Komunikacija.

# Razvoj operativnog sistema

**Uniprogramski sistemi** - računarski sistemi koji u jednom trenutku mogu da izvršavaju samo jedan korisnički program.

**Multiprogramski sistemi** - računarski sistemi koji mogu simultano da izvrše više korisničkih programa. Korisnički programi su po pravilu nezavisni, a koriste zajedničke resurse (procesor(e), memorije, I/O uređaje, datoteke, biblioteke...).

Multiprogramski sistemi su se razvili iz potrebe da se optimizuje korišćenje procesorskog vremena.

![Racionala multiprogramskog sistema.](/assets/os1/racionala.jpg "Racionala multiprogramskog sistema.")

Rani multiprogramski sistemi nisu obezbedili interakciju korisnika sa računarskim sistemom. **Vremenska podela procesora** je logična nadogradnja multiprogramskih sistema koja obezbeđuje interakciju sa korisnicima. Postoji više od jednog korisnika koji komunicira sa sistemom. Promena poseda CPU između korisnika je toliko brza da korisnik stiče utisak da samo on komunicira sa sistemom, ali u stvari on se deli između više korisnika. Svaki korisnik naizvmenično dobija kratak interval procesorskog vremena. Vremenska podela procesora je kompleksnija od multiprogramiranja zato što mora da obezbedi mehanizme za sinhronizaciju i komunikaciju između poslova različitih korisnika.

![Vremenska podela procesora između više (interaktivnih) korisnika.](/assets/os1/podela.jpg "Vremenska podela procesora između više (interaktivnih) korisnika.")

## Proces

Proces predstavlja instancu programa pri izvršavanju. Program nije proces i predstavlja **pasivni entitet**, fajl koji sadrži spisak instrukcija i koji se nalazi na hdd-u (najčešće se naziva izvršni fajl). Proces je **aktivni entitet** sa brojačem instrukcija postavljenim na sledeću instrukciju koja treba da se izvrši u kodu i skupom potrebnih resursa. Program postaje proces kada se izvršni fajl učita u glavnu memoriju.

* **Program**: skup instrukcija koje računar može da protumači i izvrši.
  - statičan,
  - nema stanja.
* **Proces**: program u izvršenju.
  - dinamički,
  - može da se kreira, izvrši i prekine,
  - prolazi kroz različita stanja (blokiran, izvršava se, spreman ...),
  - potrebni su mu resursi da bi OS mogao da ga alocira,
  - jedan ili više procesa mogu da izvršavaju isti kod.

Primer koji ilustruje razliku između procesa i programa:
{% highlight c %}
  int main()
  {
    int i, prod = 1;
    for (i = 0; i < 100; i++)
      prod = prod * i;
    return 0;
  }
{% endhighlight %}
Program sadrži jedan izraz množenja (prod = prod * i), ali proces će izvršiti 100 množenja, po jednom u svakoj iteraciji "for" petlje.

Proces roditelj kreira decu procese, koji mogu da kreiraju nove procese i tako formiraju stablo procesa.

Kada se pokrene operativni sistem, najčešće se kreira nekoliko procesa. Neki od tih procesa su prednji - mogu da komuniciraju sa korisnikom i obavljaju poslove za njih. Drugi su pozadinski procesi - nisu povezani sa korisnicima već imaju neku specifičnu funkciju. Takvi procesi ostaju u pozadini, aktiviraju se po potrebi i obavljaju funkcije kao što je npr. sinhronicacija sistemskog časovnika, štampanje itd. Ovi procesori se nazivaju demoni (deamons).

Pored procesa koji se kreiraju pri pokretanju operativnog sistema, novi procesi mogu da se kreiraju i kasnije. Najčešće se kreiraju tako što aktivan proces izdaje sistemski poziv za kreiranje jednog ili više novih procesa.

U interaktivnim sistemima korisnik pokreće program kucanjem odgovarajuće komande u terminalu ili duplim klikom na ikonu. Na Unix sistemima postoji samo jedan sistemski poziv za kreiranje procesa: **fork**. Ovaj poziv klonira pozivajući proces. Na primer, kada korisnik otkuca komandu u terminalu, recimo pstree, terminal forkuje proces dete i to dete izvrši program pstree.

Da bi smo u programskom jeziku C kreirali novi proces potrebno je pozvati funkciju **pid_t fork(void)** standardne biblioteke unistd. Dete proces se u odnosu na proces roditelj razlikuje, između ostalog, i po tome što dobija novi identifikator procesa. Nakon poziva funkcije fork() oba procesa (i roditelj i dete) imaju iste kopije podataka i nastavljaju da izvršavaju isti kod.

pid_t je generički tip procesa. Na Unix sistemima je short. fork() može da vrati samo tri stvari:

* 0 indikator procesa deteta
* -1 proces dete nije kreiran
* PID procesa deteta se vraća roditelju.

Iako ovo deluje prilično jednostavno, da bi i stvarno bilo jednostavno potrebno je znati nekoliko stvari. Pre svega, kada se na Unix sistemu proces "završi", on se više ne izvršava, ali mali ostatak podataka ostaje u memoriji i čeka da ga proces roditelj pokupi. Ovaj ostatak, između ostalog, sadrži i povratnu vrednost procesa. To znači da proces roditelj, nakon što pozove fork(), mora da sačeka da proces dete izađe (pozivom funkcije exit()). Pozivom funkcije wait() čeka bilo koje dete, a funkcijom waitpid() čeka neko konkretno dete. Ovaj čin čekanja dozvoljava da ostatak procesa deteta nestane.
Funkcija **void exit(int status)** je definisana u biblioteci stdlib, a funkcije **pid_t wait(int *status)** i **pid_t waitpid(pid_t pid, int *status, int options)** u bibliotekama sys/types i sys/wait.

## Primer fork1.c
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(void)
{
	pid_t pid;
	int rv;
	switch (pid = fork())
	{
		case -1:
			perror("fork"); // nesto nije kako treba
			exit(1); //roditelj izlazi
		case 0:
			printf(" DETE: Ovo je proces dete!\n");
			printf(" DETE: Moj PID je %d\n", getpid());
			printf(" DETE: PID mog roditelja je %d\n", getppid());
			printf(" DETE: Unesi moj izlazni status: ");
			scanf(" %d", &rv);
			printf(" DETE: Ja zavrsavam!\n");
			exit(rv);
		default:
			printf("RODITELJ: Ovo je proces roditelj!\n");
			printf("RODITELJ: Moj PID je %d\n", getpid());
			printf("RODITELJ: PID mog deteta je %d\n", pid);
			printf("RODITELJ: Ja sada cekam da moje dete izadje (exit())...\n");
			wait(&rv);
			printf("RODITELJ: Dete je izaslo sa statusom %d\n", WEXITSTATUS(rv));
			printf("RODITELJ: Ja zavrsavam\n");
	}
	return 0;
}
{% endhighlight %}

Naravno, postoji izuzetak od pravila da roditelj mora da sačeka da dete izađe. Roditelj može da ignoriše signal SIGCHLD i onda ne mora da poziva funkciju wait(). Ovo može da se postigne na sledeći način:

{% highlight c %}
main()
{
	signal(SIGCHLD, SIG_IGN); // sada ne mora da se ceka
  .
  .
  .
	fork(); fork(); fork();
}
{% endhighlight %}

Kada proces dete izađe, a nije sačekan (wait()), najčešće se u **ps** listi prikazuje kao "\<defunct\>". Takav status ima sve dok roditelj ne pozove wait(). Ako roditelj završi pre nego što pozove wait() za dete (pod pretpostavkom da ne ignoriše SIGCHLD), dete dobija novog roditelja. Taj roditelj je **init** proces (PID 1). Ovo nije problem ako dete još uvek nije završilo. Ako je dete već defunct onda ono postaje zombi proces. Na nekim sistemima init proces periodično uništava zombi procese. Na drugim sistemima init proces odbija da postane roditelj defunc procesima i odmah ih uništava.

# Kontrolni blok procesa

U operativnom sistemu svaki proces je predstavljen kontrolnim blokom procesa (PCB) ili zadataka. PCB je struktura podataka jezgra operativnog sistema koja fizički predstavlja proces u memoriji računara. Sadrži mnoge informacije povezane sa specifičnim procesom od kojih su najznačajnije:

* **Identifikator** - jedinstveni indetifikator procesa (pid),
* **Stanje** - ako se proces trenutno izvršava on je u stanju izvršenja (može biti blokiran, na čekanju isl.)
* **Prioritet** - nivo prioriteta relativno u odnosu na ostale procese
* **Brojač instrukcija** - Adresa sledeće instrukcije u programu koja će biti izvršena
* **Memorijski pokazivači** - Uključuje memorijske pokazivače ka programskom kodu i podacima koji su pridruženi procesu i ka memorijskim blokovima koji deli sa drugim procesima.
* **Konteksni podaci** - podaci koji su prisutni u registrima u trenutku kada se proces izvršava
* **Statusne informacije I/O uređaja**: Uključuje I/O zahteve, I/O uređaje priključene procesu, listu fajlova koje proces koristi isl.
* **Informacije naloga** - mogu da sadrže koliko je proces upotrebio procesorskog vremena, vremenska ograničenja i slično.

![Kontrolni blok procesa.](/assets/os1/bcp.jpg "Kontrolni blok procesa.")

## Stanje procesa

Svaki proces može biti u jednom od sledećih stanja:

![Stanja procesa.](/assets/os1/pStanja.jpg "Stanja procesa.")

* **New**: proces je kreiran.
* **Running**: izvršavaju se instrukcije.
* **Waiting**: Proces čeka da se neki događaj aktivira (npr. da se primi neki signal ili da I/O uređaj završi posao).
* **Ready**: Proces čeka da se dodeli procesoru.
* **Terminated**: Proces je završio sa radom.
