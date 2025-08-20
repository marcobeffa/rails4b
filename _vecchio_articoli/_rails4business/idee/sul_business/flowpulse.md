---
layout: default
title: Flowpulse 
parent: Sul business
grand_parent: Idee
nav_fold: true
has_children: true
nav_order: 2
---

# Flowpulse
 

Parte 1 note e business


1️⃣ FlowPulse Core 🏗️
👉 Gestisce Business, Branch e API comuni

Modelli: Business, Branch
Funzionalità: Creazione di brand, autenticazione, API centralizzate
API:
GET /businesses
POST /branches
📌 Esempio di nome app: flowpulse_core

2️⃣ ContentPulse 📰
👉 Gestisce contenuti, corsi, guide, post

Modelli: Content, Tag, Taggable
Funzionalità: Creazione e gestione dei contenuti, tag, visibilità, accesso
API:
GET /contents?visibility=public
POST /contents
📌 Esempio di nome app: content_pulse

3️⃣ ServicePulse 💼
👉 Gestisce i servizi offerti e i professionisti

Modelli: Service, Professional, ListService
Funzionalità: Creazione di servizi, assegnazione a professionisti, catalogo servizi
API:
GET /services
POST /professionals
GET /list_services?professional_id=1
📌 Esempio di nome app: service_pulse

4️⃣ CenterPulse 🏥
👉 Gestisce i centri fisici dove si erogano servizi

Modelli: Center, CenterService
Funzionalità: Creazione centri, assegnazione servizi ai centri, ricerca centri
API:
GET /centers
POST /centers
GET /center_services?center_id=1
📌 Esempio di nome app: center_pulse

5️⃣ BookingPulse 📅
👉 Gestisce le prenotazioni dei servizi

Modelli: PrenotationCart
Funzionalità: Prenotazione servizi, gestione appuntamenti, pagamenti
API:
POST /prenotation_carts
GET /prenotation_carts?user_id=1
📌 Esempio di nome app: booking_pulse

content_pulse, service_pulse, booking_pulse, center_pulse, business_pulse

link creare una rete linkable1 linable2

1impegno (calendario personale)