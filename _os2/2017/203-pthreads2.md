---
layout: lekcija
title: Kriptografija
main_category: Materijali za vežbe
sub_category: Pthreads
image: kriptografija.png
active: true
comment: true
archive: false
---

# Dekodiranje fajla

* Predlog rešenja
{% highlight c %}
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#include <pthread.h>


char *buffer;
struct stat s;

void *doWork(void *arg)
{
	long start = (long)arg;
	char i = (char)start;
	for (; i < (start + 26); i++)
	{
		if (((buffer[0] ^ i) == 'O') && ((buffer[1] ^ i) == 'S') && ((buffer[2] ^ i) == '2') && ((buffer[3] ^ i) == '.') &&
			((buffer[s.st_size - 5] ^ i) == 'O') && ((buffer[s.st_size - 4] ^ i) == 'S') &&
			((buffer[s.st_size - 3] ^ i) == '2') && ((buffer[s.st_size - 2] ^ i) == '.'))
		{
			break;
		}
	}
	if (i == (start + 26))
		pthread_exit((void *)-1);
	else
		pthread_exit((void *)i);
}

int main()
{
  pthread_t thread[2];
  pthread_attr_t attr;
  void *status;
  char code = -1;
  long i;
  FILE *out;

  int fd = open("code.txt", O_RDONLY);
  if (fd < 0)
    return EXIT_FAILURE;

  fstat(fd, &s);
  buffer = mmap(0, s.st_size, PROT_READ, MAP_PRIVATE, fd, 0);

  if (buffer == (void *)-1)
  {
    close(fd);
    return EXIT_FAILURE;
  }

  pthread_attr_init(&attr);
  pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

  pthread_create(&thread[0], &attr, doWork, (void *)65);
  pthread_create(&thread[1], &attr, doWork, (void *)97);

  pthread_join(thread[0], &status);
  code = ((char)status)== -1 ? -1 : (char)status;
  pthread_join(thread[1], &status);
  code = ((char)status)== -1 ? -1 : (char)status;

  if (code != -1)
  {
    printf("%c\n", code);
    out = fopen("izlaz.txt", "w");
    for (i = 0; i < s.st_size; i++)
      fputc(buffer[i] ^ code, out);
    fclose(out);
  }

  munmap(buffer, s.st_size);
  close(fd);

  pthread_exit(NULL);
}
{% endhighlight %}
* [Kodirani fajl](/assets/os2/code.txt)

# Domaći zadatak


Ulaz je tekstualna datoteka XOR kodirana sa šifrom od 16 bita. Potrebno je napisati program koji kodira fajl i program koji korišćenjem "grube sile" ("brute force") i pthread dekodira datoteku u originalni tekst. Kontrolni mehanizam kojim prepoznajemo ispravno dešifrovan tekst jesu 2 identične reči na početku i kraju teksta (OS2.). Broj mogućih šifara u ovom slučaju je 2^16.

Šifrovanje sa 16 bita je slično kao i za 8 bita. Komplikuje se postupak u delu realnog izvođenja operacija jer je lakše sa 8 bita koje u ASCII sistemu predstavljaju jedan znak. Ako je šifra sa 16 bita tada njome možemo da kodiramo 2 znaka odjednom. Dakle ako imamo znakove A i B, njihovi ASCII kodovi su 65 i 66, a ako su bajtovi jedan do drugog imamo redom:

0100000101000010 (A je prvih 8 bita)

Ako je šifra 16 bita recimo 0011110011001110 imamo:

0100000101000010

0011110011001110

----------------

0111110110001100

Kako se uzimaju po 2 bajta datoteka treba ili da se dopuni znakom (poželjnije rešenje) ili da se pretpostavi da je broj bajtova teksta paran.
