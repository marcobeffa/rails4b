---
title: Certificate role
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 20
---


# Certificate role
- rifarmi tutto da capo, 
- mi rifai tutte migration view controller model aggiornati, 
- ci deve essere la possibilità di vedere:
    - privatamente 
    - tra chi è condivisa con una o più persone interessate 
    -  pubblicamente: 

Al posto di journeyshare non era meglio ticketjourney? o si ma il punto che un 
- Journeyshare è l'iscrizione il biglietto per un journey  
- Partecipation event è solo per quell'evento, 

esempio igiene posturale ourney lez1 lez2 lez3 lez4 eventdate per i clienti ma poi ci sono tutti gli eventdate fatti da chi lavora per prepararlo e 

- la EventParticipation  è diverso dal partecipation journey perchè tutti partecipano ma con ruoli diversi.. 
- partecipationjourney uno segna anche con che ruolo partecipa 
- partecipa solo agli eventpartecipation dove ci sara la resourcenote con il ruolo della partecipation journey con cui è iscritto... 

-  Un event può avere molte partecipationevent in base a quello che è specificato nelle  step  (num-partecipanti:5 ruoli:utente)

- la partecipazione a un evento diventa anche condivisione dei taxbranch e degli eventi in base al ruolo che è segnato nel resources...  perchè una step resources può essere legata ai taxbarnch.


- step (resources) indica 
   - ruolo 
   - n_ruolo 15 
   - prezzo (retribuzione)
   (per quanto riguarda gli uteni che consumanon il services invece è indicato nel services il prezzo e nel yourney quanti  )
   - durata 
   - note 
   -  importanza 
   - urgenza 
   - energia(faccine) e 
   - position 
si può associare a eventdate_id a taxbranch_id

- il services può essere legato a taxbranch e a lead creatore
-  journey  può esser legato a services, e a taxbranch
-  step resources sono tutti gli step necessari per il processo del servizio
 → step di lavoro / compito strutturato, 
 legato al contenuto e/o all’evento, 
 step resources può essere che può essere di tipo 
 - workprogettazione
 - workcostruzione (preparazione)
 - workerogazione (erogazione)
 - partecipazione  (utenti)
  nel journey è indicato il numero di partecipanti che possono partecipare al servizio 
   e il prezzo nel servizio 
   
  step resource terrei:   
  belongs_to :journey
  belongs_to :taxbranch, optional: true
  belongs_to :eventdate, optional: true

service avrà dei journey in base a quanto viene ripetuto il servizio in oltre metterei visto che il journey può essere ripetuto qual è l'ultimo journey di riferimento, nel senso ogni journey è l'erogazione che nel caso sia ciclica verrà adattata, e viene mantenuto il ciclo migliore perciò deve essere cicle journey (ripetibile attivo) 

Un percorso concreto/progetto (es. “Igiene Posturale – Edizione Novembre”).
Ha owner, in taxbranch stato, visibilità, avanzamento, ecc.

- Services belongs_to un taxbranch_id  taxbranch has_one services può avere un Service (pacchetto vendibile)  

si può all'inizio anche solo validare un progetto che sarebbe un journey che è legato a un taxbranch che non ha ancora un services

da journey  si creano una serie di resources che è il piano per validare un progetto 

mettiamo di voler creare il journey 60 gg alla scrivania (journey) 

scrivere il progetto in un trextbranch e poi di creare tutti gli step resources per il journey 

e poi si legano gli step resource.event_id evento può avere più step resources e si fanno tutti 

e quando si finiscono gli step resource si vede se è andato bene 
si fanno i calcoli si può aggiungere un sercives 

i role  li mettiamo tutti nel step resources 
e mettiamo anche le aree: 
 - Prodotti /servizi - Reparto Contenuti - reparto rails programmazione - Reparto Eventi servizi formazione con i clienti - reparto admin e segreteria! 
 
 se per un services sono necessari più journey si apre un nuovo taxbranch e si annida
 
 un journey può avere una lista di step resources che possono poi a loro volta legarsi a step resoucer.taxbranch_id

journey id senza parent e journey.taxbranch_id e a journey.service_id id e a journey.lead_id



-------
-------
-------
-------

# db/migrate/202511150001_add_workflow_fields_to_taxbranches.rb
class AddWorkflowFieldsToTaxbranches < ActiveRecord::Migration[8.1]
  def change
    add_column :taxbranches, :service_certificable, :boolean
    add_column :taxbranches, :certificate_role, :string
    add_column :taxbranches, :status,      :integer, null: false, default: 0
    add_column :taxbranches, :visibility,  :integer, null: false, default: 0
    add_column :taxbranches, :published_at,:datetime
  end
end

1. Service (se non l’hai ancora fatto così)
rails g scaffold Service \
  name:string \
  description:text \
  n_eventdates_planned:integer \
  price_ticket_euro:decimal \
  price_ticket_dash:decimal \
  min_tickets:integer \
  max_tickets:integer \
  open_by_journey:boolean \
  taxbranch:references \
  lead:references \
  meta:jsonb

2. Journey (progetto/edizione reale)
rails g scaffold Journey \
  title:string \
  taxbranch:references \
  service:references \
  lead:references \
  importance:integer \
  urgency:integer \
  energy:integer \
  progress:integer \
  notes:text \
  price_estimate_euro:decimal \
  price_estimate_dash:decimal \
  meta:jsonb \
  cyclic:boolean \
  template_journey_id:integer \
  is_template:boolean \
  start_ideate:datetime \
  start_realized:datetime \
  start_erogation:datetime \
  complete:datetime

3. Step (step di lavoro/partecipazione)
rails g scaffold Step \
  journey:references \
  taxbranch:references \
  eventdate:references \
  role_name:string \
  area:string \
  role:string \
  role_count:integer \
  compensation_euro:decimal \
  compensation_dash:decimal \
  duration_minutes:integer \
  importance:integer \
  urgency:integer \
  energy:integer \
  position:integer \
  step_kind:integer \
  notes:text \
  meta:jsonb

4. rails g scaffold Contact \
  first_name:string \
  last_name:string \
  lead:references \
  email:string \
  phone:string \
  date_of_birth:date \
  place_of_birth:string \
  fiscal_code:string \
  vat_number:string \
  billing_name:string \
  billing_address:string \
  billing_zip:string \
  billing_city:string \
  billing_country:string \
  meta:jsonb
  
 5.  Enrollment Journey (ticket per journey, con ruolo)
rails g scaffold Enrollment \
  service:references \
  journey:references \
  contact:references \
  role_name:string \
  status:integer \
  mode:integer \
  request_kind:integer \
  requested_by_contact:references \
  invited_by_contact:references \
  price_euro:decimal \
  price_dash:decimal \
  notes:text \
  meta:jsonb --skip-test-framework

6. Event Booking (partecipazione a evento, legata a journey)
rails g scaffold Booking \
  service:references \
  eventdate:references \
  contact:references \
  enrollment:references \
  step:references \
  status:integer \
  mode:integer \
  participant_role:integer \
  requested_by_contact:references \
  invited_by_contact:references \
  price_euro:decimal \
  price_dash:decimal \
  notes:text \
  meta:jsonb --skip-test-framework


6. rails g scaffold Payment \
  contact:references \
  payable:references{polymorphic} \
  method:integer \
  status:integer \
  amount_euro:decimal \
  amount_dash:decimal \
  currency:string \
  external_id:string \
  paid_at:datetime \
  kind:integer \
  refund_amount_euro:decimal \
  refund_due_at:datetime \
  parent_payment:references \
  notes:text \
  meta:jsonb --skip-test-framework


7. Aggiornamento Eventdate (se non l’hai già collegato a Service)
rails g migration AddFieldsToEventdates \
  service:references \
  event_type:integer \
  mode:integer \
  max_participants:integer \
  location:string \
  visibility:integer



8. rails d migration RemoveHomeNavToTaxbranch

8. rails g migration AddLinkToTaxbranch link_child_taxbranch:references




in effetti ho delle eventdate che hanno degli step (che indicano le risorse perchè in effetti un evento può avere bisogno anche di più di uno step  (che indica una risorsa può avere bisogno)  ) però dall'altro canto come posso fare ad evitare che uno step abbia bisogno di più eventdate... lo step dovrebbe essere qualcosa di atomico  eventdate -> step  e taxbranch -> Journey -> step  e  / o. taxbranch -> Services -> Journey -> step e Journey -> evendate tramite gli step giusto?



Journey: è il progetto/edizione reale.

Step: è l’unità atomica di lavoro/partecipazione, dove dici:

chi serve (role_name, area, role, role_count)

quanto dura

quanto viene pagato

quale eventdate occupa

Eventdate: è solo uno slot di calendario (data, ora, luogo, modalità), che diventa “vivo” solo quando ci sono degli Step.


