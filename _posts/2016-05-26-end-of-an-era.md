---
layout:     post
title:      "Iceweasel - The end of an era."
date:       2016-05-26 00:00:00
author:     "surF"
header-img: "img/post-bg-02.jpg"
---

Today, while upgrading my Debian box, I discovered something was wrong (yeah, I try to always check what is going on while updating... you know... you never know :) ). Actually this is an interesting output
```c
Selezionato il pacchetto firefox-esr-l10n-it non precedentemente selezionato.
Preparativi per estrarre .../firefox-esr-l10n-it_45.1.1esr-1_all.deb...
Estrazione di firefox-esr-l10n-it (45.1.1esr-1)...
Preparativi per estrarre .../iceweasel-l10n-it_1%3a45.1.1esr-1_all.deb...
Estrazione di iceweasel-l10n-it (1:45.1.1esr-1) su (1:44.0.2-1)...
Preparativi per estrarre .../iceweasel_45.1.1esr-1_all.deb...
Estrazione di iceweasel (45.1.1esr-1) su (44.0.2-1)...
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel/searchplugins/common": Directory non vuota
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel/searchplugins": Directory non vuota
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel/profile/chrome": Directory non vuota
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel/profile": Directory non vuota
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel/pref": Directory non vuota
dpkg: attenzione: impossibile eliminare la vecchia directory "/etc/iceweasel": Directory non vuota
Selezionato il pacchetto firefox-esr non precedentemente selezionato.
Preparativi per estrarre .../firefox-esr_45.1.1esr-1+b1_amd64.deb...
Rimozione di "deviazione di /usr/bin/firefox in /usr/bin/firefox.real da iceweasel"
Viene aggiunto "deviazione di /usr/bin/firefox in /usr/bin/firefox.real da firefox-esr"
Estrazione di firefox-esr (45.1.1esr-1+b1)...
```
So you may imagine how something came up to my mind... what is going on??

It really looks like I am loosing some convos:
```
https://lwn.net/Articles/676799/
```
you should check it out. 

