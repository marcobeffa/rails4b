---
layout: default
title: Flowpulse aggiornamenti
# parent: Flowpulse 
parent: Sul business
grand_parent: Idee
nav_fold: true
has_children: true
has_toc: true
nav_order: 9999
back_to_top: true
back_to_top_text: "Back to top"
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>


# Progetto Produzione Calendario 
### Business Manager 1impegno
Creare un mvp per ampliare flowpulse, ho già, note.flowpulse.net, branch.flowpulse.note
 


#  flowpulse_business 

business.flowpulse.net

## sezione 1  

Mappa del business:

| Business |
| Project (nome: '' modelli_progetto: '') |
| ruoli linee di produzione {nome:"", descizione: , professione: "" } |
| TasTask {nome: "", descrizione: "", perchè: "" come: "" cosa: "", tempo_stimato: "" costo_stimato: ""} | 
| materiale necessario |

### tipo progetto
Si può implementare sull'app flowpulse_branch e viene chiamata dall'api, qui di seguito alcuni dei modelli progetto possibile

Modelli progetto:

| Eventi | |
| Contenuti | |
|  | Blog  | 
|  | Youtube  | 
|  | altro social | 
| Piattaforma Corsi online autonomia | |
| Accademia con livelli e insegnanti | |
| Corsi professionista in gruppo | |
| Consulenza professionista individuale | |
| Community Forum | |
| Costruzione percorsi per Soluzione problemi | |
| | Test Questionari |
| | Valutazioni | 
| | Servizi professionisti | 
| | Programmi | 
| | Schede | 
| | recenzioni | 

## Sezione 2 per implementare il ciclo di produzione
manager.flowpulse.net

| Sceegli Business Progetto Apri Ciclo di produzione |
| Assegna ruoli - attiva linea di produzione |
| Assegna task {responsabile: "", costo_reale: "", concluso_data: ""  } |
| request_event {user_id: "", task_id: "" costo_reale: "",  concludere_entro_data: "", accettato_rifiutato: "", compenso: "" } |
| chiudi la linea |
| Chiudi il ciclo |


# app 1impegno 

L'app 1impegno è per la gestione calendario. 

Parla con flowpulse_business è la registrazione degli eventi fatti da chi partecipa al progetto e il calendario attività utente (1impegno.it)

1impegno.it

| calendario per gli utenti |
| crea nome attività  [salute, produttività, formazione, altro] |
| ruoli attivi |
| event { tipo: [:prenotazioni, :evento], request_event: "", data_inizio: "", data_fine,"" luogo: "", perchè: "" come: "" cosa: "",  }|


## Da valutare in futuro
risorse.flowpulse.net

Da valutare la gestione delle altre risorse: 

- soldi transazioni
- contatti 
- luoghi  
- energia 


-------

# Flowpulse Istruzioni per Sviluppo e Integrazione MVP
## Struttura generale del sistema
### App principali
- `flowpulse_branch`: struttura dei progetti e modelli
- `manager.flowpulse.net`: gestione del ciclo produttivo
- `1impegno.it`: calendario e attivit utenti
---
## Step consigliati da Senior Developer
### 1. Definire JSON API coerenti tra le app
**Esempio: API progetto**
GET `/api/v1/projects/:id.json` in `flowpulse_branch`:
```json
{
"id": 12,
"name": "Corso Igiene Posturale",
"tipo": "corso_online",
"tasks": [...],
"ruoli": [...],
"materiali": [...]
}
```
---
### 2. Design system minimale condiviso
Componenti HTML/JS comuni:
- Pulsanti (`.btn-primary`)
- Card (task, ruoli, attivit)
- Status tag (pending, done, attivo)
- Form modulari
Suggerito: Tailwind CSS o UnoCSS.
---
### 3. Struttura dei modelli Rails
#### `flowpulse_branch`
```ruby
class Business < ApplicationRecord
has_many :projects
end
class Project < ApplicationRecord
belongs_to :business
has_many :tasks
has_many :roles
enum tipo: [:corso, :evento, :piattaforma]
end
class Role < ApplicationRecord
belongs_to :project
end
class Task < ApplicationRecord
belongs_to :project
end
```
#### `manager.flowpulse.net`
```ruby
class ProductionCycle < ApplicationRecord
belongs_to :project
has_many :production_lines
end
class ProductionLine < ApplicationRecord
belongs_to :production_cycle
has_many :assigned_tasks
end
class AssignedTask < ApplicationRecord
belongs_to :task
belongs_to :user
enum status: [:pending, :in_progress, :done]
end
class RequestEvent < ApplicationRecord
belongs_to :assigned_task
enum state: [:requested, :accepted, :rejected]
end
```
#### `1impegno.it`
```ruby
class UserEvent < ApplicationRecord
belongs_to :user
enum tipo: [:prenotazione, :evento, :attivit]
end
```
---
### 4. UX: interfaccia visuale manager
- Kanban board: colonne = linee produzione, card = task
- Pulsanti: [Assegna], [Accetta richiesta], [Completa]
- Badge: stato, utente, deadline, costo
---
### 5. Metriche e analisi
- Colonne aggiuntive:
- `tempo_effettivo`, `costo_effettivo`, `valutazione_efficienza`, `feedback`
- Dashboard futura
---

### 6. Roadmap MVP

| Fase | Obiettivo | App coinvolte |
|------|-----------|----------------|
| MVP 1 | Visualizza progetto + task | `flowpulse_branch` |
| MVP 2 | Apri ciclo produzione + assegna task | `manager.flowpulse.net` |
| MVP 3 | Calendario personale + attivit | `1impegno.it` |
| MVP 4 | Integrazione API | tutte |
| MVP 5 | UI condivisa + analisi | tutte |



## Miglioramenti aggiuntivi
- Modello `Resource` (luoghi, materiali, denaro, persone)
- Aggiunta clonazione progetto da template JSON
- File demo `seed/demo.json`
- Versionamento API `/api/v1/...`
---
## Pronto per partire
Suggeriti prossimi step:
1. Crea struttura modelli
2. Definisci API in `flowpulse_branch`
3. Setup demo project
4. Visualizza task assegnati in `manager.flowpulse.net`
5. Registra attivit utente in `1impegno.it`
Fammi sapere se vuoi esempi, codice seed, UI mockup o API intere.