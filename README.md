# MessedUP - HackMyVM (Medium)

![MessedUP.png](MessedUP.png)

## Übersicht

*   **VM:** MessedUP
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=MessedUP)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 30. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/MessedUP_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, die User- und Root-Flags der Maschine "MessedUP" zu erlangen. Die Maschine wies eine stark fehlkonfigurierte Netzwerkumgebung auf, bei der Dienste auf Nicht-Standard-Ports liefen (z.B. HTTP auf Port 22, SSH auf Port 80, FTP auf Port 8080). Der initiale Angriffsvektor war ein anonymer, beschreibbarer FTP-Zugang, über den eine Hinweismeldung (`idiot.txt`) gefunden wurde. Diese Nachricht enthielt Hinweise auf einen TFTP-Server und weitere herunterladbare Dateien. Obwohl der genaue Weg zum Shell-Zugriff im Bericht nicht detailliert ist, führte die Analyse der heruntergeladenen Dateien (ZIP/MP4) und der Hinweise vermutlich zur Kompromittierung des Systems. Die Flags wurden anschließend über das Netzwerk exfiltriert.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `wget`
*   `hydra` (versucht)
*   `curl`
*   `unzip`
*   `nc` (netcat)
*   Standard Linux-Befehle (`cat`, `ls`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "MessedUP" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Service Enumeration:**
    *   IP-Adresse des Ziels (192.168.2.116) mit `arp-scan` identifiziert.
    *   `nmap`-Scan offenbarte eine stark fehlkonfigurierte Port-/Dienstzuordnung:
        *   Port 22/tcp lief ein SimpleHTTPServer (Python).
        *   Port 80/tcp lief OpenSSH 7.9p1.
        *   Port 111/tcp (rpcbind) und 2049/tcp (NFS) waren offen.
        *   Port 8080/tcp lief vsftpd (FTP) mit erlaubtem anonymen, beschreibbaren Zugriff.
        *   Weitere RPC-bezogene Ports waren offen.
    *   `gobuster` auf Port 80 (SSH) schlug erwartungsgemäß fehl.

2.  **Information Disclosure & Initial Foothold Vector:**
    *   Anonymer FTP-Login auf Port 8080 war erfolgreich. Die Datei `idiot.txt` wurde heruntergeladen.
    *   `idiot.txt` enthielt eine Nachricht, die auf den Benutzer `skinny`, einen TFTP-Server und eine Datei `125.zip` hinwies.
    *   Ein Brute-Force-Versuch auf FTP für `skinny` mit `hydra` scheiterte.
    *   Zugriff auf `http://192.168.2.116:22/` (SimpleHTTPServer) zeigte ein Directory Listing mit den Dateien `hole.jpg`, `Just hanging out with a bear - WTF.mp4` und `ST returns.zip`.
    *   Die Datei `ST returns.zip` wurde heruntergeladen und enthielt `ST returns.mp4`.
    *   *Der genaue Weg zum Shell-Zugriff ist im Log nicht dokumentiert, aber es wird angenommen, dass die Analyse der heruntergeladenen Dateien (MP4, potenziell `125.zip` von TFTP) oder NFS-Exploitation zu einem Shell-Zugriff führte.*

3.  **Data Exfiltration (Flags):**
    *   Nach Erlangung eines Shell-Zugriffs (Benutzerkontext und Methode nicht spezifiziert im Log) wurden die Inhalte der User- und Root-Flag-Dateien mittels `cat` und Pipe zu `nc` an einen Listener auf dem Angreifer-System gesendet.
    *   Ein Netcat-Listener auf dem Angreifer (`nc -lp 4444 > flags.txt`) empfing die Flags.
    *   User-Flag (`/home/skinny/user.txt`): `E9619156E7290E79C226FBF1451F400A5EA35107`.
    *   Root-Flag (`/root/root.txt`): `54CE6117EBEBCB4F10781C11FD2896ED191F2182`.

## Wichtige Schwachstellen und Konzepte

*   **Stark fehlerhafte Port-/Dienstzuordnung:** Dienste liefen auf Nicht-Standard-Ports (HTTP auf 22, SSH auf 80, FTP auf 8080), was die initiale Enumeration erschwerte, aber auch auf ein unsicheres System hindeutete.
*   **Anonymer, beschreibbarer FTP-Zugang:** Ermöglichte das Herunterladen von Hinweisen und potenziell das Hochladen von Payloads.
*   **Information Disclosure:** Die Datei `idiot.txt` auf dem FTP-Server enthielt Hinweise auf Benutzer, andere Dienste (TFTP) und Dateien.
*   **Directory Listing auf HTTP-Server:** Der SimpleHTTPServer auf Port 22 erlaubte das Auflisten und Herunterladen von Dateien.
*   *(Nicht dokumentiert, aber angenommen)* **Ausnutzung von TFTP/NFS oder Schwachstellen in heruntergeladenen Dateien:** Der genaue Vektor zum Shell-Zugriff wurde nicht gezeigt, aber die Hinweise deuteten auf diese Möglichkeiten hin.

## Flags

*   **User Flag (`/home/skinny/user.txt`):** `E9619156E7290E79C226FBF1451F400A5EA35107`
*   **Root Flag (`/root/root.txt`):** `54CE6117EBEBCB4F10781C11FD2896ED191F2182`

## Tags

`HackMyVM`, `MessedUP`, `Medium`, `Misconfiguration`, `Non-Standard Ports`, `Anonymous FTP`, `Information Disclosure`, `TFTP`, `NFS`, `Linux`, `Privilege Escalation` (impliziert)
