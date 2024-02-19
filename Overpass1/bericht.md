## Penetrationstest-Bericht: Overpass 1 CTF auf tryhackme.com

### Einleitung:
Dieser Bericht dokumentiert den Penetrationstest der Overpass 1 CTF-Maschine auf tryhackme.com. Das Hauptziel war es, Sicherheitslücken in der Zielsystemumgebung zu identifizieren und auszunutzen, um Zugriff zu erlangen und die Flags zu extrahieren.

### Zusammenfassung der Ergebnisse:
- Erfolgreicher Zugriff auf das System über eine Sicherheitslücke in der Webanwendung.
- Kompromittierung von Benutzer- und Root-Zugriff durch Ausnutzung von Schwachstellen in der Konfiguration und den geplanten Aufgaben des Systems.
- Extraktion der Benutzerflagge und der Root-Flagge.

### Beschreibung des Angriffsvektors:
Der Angriff begann mit einer Nmap-Scans, der offene Ports und Dienste auf der Zielsystem-IP identifizierte. Der Port 80 war besonders interessant, da er eine Webanwendung hostete. Mit Gobuster wurde nach versteckten Verzeichnissen gesucht und eine potenziell anfällige Admin-Login-Seite entdeckt.

Nach Analyse des JavaScript-Codes auf der Login-Seite wurde eine Sicherheitslücke identifiziert, die es ermöglichte, die Anmeldung durch Manipulation von Cookies zu umgehen. Dadurch erhielt der Angreifer Zugriff auf die Admin-Seite, auf der sensible Informationen und ein verschlüsselter SSH-Private-Key für den Benutzer "James" gefunden wurden.

Der SSH-Private-Key wurde entschlüsselt und für den Zugriff auf den SSH-Server verwendet, wodurch der Benutzerzugriff erlangt wurde. Durch Überprüfung der geplanten Aufgaben in der Crontab-Datei wurde eine Möglichkeit gefunden, eine rückständige Webanwendung zu kompromittieren und eine Reverse-Shell zu erhalten. Die Webanwendung lud regelmäßig ein Skript herunter und führte es aus, was dem Angreifer die Möglichkeit gab, Code auf dem Server auszuführen.

### Weitere Analysen und Ergebnisse:
Während des Tests wurden potenzielle Schwachstellen wie unsichere Passwörter, fehlende Authentifizierungsmaßnahmen und unzureichende Absicherung von geplanten Aufgaben identifiziert. Die Kombination dieser Schwachstellen ermöglichte es dem Angreifer, umfassenden Zugriff auf das System zu erlangen und sensible Daten zu kompromittieren.

### Erklärung der Tools und Techniken:
- Nmap wurde verwendet, um offene Ports und Dienste auf der Ziel-IP zu identifizieren.
- Gobuster wurde eingesetzt, um nach versteckten Verzeichnissen auf der Webanwendung zu suchen.
- John the Ripper wurde verwendet, um den verschlüsselten SSH-Private-Key zu knacken.
- Eine Reverse-Shell wurde durch Manipulation der Crontab-Aufgabe und Bereitstellung eines bösartigen Skripts erreicht.

### Risikobewertung:
Die identifizierten Schwachstellen stellen ein erhebliches Risiko für die Sicherheit der Zielsystemumgebung dar. Ein Angreifer könnte sensible Daten stehlen, Systeme kompromittieren und unbefugten Zugriff erlangen, was zu erheblichen Betriebsunterbrechungen und Reputationsverlusten führen könnte.

### Empfehlungen:
- Implementierung von starken Authentifizierungsmethoden und regelmäßiger Überprüfung von Passwörtern.
- Aktualisierung und Absicherung von Webanwendungen und geplanten Aufgaben, um das Risiko von Code-Injektion und unerwünschter Ausführung zu minimieren.
- Regelmäßige Sicherheitsüberprüfungen und Schulungen für Mitarbeiter zur Sensibilisierung für Sicherheitsrisiken und bewährte Sicherheitspraktiken.

### Abschluss:
Der Penetrationstest auf der Overpass 1 CTF-Maschine hat gezeigt, wie Sicherheitslücken in Webanwendungen und geplanten Aufgaben ausgenutzt werden können, um Zugriff auf Systeme zu erhalten. Durch die Identifizierung und Behebung dieser Schwachstellen kann das Sicherheitsniveau verbessert und das Risiko von Sicherheitsverletzungen minimiert werden.
