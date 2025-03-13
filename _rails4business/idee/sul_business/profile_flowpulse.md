---
layout: default
title: Profile
# parent: Flowpulse 
parent: Sul business
grand_parent: Idee
nav_fold: true
has_children: true
nav_order: 1
---

parent: Work in progress
grand_parent: Idee
## FlowPulse Profile gestisci il tuo profilo

L'app **FlowPulse Profile** è il primo modulo per la gestione degli utenti e dell'autenticazione. 

Questo modulo è responsabile di registrare e autenticare gli utenti, consentendo l'accesso sicuro agli altri moduli dell'ecosistema **FlowPulse**. Utilizza il framework **Rails Authentication** insieme a **JWT** per generare token di accesso, e integra la gestione dei profili utente, consentendo la personalizzazione e il salvataggio dei dati relativi agli utenti.

**Caratteristiche principali:**

- **Gestione utenti**: Permette di registrare e autenticare gli utenti tramite email e password.
- **Autenticazione con JWT**: Utilizza JSON Web Tokens (JWT) per generare e gestire i token di accesso.
- **Profili utente**: Ogni utente ha un profilo personalizzato con informazioni come username, contatti e avatar.

### Mockup (da aggiungere il link alla foto)
cerca un app per fare i mockup

### Tecnologie utilizzate:
- **Rails 8**
- **cursorules**
- **rails template**
- **CSS Zero** (per il layout minimalista)
- **Stimulus**
- **Hotwire** 
- **Turbo**
- **Rails Authentication**
- **JWT**
- **Importmap** (gestione delle dipendenze)

---





### Comandi per la creazione dell'app FlowPulse Profile

Per creare l'app **FlowPulse Profile**, segui i passaggi qui sotto:

1️⃣ **Creare una nuova app Rails con PostgreSQL:**

<pre>
rails new flowpulse_profile -d postgresql
</pre>

2️⃣ **Aggiungere e installare il gemma CSS Zero per il layout minimalista:**

<pre>
gem "css-zero", "~> 1.1"

bundle add css-zero
rails generate css_zero:install
rails generate css_zero:add layouts
</pre>

3️⃣ **Generare l'autenticazione per l'app (con Rails Authentication):**

<pre>
rails g authentication
</pre>

4️⃣ **Creare un utente di esempio (con email e password):**

<pre>
User.create(email_address: "you@example.com", password: "test-password-123")
</pre>

5️⃣ **Generare il modello "Profile" per memorizzare i dati utente:**

<pre>
rails g scaffold Profile user:references username:string descriptions:text avatar_img_url:string visibility:integer
</pre>

<pre>
class CreateProfiles < ActiveRecord::Migration[8.0]
  def change
    create_table :profiles do |t|
      t.references :user, null: false, foreign_key: true
      t.string :username, null: false
      t.text :descriptions
      t.string :avatar_img_url
      t.integer :visibility

      t.timestamps
    end

    add_index :profiles, :username, unique: true
  end
end

</pre>
---
<pre>
# app/controllers/profiles_controller.rb
</pre>
---
<pre>
# app/models/profile.rb
class Post < ApplicationRecord
  enum :visibility { privato: 0, gruppo: 1, pubblico: 2 }

  validates :visibility, presence: true
  validates :username, presence: true, uniqueness: true, 
                       format: { with: /\A[a-z0-9_]+\z/, message: "può contenere solo lettere minuscole, numeri e underscore" }

end
</pre>
---
<pre>
# app/views/profiles/_form.html.erb

<div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :visibility, class: "text-sm font-medium leading-none" %>
  <%= form.select :visibility, Profile.visibilities.keys.map { |v| [v.humanize, v] }, {},  class: "input" %>
</div>
</pre>
6️⃣ 