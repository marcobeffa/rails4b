---
layout: default
title: 1impegno 
parent: Sul business
grand_parent:  Idee
nav_fold: true
has_children: true
nav_order: 5
---

1 impegno traccia le tue risose...

- tempo 
- soldi 
- attenzione 
- energia 
- contatti 

# 🚀 Rails 8 MVP con TailwindCSS 4 e Flowbite

Questa guida ti permette di creare rapidamente un MVP in **Rails 8** con **TailwindCSS 4** e **Flowbite**.  
L'app gestisce utenti, contatti, task e inviti in meno di un'ora! ⏳

---

## **1️⃣ Creazione del progetto Rails 8**

da inserire postgresql
```sh
rails new impegno1
cd impegno1
```

## **1️⃣ Tailwindcss**
[Tailwwindcss 4](https://tailwindcss.com/docs/installation/framework-guides/ruby-on-rails)
```sh
./bin/bundle add tailwindcss-ruby
./bin/bundle add tailwindcss-rails
./bin/rails tailwindcss:install
```
```css
app/assets/tailwind/application.css
@import "tailwindcss";
```
## **2️⃣ Generazione del sistema di autenticazione**


```sh
rails g controller Pages home

```

## **1️⃣ Flowbite**
```sh

mkdir vendor/assets
mkdir vendor/assets/stylesheets
mkdir vendor/javascript/

curl https://cdn.jsdelivr.net/npm/flowbite@2.5.2/dist/flowbite.min.css > vendor/assets/stylesheets/flowbite.css

curl https://cdn.jsdelivr.net/npm/flowbite@2.5.2/dist/flowbite.turbo.min.js > vendor/javascript/flowbite.turbo.min.js




```
```sh
# importmap.rb
pin "flowbite", to: "flowbite.turbo.min.js"
# application.js 
import "flowbite"
```





---

## **2️⃣ Generazione del sistema di autenticazione**
```sh

rails g authentication 

rails g scaffold Profile user:references username tel links nome cognome

```
```sh
class CreateProfiles < ActiveRecord::Migration[6.0]
  def change
    create_table :profiles do |t|
      t.references :user, null: false, foreign_key: true
      t.string :username, null: false
      t.string :tel
      t.text :links
      t.string :nome
      t.string :cognome

      t.timestamps
    end
    add_index :profiles, :username, unique: true  # Aggiungi questa riga per rendere il campo 'username' unico
  end
end


```

```sh
rails c
User.create(email_address: "mario@mario.it", password: "123456")
```

------------------- 3 mar 2025 ------ 
## **3️⃣ Creazione dei scaffoldli principali**

### **Contact (Gestione contatti)**
```sh
rails g scaffold Contact profile:references nome cognome data_nascita:datetime luogo_nascita_indirizzo:text riferito_profile_id:integer
```

### **Location (Luoghi)**
```sh
rails g scaffold Location profile:references nome descrizione lat:float long:float google_maps_url
```

```rb
class CreateLocations < ActiveRecord::Migration[6.1]
  def change
    create_table :locations do |t|
      t.references :profile, null: false, foreign_key: true
      t.string :nome
      t.text :descrizione
      t.decimal :lat, precision: 10, scale: 6
      t.decimal :long, precision: 10, scale: 6
      t.string :google_maps_url

      t.timestamps
    end
  end
end

```

### **Task (Impegni o attività dell'utente)**
```sh
rails g scaffold Task profile:references data_inizio:datetime data_fine:datetime location:references star:integer evento:text 
```

### **Invite (Gestione inviti ai task)**
```sh
rails g scaffold Invite contact:references task:references by_contact_id:integer request:boolean accettato:datetime 
```

### **Invite (Gestione inviti transaction)**
```sh
rails g scaffold Transaction invites:references profile:references contact:references dash:boolean data_transazione:datetime valore:decimal valuta:text
```
```rb
class CreateTransactions < ActiveRecord::Migration[6.1]
  def change
    create_table :transactions do |t|
      t.references :invite, null: false, foreign_key: true
      t.references :profile, null: false, foreign_key: true
      t.references :contact, null: false, foreign_key: true
      t.boolean :dash
      t.datetime :data_transazione
      t.decimal :valore, precision: 10, scale: 2
      t.text :valuta

      t.timestamps
    end
  end
end
```
Esegui le **migrazioni**:
```sh
rails db:migrate
```

```sh
rails g migration AddVisibilityToTask visibility:integer confermato:boolean
```

## **5️⃣ Configurazione dei modelli**

### **app/models/profile.rb**
```ruby
class Profile < ApplicationRecord
  belongs_to :user
  has_many :contacts
  has_many :tasks
end
```

### **app/models/contact.rb**
```ruby
class Contact < ApplicationRecord
  belongs_to :profile
end
```

### **app/models/task.rb**
```ruby
class Task < ApplicationRecord
  belongs_to :profile
  has_many :invites
end
```

### **app/models/invite.rb**
```ruby
class Invite < ApplicationRecord
  belongs_to :contact
  belongs_to :task
end
```

---

## **6️⃣ Aggiunta delle rotte (config/routes.rb)**
Modifica **config/routes.rb**:
```ruby
Rails.application.routes.draw do
  resources :tasks
  resources :contacts
  resources :invites
  root "tasks#index"
end
```

---

## **7️⃣ Configurazione delle viste con Tailwind e Flowbite**

Aggiorna le viste CRUD nei file in **app/views** per utilizzare Flowbite.  
Esempio per il pulsante di salvataggio in **app/views/tasks/_form.html.erb**:

```erb
<button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
  Salva Task
</button>
```

---

## **8️⃣ Controller aggiornati con autorizzazione base**

### **app/controllers/tasks_controller.rb**
```ruby
class TasksController < ApplicationController
  before_action :set_task, only: %i[ show edit update destroy ]

  def index
    @tasks = Task.all
  end

  def show; end

  def new
    @task = Task.new
  end

  def create
    @task = Task.new(task_params)
    if @task.save
      redirect_to @task, notice: "Task creato con successo!"
    else
      render :new, status: :unprocessable_entity
    end
  end

  def edit; end

  def update
    if @task.update(task_params)
      redirect_to @task, notice: "Task aggiornato!"
    else
      render :edit, status: :unprocessable_entity
    end
  end

  def destroy
    @task.destroy
    redirect_to tasks_url, notice: "Task eliminato."
  end

  private

  def set_task
    @task = Task.find(params[:id])
  end

  def task_params
    params.require(:task).permit(:nome, :descrizione, :data_inizio, :data_fine, :luogo, :profile_id)
  end
end
```

---

## **9️⃣ Helpers per formattazione date e testo**

### **app/helpers/application_helper.rb**
```ruby
module ApplicationHelper
  def format_date(date)
    date.strftime("%d %B %Y - %H:%M")
  end
end
```

---

## **🔟 Avvio del server**
```sh
rails s
```
Ora puoi testare l’MVP 🚀

---

## **📌 CONCLUSIONI**
- Questo MVP ti permette di gestire utenti, task e inviti in poco tempo.
- Con **TailwindCSS + Flowbite**, hai uno stile moderno e interattivo.
- Per un'UX migliore, puoi integrare **Turbo Frames e StimulusJS**.

💡 **Se vuoi aggiungere autenticazione più avanzata, considera Devise.**

🔥 **Buona programmazione! 🚀**
