---
layout: lekcija
title: Filozofi
main_category: Materijali za vežbe
sub_category: Pthreads
image: philosophy.png
active: true
comment: true
archive: false
---

Pet filozofa sedi za okruglim stolom. Svaki filozof provodi svoj život tako što naizmenično razmišlja i
jede. Filozofi nisu veoma spretni, moraju da koriste dva štapića kada jedu. Nažalost, postoji samo pet štapića, između svaka dva susedna filozofa po jedna. Filozofi mogu da dohvate samo štapić koja je neposredno levo i neposredno desno od njih. Posude sa pirinčem se neprestano pune i filozofi su uvek gladni posle razmišljanja. Napisati C program koristeći pthread koji simulira ponašanje filozofa.

![Filozofi](/assets/os2/philtable.png "Filozofi"){:style="width: auto;"}

{% highlight c %}
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define BROJ_FILOZOFA 5

pthread_mutex_t mutexi[BROJ_FILOZOFA];

void *doWork(void *t)
{
	long i = (long) t;

	while (1)
	{
		printf("Filozof %ld misli.\n", i);
		sleep(2); //mislim

		pthread_mutex_lock(&mutexi[i]);
		pthread_mutex_lock(&mutexi[(i + 1)%BROJ_FILOZOFA]);
		printf("Filozof %ld jede.\n", i);
		sleep(1); //jedem
		pthread_mutex_unlock(&mutexi[i]);
		pthread_mutex_unlock(&mutexi[(i + 1)%BROJ_FILOZOFA]);
	}

	pthread_exit(NULL);
}

int main(int argc, char **argv)
{
	long i;
	pthread_t niti[BROJ_FILOZOFA];
	pthread_attr_t attr;

	pthread_attr_init(&attr);
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_init(&mutexi[i], NULL);

	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_create(&niti[i], &attr, doWork, (void *)i);


	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_join(niti[i], NULL);


	pthread_attr_destroy(&attr);
	for (i = 0; i < BROJ_FILOZOFA; i++)
		pthread_mutex_destroy(&mutexi[i]);

	pthread_exit(NULL);

	return 0;
}
{% endhighlight %}
