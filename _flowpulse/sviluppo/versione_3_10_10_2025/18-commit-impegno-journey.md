---
title: Comit impegno journey
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 17
---


# Commit impegno journey
non so se è meglio mettere ripo hjourney -> step e ogni step  deve avere engagement qualcosa che comprenda il ruolo il costo e il guadagno in euro e in dash e che risultato porta magari per un altra tappa, il tempo necessario, l'ordine in cui essere eseguito, cosa essere eseguito, importante urgente l'attenzione necessaria, e l'energia che serve faccine felice dolore... ma io vorrei sottolineare sia l'azione registrata ma prima  l'impegno  progettato  per raggiungere un journey che poi verrà seguita e registrata anche da altri!



Grande — ti propongo una struttura chiara che mette prima l’impegno progettato e poi l’azione registrata, includendo ruoli, costi/ricavi (EUR & DASH), priorità, energia/attenzione, tempo e collegamenti tra tappe. È pensata per essere seguita e registrata anche da altri.

Struttura concettuale

Journey → il percorso (template o istanza).

Step → la tappa del journey (ciò che va fatto).

Engagement (template di ingaggio) → come quella tappa coinvolge i ruoli e l’economia.

Pledge (impegno progettato) → la promessa/programma di esecuzione per una persona/gruppo.

Commit (azione registrata) → il fatto eseguito (log di esecuzione, risultati).

OutcomeLink → collegamento condizionale alla prossima tappa in base al risultato.

Modelli (Rails-style, sintetici)
Journey

name, description, category, visibility:enum

template:boolean (true = blueprint; false = istanza)

Relazioni: has_many :steps, has_many :pledges, has_many :commits

Step

journey:references

Progettazione/esecuzione

position:integer (acts_as_list)

duration_minutes:integer

importance:enum (low/med/high)

urgency:enum (low/med/high)

attention:enum (bassa/media/alta)

energy:enum (scarica/ok/carica) — anche iconizzabile (🙂😐☹️)

Prerequisiti & flusso

prerequisite_step_id:integer (FK opzionale)

meta:jsonb (campi liberi: attrezzi, luogo, note)

Relazioni: has_many :engagements, has_many :outcome_links

Engagement (template economico/organizzativo dello step)

step:references

role_name:string (es. tutor, operatore, centro, segnalatore, piattaforma)

Economia

price_eur_cents:integer, price_currency:string

price_dash_amount:decimal(18,8), price_dash_address:string

revenue_eur_cents:integer (quota per il ruolo)

revenue_dash_amount:decimal(18,8)

(facoltativo: revenue_share_pct:integer)

Operatività

effort_minutes:integer (tempo richiesto al ruolo)

capacity:int (max utenti gestibili in questo step da quel ruolo)

requirements:jsonb (certificazioni, strumenti)

Nota: puoi avere più Engagement per ruolo diverso nello stesso step (coerente con il tuo revenue-sharing).

Pledge (impegno progettato)

lead:references, journey:references, step:references (opz.)

intent:string (scopo dichiarato)

planned_at:datetime, due_at:datetime

status:enum (planned, approved, in_progress, paused, canceled, completed)

Priorità personali al momento dell’impegno

importance_override:enum, urgency_override:enum

energy_target:enum, attention_target:enum

meta:jsonb (es. contesto: “in ufficio”, “prima del turno”)

Relazioni: potrà essere “seguito” da altri (observer/follower)

Commit (azione registrata)

lead:references, journey:references, step:references, pledge:references

started_at, finished_at, status:enum (started, completed, skipped, failed)

Metriche/rilevazioni

pain_before:integer, pain_after:integer (0–10)

mood_before:integer, mood_after:integer (🙂 scale)

notes:text, metrics:jsonb (HRV, ripetizioni, RPE…)

dash_txid:string (se la tappa prevedeva pagamento in DASH)

Log perfetto per la “storia” del journey.

OutcomeLink (risultato → prossima tappa)

from_step_id, to_step_id

condition:jsonb (es. { "pain_after_lte": 3 } o { "commit_status": "completed" })

Serve a dire: “se ottieni X, vai alla tappa Y”.

Flussi tipici

Progettazione

Definisci Journey (template) → crea Steps → per ogni Step definisci uno o più Engagement (ruoli+economia).

Collega Steps con OutcomeLink (flow condizionale).

Impegno

L’utente (o il team) crea un Pledge: sceglie la tappa, dichiara un intent, pianifica tempi, adegua priorità/energia.

Esecuzione

Esegue → crea un Commit: logga tempi, stato, metriche (dolore, umore, note, TX DASH se previsto).

Avanzamento

Il sistema legge OutcomeLink e propone la prossima tappa in base ai risultati del Commit.

Dettagli che cercavi (tutti coperti)

✅ Ruolo → in Engagement.role_name

✅ Costo/guadagno (EUR & DASH) → Engagement.price_* e Engagement.revenue_*

✅ Risultato che porta ad altra tappa → OutcomeLink condizionale

✅ Tempo necessario → Step.duration_minutes, Engagement.effort_minutes

✅ Ordine d’esecuzione → Step.position + prerequisite_step_id

✅ Cosa dev’essere eseguito → Step.meta (lista attività/attrezzi) + descrizione

✅ Importante/Urgente → Step.importance, Step.urgency (e override nel Pledge)

✅ Attenzione/Energia → Step.attention, Step.energy (e target nel Pledge)

✅ Faccine dolore/umore → Commit.pain_before/after, mood_before/after

✅ Prima l’impegno, poi l’azione → coppia Pledge → Commit

✅ Seguito/replicabile da altri → rendi il Journey template condivisibile; i Pledge possono essere copiati, e i Commit sono pubblicabili/seguibili (followers/observers).

Snippet (migr. super-sintetiche)
create_table :journeys do |t|
  t.string :name; t.text :description; t.string :category
  t.integer :visibility, default: 0; t.boolean :template, default: true
  t.jsonb :meta, default: {}
  t.timestamps
end

create_table :steps do |t|
  t.references :journey, null: false
  t.string :title; t.text :description
  t.integer :position
  t.integer :duration_minutes
  t.integer :importance, default: 1
  t.integer :urgency, default: 1
  t.integer :attention, default: 1
  t.integer :energy, default: 1
  t.bigint :prerequisite_step_id
  t.jsonb :meta, default: {}
  t.timestamps
end
add_index :steps, :prerequisite_step_id

create_table :engagements do |t|
  t.references :step, null: false
  t.string :role_name
  t.integer :price_eur_cents; t.string :price_currency, default: "EUR"
  t.decimal :price_dash_amount, precision: 18, scale: 8
  t.string :price_dash_address
  t.integer :revenue_eur_cents
  t.decimal :revenue_dash_amount, precision: 18, scale: 8
  t.integer :effort_minutes
  t.integer :capacity
  t.jsonb :requirements, default: {}
  t.timestamps
end

create_table :pledges do |t|
  t.references :lead; t.references :journey; t.references :step
  t.string :intent
  t.datetime :planned_at; t.datetime :due_at
  t.integer :status, default: 0
  t.integer :importance_override; t.integer :urgency_override
  t.integer :energy_target; t.integer :attention_target
  t.jsonb :meta, default: {}
  t.timestamps
end

create_table :commits do |t|
  t.references :lead; t.references :journey; t.references :step; t.references :pledge
  t.datetime :started_at; t.datetime :finished_at
  t.integer :status, default: 0
  t.integer :pain_before; t.integer :pain_after
  t.integer :mood_before; t.integer :mood_after
  t.text :notes
  t.jsonb :metrics, default: {}
  t.string :dash_txid
  t.timestamps
end

create_table :outcome_links do |t|
  t.bigint :from_step_id; t.bigint :to_step_id
  t.jsonb :condition, default: {}
  t.timestamps
end
add_index :outcome_links, :from_step_id
add_index :outcome_links, :to_step_id

Enum suggeriti (model)
class Step < ApplicationRecord
  enum importance: { low: 0, medium: 1, high: 2 }
  enum urgency:    { low: 0, medium: 1, high: 2 }
  enum attention:  { bassa: 0, media: 1, alta: 2 }
  enum energy:     { scarica: 0, ok: 1, carica: 2 }
end

class Pledge < ApplicationRecord
  enum status: { planned: 0, approved: 1, in_progress: 2, paused: 3, canceled: 4, completed: 5 }
end

class Commit < ApplicationRecord
  enum status: { started: 0, completed: 1, skipped: 2, failed: 3 }
end

Micro-UX (Eisenhower & stato)

Mostra badge Importante/Urgente (matrice di Eisenhower).

Slider/emoji per Energia e Attenzione nel Pledge.

Nel Commit, widget rapido Dolore/Umore prima/dopo.

Se vuoi, te lo converto subito in scaffold Rails 8.1 con validations, indici, enum e un paio di view partial pronte (cards Tailwind per Step, Pledge e Commit).




🚆 1. Journey → il percorso

È il viaggio completo, la “linea” della metropolitana.

Può essere:

un programma (es. Igiene Posturale Base),

un processo aziendale (es. Attivazione Corso in un Centro),

o un percorso personale (es. Riattiva la Tua Fisiologia).

In sintesi:

Ha un obiettivo generale (meta del viaggio).

Contiene più step (le fermate).

Può essere un template (modello da riutilizzare) o un’istanza (viaggio reale di una persona o gruppo).

📘 Esempio:

Journey “Igiene Posturale Base” → 4 lezioni settimanali + valutazione finale.

🧭 2. Step → la tappa del journey

È una fermata del percorso: ciò che va effettivamente fatto.
Può rappresentare:

una lezione o esercizio,

una fase operativa,

un checkpoint di valutazione.

Ogni Step definisce:

cosa bisogna fare (contenuto, obiettivo, istruzioni);

quanto tempo serve;

quanto è importante e urgente;

che tipo di attenzione o energia richiede;

a quale tappa successiva porta.

📘 Esempio:

Step 1 → “Valutazione articolare”
Step 2 → “Lezione 1: mobilità del bacino”
Step 3 → “Lezione 2: respirazione e diaframma”

💼 3. Engagement → il modello di ingaggio (ruoli + economia)

È il contratto implicito tra la tappa e i ruoli coinvolti.
Descrive come quella tappa viene gestita da diversi attori e come vengono distribuite le risorse.

Ogni Step può avere più Engagement:

Ruolo	Cosa fa	Guadagno	Costo utente
Tutor	Organizza gruppo	30€	-
Insegnante	Tiene la lezione	60€	-
Centro	Offre la sala	20%	-
Utente	Partecipa	-	150€
PosturaCorretta	Coordina il sistema	10%	-

📘 Esempio:

L’Engagement dello Step “Lezione 1” stabilisce che l’insegnante guadagna 60€, il centro 20%, e PosturaCorretta il 10%.

✋ 4. Pledge → l’impegno progettato

È la decisione consapevole di intraprendere una tappa.
Quando qualcuno prende un pledge, si impegna a fare uno step in un certo tempo e con una certa intenzione.

Include:

chi lo prende (utente o gruppo),

per quale step/journey,

con quale intent (“voglio migliorare la mobilità lombare”),

entro quando lo farà,

quanta attenzione/energia dedica,

eventuali ruoli assegnati o tutor collegati.

📘 Esempio:

Antonio prende il pledge per la Lezione 1.
“Mi impegno a completarla entro domenica, con l’obiettivo di rilassare la schiena.”

Questo è l’impegno progettato, prima dell’azione vera.

✅ 5. Commit → l’azione registrata

È il fatto compiuto, la traccia concreta di un’azione.

Quando l’utente (o gruppo) realizza davvero ciò che aveva promesso nel pledge, il sistema registra un commit.

Il commit conserva:

la data/ora di esecuzione,

quanto tempo è durato,

i risultati (es. dolore prima/dopo, umore),

eventuali note, valori fisiologici, o metriche,

il collegamento a eventuali pagamenti in euro o dash.

📘 Esempio:

Antonio completa la Lezione 1.
Tempo: 45 min
Dolore prima: 6 → dopo: 3
Energia percepita: media → alta
Commit salvato e visibile al tutor.

🔄 6. OutcomeLink → il collegamento condizionale

Serve per collegare le tappe tra loro in modo dinamico.

Un OutcomeLink definisce cosa succede dopo una tappa:

Se il risultato è buono → passa alla successiva.

Se serve approfondimento → ripeti o passa a una variante.

Se un certo parametro è sotto soglia → attiva un altro journey.

📘 Esempio:

Se “Dolore dopo < 3” → vai a “Lezione 2”
Se “Dolore dopo > 5” → consiglia “Visita individuale”

È ciò che trasforma il sistema in una rete intelligente di percorsi adattivi, non lineari.

🌐 Riassunto visivo
Journey (es. Igiene Posturale)
│
├── Step 1: Valutazione iniziale
│     ├── Engagement (ruoli, costi, ricavi)
│     ├── Pledge (Antonio si impegna)
│     └── Commit (Antonio esegue)
│     └── OutcomeLink → Step 2
│
├── Step 2: Mobilità del bacino
│     └── …
│
└── Step 3: Respirazione