---
layout: default
title: PosturaCorretta
parent: Web app rails
nav_fold: true
has_children: true
nav_order: 7
---


## PosturaCorretta.org
🙂 users [🔒 ❌]
├── 👩🏻‍🏫 Tutors [🔒 ❌]
│   ├── 📆 appointements [🔒 ❌]
│   └── 📄 assigneds [🔒 ❌]
├── ✅ Done [🔒 ❌]
└── ✨ Reviews [🔒 ❌]

rails g authentication
rails g migration AddStatoToUsers come_ti_senti:integer last_tutor_id:integer
rails g scaffold Tutor name:string user:references admin:boolean 
rails g scaffold Appointment user:references tutor:references title:string description:text autonomia:boolean date_start:datetime date_end:datetime price:decimal parent_id:integer
rails g scaffold Assigned branch_id:integer structure_id:integer record_id:integer task_id:integer engagement_id:integer tipo:string titolo:string prezzo:decimal
rails g scaffold Done user:references assigned:references data:datetime note:text
rails g scaffold Review user:references assigned:references recensione:text stelle:integer



🛒 eCommerce Piattaforma
├── 👤 Users
│   ├── Cliente
│   └── Professionista
│
├── 💳 Orders (Acquisti)
│   ├── Tipo: Consulenza
│   ├── Tipo: Evento
│   └── Stato: pagato / in attesa / completato
│
├── 📅 Appointments (per consulenze o eventi)
│   ├── Collegato all’Order
│   ├── Data e orario
│   └── Stato: previsto / completato / valutato
│
├── 🎓 Corsi
│   ├── Accesso: bloccato / sbloccato
│   ├── Collegati a: esito consulenza o partecipazione evento
│   └── Contenuti: lezioni, esercizi, diario
│
└── ⚙️ Logica di sblocco
    ├── Se consulenza completata → sblocca corso
    ├── Se evento frequentato → sblocca corso
    └── Opzione: corso incluso nel prezzo o attivabile successivamente