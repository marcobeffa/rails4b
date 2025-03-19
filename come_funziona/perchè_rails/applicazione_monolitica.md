---
layout: default
title: Applicazione Monolitica
parent: Ruby on rails
grand_parent: Come funziona
nav_fold: true
has_children: true
nav_order: 1
---

## **[Riassunto articolo the majestic monolith](https://signalvnoise.com/svn3/the-majestic-monolith/)**



Il testo discute due principali modelli architetturali per lo sviluppo software: **Microservices/Service-Oriented Architecture (M/SOA)** e **Majestic Monolith**.

### **Pattern e contesto**
- Esistono pattern focalizzati sul codice e altri che riguardano il modo in cui il codice viene scritto all'interno di un'organizzazione.
- L’architettura a **microservizi (M/SOA)** è adatta per aziende enormi come Amazon o Google, permettendo lo sviluppo parallelo con team separati che lavorano su servizi autonomi.
- Tuttavia, molte aziende più piccole adottano i microservizi senza reali necessità, finendo per complicarsi la vita inutilmente.

### **Il problema della distribuzione prematura**
- La distribuzione del codice su più servizi introduce complessità inutili quando non è strettamente necessaria.
- Gestire più servizi significa affrontare problemi di downtime, sincronizzazione e costi di manutenzione elevati.

### **La soluzione: Il Majestic Monolith**
- Un **monolite ben progettato** integra più funzioni in un unico sistema senza suddivisioni forzate.
- Riduce la complessità, elimina astrazioni inutili e facilita lo sviluppo per team piccoli.
- Basecamp ne è un esempio: una grande applicazione gestita con un piccolo team (12 programmatori e 7 designer), sviluppata per più piattaforme senza suddivisione in microservizi.

### **I vantaggi del Majestic Monolith**
- **Maggiore comprensibilità del codice:** Ogni sviluppatore ha una visione chiara del sistema.
- **Efficienza:** Evita il costo di gestione e comunicazione tra microservizi.
- **Scalabilità interna:** Non tutti i sistemi devono essere distribuiti per scalare.
- **Codebase più pulita e manutenibile.**

### **Filosofia del design**
- Il codice deve essere bello, comprensibile e conciso.
- Un monolite ben scritto permette agli sviluppatori di migliorarsi, mentre un’architettura frammentata può portare a un codice caotico.

---

## **Albero dei Concetti**

```
📌 Pattern Software
├── 🔹 Code-centric patterns (migliorano il codice)
├── 🔹 Organization-centric patterns (modelli organizzativi)
│
📌 Microservices/Service-Oriented Architecture (M/SOA)
├── ✅ Pro:
│   ├── Scalabilità per grandi aziende
│   ├── Team indipendenti, sviluppo parallelo
│   ├── Evoluzione separata dei servizi
│
├── ❌ Contro:
│   ├── Complessità nella gestione dei servizi
│   ├── Sincronizzazione e downtime problematici
│   ├── Necessita di risorse e team numerosi
│
📌 Majestic Monolith
├── 🎯 Principi:
│   ├── Sistema integrato senza microservizi inutili
│   ├── Minimo livello di astrazione
│   ├── Codebase leggibile e manutenibile
│
├── ✅ Pro:
│   ├── Maggiore semplicità per team piccoli
│   ├── Minor costo di gestione
│   ├── Facilità di comprensione e modifica del codice
│   ├── Manutenzione più efficace
│
├── 🚀 Caso di studio: **Basecamp**
│   ├── Team di 12 sviluppatori e 7 designer
│   ├── Applicazione monolitica su 6 piattaforme (Web, iOS, Android, Mac, Windows, Email)
│   ├── 200 controller, 900 metodi, 190 classi
│
📌 Conclusione
├── Non copiare i pattern delle big tech se non sei una big tech
├── Se hai un team piccolo, un monolite ben progettato è la soluzione migliore
├── Scrivi codice chiaro, leggibile e manutenibile
```

---

### **Messaggio finale**
💡 **Se hai un team piccolo e un prodotto che non richiede distribuzione massiva, dimentica i microservizi e rendi il tuo monolite maestoso! 🚀**

