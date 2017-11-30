---
layout: lekcija
title: Emiteri
main_category: Materijali za vežbe
sub_category: Parametarski posao
image: emiteri.png
active: true
comment: true
archive: false
---

# Emiteri i mrežni uređaji

Neka je dat fajl sa 100.000 linija, korisnici.txt, u kome je u svakoj liniji upisana x i y koordinata korisnika koji poseduju mobilni telefon. Pored pomenutog fajla data su i tri fajla sa koordinatama emitera različitih klasa (A, B, C): emiteriA.txt, emiteriB.txt, emiteriC.txt i fajl opsezi.txt u kome se u tri reda nalaze dometi emitera za svaku klasu A, B i C respektivno.

Za date fajlove, potrebno je napisati programsko rešenje koje će se izvršavati na klasteru i upisati u fajl "nemajuDomet.txt" koordinate onih korisnika koji nemaju domet i mora sadržati sledeće fajlove:

* obrada.c - za date fajlove štampa koordinate onih korisnika koji nemaju domet,
* posao.sub - PBS komandni fajl za pokretanje posla na klasteru, treba imati u vidu da se radi o parametarskom poslu,
* pokreni.sh - zadužen je za pokretanje posla i skupljanje rezultata

Programsko rešenje mora podeliti posao na 4 procesora i rešenje upisati u fajl "nemajuDomet.txt".

**Napomena**: Za potrebe testiranja napraviti programski kod u C-u koji će generisai slučajne vrednosti za koordinate u opsegu [-1000, 1000]. Korisnika ima 100 000, emitera klase A ima 10, klase B 20 i klase C 30. U fajl opseg.txt upisati redom vrednosti 130, 100, 70.

## Predlog resenja

obrada.c
{% highlight c %}
#include <stdio.h>
#include <math.h>

typedef struct emiter{
	double x;
	double y;
} Emiter;

int proveriKorisnika(double x, double y, Emiter E, double radius)
{
	double rez=sqrt((E.x-x)*(E.x-x)+(E.y-y)*(E.y-y));
	if(rez<=radius) return 1;
	return 0;
}

void main(int argc, char **argv){
	Emiter A[10];
	Emiter B[20];
	Emiter C[30];
	double radiusA,radiusB,radiusC;
	long i;

	FILE *f=fopen("emiteriA.txt","r");
	for(i=0;i<10;i++)
	{
		double x;double y;
		fscanf(f,"%lf%lf",&x,&y);
		A[i].x=x;
		A[i].y=y;
	}
	fclose(f);

	f=fopen("emiteriB.txt","r");
	for(i=0;i<20;i++)
	{
		double x;double y;
		fscanf(f,"%lf%lf",&x,&y);
		B[i].x=x;
		B[i].y=y;
	}
	fclose(f);

	f=fopen("emiteriC.txt","r");
	for(i=0;i<30;i++)
	{
		double x;double y;
		fscanf(f,"%lf%lf",&x,&y);
		C[i].x=x;
		C[i].y=y;
	}
	fclose(f);
	f=fopen("opseg.txt","r");
	fscanf(f,"%lf",&radiusA);
	fscanf(f,"%lf",&radiusB);
	fscanf(f,"%lf",&radiusC);
	fclose(f);

	f=fopen(argv[1],"r");
	while(1){
		double x;double y;
		if(fscanf(f,"%lf%lf",&x,&y)!=2) break;
		for(i=0;i<30;i++){
			if(i<10)
				if(proveriKorisnika(x,y,A[i],radiusA)==1) break;
			if(i<20)
				if(proveriKorisnika(x,y,B[i],radiusB)==1) break;

			if(proveriKorisnika(x,y,C[i],radiusC)==1) break;

		}
		if(i==30) printf("%lf %lf\n",x,y);
	}
	fclose(f);
}
{% endhighlight %}
posao.sub
{% highlight bash %}
#!/bin/bash
#PBS -N emiteri
#PBS -q batch
#PBS -l nodes=1:ppn=1
#PBS -r n

cd $PBS_O_WORKDIR

./obrada $PBS_ARRAYID.txt > izlaz$PBS_ARRAYID.txt
{% endhighlight %}
pokreni.sh
{% highlight bash %}
#!/bin/bash
split -l 25000 korisnici.txt f
i=0
for fajl in `ls f*`
do
	mv $fajl $i.txt
	i=`echo $i+1|bc -l`
done
gcc -o obrada obrada.c -lm
chmod +x obrada
qsub -t 0-3 posao.sub
while [ `qstat | wc -l` -gt 0 ]
do
sleep 2
done
rm nemajuDomet.txt 2> /dev/null
for fajl in `ls izlaz*`
do
	cat $fajl >> nemajuDomet.txt
	rm $fajl
done
{% endhighlight %}
