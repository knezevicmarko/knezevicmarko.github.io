---
layout: lekcija
title: Lekcija 02
main_category: Materijali za vežbe
sub_category: Linux
image: linux.png
active: false
comment: false
---

Ovo je prva lekcija iz Linux dela.
{% highlight bash %}
marko@marko-P5E3-Premium:~$ sudo fdisk -l /dev/sda

Disk /dev/sda: 465,8 GiB, 500107862016 bytes, 976773168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0c210c20

Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048 235522047 235520000 112,3G  7 HPFS/NTFS/exFAT
/dev/sda2       235524094 976773119 741249026 353,5G  5 Extended
/dev/sda5       307208192 976773119 669564928 319,3G  7 HPFS/NTFS/exFAT
/dev/sda6       235524096 243335167   7811072   3,7G 82 Linux swap / Solaris
/dev/sda7       243337216 307195903  63858688  30,5G 83 Linux

Partition table entries are not in disk order.
{% endhighlight %}

{: .w3-table .w3-bordered .w3-striped .w3-card-4 .w3-margin}
| Tables   |      Are      |  Cool |
|----------|:-------------:|------:|
| col 1 is |  left-aligned | $1600 |
| col 2 is |    centered   |   $12 |
| col 3 is | right-aligned |    $1 |


Najznačajnija hardverska inovacija koja je omogućila multiprogramske sisteme je I/O (Input/Output) procesor (kontroleri, kanali). I/O procesor može da izvrši specijalizovani I/O program bez intervencije CPU-a. CPU mora samo da definiše niz aktivnosti za I/O procesor. Nakon toga I/O procesor izvršava I/O instrukcije i prekida CPU samo kada je čitava sekvenca I/O instrukcija izvršena. Za posledicu ovakvog načina upravljanja I/O uređajima imamo rasterećenje CPU-a pri sporoj interakciji poslova sa I/O uređajima.
