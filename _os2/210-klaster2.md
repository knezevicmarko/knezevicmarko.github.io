---
layout: lekcija
title: Emiteri
main_category: Materijali za vežbe
sub_category: Parametarski posao
image: emiteri.png
active: true
comment: true
---

# Emiteri i mrežni uređaji

Neka je dat fajl sa 100.000 linija, korisnici.txt, u kome je u svakoj liniji upisana x i y koordinata korisnika koji poseduju mobilni telefon. Pored pomenutog fajla data su i tri fajla sa koordinatama emitera različitih klasa (A, B, C): emiteriA.txt, emiteriB.txt, emiteriC.txt i fajl opsezi.txt u kome se u tri reda nalaze dometi emitera za svaku klasu A, B i C respektivno.

Za date fajlove, potrebno je napisati programsko rešenje koje će se izvršavati na klasteru i upisati u fajl "nemajuDomet.txt" koordinate onih korisnika koji nemaju domet i mora sadržati sledeće fajlove:

* obrada.c - za date fajlove štampa koordinate onih korisnika koji nemaju domet,
* posao.sub - PBS komandni fajl za pokretanje posla na klasteru, treba imati u vidu da se radi o parametarskom poslu,
* pokreni.sh - zadužen je za pokretanje posla i skupljanje rezultata

Programsko rešenje mora podeliti posao na 4 procesora i rešenje upisati u fajl "nemajuDomet.txt".

**Napomena**: Za potrebe testiranja napraviti programski kod u C-u koji će generisai slučajne vrednosti za koordinate u opsegu [-1000, 1000]. Korisnika ima 100 000, emitera klase A ima 10, klase B 20 i klase C 30. U fajl opseg.txt upisati redom vrednosti 130, 100, 70.

## Predlog rešenja

obrada.c
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

//Emiter je odredjen svojim koordinatama
typedef struct
{
  double x;
  double y;
}Emiter;

//metod koji racuna rastojanje izmedju dve tacke u 2D prostoru
double rastojanje(double x1, double x2, double y1, double y2)
{
  double dx = x2 - x1;
  double dy = y2 - y1;
  return sqrt(dx * dx + dy * dy);
}

int main(int argc, char *argv[])
{
  double x,y;

  //emiteri klase A
  Emiter emiteriA[10];
  //emiteri klase B
  Emiter emiteriB[20];
  //emiteri klase C
  Emiter emiteriC[30];
  //niz za cuvanje opsega emitera (opsezi[0] -> opseg emitera klase A...)
  double opsezi[3];

  //ucitavanje opsega iz fajla
  FILE *O = fopen("opseg.txt","r");
  fscanf(O,"%lf",&opsezi[0]);
  fscanf(O,"%lf",&opsezi[1]);
  fscanf(O,"%lf",&opsezi[2]);

  //ucitavanje emitera klase A
  FILE *A = fopen("emiteriA.txt","r");
  long i;
  for(i=0;i<10;i++)
  {
    fscanf(A,"%lf%lf",&x,&y);
    emiteriA[i].x = x;
    emiteriA[i].y = y;
    //printf("A[%ld] = (%lf,%lf)\n", i, emiteriA[i].x, emiteriA[i].y);
  }

  //ucitavanje emitera klase B
  FILE *B = fopen("emiteriB.txt","r");
  for(i=0;i<20;i++)
  {
    fscanf(B,"%lf%lf",&x,&y);
    emiteriB[i].x = x;
    emiteriB[i].y = y;
  }

  //ucitavanje emitera klase C
  FILE *C = fopen("emiteriC.txt","r");
  for(i=0;i<30;i++)
  {
    fscanf(C,"%lf%lf",&x,&y);
    emiteriC[i].x = x;
    emiteriC[i].y = y;
  }

  FILE *F = fopen(argv[1],"r");
  while(!feof(F))
  {
    /******************************************************************
    ako se uspesno ucita broj -
    ovim izbegavamo ucitavanje prazne linije na kraju fajla

    pored ovog nacina za ucitavanje mozemo poslati u argumentu
    koliko fajl ima linija i onda koordinate ucitati FOR petljom
    ******************************************************************/

    if(fscanf(F,"%lf%lf",&x,&y) == 2)
    {
      int uOpsegu=0;
      long e;

      // proveravamo je li u opsegu nekog emitera vrste A
      for(e = 0; e < 10 && uOpsegu == 0; e++)
      {
        double Xe,Ye;
        Xe = emiteriA[e].x;
        Ye = emiteriA[e].y;
        if(rastojanje(Xe,x,Ye,y) < opsezi[0])
        uOpsegu = 1;
      }

      // proveravamo je li u opsegu nekog emitera vrste B
      for(e = 0; e < 20 && uOpsegu == 0; e++)
      {
        double Xe,Ye;
        Xe = emiteriB[e].x;
        Ye = emiteriB[e].y;
        if(rastojanje(Xe,x,Ye,y) < opsezi[1])
          uOpsegu=1;
      }

      // proveravamo je li u opsegu nekog emitera vrste C
      for(e = 0; e < 30 && uOpsegu == 0; e++)
      {
        double Xe,Ye;
        Xe = emiteriC[e].x;
        Ye = emiteriC[e].y;
        if(rastojanje(Xe,x,Ye,y)<opsezi[2])
          uOpsegu=1;
      }

      if(uOpsegu==0)
      {
        // korisnik nije u opsegu
        printf("%lf %lf\n",x,y);
      }
    }
  }

  fclose(O);
  fclose(A);
  fclose(B);
  fclose(C);
  return 0;
}
{% endhighlight %}
posao.sub
{% highlight bash %}
#!/bin/bash
#PBS -N emiteri
#PBS -q batch
#PBS -r n
#PBS -l nodes=1:ppn=1

cd $PBS_O_WORKDIR

./obrada $PBS_ARRAYID.txt > izlaz$PBS_ARRAYID.txt
{% endhighlight %}

pokreni.sh
{% highlight bash %}
#!/bin/bash

gcc obrada.c -o obrada -lm

i=0
split -l 25000 korisnici.txt fajl
for f in fajl*; do
  mv $f $i.txt
  i=`echo "$i+1" | bc -l`
done

qsub -t 0-3 posao.sub

while [ `qstat|wc -l` -gt 0 ]
do
  sleep 2
done

file="nemajuDomet.txt"

if [ -f $file ]; then
  rm $file
fi

for f in izlaz*
do
  cat $f >> $file
done
{% endhighlight %}

## Domaći zadatak

Neka je dato 5 fajlova (brojevi1.txt, brojevi2.txt, ..., brojevi5.txt) sa decimalnim brojevima (ne zna se unapred koliko ih ima). Potrebno je preko cluster arhitekture odrediti 2 najveća broja. Za potrebe rešenja napraviti skript pokreni.sh koji će pokrenuti parametarski posao na 5 procesora i ispisati rešenje u fajl rešenje.txt.
