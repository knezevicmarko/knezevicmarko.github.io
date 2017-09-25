---
layout: lekcija
title: PowerShell
main_category: Materijali za vežbe
sub_category: PowerShell
image: powershell.png
active: true
comment: true
archive: true
---

# Materijali

* [Prezentacija.](/assets/os1/powershell/PowerShell.pdf)
* [Mastering PowerShell](http://eddiejackson.net/web_documents/Mastering-PowerShell.pdf)
* [The Administrator Crash Course - Windows PowerShell v2](http://eddiejackson.net/web_documents/The_Administrator_Crash_Course_Windows_PowerShell%20v2.pdf)

# Primer (II kolokvijum 2015/16 1. zadatak)

Napisati powershell skriptu koja prima promenljiv broj parametara (minimalno 3). Prva dva parametra predstavljaju putanje do direktorijuma, dok ostali parametri definišu listu ekstenzija. Skripta treba da iskopira sve datoteke koje imaju neku od zadatih ektenzija iz foldera zadatog kao prvi parametar u folder koji je zadat kao drugi parametar na sledeći način:

Za svaku zadatu ekstenziju za koju postoji neka datoteka kreira se novi folder koji nosi naziv ekstenzije, a potom se u njega kopiraju sve datoteke sa tom ekstenzijom.Skripta treba da proveri postojanje zadatih direktorijuma, minimalni broj parametara, kao i da direktorijum zadat kao prvi parametar ne bude isti kao direktorijum zadat kao drugi parametar.

## Predlog rešenja

{% highlight posh %}
if ($args.length -lt 3)
{
	echo "Broj parametara mora da bude minimalno 3."
	exit 1
}

if ( ! (Test-Path $args[0] -pathType container))
{
	echo "Prvi argument mora da bude direktorijum."
	exit 1
}

if ( ! (Test-Path $args[1] -pathType container))
{
	echo "Drugi argument mora da bude direktorijum."
	exit 1
}

if ( $args[0] -eq $args[1] )
{
	echo "Folderi moraju biti razliciti."
	exit 1
}

foreach ($ext in $args[2..$args.length])
{
	$path = $args[0] + "\*." + $ext
	$files = dir $path
	if ($files.length -gt 0)
	{
		$newDir = $args[1] + "\" + $ext
		if (Test-Path $newDir)
		{
			Remove-Item $newDir -Force
		}
		New-Item -ItemType directory $newDir
		Copy-Item $path $newDir
	}
}
{% endhighlight %}
