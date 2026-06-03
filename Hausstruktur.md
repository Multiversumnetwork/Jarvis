# Hausstruktur für Home Assistant / Jarvis

> Arbeitsdatei zur Definition der Etagen, Bereiche, angrenzenden Bereiche und Aliase.  
> Die Aliase und Nachbarschaften werden später zur Generierung der YAML-Struktur verwendet.

---

## Untergeschoss

### Garage

Typ: Innenbereich / Technik / Fahrzeug  
Angrenzend an:

- Atelier
- Einfahrt
- 

Aliase:

- 
- 
- 

---

### Waschküche

Typ: Innenbereich / Haushalt  
Angrenzend an:

- Atelier
- Vorratskammer
- Rümpelkammer

Aliase:

- 
- 
- 

---

### Vorratskammer

Typ: Innenbereich / Lager  
Angrenzend an:

- Waschküche
- Atelier
- 

Aliase:

- 
- 
- 

---

### Studio

Typ: Innenbereich / Arbeit / Kreativraum  
Angrenzend an:

- Ziergarten
- Waschküche
- Café d'Udange

Aliase:

- 
- 
- 

---

### Rümpelkammer

Typ: Innenbereich / Lager  
Angrenzend an:

- Waschküche
- Atelier
- 

Aliase:

- 
- 
- 

---

### Atelier

Typ: Innenbereich / Kreativraum  
Angrenzend an:

- Garage
- Rümpelkammer
- Waschküche
- Vorratskammer

Aliase:

- 
- 
- 

---

### Café d'Udange

Typ: Außenbereich / Aufenthalt / Garten  
Angrenzend an:

- Studio
- Ziergarten
- Gemüsegarten

Aliase:

- 
- 
- 

---

### Gemüsegarten

Typ: Außenbereich / Garten  
Angrenzend an:

- Waschküche
- Ziergarten
- Einfahrt
- Café d'Udange

Aliase:

- 
- 
- 

---

### Ziergarten

Typ: Außenbereich / Garten  
Angrenzend an:

- Einfahrt
- Gemüsegarten
- Studio
- Café d'Udange

Aliase:

- 
- 
- 

---

### Einfahrt

Typ: Außenbereich / Zugang / Fahrzeug  
Angrenzend an:

- Ziergarten
- Garage
- Gemüsegarten

Aliase:

- 
- 
- 

---

### Blumenwiese

Typ: Außenbereich / Garten  
Angrenzend an:

- 
- 
- 

Aliase:

- 
- 
- 

---

## Erdgeschoss

### Eingang

Typ: Übergang / Zugang  
Angrenzend an:

- Küche
- Esszimmer
- Flur

Aliase:

- 
- 
- 

---

### Flur

Typ: Innenbereich / Übergang  
Angrenzend an:

- Klo
- Küche
- Schlafzimmer
- Treppe

Aliase:

- 
- 
- 

---

### Schlafzimmer

Typ: Innenbereich / Schlafbereich  
Angrenzend an:

- Dusche
- Flur
- 

Aliase:

- 
- 
- 

---

### Dusche

Typ: Innenbereich / Bad  
Angrenzend an:

- Schlafzimmer
- 
- 

Aliase:

- 
- 
- 

---

### Küche

Typ: Innenbereich / Wohnen  
Angrenzend an:

- Lounge
- Esszimmer
- Eingang
- Flur

Aliase:

- 
- 
- 

---

### Lounge

Typ: Innenbereich / Wohnen / Zwischenebene  
Angrenzend an:

- Küche
- Treppe
- 

Aliase:

- 
- 
- 

---

### Esszimmer

Typ: Innenbereich / Wohnen  
Angrenzend an:

- Terrasse
- Küche
- Eingang
- Büro

Aliase:

- 
- 
- 

---

### Büro

Typ: Innenbereich / Arbeit  
Angrenzend an:

- Esszimmer
- 
- 

Aliase:

- 
- 
- 

---

### Klo

Typ: Innenbereich / Bad  
Angrenzend an:

- Flur
- 
- 

Aliase:

- 
- 
- 

---

### Terrasse

Typ: Außenbereich / Aufenthalt  
Angrenzend an:

- Esszimmer
- Spielwiese
- 

Aliase:

- 
- 
- 

---

### Parkplatz

Typ: Außenbereich / Fahrzeug  
Angrenzend an:

- 
- 
- 

Aliase:

- 
- 
- 

---

### Spielwiese

Typ: Außenbereich / Garten  
Angrenzend an:

- Terrasse
- 
- 

Aliase:

- 
- 
- 

---

## Obergeschoss

### Balkon

Typ: Außenbereich / Terrasse  
Angrenzend an:

- Pallier
- 
- 

Aliase:

- 
- 
- 

---

### Treppe

Typ: Übergang / Verbindung  
Angrenzend an:

- Flur
- Lounge
- Pallier

Aliase:

- 
- 
- 

---

### Pallier

Typ: Innenbereich / Übergang  
Angrenzend an:

- Treppe
- Balkon
- Badezimmer
- Schlafzimmer C
- Speicher

Aliase:

- 
- 
- 

---

### Badezimmer

Typ: Innenbereich / Bad  
Angrenzend an:

- Pallier
- Speicher
- Schlafzimmer C

Aliase:

- 
- 
- 

---

### Speicher

Typ: Innenbereich / Lager  
Angrenzend an:

- Pallier
- Badezimmer
- 

Aliase:

- 
- 
- 

---

### Schlafzimmer C

Typ: Innenbereich / Schlafbereich  
Angrenzend an:

- Pallier
- Badezimmer
- 

Aliase:

- 
- 
- 

---