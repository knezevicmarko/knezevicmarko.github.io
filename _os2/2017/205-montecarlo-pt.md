---
layout: lekcija
title: PI
main_category: Materijali za vežbe
sub_category: Pthreads
image: pi.png
active: true
comment: true
archive: false
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
