# Aufgabenstellung

Projektaufgabe:
**Krankenhaus Abteilungsverwaltung**
In einem Krankenhaus soll eine Datenbank zur Abteilungsverwaltung eingerichtet werden. Wichtige
Informationen über Patienten, Ärtze und Schwestern sollen damit abfragbar sein.
Die Datenbank soll in der Lage sein, folgende Informationen zu liefern:
- Welche Ärzte behandeln welche Patienten?
- Welches Zimmer kann einem neuen Patienten zugeordnet werden?
- Wie viele freie Zimmer hat die Chirugie heute?
- Welche Ärzte arbeiten in der HNO-Abteilung?
- Für welches Zimmer ist Oberschwester "Hilde" zuständig?


Hinweise: Ein Zimmer ist genau zu einer Abteilung zugeordnet. Jeder Arzt und jedes Mitglied des
Pflegepersonals ist genau einer Abteilung zugeordnet.
Entitäten aus dem Text: Patient, Arzt, Pflegepersonal, Zimmer, Abteilung


- Erstellen Sie ein entsprechendes ER-Diagramm mit Attributen.  
- Überführen Sie das ER-Diagramm in das relationale Modell.  
- Danach erstellen Sie die Datenbank mit SQL-Anweisungen.  
Speichern Sie die Anweisungen in einer Textdatei.  
- Formulieren Sie Abfragen und speichern diese als VIEW ab, die die oben genannten Fragen beantworten.  
- Erstellen Sie ein Dokument (eine Seite) indem Sie Ihre Vorgehensweise beschreiben.  
- In einer Präsentation (maximal 15 Minuten) mit 8-10 Folien beschreiben
Sie Ihr Datenbankmodell, begründen Ihre Vorgehensweise,  
beschreiben Ihre Umsetzung in SQL und geben ein Fazit,  
ob die oben genannten Fragen umgesetzt sind.  

# Dokumentation

1) Konzeption und Erstellen des ER-Diagrammes
2) Erstellung des Relationenmodells
3) Erstellung der SQL Befehle

## 1. Konzeption und Erstellen des ER-Diagrammes
Erstellen eines ersten ER-Entwurfes anhand der Anforderungen.

![ER Erstentwurf](img/ER0.png)

Überprüfen ob alle Anforderung von diesem Entwurf abgedeckt werden.   
Anpassung des Modells an die Anforderungen nötig:  

![ER Finale Version](img/ER1.png)

## 2. Erstellung des Relationenmodells
Auflösen der n-m Beziehungen in gesonderten Tabellen mit passenden attributen.  
- behandlung_arzt zwischen Tabellen aufenthalt und arzt
- behandlung_pflegekraft zwischen Tabellen aufenthalt und pflegekraft

Festlegen der Primär und Fremdschlüssel und ihrer Zugehörigkeit.  
erstellen des Relationenmodells:

![Relationen Modell](img/RM0.png)

Überprüfen ob alle Anforderung von diesem Entwurf abgedeckt werden.

## 3. Erstellung der SQL Befehle

```

--- Create DATABASE ---
DROP DATABASE IF EXISTS kh_abteilungsverwaltung;
CREATE DATABASE kh_abteilungsverwaltung;
USE kh_abteilungsverwaltung;

--- Create Tables ---
CREATE TABLE 
	abteilung(id INT AUTO_INCREMENT PRIMARY KEY, 
		bezeichnung VARCHAR(60)
);

CREATE TABLE 
	pflegekraft(id INT AUTO_INCREMENT PRIMARY KEY, 
		vorname VARCHAR(60), nachname VARCHAR(60),
        abteilung_id INT, 
        FOREIGN KEY (abteilung_id) REFERENCES abteilung(id)
);

CREATE TABLE 
	arzt(id INT AUTO_INCREMENT PRIMARY KEY, 
		vorname VARCHAR(60), nachname VARCHAR(60),
        abteilung_id INT, 
        FOREIGN KEY (abteilung_id) REFERENCES abteilung(id)
);

CREATE TABLE 
	zimmer(id INT AUTO_INCREMENT PRIMARY KEY, 
		stockwerk INT,
        abteilung_id INT, 
        FOREIGN KEY (abteilung_id) REFERENCES abteilung(id)
);

CREATE TABLE 
	patient(id INT AUTO_INCREMENT PRIMARY KEY, 
		vorname VARCHAR(60), nachname VARCHAR(60), krankenkasse VARCHAR(30)
);

CREATE TABLE 
	aufenthalt(id INT AUTO_INCREMENT PRIMARY KEY, 
		datum_aufnahme DATE,
        datum_entlassung DATE,
        zimmer_id INT,
        pazient_id INT,
        FOREIGN KEY (zimmer_id) REFERENCES zimmer(id),
        FOREIGN KEY (patient_id) REFERENCES patient(id)
);
CREATE TABLE 
	behandlung_pflegekraft(id INT AUTO_INCREMENT PRIMARY KEY, 
		pflegekraft_id INT,
        aufenthalt_id INT, 
        FOREIGN KEY (pflegekraft_id) REFERENCES pflegekraft(id),
        FOREIGN KEY (aufenthalt_id) REFERENCES aufenthalt(id)
);

CREATE TABLE 
	behandlung_arzt(id INT AUTO_INCREMENT PRIMARY KEY, 
		arzt_id INT,
        aufenthalt_id INT, 
        FOREIGN KEY (arzt_id) REFERENCES arzt(id),
        FOREIGN KEY (aufenthalt_id) REFERENCES aufenthalt(id)
);


```


### Erstellen der geforderten Abfragen

```
--- Welche Ärzte behandeln welche Patienten ---
SELECT arzt.id, arzt.nachname, arzt.vorname, 
        patient.id, patient.nachname, patient.vorname
    from arzt 
    INNER JOIN behandlung_arzt ON arzt.id=arzt_id
    INNER JOIN aufenthalt ON aufenthalt.id=aufenthalt_id
    INNER JOIN patient ON patient.id=patient_id
    WHERE datum_entlassung IS NOT null                      # filter out already released patients and get only the ones currently in treatment
    GROUP BY arzt.id, patient.id                            # filter out double entries where a doc did multiple procedures on the same patient
;

--- Welches Zimmer kann einem neuen Patienten zugeordnet werden? ---
SELECT *                                                    #Liste aller freien Zimmer gesucht
from zimmer 
LEFT JOIN (
    SELECT zimmer.id AS id                                  # alle zimmer die momentan belegt sind
    FROM zimmer
    INNER JOIN aufenthalt ON zimmer.id=zimmer_id
    WHERE datum_entlassung IS NULL
) AS t2 On zimmer.id = t2.id
WHERE t2.id is null                                         # alle zimmer die momentan frei sind
;


--- Wie viele freie Zimmer hat die Chirugie heute? ---
SELECT COUNT(*)                                             # modify query from above
from zimmer 
LEFT JOIN (
    SELECT zimmer.id AS id
    FROM zimmer
    INNER JOIN aufenthalt ON zimmer.id=zimmer_id
    WHERE datum_entlassung IS NULL
) AS t2 On zimmer.id = t2.id
INNER JOIN abteilung ON abteilung.id=abteilung_id
WHERE t2.id is null AND abteilung.bezeichnung
;

--- Welche Ärzte arbeiten in der HNO-Abteilung ---
SELECT *
FROM arzt
INNER JOIN abteilung ON abteilung.id= abteilung_id
WHERE abteilung.bezeichnung="HNO"
;

--- Für welche Zimmer ist Oberschwerster "Hilde" zuständig---
select zimmer.id, zimmer.stockwerk, pflegekraft.nachname
FROM pflegekraft
INNER JOIN abteilung ON pflegekraft.abteilung_id=abteilung.id
INNER JOIN zimmer    ON zimmer.abteilung_id     =abteilung.id
;

```

### einlesen von Datensätzen zum testen der Datenbank

```



```