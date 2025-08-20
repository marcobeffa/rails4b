---
layout: default
title: Web app Creazione e aggiornamenti
parent: Attività chiave
grand_parent: PosturaCorretta 1
nav_fold: true
has_children: true
nav_order: 4
---

#  Web app Creazione e aggiornamenti componenti

Componenti necessari
### Strumenti necessari

| data inizio -- -- -- data Fine | Cosa | 
| | Landing Page | 
| | Questionario type form | Materiale necessario per la consulenza | 
| | Sistema - Note (per gli articoli)  |
| | Sistema - Project (per l'indice e la mappa) |
| | Corso Igiene Posturale Online |
| | Corso Principi di fisioterapia Online |
| | Articoli Blog e sistema contenuti base |
| | Corsi online  igiene posturale |
| | Corsi online principi di fisioterapia |
| | Area personale |
| | Listino Servizi Prodotti (Tipo carrello) |  Consulenza - 80 € | Programmi Schede Corsi |
| | consulenze tutor posturacorretta 
| | prenotazioni visite trattamenti con professionisti benessere Integrato
| | prenotazione consulenze percorsi posturacorretta | 
| | Iscrizione per l'accesso ai contenuti privati  |
| | Calendario Prenotazioni -  Eventi - Appuntamenti |
| | Pagamenti - Fatturazione - Dati |
| | Recensioni |
| | Altri test |
| | Messaggi |
| | Professionisti Consigliati | Centri | Metodiche


### Struttura:


- Landing Page
- Servizi
    - Le consulenze 

- Contenuti 
    - Corsi online:
    - Corsi per Patologia
    - Corsi Postura e Fisiologia
    - Corsi attività specifica
    - Percorso Generale Postura e fisiologia
        - Recupero e Terapie Manuali
            - Igiene Posturale
            - Principi di fisioterapia
  
  - Blog
  - Questionari
  - Valutazioni
  - Schede Esercizi
  - Schede Trattamenti
  - Programma 

- Mappa professionisti (benessere integrato)
- profilo professionisti
- Prenotazioni servizi con professionisti accreditati

- Area privata
  - Iscrizioni
  - Prenotazioni
  - Pagamenti
  - Recensioni
  - Messaggi





------- 
### Struttura database 
- User 
- Profile
- Contact 

- Project profile:references name parent_id child_id position 

- Note profile:references project:references visibility
- Service name price autonomia gruppo singolo online offline
    - Consulenza

- Iscription profile:references professional:references contact:references service:references project:references 

( in teoria dopo l'iscrizione c'è la prenotazione oppure prenoti e poi metti le iscrizioni tipo lista comanda 
Prenota -> Evento
)
- Event data_inizio:datetime data_fine:datetime 

- Professional username nome cognome profile:references contact:references


-------- 