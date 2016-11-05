---
layout: lekcija
title: PI
main_category: Materijali za vežbe
sub_category: Pthreads
image: pi.png
active: true
comment: true
---

Aproksimacija vrednosti broja $$ \pi $$ primenom Monte Carlo metode.

![By CaitlinJo [CC BY 3.0 (http://creativecommons.org/licenses/by/3.0)], via Wikimedia Commons](/assets/os2/montecarlo.gif "By CaitlinJo [CC BY 3.0 (http://creativecommons.org/licenses/by/3.0)], via Wikimedia Commons"){:style="width: auto;"}

{% highlight c %}
#include<stdio.h>
#include<stdlib.h>
#include<time.h>
#include<math.h>

int main()
{
    double pi = 0.0, x, y;
    clock_t cStart = clock();

    srand(time(NULL));
    double u;
    unsigned long n = 200000000, i, unutar = 0;

    for(i = 0; i < n; i++)
    {
        x = rand() * 1.0 / RAND_MAX;
        y = rand() * 1.0 / RAND_MAX;
        if ((x*x + y*y) <= 1.0)
            unutar += 1;
    }

    pi = 4.0 * (unutar * 1.0 / n);

    clock_t cEnd = clock();

    printf("PI: %.10lf za clock: %lf\n", pi, (cEnd - cStart) * 1.0 / CLOCKS_PER_SEC);
    return 0;
}
{% endhighlight %}

Sporija varijanta generatora slučajnih brojeva.

{% highlight c %}
#define NEW_RAND f1 *=171; f1%=30269; f2*=172; f2%=30307; f3*=170; f3%= 30323; u=((((f1 * 1.0))/30269.0)  + ((f2*1.0)/30307.0)  + ((f3 * 1.0)/30323.0)); u = u - floor(u);

unsigned long f1=100 + rand(),f2=20  + rand(), f3=2500 + rand();
double u;

for(.....){
   NEW_RAND
   x = u;
}
{% endhighlight %}

## Domaći zadatak 1.

Dati serijski algoritam, korišćenjem pthread biblioteke, prilagoditi za višenitno izvšenje (N niti).

## Domaći zadatak 2.

**Varijanta A**

Potrebno je napisati C program koji korišćenjem pthread niti simulira procesiranje prometa filijala. Program startuje 2 niti. Svaka od niti učitava promet iz pridruženih datoteka f0.txt i f1.txt. Datoteke su tekstualne i svaka linija je formata

id iznos

gde je id prirodan broj >= 0 i označava identitet klijenta (0, 12, 567,6,...), a iznos je ceo broj i označava promet za datog klijenta u datoj filijali. Na primer, ako je iznos 345, klijentu se povećava stanje računa za navedeni iznos, a ako je -1207 klijentu se umanjuje stanje računa za navedeni iznos. Početno stanje svakog klijenta je 0. Nije unapred poznato koliko je klijenata i koji su. Potrebno ih je dinamički registrovati kad se pojave i pratiti promene na njihovim računima.

**Varijanta B** - dodatni uslov bonusa

Prilikom promena na računima pažnju obratiti na bonuse koje klijenti dobijaju. Naime, ako je posle neke promene (uplata iz filijale, iznos > 0) novo stanje sume svih računa prešlo iz "jedne hiljade" u "neku drugu", svim klijentima jednokratno uplatiti 10 dinara bonusa. Dakle ako je:

- tekuće stanje 3450 i uplatom imamo 4001 dati svima bonus 10 dinara
- tekuće stanje 3450 i uplatom imamo 19871 dati svim bonus 10 dinara
- tekuće stanje 3450 i uplatom imamo 4000, tada nema bonusa
- tekuće stanje 3450 i uplatom imamo 3650, tada nema bonusa

Ne reagovati novim bonusima u slučaju da dodati bonusi proizvode prelazak između hiljada iznosa. Bonusi dolaze samo od direktnih uplata iz filijala. Ako imamo 100 klijenata i poslednjom uplatom jednom klijentu 300 sa sume svih stanja 14860 prelazimo na 15160, tada se svima na račune uplaćuje još po 10 dinara (ukupno 1000 dinara), pa je nova suma stanja 16160. Na nju ne treba reagovati bonusom jer nije direktna posledica standardne uplate već
bonusa.

U obe varijante, na kraju obrade prometa filijala program ispisuje tekuće stanje svih klijenata posle svih promena.
