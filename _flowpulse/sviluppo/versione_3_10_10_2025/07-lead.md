---
title: Lead
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 10
---

# Lead
🧩 1. Scaffold di base
rails g scaffold Lead \
  name:string \
  surname:string \
  username:references \
  email:string \
  phone:string \
  token:string:index \
  user:references \
  parent_id:integer:index \
  referral_lead_id:integer:index \
  meta:jsonb \
  --no-jbuilder

se ho class CreateLeads < ActiveRecord::Migration[8.0]
  def change
    create_table :leads do |t|
      t.string :name
      t.string :surname
      t.string :username, null: false
      t.string :email
      t.string :phone
      t.string :token
      t.references :user, null: true, foreign_key: true
      t.integer :parent_id
      t.integer :referral_lead_id
      t.jsonb :meta

      t.timestamps
    end
    add_index :leads, :token
    add_index :leads, :parent_id
    add_index :leads, :referral_lead_id
  end
end 

- Voglio fare un sistema di registrazione senza usare email o messaggi automatici ma con una verifica manuale

La registrazione avviene tramite lead in automatico si crea l'user

l'user ha:
- email_address 
- password superadmin:boolean 
- state_registration:integer 
- superadmin:boolean 

enum state [pending, approved]

alla creazione dell'user lo stato è pending e l'user è limitato a una serie di cose, 

se l'utente non viene approvato da referral_lead_id questo per eliminare i robot ma senza email e così la verifica viene fatta in manuale  con la conferma quando fatto la conferma cambia stato dell'@user.state_registration definitivamente abilitato! 

pulizia automatica degli inattivi. 

su @user non ho username lo tengo solo su lead.. 

non vorrei mettere l'username su @user e vorrei fare in modo che funzioni come rails g authentication l'ha impostato con rails 8.1

l'email obbligatoria e rendere registrazione e riconoscimento degli account veri più leggera possibile
  
non ho ancora fatto la migration!  però ho già class CreateUsers < ActiveRecord::Migration[8.0]
  def change
    create_table :users do |t|
      t.string :email_address, null: false
      t.string :password_digest, null: false

      t.timestamps
    end
    add_index :users, :email_address, unique: true
  end
end
e class CreateSessions < ActiveRecord::Migration[8.0]
  def change
    create_table :sessions do |t|
      t.references :user, null: false, foreign_key: true
      t.string :ip_address
      t.string :user_agent

      t.timestamps
    end
  end
end
 con rails g authentication


 poi


 /signup?ref=<%= current_user.lead.token %>