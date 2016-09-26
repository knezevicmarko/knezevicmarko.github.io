---
layout: lekcija
title: Operativni Sistem Uvod
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: linux.png
active: false
comment: true
---
# Operativni sistem

Računarski softver se može, na grubo, podeliti u dva tipa:

* Aplikacioni softver - obavlja posao koji je korisnik zadao.
* Sistemski softver - obavlja operacije samog računara.

Najbitniji sistemski program je operativni sistem, čiji je posao da kontroliše sve računarske resurse i obezbedi osnovu na kojoj se može razvijati aplikacioni softver. Operativni sistem se ponaša kao posrednik između korisnika računara i računarskog hardvera.

**Slika** Abstraktni pogled na komponente računarskog sistema.

Računarski sistem se može podeliti na četri komponente: hardver, operativni sistem, aplikacioni program i koristnik.

Operativni sistem obavlja posao sličan državnoj upravi. Kao i državna uprava, on nema korisnu funkciju sam po sebi, već obezbeđuje okruženje u kome drugi programi mogu da radi korisne stvari.

# Načini posmatranja operativnog sistema

## Operativni sistem kao proširenje hardvera ili virtuelna mašina (ili kao sloj između korisnika i kompjutera)

Operativni sistem sakriva hardverske detalje od korisnika i obezbeđuje prikladan interfejs za korišćenje sistema. Ako ovako posmatramo operativni sistem onda je njegova funkcija da obezbedi korisniku "prihvatljiviji" oblik komunikacije sa hardverskim komponentama tj. da se ponaša kao proširenje hardvera (virtuelna mašina). Mesto operativnog sistema je prikazano na slici ispod.

**Slika** Mesto operativnog sistema u računarkom sistemu.

## Operativni sistem kao upravljač resursa

Računarski sistem ima mnoge resurse. Moderan računar se sastoji od procesora, memorije, tajmera, diska, miša, mrežnog interfejsa, štampača i mnogih drugih uređaja. Ako posmatramo OS kao upravljač resursa, onda je njegova funkcija da obezbedi uređeno i kontrolisano dodeljivanje procesora, memorije i ulazno / izlaznih (I/O) uređaja između više programa koji se takmiče za njihovo korišćenje.

**Slika** Resursi kojima upravlja operativni sistem.

Deo operativnog sistema se nalazi u radnoj memoriji. Ovo je jezgro operativnog sistema. Ostatak glavne memorije sadrži korisničke programe i podatke. Dodeljivanje resorsa (glavne memorije) je kontrolisano od strane OS i hardvera za upravljanje memorijom u procesoru.

# Organizacija računarskog sistema

Moderan računarski sistem opšte upotrebe se sastoji od jednog ili više centralnih procesorskih jedinica i većeg broja kontrolora uređaja povezanih preko magistrale koja obezbeđuje i pristup deljenoj memoriji.

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
**Slika** Racionala multiprogramskog sistema

Rani multiprogramski sistemi nisu obezbedili interakciju korisnika sa računarskim sistemom. **Vremenska podela procesora** je logična nadogradnja multiprogramskih sistema koja obezbeđuje interakciju sa korisnicima. Postoji više od jednog korisnika koji komunicira sa sistemom. Promena poseda CPU između korisnika je toliko brza da korisnik stiče utisak da samo on komunicira sa sistemom, ali u stvari on se deli između više korisnika. Svaki korisnik naizvmenično dobija kratak interval procesorskog vremena. Vremenska podela procesora je kompleksnija od multiprogramiranja zato što mora da obezbedi mehanizme za sinhronizaciju i komunikaciju između poslova različitih korisnika.
**Slika** Vremenska podela procesora između više (interaktivnih) korisnika.
