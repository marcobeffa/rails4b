---
layout: default
title: richiesta accesso nuovi utenti
parent: Posturacorretta
grand_parent: Web app rails
nav_fold: true
has_children: true
nav_order: 7
---


## Richiesta accesso nuovi utenti 17 aprile 2024

Aggiungere tutti i moduli e il login unico per far comunicare i moduli

[aggiornamenti flowpulse](/rails4business/idee/sul_business/flowpulse_aggiornamenti/#flowpulse-istruzioni-per-sviluppo-e-integrazione-mvp)

rails g scaffold Treepage slug:string title:string description:string hash_tree:jsonb external_post_id:integer external_post_content:jsonb content:boolean branch_id:integer published:boolean visibility:integer



💻 piattaforma cms Business [🔒 ❌]
├── Treepages [🔒 ❌]
├── 🙂 users [🔒 ❌]
│   ├── 👩🏻‍🏫 Tutors [🔒 ❌]
│   │   ├── 📆 appointements - autonomia - tutor - professionista [🔒 ❌]
│   │   └── 📄 assigneds - anche il professionista [🔒 ❌]
│   ├── 🧑🏻‍⚕️ Professionals (attivo non attivo linkato alla sua pagina) [🔒 ❌]
│   ├── ✅ Done [🔒 ❌]
│   ├── ✨ Reviews [🔒 ❌]
│   └── 🗣️ conversation [🔒 ❌]
│       └── 💬 Message [🔒 ❌]
└── Dashboard [🔒 ❌]
    ├── Done Salvati con i parent [🔒 ❌]
    ├── Prescritti assigned [🔒 ❌]
    ├── Appuntamenti [🔒 ❌]
    └── 4 percorsi [🔒 ❌]


rails g authentication 


```sh
rails c
User.create(email_address: "mario@mario.it", password: "123456")


```

rails g scaffold Inviteplatform user:references user_email_inviting:string password:string accept_user_id:integer

rails g scaffold Profile user:references  nome cognome data_nascita:datetime luogo_nascita:text lat:float long:float come_ti_senti:integer last_tutor_id:integer professione:string bio:text hobbies:text 

rails g scaffold Taxdata user:references ragione_sociale:string partita_iva:string codice_fiscale:string indirizzo:string cap:string città:string provincia:string nazione:string pec:string codice_sdi:string telefono:string email:string


rails g scaffold Tutor user:references profile:references bio:text specialties:text 


rails g scaffold Professional tutor:references user:references active:boolean profile_url:string treepage:references nome cognome professione:text formazione:text bio:text

rails g scaffold Pathprofessional professional:references tutor:references path_auth:integer

rails g scaffold Appointment tutor:references user:references professional:references mode:integer scheduled_at:datetime durata_min:integer note:text parent_id:integer 

rails g scaffold Assigned appointment:references user:references content:text status:integer treepages:references parent_id:integer

rails g scaffold Done user:references treepage:references assigned:references note:text start_at:datetime end_at:datetime parent_id:integer link:text

rails g scaffold Review user:references tutor:references professional:references appointment:references review:text stars:integer target_user_id:integer rating:integer comment:text

rails g scaffold Conversation user:references appointment:references subject:string stato:integer tipo_conversazione:integer percorso:integer parent_id:integer

(per gli user: si può aprire una conversazione solo se l'ultima è chiusa)

rails g scaffold Participant conversation:references user:references tutor:references professional:references pathprofessional:references  

rails g scaffold Message conversation:references participant:references body:text 



--- 

Rails.application.credentials.dig(:active_record_encryption, :primary_key)


````rb
class Inviteplatform < ApplicationRecord
  encrypts :user_email_inviting, deterministic: true
  encrypts :password
  belongs_to :user, optional: true

  enum :richiesta_percorso, {  stop_al_dolore: 0,  benessere_integrato: 1,  accademia: 2, giardino_del_corpo: 3, creator: 4, digital: 5, investi: 6, sponsor: 7, sostieni: 8, prof_salute_benessere: 9, insegnante: 10, tutor: 11  }
  validates :password, presence: true
  validates :user_email_inviting, presence: true
end
````

se non funziona con 
bin/rails credentials:edit

active_record_encryption:
  primary_key: "x8M5EwrfT6mjDJ9IQFzjFSIGhXxXCl8fsZD4rCwMZkA="
  deterministic_key: "RbLzP5MRrmkDPZnE9tLKQyeJm1Pfj61rwMNz+fREH+g="
  key_derivation_salt: "IfVb8U+MNSHdfD/JkWrhzq+bdVWRzVrdMLPk4QO3zcs="

RAILS_ACTIVE_RECORD_ENCRYPTION_PRIMARY_KEY=... \
RAILS_ACTIVE_RECORD_ENCRYPTION_DETERMINISTIC_KEY=... \
RAILS_ACTIVE_RECORD_ENCRYPTION_KEY_DERIVATION_SALT=... \


validate :email_unica_se_non_invitato

  def email_unica_se_non_invitato
    return if user_email_inviting.blank?

    # trova tutte le richieste con la stessa email
    richieste_stesse_email = Inviteplatform.where(user_email_inviting: user_email_inviting)

    # se almeno una è già associata a un user_id (cioè è stata accettata)
    if richieste_stesse_email.where.not(user_id: nil).exists?
      errors.add(:user_email_inviting, "è già stata usata da un utente accettato")
    end

    # se nessuna ha user_id ma questa richiesta non ha invitante
    if invitante_o_verificatore_user_id.nil? && richieste_stesse_email.where(invitante_o_verificatore_user_id: nil).where.not(id: self.id).exists?
      errors.add(:user_email_inviting, "è già stata usata in una richiesta senza invito")
    end
  end