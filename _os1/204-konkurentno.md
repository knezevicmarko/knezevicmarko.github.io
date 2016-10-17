---
layout: lekcija
title: Komunikacija i sinhronizacija
main_category: Materijali za vežbe
sub_category: Konkurentno programiranje
image: semaphore.png
active: true
comment: true
---

# Tipični problemi komunikacije i sinhronizacije procesa

## a) Pristup zajedničkoj varijabli (Shared Variable)

![Komunikacija i sinhronizacija pristupa zajedničkoj varijabli.](/assets/os1/zVarijabla.jpg "Komunikacija i sinhronizacija pristupa zajedničkoj varijabli.")

{: .w3-table .w3-margin .w3-card-4}
|------------------|------------------|
| P :: *var* x : integer; | Q :: *var* y : integer; |
| s11; | s21; |
| v := x; | y := v; |
| s12; | s22; |

* v - zajednička varijabla,
* x - lokalna varijabla procesa P,
* y - lokalna varijabla procesa Q.

## b) Pristup zajedničkom resursu (Shared Resource)

![Komunikacija i sinhronizacija pristupa zajedničkom resursu.](/assets/os1/zResurs.jpg "Komunikacija i sinhronizacija pristupa zajedničkom resursu.")

{: .w3-table .w3-margin .w3-card-4}
|------------------|------------------|
| P :: *var* x : slot; | Q :: *var* y : slot; |
| s11; | s21; |
| write(x); | read(y); |
| s12; | s22; |

## v) Kritični region (Critical Region, Critical Section)

* Generalizacija a) i b)

![Komunikacija i sinhronizacija pristupa kritičnom regionu.](/assets/os1/zRegion.jpg "Komunikacija i singronizacija pristupa kritičnom regionu.")

* cr - jedna naredba ili grupa naredbi (segment koda) zajednička za oba procesa.

{: .w3-table .w3-margin .w3-card-4}
|------------------|------------------|
| P :: | Q ::|
| s11; | s21; |
| cr; | cr; |
| s12; | s22; |

{: .w3-table .w3-margin .w3-card-4}
|------------------|------------------|
| P :: | Q ::|
| s11; | s21; |
| cr1; | cr2; |
| s12; | s22; |

Segment koda cr1 i cr2 sadrže ili zajedničke varijable ili pristup zajedničkom resursu (zajedničkom kodu) ili kombinaciju ova dva.

### Primer

![Primer.](/assets/os1/zpRegion.jpg "Primer.")

![Primer kritičnog regiona.](/assets/os1/krPrimer.jpg "Primer kritičnog regiona.")

**Zaključak:** Ne sme se dozvoliti da dva procesa istovremeno pritupe zajedničkim varijablama / resursima (da uđu istovremeno u kritični region).

# Isključivanje istovremenog pristupa (Mutual Exclusion)

Kako obezbediti isključenje istovremenog pristupa kritičnom regionu ako precesi rade asinhrono i ako ne želimo da vodimo računa o brzini njihovog rada?

## a) Signalizacija (sinhronizacija)

![Signalizacija.](/assets/os1/signalizacija.jpg "Signalizacija.")

{: .w3-table .w3-margin .w3-card-4}
|------------------|------------------|
| P :: | Q ::|
| s11; | s21; |
| Čekaj na signal iz Q; | Signaliziraj P da može da nastavi; |
| s12; | s22; |

## b) Primo - predaja podataka (Producer - Consumer Relation)

![Primo-predaja podataka.](/assets/os1/ppredaja.jpg "Primo-predaja podataka.")

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  put(x);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y: slot;
while true do
begin
  get(y);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

gde je:

* P - proces davač (Producer),
* Q - proces prijemnik (Consumer) i
* b - bafer za razmenu podataka (bafer može da bude jednostruki ili višestruki)
{% highlight pascal %}
type slot = array 1 .. M of integer;
{% endhighlight %}

### Jednostruki bafer (Single buffer)

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>{% highlight pascal %}
var B : slot;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">prostor bafera</td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
procedure put (x : slot)
var i : integer;
begin
  for i := 1 to M do B[i] := x[i];
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Operacija stavljanja podataka u bafer.</td>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
procedure get (var x : slot)
var i : integer;
begin
  for i := 1 to M do x[i] := B[i];
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Operacija uzimanja podataka iz bafera.</td>
    </tr>
  </tbody>
</table>

### Višestruki bafer (Bounded Buffer) - Konačni bafer

Tehnika višestrukog bafera koristi se kod problema pristupa zajedničkoj memoriji i problema obaveštavanja.

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>{% highlight pascal %}
var B : array 0 .. N - 1 of slot;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">prostor bafera</td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
last : 0 .. N - 1;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">ukazatelj kraja bafera</td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
count : 0 .. N;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">brojac bafera</td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
procedure put (x : slot)
var i : integer;
begin
  for i := 1 to M do B[last, i] := x[i];
  last := (last + 1) mod N;
  count := count + 1;
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Operacija stavljanja podataka u bafer.</td>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
procedure get (var x : slot)
var i : integer;
begin
  for i := 1 to M do x[i] := B[(last - count) mod N, i];
  count := count - 1;
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Operacija uzimanja podataka iz bafera.</td>
    </tr>
  </tbody>
</table>

![Višestruki bafer.](/assets/os1/vBafer.jpg "Višestruki bafer.")

Proces P proizvodi podatke za proces Q. P ne sme da pošalje nove podatke u bafer sve dok Q ne uzme prethodne podatke iz bafera.

**P čeka da Q "isprazni" bafer.**
**Q čeka da P "napuni" bafer.**

**Q treba da obavesti P da je ispraznio bafer.**
**P treba da obavesti Q da je napunio bafer.**

Ovo je problem isključenja uzajamnog pristupa kombinovan sa problemom sinhronizacije.

# Arhitektonska sredstva za kontrolu pristupa kritičnom regionu

## a) Zabrana prekida

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>P</td>
      <td></td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
s11;
disable;
cr1;
enable;
s12;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">zabrana prekida<br/>kritični region<br/>ukidanje zabrane prekida</td>
    </tr>
    <tr>
      <td>Q</td>
      <td></td>
    </tr>
    <tr>
      <td>{% highlight pascal %}
s21;
disable;
cr2;
enable;
s22;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Aplikacioni programer mora<br/>da pazi da kritične regione<br/>obavezno "opkoli" parom<br/>mašinskih naredbi DISABLE/ENABLE.</td>
    </tr>
  </tbody>
</table>

1) Metod je neprihvatljiv u slučaju "dugih" kritičnih regiona (krajnje je opasno zabraniti prekide na duže vreme jer onemogućava multiprocesuiranje.)

2) Aplikacioni programer mora da se spušta na nizak jezički nivo (Assembler).

3) Metod nije dovoljan u slučaju višeprocesorskih sistema (zabrana prekida je na nivou jednog procesora.)

## Čekanje u petlji (Busy Wait)

![Čekanje u petlji.](/assets/os1/busyWait.jpg "Čekanje u petlji.")

gde su:

* l - logički indikator:
  - l = true - kritični region zauzet,
  - l = false - kritični region slobodan,
* w1/2 - jalova naredba koja zadržava procesor za neko vreme.

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
while l do w1;
l := true;
cr1;
l := false;
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
while l do w2;
l := true;
cr2;
l := false;
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

### TEST-AND-SET naredba

![TEST-AND-SET naredba.](/assets/os1/testAndSet.jpg "TEST-AND-SET naredba.")

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>
{% highlight pascal %}
function testset(var l : boolean)
: boolean;
begin
  testset := l;
  l := true;
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">Ova funkcija predstavlja jednu mašinsku naredbu (koja ne može da se prekine)</td>
    </tr>
  </tbody>
</table>

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
while testset(l) do w1;
cr1;
l := false;
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
while testset(l) do w2;
cr2;
l := false;
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

U slučaju višeprocesorskog sistema (sa zajedničkom memorijom) potrebno je obezbediti isključivost istovremenog pristupa zajedničkoj memoriji. To se takođe postiže hardverski:

{% highlight pascal %}
function testset(var l : boolean) : boolean;
begin
  zabrana pristupa zajedničkoj memoriji; (MEMORY lock)
  testset := l;
  l := true;
  dozvola pristupa zajedničkoj memoriji; (MEMORY unlock)
end;
{% endhighlight %}

4) Metod čekanja u petlji dovodi do bespotrebnog "traćenja" vremena procesora (procesor bi mogao da radio nešto korisnije dok se čeka na dozvolu pristupa).

5) Varijabla l je vidljiva u aplikacionom programu, to je opasno: opterećuje aplikacionog programera. Takođe važi nedostatak 2.

6) Metod ne zadovaoljava sistem sa prijemnicom (A -> R)(Running -> Ready): Neka P ima niži prioritet od Q i neka Q prekine P u trenutku dok se nalazi u kritičnom regionu, tada će Q beskonačno čekati u petlji, a P nikada neće moći da nastavi sa radom.

Zato treba kombinovati a) i b):
<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
disable;
while testset(l) do w1;
cr1;
l := false;
enable;
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
disable;
while testset(l) do w2;
cr2;
l := false;
enable;
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>
Ipak ostaju nedostaci: 1), 2), 4) i 5).

# Sistemska sredstva za komunikaciju i sinhronizacija procesa

Primitivni koncepti (1970-ih godina):

- a) Regioni (Regions),
- b) Semafori (Semaphores),
- v) Događaji (Events) i
- g) Uslovi (Conditions).

Strukurni koncepti (1980-ih godina):

- d) Monitori (Monitors).

## a) Regioni (Regions)

Baza podataka:

1. jedan logički indikator:
  - busy = true - kritični region zauzet i
  - busy = false - kritični region slobodan
2. jedna procesna lista (B) - lista procesa koji čekaju na ulaz u kritični region.

Primitivne operacije:

1. Enter - zahtev za ulaz u kritični region i
2. Exit - izlaz iz kritičnog regiona.

Početno stanje:

1. busy = false i
2. B prazan.

Semantika primitivnih operacija:
{% highlight pascal %}
ENTER ::
  if busy then
    {A -> B
     R -> A}
  else
    busy := true;
{% endhighlight %}

{% highlight pascal %}
EXIT ::
  busy := false
  if not empty(B) then
    { A -> R
      B -> R
      R -> A}
{% endhighlight %}

Primena:

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
enter(reg);
cr1;
exit(reg);
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
enter(reg);
cr2;
exit(reg);
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

gde je:

* reg - ime (identifikacioni broj) objekta tipa "region" koji je dodeljen kritičnom regionu cr1 i cr2.

Ovim su uklonjeni nedostaci 1) - 6).

Aplikacioni programer je dužan da:

1. locira kritične regione koji imaju pristup zajedničkim podacima/resursima,
2. svim takvim kritičnim regionima dodeli po jedan objekat tipa "region" i
3. sve kritične regione opkoli odgovarajućim parom primitiva Enter/Exit.

## b) Semafori (Semaphores)

(1968. G. Diijkstra "Cooperating Sequential Processes")

Semafori su objekti operativnog sistema sa sledećim karakteristikama:

Baza podataka:

1. semafor varijabla (se{0, 1, 2, ..., N}),
2. jedna procesna lista (B) - lista procesa blokiranih na semaforu).

Primitivne operacije:

1. P (Probeer - pokušaj) - dekrementiranje semafor varijable i
2. V (Verhoog - inkrementiraj) - inkremetiranje semafor varijable.

Početno stanje:
1. S - može da se inicijalizuje na bilo koju vrednost između 0 i N (N može da predstavlja broj jedinica nekog resursa) i
2. B - prazna.

Semantika P i V operacija:

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>V :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
if s > 0 then s := s - 1;
else
  {
    A -> B
    R -> A
  }
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s := s + 1;
if not empty(B) then
  {
    A -> R
    B -> R
    R -> A
  }
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Ukoliko je N = 1, tj. se {0, 1} takav semafor je **binarni** semafor.

Primer kritičnog regiona

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
p(sem);
cr1;
v(sem);
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
p(sem);
cr2;
v(sem);
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

gde je:

* sem - ime (identifikacioni broj) semafora koji je dodeljen kritičnom regionu. Ovde se koristi binarni semafor koji je inicijalizovan sa s = 1.

Primer signalizacije:

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
s11;
p(sem);
s12;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
s21;
v(sem);
s22;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Ako je proces P prvi počeo sa radom izvršiće naredbu s11 i preći će u blokirano stanje. Čekaće sve dok mu proces Q ne pošalje signal preko semafora sem.

Ovde se takođe koristi binarni semafor koji je inicijalizovan sa s = 0.

Primer primo-predaje preko jednostrukog bafera:

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvodi x;
  p(sem1);
  put(x);
  v(sem2);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  p(sem2);
  get(y);
  v(sem1);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

gde su:

* sem1 i sem2 binarni semafori inicijalizovani sa:
  - sem1 -> 1 i
  - sem2 -> 0.

Svakom jednostrukom baferu treba, u operativnom sistemu, pridružiti po 2 binarna semafora za 2 uslova:

- bafer nije pun (sem1) i
- bafer nije prazan (sem2)

Primer primo-predaje preko konačnog bafera

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  p(space);
  p(mutex);
  put(x);
  v(mutex);
  v(count);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  p(count);
  p(mutex);
  get(y);
  v(mutex);
  v(space);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Svakom konačnom baferu treba, u operativnom sistemu, pridružiti 3 semafora:

* 1 binarni semafor (mutex) za kontrolu pristupa baferu (proverava da li je kritični region zauzet) i
* 2 celobrojna semafora za signalizaciju:
  - semafor čija semafor varijabla broji prazna mesta u baferu (space) i
  - semafor čija semafor varijabla broji zauzeta mesta u baferu (count).

Kada vrednost varijable space padne na 0 to znači da je bafer pun i da u njega ne može da se piše. Kada vrednost varijable count padne na 0 to znači da je bafer prazan i da iz njega ne može da se čita.

Inicijalizacija semafora:

* mutex -> 1,
* space -> N i
* count -> 0.

**Za domaći**: Realizovati primo-predaju konačnog bafera samo pomoću binarnih semafora, s tim da se brojač punih mesta u baferu eksplicitno pojavi u aplikacionom programu.

## v) Događaji (Events)

(tehnika koja se koristi za sinhronizaciju i koordinaciju procesa u DEC-OS familiji (VMS OS) operativnih sistema)

Događaji su objekti operativnog sistema kojima se vrši:

1. Kontrola kritičnog regiona
2. Signalizacija

Baza podataka:

1. logički indikator događaja (eflg - Event Flag) - kada je eflg = true događaj je nastupio i proces nastavlja sa radom, u suprotnom se blokira i čeka na događaj i
2. jedna procesna lista (B) - lista procesa koji čekaju na događaj.

Primitivne operacije:

1. WAITEVENT - čekanje na događaj,
2. SETEVENT - signalizacija događaja i
3. CLREVENT - brisanje događaja.

Početno stanje:

1. Logička indikacija događaja eflg = false i
2. B prazan.

Semantika primitivnih operacija:
{% highlight pascal %}
WAITEVENT ::
  if not eflg then
    {
      A -> B
      R -> A
    }
{% endhighlight %}
Ako nije nastupio događaj aktivan proces se stavlja u listu blokiranih procesa, a neki drugi proces koji je do tada bio u listi spremnih procesa se aktivira.

{% highlight pascal %}
SETEVENT ::
  eflg := true;
  if not empty(B) then
    {
      A -> R
      B -> R
      R -> A
    }
{% endhighlight %}

{% highlight pascal %}
CLREVENT ::
  eflg := false;
{% endhighlight %}

Primer primo-predaje preko jednostrukog bafera

Svakom jednostrukom baferu treba pridružiti po dva objekta tipa "događaj":

* empty - bafer prazan i
* full - bafer pun.

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
SETEVENT(empty);
while true do
begin
  proizvedi x;
  WAITEVENT(empty);
  put(x);
  CLREVENT(empty);
  SETEVENT(full);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  WAITEVENT(full);
  get(y);
  CLREVENT(full);
  SETEVENT(empty);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

* SETEVENT(empty) - signalizira se da je bafer prazan
* WAITEVENT(empty) - čeka se na događaj da bafer postane prazan
* CLREVENT(empty) - poništava se čekanje na događaj prazan bafer
* SETEVENT(full) - signalizira se da je bafer pun
* WAITEVENT(full) - čeka se na događaj da je bafer pun
* CLREVENT(full) - poništava se čekanje da bafer postane pun

### Generalizacija događaja (kombinovani događaji)

* Konjukcija događaja: WAITAND(e1, e2, ..., en)(Wait for logical AND) - čeka se na događaj koji nastupa ako se dese svi događaji e1, e2, ..., en.
* Disjunkcija događaja: WAITOR(e1, e2, ..., en))Wait for logical OR) - čeka se da nastupi bilo koji od događaja e1, e2, ..., en.

![Primer jednog predajnika i n prijemnika.](/assets/os1/pre1priN.jpg "Primer jednog predajnika i n prijemnika.")

P može da pošalje u bafer podatke tek pošto svi prijemnici uzmu prethodni podatak iz bafera (B je jednostruki bafer).

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
SETEVENT(empty1, empty2, ..., emptyn);
while true do
begin
  proizvedi x;
  WAITAND(empty1, empty2, ..., emptyn);
  put (x);
  CLREVENT(empty1, empty2, ..., emptyn);
  SETEVENT(full1, full2, ..., fulln);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  WAITAND(fulli);
  get(y);
  CLREVENT(fulli);
  SETEVENT(emptyi);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>
Ako jednostruki bafer ima n prijemnika potrebno je koristiti 2 x n objekta tipa "događaj".

## g) Uslovi (Conditions)
(Brinch-Hansen, Hoare, 1972.-1974.)

Uslovi su objekti operativnog sistema.

Baza podataka:

1. jedna procesna lista (B) - lista procesa koji čekaju na uslov.

Primitivne operacije:

1. WAIT - čekanje na uslov (blokiraj proces).
2. SIGNAL - signalizacija uslova (odblokiraj proces).

Početno stanje:

1. B prazna.

Semantika primitivnih operacija:
{% highlight pascal %}
WAIT ::
{
  A -> B
  R -> A
}
{% endhighlight %}
{% highlight pascal %}
SIGNAL ::
  if not empty(B) then
  {
    A -> R
    B -> R
    R -> A
  }
{% endhighlight %}
Za aplikativne programere ovo nije zgodan metod.

## Monitori (strukturna sredstva)
(Brinch-Hansen, 1972.-73., Hoare, 1974.)

### a) Ugnježdavanje regiona i uslova

Direktnom primenom regiona (Enter/Exit) i uslova (Wait/Signal) na konačni bafer dobija se sledeće rešenje za predajnik i prijemnik:

**Rešenje 1.**
<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  ENTER(mutex);
  if count = N then WAIT(nonfull);
  B[last] := x;
  last := (last + 1) mod M;
  count := count + 1;
  SIGNAL(nonempty);
  EXIT(mutex);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  ENTER(mutex);
  if count = 0 then WAIT(nonempty);
  y := B[(last - count) mod N];
  count := count - 1;
  SIGNAL(nonfull);
  EXIT(mutex);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Deo koda u kome su vidljive varijable count, last i B predstavlja kritični region procesa P i Q. Oni se štite primitivama Enter i Exit.

Početne vrednosti:

* count = 0,
* last = 0 i
* B - nedefinisano.

Konačni bafer traži 3 objekta u operativnom sistemu:

- 1 region (mutex) za kontrolu pristupa u kritičnom regionu,
- 1 uslov (nonfull) - bafer nije pun i
- 1 uslov (nonempty) - bafer nije prazan.

Uslovi nonfull i nonempty su lokalni za kritični region.

**Rešenje 1. nije ispravno!**

Operacije WAIT(nonfull) i WAIT(nonempty) ne oslobađaju kritični region. Zato treba kombinovati operacije WAIT i EXIT.

Ako je proces blokiran na operaciji WAIT protrebno je osloboditi kritični region kako bi drugi procesi mogli da pristupe u kritični region i eventualno ostvare uslov na koji čeka prvi proces.

Ukoliko je SIGNAL zadnja operacija u kritičnom regionu, a neposredno ispred operacije EXIT, moguće je kombinovati SIGNAL i EXIT kao jednu jedinstvenu operaciju.

Ovakvo kombinovanje objekata predstavlja "ugnježdavanje" **regiona i uslova**.

**Rešenje 2.** Semantika modifikovanih primitivnih funkcija:
{% highlight pascal %}
ENTER(r) ::
if busy then
{
  A -> Br
  R -> A
}
else busy := true;
{% endhighlight %}
{% highlight pascal %}
WAIT(r, c) ::
{
  A -> Bc
}
if not empty(Br) then
  {
    Br -> R
  }
else busy := false;
{
  R -> A
}
{% endhighlight %}
{% highlight pascal %}
SIGNAL(r, c) ::
if not empty(Bc) then
  {
    Bc -> R
    A -> R
    R -> A
  }
else if not empty(Br) then
  {
    Br -> R
    A -> R
    R -> A
  }
else busy := false;
{% endhighlight %}
gde su:

* Br - lista blokiranih procesa koji čekaju dozvolu za ulazak u kritični region r i
* B c - lista blokiranih procesa koji čekaju da se ostvari (signalizuje) uslov c.

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  ENTER(mutex);
  if count = N then WAIT(mutex, nonfull);
  B[last] := x;
  last := (last + 1) mod N;
  count := count + 1;
  SIGNAL(mutex, nonempty);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  ENTER(mutex);
  if count = 0 then WAIT(mutex, nonempty);
  y := B[(last - count) mod N];
  count := count - 1;
  SIGNAL(mutex, nonfull);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Nedostaci rešenja 2.:

* zajedničke varijable count, last i B su vidljive u aplikacionom programu,
* aplikacioni programer je nepotrebno opterećen mehanizmom bafer i
* aplikacioni programer je prinuđen da koristi relativno veliki broj sinhronizacionih primitiva (ovde: ENTER, WAIT, SIGNAL).

**Rešenje:** Konačni bafer treba realizovati kao novi objekat operativnog sistema.

Baza podataka:

1. B - prostor za smeštanje podataka,
2. count - broj "porcija" podataka koje se nalaze u baferu i
3. last - ukazatelj kraja bafera.

Primitivne operacije:

1. SEND(b, x) - slanje podataka x u bafer B i
2. RECEIVE(b, x) - uzimanje podataka x iz bafera B.

Početno stanje:

1. count = 0 i
2. last = 0.

Semantika primitivnih operacija:

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>SEND(b, x) :: </th>
      <th>RECEIVE(b, x) :: </th>
    </tr>
    <tr>
      <td>
      Ako je bafer pun pozivni proces biće blokiran i čekaće sve dok se u baferu ne stvori mesto barem za jednu porciju podataka
      </td>
      <td>
      Ako je bafer prazan pozivni proces biće blokiran i čekaće sve dok se u baferu ne pojavi barem jedna porcija podataka.
      </td>
    </tr>
  </tbody>
</table>

Ovakav objekat zove se monitor (monitor tipa "konačni bafer").

![Monitor.](/assets/os1/monitori.jpg "Monitor.")

Monitor je objekat sa hijerarhijskom strukturom. Njemu su podređeni objekti tipa "uslov" i objekti tipa "region". Aplikacioni programer od monitora vidi samo primitive SEND i RECEIVE.

Primo-predaja preko monitora tipa "konačni bafer":

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  SEND(b, x);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  RECEIVE(b, y);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

### b) Opšta definicija monitora

Monitor je pasivni objekt koji objedinjuje u jednu jedinstvenu funkcionalnu celinu tri stvari:

* podatke (varijable monitora),
* operacije nad podacima (procedure) i
* inicijalnu operaciju (inicijalni kod).

Varijable monitora:

* lokalne su za monitor (nisu vidljive za druge objekte (procese i monitore)),
* permanentne su za konkurentni program (punovažne su sve dok egzistira monitor),
* njima mogu direktno da pristupe samo procedure monitora i
* drugi objekti mogu da pristupe varijablama monitora samo preko procedura monitora.

Procedure monitora:

* procesi (i drugi monitori) mogu da pristupe monitoru samo preko njegovih procedura (pristup u monitor = izvršavanje njegove procedure),
* dva procesa ne mogu istovremeno da pristupe u isti monitor (mutual exclusion) i
* ako je neki proces P pristupio u monitor i ako drugi proces Q želi da pristupi u isti monitor tada će proces Q biti automatski blokiran, sve dok proces P ne napusti monitor.

Inicijalni kod:

* sekvencijalni program (jedna ili više naredbi) pomoću koga se definišu početne vrednosti varijabli monitora i
* inicijalni kod izvršava se u trenutku uvođenja (kreiranja) monitora u konkurentni program.

### v) Deklaracija monitora
Operativni sistem može u sebi da ima ugrađen određeni broj standardnih tipova monitora. Ukoliko to nije slučaj, tada monitore mora da definiše sam aplikacioni programer, kao i svaku drugu varijablu ili
proceduru. Način definicije i deklaracije monitora zavisi od jezika na kome se piše aplikacioni program. Ovde će se koristiti notacija HOARE-a, koja je predložena po uzoru na klase O.J.DAHLA (jezik SIMULA).

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>
{% highlight pascal %}monitor M;{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">
      Zaglavlje deklaracije monitora.
      </td>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
begin
var v1:t1; v2:t2; ...; vn:tn;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">
        Deklaracija varijabli monitora.
      </td>
    </tr>
    <tr>
    <td>
{% highlight pascal %}
procedure p1 <formalni parametri>;
lokalne varijable procedure;
begin
  telo procedure;
end;
.
.
.
procedure pn <formalni parametri>;
lokalne varijable procedure;
begin
  telo procedure;
end;
{% endhighlight %}
    </td>
    <td style="vertical-align: middle;">
    Deklaracija procedura (funkcija) monitora.
    </td>
    </tr>
    <tr>
    <td>
{% highlight pascal %}
  begin
    inicijalni kod
  end
end
{% endhighlight %}
    </td>
    <td style="vertical-align: middle;">
    Inicijalni kod monitora.
    </td>
    </tr>
  </tbody>
</table>
gde su:

* M - ime monitora,
* vi - imena varijabli monitora,
* ti - tipovi varijabli monitora i
* pk - imena procedura monitora.

Pozivi monitorove procedure iz procesa ili nekog drugog monitora (M.pk):
<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>P :: </th>
      <th>Q :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
var x : slot;
while true do
begin
  proizvedi x;
  buffer.send(b, x);
end;
{% endhighlight %}
      </td>
      <td>
{% highlight pascal %}
var y : slot;
while true do
begin
  buffer.receive(b, y);
  utrosi y;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Ista notacija se koristi za poziv primitiva WAIT i SIGNAL u okviru monitorovih procedura:

- C.WAIT
- C.SIGNAL

gde je C ime uslova.

Uslovi koji su lokalni za monitor takođe mogu da se tretiraju kao varijable monitora (varijable tipa "condition") - var c : condition;

Predpostavljaće se da jezgro operativnog sistema u sebi sadrži objekte tipa uslov i region (tj. da operativni sistem podržava objekte tipa monitor).

### g) Primeri primene monitora

#### Konačni bafer
{% highlight pascal %}
monitor buffer;
begin
  var B: array [0 .. N-1] of slot;
      last : 0 .. N-1;
      count : 0 .. N;
      nonempty, nonfull : condition;

  procedure SEND (x : slot);
  begin
    if count = N then nonfull.wait;
    B [last] := x;
    last := (last + 1) mod N;
    count := count + 1;
    nonempty.signal;
  end;

  procedure RECEIVE(var x : slot);
  begin
    if count = 0 then nonempty.wait;
    x := B [(last - count) mod N];
    count := count - 1;
    nonfull.signal
  end

  begin
    count := 0;
    last := 0
  end
end
{% endhighlight %}

#### Resursni monitor
{% highlight pascal %}
monitor resource;
begin
  var busy : boolean
      nonbusy : condition;

  procedure request;
  begin
    if busy then nonbusy.wait;
    busy := true
  end;

  procedure release;
  begin
    busy := false;
    nonbusy.signal
  end

  begin
    busy := false;
  end;
end
{% endhighlight %}

#### Dinamička alokacija prostora

![Dinamička alokacija prostora](/assets/os1/alokacija.jpg "Dinamička alokacija prostora")

{% highlight pascal %}
monitor allocator;
begin
  var busy : array [1 .. N] of boolean;
      nonempty : condition;

  procedure request (var =: (0..N);
  begin
    i := first (busy);
    if not i > 0 then nonempty.wait;
    busy [i] := true
    end;

  procedure release (i : (0..N));
  begin
    busy [i] := false;
    nonempty.signal
  end

  for j := 1 to N do busy [j] := false;
end
{% endhighlight %}

#### Primo-predaja velikih količina podataka

Ukoliko se razmenjuju veće količine podataka (datotetke za pseudo U/I, itd.) tada konačni bafer, postaje neracionalan. Treba omogućiti da predajnik i prijemnik istovremeno vrše prenos podataka.

![Primo-predaja velikih količina podataka.](/assets/os1/ppVelike.jpg "Primo-predaja velikih količina podataka.")

Konačni bafer služi za prenos adrese podataka, a podaci se razmenjuju preko blokova iz POOL-a.

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <td>
{% highlight pascal %}
P ::
var b : blockaddress;
while true do
  A.request(b);
  napuni blok b;
  B.send(b);
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">
      Traži i rezerviše blok. I nakon što napuni blok javlja procesu Q da je dobio podatke (tj. šalje mu adresu bloka u kome su podaci.)
      </td>
      <td>
{% highlight pascal %}
Q ::
var c : blockaddress;
while true do
  B.receive(c);
  isprazni blok c;
  A.release(c);
end;
{% endhighlight %}
      </td>
      <td style="vertical-align: middle;">
      Uzima adresu bloka. I nakon što isprazni blok oslobađa prostor (vraća blok u POOL)
      </td>
    </tr>
  </tbody>
</table>
Jedan alokator može da opslužuje veći broj primo-predajnih parova (information stream).

![Alokator opslužuje više primo-predajnih parova.](/assets/os1/ppVise.jpg "Alokator opslužuje više primo-predajnih parova.")

Da li FIFO strategija dodele prostora (objekat tipa uslov monitora A) zadovaoljava u slučaju da se brzine primo-predajnih parova znatno razlikuju?

Ovaj problem zahteva generalizaciju objekta tipa "uslov": uvođenje priorioteta c.wait(p).

#### Sistem za rezervaciju karata

L letova. Svakom letu odgovara slog u datoteci letova. W1 , W2 , ..., Wm su procesi koji ažuriraju slogove letova (WRITERS), a R1 , R2 , ..., RM su procesi koji samo čitaju slogove letova (READERS).

Da bi se izbegla zbrka neophodno je uvesti kontrolu pristupa datoteci letova. Usvajaju se sledeća pravila:

* novi R ne može da započne čitanje ako neki W vrši ažuriranje ili ako neki W čeka na pristup slogu leta i
* svi R koji čekaju kraj ažuriranja sloga leta imaju prvenstvo nad sledećim W koji želi da ažurira isti slog leta.

Rešenje: Svakom slogu leta dodeljuje se po jedan monitor istog tipa sa 4 procedure:

- startw - hoću da ažuriram,
- endw - završio sam ažuriranje,
- startr - hoću da čitam i
- endr - završio sam čitanje.

![Sistem za rezervaciju karata avio letova.](/assets/os1/avioKarte.jpg "Sistem za rezervaciju karata avio letova.")

<table class="w3-table w3-margin w3-card-4">
  <tbody>
    <tr>
      <th>Wi :: </th>
      <th>Rj :: </th>
    </tr>
    <tr>
      <td>
{% highlight pascal %}
while true do
begin
  M[l].startw;
  Azuriraj slog za let l;
  M[l].endw;
end;
{% endhighlight %}
    </td>
    <td>
{% highlight pascal %}
while true do
begin
  M[l].startr;
  Citaj slog za let l;
  M[l].endr;
end;
{% endhighlight %}
      </td>
    </tr>
  </tbody>
</table>

Za ovaj problem potrebno je objekte tipa "uslov" proširiti sa još jednom
primitvnom operacijom - **c.empty**, gde je:

- c - ime uslova,
- empty - funkcija koja vraća vrednost true ako u listi blokiranih procesa koji čekaju na uslov c nema niti jedan proces (lista prazna). Ako čeka barem jedan proces (lista nije prazna) tada empty vraća vrednost false.

{% highlight pascal %}
monitor M;
begin
  var r : integer;          (Broj R koji citaju slog leta)
      busy : boolean;       (= true ako neki W azurira slog leta)
      okr : condition;      (uslov: R-ovi mogu da pristupe.)
      okw : condition;      (uslov: W-ovi mogu da pristupe.)

  procedure startr;
  begin
    if busy or not okw.empty then okr.wait;
    r := r + 1;
    okr.signal              (Ako jedan R pocne mogu i svi ostali)
  end;

  procedure endr;
  begin
    r := r - 1;
    if not r > 0 then okw.signal;
  end;

  procedure startw;
  begin
    if r > 0 or busy then okw.wait;
    busy := true;
  end;

  procedure endw;
  begin
    busy := false;
    if not okr.empty then okr.signal;
    else okw.signal;
  end;

  r := 0;
  busy := false;
end;
{% endhighlight %}
