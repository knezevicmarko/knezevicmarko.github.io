---
layout: lekcija
title: Uvod
main_category: Materijali za vežbe
sub_category: Linux
image: homework.png
active: false
comment: true
---

# 1. Istorijat

* **UNIX je operativni sistem opšte namene** (i serveri i radne stanice) koji je svoj životni put započeo 1969. godine u Bell Labs. Grupa AT&T programera koja je najodgovornija za razvoj UNIX-a (tada se zvao UNICS, ili Uniplexed Operating and Computing System) su Ken Thompson, Dennis Ritchie, Brian Kernighan, Douglas McIlroy, i Joe Ossanna. Programski jezik C je konstruisan da bi se napisao UNIX.
* **Danas je UNIX zajednički naziv za široku familiju operativnih sistema** koji zadovoljavaju propisane standarde. Neka od UNIX izdanja su komercijalna, kao IBM AIX i HP-UX, dok neka izdanja karakteriše otvoreni kod, kao Linux i Free-BSD. Na slici je prikazan uprošćen dijagram razvoja UNIX-olikih sistema.
**slika razvoj UNIX-olikih sistema**
* Alternativa kvalitetnim, ali skupim UNIX sistemima je Linux, koji spaja odličnu pouzdanost i performanse sa besplatnim i potpuno dostupnim izvornim kodom. Pored tradicionalnog CLI (Command Line Interface) pristupa, danas je za Linux dostupno više grafičkih okruženja, kao što su GNOME, KDE, XFCE, itd. Linux 2009. godine pokreće više od polovine svih servera na Internetu, a polako se probija i na desktop tržište. Ovaj kurs se bazira upravo na Linuxu.
* **Verzije se među sobom razlikuju manje ili više.** Većina verzija je kompatibilna sa jednom od dve verzije SVR4 (System V) i BSD.
* **Pokušaji standardizacije:** najuspešniji - standard 1003 (IEEE 1003) – poznat kao POSIX (Portable Operating Systems Interface) standard. POSIX je u jednom trenutku spojen sa Single Unix Specification (SUS) standardom, ali je zadržan isti naziv.

# 2. Distribucija

* **1984. Richard Stallman je otpočeo sa GNU (Gnu's Not Unix)** projektom sa ciljem kreiranja operativnog sistema koji liči na Unix, a može biti slobono distribuiran. Iz GNU projekta potekla je *gcc* kompajler kolekcija, *bash shell* itd.
* **1991. Linus Torvalds, Finski diplomac, je započeo rad na Unix-like sistemu - Linux.** Linux je samo kernel(jezgro OS-a), dok su file sistem, shell itd. kreacije drugih (često GNU organizacije). Licenca pod kojom se Linux distribuira je [General Public License (GPL)](http://gnu.org/licenses/licenses.html). Zahtev ove licence je da **sve izmene koje se prave na nekom GPL kodu budu dostupne zajednici.**
* **Od 1994. BSD Unix se distribuira pod [BSD licencom](http://opensource.org/licenses/bsd-license.php)** koja dozvoljava slobodne izmene bez zahteva da izvorni kod bude dostupan drugima. Nejgove poznate verzije NetBSD, FreeBSD i OpenBSD projects. On čini i osnovu Darwin tehnologije (**na njoj je baziran Mac OS X**).
* **Linux distribucija** je skup biblioteka i raznorodnih softverskih alata izgrađenih oko Linux kernela. Većina distribucija sastoji se iz tzv. paketa, koji pored fajlova sadrže i međuzavisnosti sa drugim paketima. Većina tog softvera je otvorenog koda, pod GNU ili BSD   licencom. Moderne distribucije poseduju i preko 30000 paketa! Neke od Linux distribucija su:
  * **Komercijalne** - Red Hat Enterprise Linux (RHEL), Suse Linux Enterprise Server (SLES).
  * **Sa komercijalnom podrškom** - Fedora (Red Hat), openSUSE (Novell), Ubuntu (Canonical).
  * **Razvijene od strane zajednice** - Debian, Gentoo, Arch, Slackware.
* **Osnovne komponente** svake distribucije su:
  * **User interface** - počeo kao command-line interface (CLI) sistem; graphical user interface (GUI) – npr. Mac OS X Aqua, zatim Linux KDE i GNOME interfejsi
  * **kernel** - samo jezgro OS-a (višekorisnički, multitasking, monolitni)
  * **shell** - command line interpreter obezbeđuje komunikaciju korsinika sa OS-om. Neki od najrasprostranjenijih su:
    * sh (Bourne shell)
    * C shell (csh); TCSH (TENEX C shell)
    * Korn shell (ksh); PDKSH (Public Domain Korn shell)
    * **bash (Bourne Again Shell)**
    * Z shell

# 3. Opšti pregled Linux sistema

## Komponente OS-a

Linux je višekorisnički, višeprocesni operativni sistem sa potpunim skupom UNIX kompatibilnih alata, projektovan tako da poštuje **relevantne POSIX standarde**. Linux sistemi podržavaju tradicionalnu UNIX semantiku i potpuno implementiraju standardni UNIX mrežni model. Linux OS se satoji od:

* **KERNELA**,
* **sistemskog softvera**,
* **korisničkih aplikacija**,
* **programskih prevodilaca i njihovih odgovarajućih biblioteka** (GCC - GNU C Compiler i C biblioteka za Linux) i
* **dokumentacije**.

Svi programi, uključujući i sistemske, funkcionišu na nivou iznad kernela, što se naziva **(1) korisnički režim rada**, dok se sistemske  aktivnosti  poput pristupa  hardveru obavljaju na nivou kernela, odnosno u **(2) sistemskom režimu rada** (supervisory mode). **Slika** Komponente OS-a.

**Kernel je modularizovan** (modularna monolitna arhitektura), odnosno uvedeni su izmenljivi drajverski moduli (loadable kernel modules), a standardizovan je i konfiguracioni interfejs.

**Moduli kernela** su delovi kernelskog koda koji može da se prevede, napuni u memoriju ili izbaci iz memorije nezavisno od ostatka kernela. Kernelski moduli implementiraju drajvere za hardverske uređaje, novi fajl sistem, mrežne protokole, itd. Moduli omogućavaju raznim programerima da napišu i distribuiraju drajvere koji ne moraju da prođu GPL licencu (karakterističan primer su video drajveri). Potrebni drajveri pune se u memoriju kao moduli kernela. Module Linux kernela čine tri komponente:

  * **upravljanje modulom** - omogućava punjenje modula u kernelsku memoriju i komunikaciju modula sa ostatkom kernela, proveru da li je modul u memoriji i da li se koristi i izbacivanje modula iz memorije (pod uslovom da se modul ne koristi),
  * **registracija drajvera** - omogućava modulu da objavi ostatku kernela da je novi drajver u memoriji i da je raspoloživ za korišćenje. Kernel održava dinamičku tabelu drajvera koji se pomoću posebnog seta programa mogu napuniti ili izbaciti iz memorije u svakom trenutku,
  * **rezolucija konflikata** - mehanizam koji služi da spreči hardverske konflikte tako što omogućava drajveru da rezerviše hardverske resurse (IRQ, DMA1, ports) i time spreči druge drajvere ili autoprobe funkciju da ih koriste.

## Komponente kernela

**Komponente kernela** su sledeće:

* **upravljanje procesima** - kreira procese i omogućava višeprocesni rad (multitasking)
* **upravljanje memorijom** - kontroliše dodeljivanje memorije i swap prostora procesima, kernelskim komponentama kao i bafersko keširanje
* **upravljanje fajl sistemima** (VFS, Virtual File System)
* **apstrakcija mrežnih servisa**
* **podrška za hardverske uređaje**, podrška za različite sisteme datoteka, podrška za TCP/IP...
**Slika** Komponente kernela.

# 4. Blok uređaji i administracija fajl sistema

## Osnovni zadaci

Četiri su osnovna tipa zadataka u administraciji disk fajl sistema:
* **formatiranje diska na niskom nivou** - većina diskova koji se danas proizvode fabrički su preformatirani
* **podela diska na particije**
* **kreiranje fajl sistema** na particijama diska
* **aktiviranje fajl sistema** - montiranjem (mounting) na odgovarajuće direktorijume, čime se formira struktura aktivnog direktorijumskog stabla. Ovaj postupak se obavlja ili automatski, prilikom podizanja sistema (definisano u fajlu /etc/fstab), ili ručno, komandom mount.
**Slika** izgled hard diska.

## Master Boot Record (MBR), boot sektor i particiona tabela

* **Informacije o svim particijama diska čuvaju se u prvom logičkom sektoru**, tj. u prvom sektoru prve staze sa prve površine diska. Ovaj sektor je poznat pod imenom Master Boot Record (MBR) i njemu BIOS pristupa prilikom boot procedure.
* **MBR sadrži mali program** koji očitava particionu tabelu, proverava koja je particija aktivna, i očitava prvi sektor aktivne particije (boot sektor). U boot sektoru se nalazi program čijim pokretanjem započinje boot-strap, odnosno punjenje RAM memorije operativnim sistemom.
* **Particionisanje   diska   je   konvencija** koje se pridržava većina operativnih sistema uključujući i UNIX i MS Windows.
* **Informacije o particionoj tabeli** mogu se dobiti pomoću komande *fdisk -l*, recimo:
{% highlight bash %}
$ sudo fdisk -l /dev/sda
{% endhighlight %}
{% highlight bash %}
Disk /dev/sda: 465,8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0c210c20

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048 234598399 234596352 111,9G  7 HPFS/NTFS/exFAT
/dev/sda2       234598400 235519999    921600   450M 27 Hidden NTFS WinRE
/dev/sda3       235522046 976773119 741251074 353,5G  5 Extended
/dev/sda5       307208192 976773119 669564928 319,3G  7 HPFS/NTFS/exFAT
/dev/sda6       235522048 243333119   7811072   3,7G 82 Linux swap / Solaris
/dev/sda7       243335168 307195903  63860736  30,5G 83 Linux

Partition table entries are not in disk order.
{% endhighlight %}

* Jedno od najvećih ograničenja MBR-a je korišćenje 32 bita za skladištenje adresa blokova. Ako hard disk ima sektore od 512 bajtova MBR može da adresira samo 2TB (2<sup>32</sup> x 512 B).

## GUID Partition Table (GPT)
