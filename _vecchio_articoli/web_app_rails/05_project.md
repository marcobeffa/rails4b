---
layout: default
title: Flowpulse project
parent: Web app rails
nav_fold: true
has_children: true
nav_order: 5
---


## Flowpulse login

Aggiungere tutti i moduli e il login unico per far comunicare i moduli

[aggiornamenti flowpulse](/rails4business/idee/sul_business/flowpulse_aggiornamenti/#flowpulse-istruzioni-per-sviluppo-e-integrazione-mvp)


```sh
rails new flowpulse_project -j importmap --database=postgresql
cd flowpulse_branch

bundle add css-zero

bin/rails generate css_zero:install

bin/rails generate css_zero:add --help

bin/rails g authentication

User.create(email_address: "mario@mario.it", password: "123456")


rails g controller Pages home 


rails g scaffold Structure user:references branch_id:integer parent_id:integer structure_branch_link_child_id:integer tree_hash:jsonb name description


rails generate stimulus tree_loader

rails g controller Branches index show

rails g scaffold Event \
  lat:decimal \
  long:decimal \
  position:text \
  gmaps_url:string \
  number_max_participants:integer \
  number_min_participants:integer \
  current_participants:integer \
  start_time:datetime \
  end_time:datetime \
  status:string \
  title:string \
  description:text \
  image_url:string \
  icon_url:string \
  recurrence_rule:text \
  repeat_until:datetime \
  parent_id:integer:index


  t.decimal :price, precision: 10, scale: 2

rails g scaffold Record \
  user:references \
  structure:references \
  structure_branch_id:integer \
  name:string \
  description:text \
  body:jsonb \
  parent_id:integer \
  task:boolean \
  event:references \
  payable_role_id:integer \
  receivable_role_id:integer \
  pay:decimal \
  receive:decimal \
  completed_at:datetime





rails g scaffold Role \
  user:references \
  name:string \
  description:text \
  parent_id:integer \
  active_task_id:integer


rails g scaffold Task \
  role:references \
  record:references \
  position:integer \
  parent_id:integer

rails g scaffold Engagement \
  task:references \
  event:references \
  user:references \
  tipo:integer \
  status:integer \
  note:text \
  contatto:jsonb \
  call_api:string \
  dati:jsonb 

  (call api per chiamare 1impegno.it o calendar.flowpulse.net)



  


```


  <pre>
    🧱 1. Project
Contiene:

structure → set di branch

records → istanze operative, usano i branch come dima

📦 2. Record
Ogni record ha:

record_type: viaggio, ruolo, task 

Chiave relazionale:

parent_journey_id

parent_role_id

parent_task_id

position 
👉 Questo ti permette di strutturare viaggi multilivello, task nested, e costruire un albero di eventi umani.

🌪️ Coinvolgimento (Engagement)
└── record_type: "task" | "ruolo" | "viaggio"
└── tipo: "invito" | "richiesta" | "prenotazione"
└── stato: "proposto" → "accettato" | "rifiutato"
└── record_id (riferimento al viaggio/evento/progetto)
└── user_id (utente coinvolto)
└── trasmesso a:
     ├── 📅 calendario.flowpulse.net → API/eventi → attesa accettazione
     └── 🫱🏻 1impegno.it → API risorse → accettazione di luogo/scambio/partecipazione

✔️ Al cambiamento di stato (da “proposto” a “accettato”):
→ viene generato un **evento definitivo**
→ viene creato (o aggiornato) un **impegno**
💡 CONSEGUENZA: Engagement NON è la fonte della verità.
È un in-between, che notifica, aspetta, e riceve risposta.

Quindi: lo tratti come un oggetto sincronizzabile via API:
json
Copia
Modifica
POST /engagements
{
  "record_id": "xyz",
  "record_type": "task",
  "user_id": 42,
  "tipo": "prenotazione",
  "status": "proposto"
}
Poi su 1impegno.it:
json
Copia
Modifica
PATCH /impegni/:id
{
  "engagement_id": 123,
  "status": "accepted",
  "location": "Parco Sempione",
  "exchange": "1h yoga in cambio di video",
  ...
}
E su calendar.flowpulse.net:
json
Copia
Modifica
POST /eventi
{
  "engagement_id": 123,
  "title": "Tappa 2: Video + Corpo",
  "datetime": "2025-04-08T16:00",
  "status": "pending"
}

  </pre>

## 🧠 Istruzioni: Creazione Record da un Nodo (Branch) in una Struttura

### ✅ 1. Rotta (routes.rb)
```ruby
resources :structures do
  resources :branches, only: [:show, :index]
end
```

> URL generato: `/structures/:structure_id/branches/:id`
> Questo apre il form per creare un nuovo `Record` relativo al nodo `:id` della struttura `:structure_id`.

---

### ✅ 2. Migration: `CreateRecords`
```ruby
class CreateRecords < ActiveRecord::Migration[8.0]
  def change
    create_table :records do |t|
      t.references :user, null: false, foreign_key: true
      t.references :structure, null: false, foreign_key: true
      t.integer :structure_branch_id
      t.string :name
      t.text :description
      t.jsonb :body
      t.integer :parent_id
      t.boolean :task, default: false
      t.references :event, null: true, foreign_key: true
      t.integer :payable_role_id
      t.integer :receivable_role_id
      t.decimal :pay, precision: 10, scale: 2
      t.decimal :receive, precision: 10, scale: 2
      t.datetime :completed_at

      t.timestamps
    end
  end
end
```

---

### ✅ 3. Controller: `BranchesController#show`
```ruby
class BranchesController < ApplicationController
  before_action :set_structure

  def index
    @records = Current.user.records.where(structure_id: @structure.id)
  end

  def show
    @structure = Current.user.structures.find(params[:structure_id])
    @records = Record.where(structure_id: @structure.id, structure_branch_id: params[:id])

    tree = JSON.parse(@structure.tree_hash, symbolize_names: true)
    node = helpers.find_node_by_id(tree, params[:id].to_i)
    @labeled_nodes = helpers.collect_labeled_nodes(node)

    @record = Current.user.records.build(
      structure_id: @structure.id,
      structure_branch_id: params[:id]
    )
    @current_node = node
  end

  private

  def set_structure
    @structure = Structure.find(params[:structure_id])
  end

  def extract_labeled_nodes(node)
    return [] if node.nil?

    nodes = []
    nodes << node if node[:label] == true
    node[:children].each do |child|
      nodes += extract_labeled_nodes(child)
    end if node[:children]

    nodes
  end
end
```

---

### ✅ 4. Helpers (`application_helper.rb` o `structures_helper.rb`)
```ruby
def find_node_by_id(tree, target_id)
  return tree if tree[:id] == target_id
  return nil unless tree[:children]

  tree[:children].each do |child|
    found = find_node_by_id(child, target_id)
    return found if found
  end
  nil
end

def collect_labeled_nodes(node)
  return [] unless node

  nodes = []
  nodes << node if node[:label] == true
  node[:children].each do |child|
    nodes += collect_labeled_nodes(child)
  end
  nodes
end
```

---

### ✅ 5. View: `branches/show.html.erb`
```erb
<h1 class="font-bold text-2xl mb-4">📌 Nuovo record per: <%= @current_node[:name] %></h1>

<%= render "records/form", record: @record, structure: @structure, labeled_nodes: @labeled_nodes %>

<div class="inline-flex flex-wrap items-center gap mt-4">
  <%= link_to "← Torna alla struttura", structure_path(@structure), class: "btn" %>
</div>
```

---

### ✅ 6. Partial: `records/_form.html.erb`
```erb
<%= form_with(model: record, local: true) do |form| %>
  <%= form.hidden_field :structure_id, value: structure.id %>
  <%= form.hidden_field :structure_branch_id, value: record.structure_branch_id %>

  <fieldset>
    <legend class="font-semibold mb-2">Dati personalizzati</legend>

    <% labeled_nodes.each do |node| %>
      <div class="mb-3">
        <%= form.label "body_\#{node[:id]}", node[:name], class: "block text-sm font-medium" %>
        <%= text_field_tag "record[body][\#{node[:id]}]", (record.body || {})[node[:id].to_s], class: "input" %>
      </div>
    <% end %>
  </fieldset>

  <div class="mt-4">
    <%= form.submit "📀 Salva record per questo nodo", class: "btn btn--primary" %>
  </div>
<% end %>
```

---

### ✅ 7. RecordsController: `create` + strong params
```ruby
def record_params
  params.require(:record).permit(:structure_id, :structure_branch_id, :note, :name, :description, body: {})
end

def create
  @record = Current.user.records.build(record_params)
  if @record.save
    redirect_to @record, notice: "Record creato con successo"
  else
    render :new
  end
end
```

---

## 🎯 Risultato

- Ogni nodo (`branch`) ha una pagina `show` da cui creare un `Record`
- Il form mostra solo i campi `label: true` del sottoalbero selezionato
- Il `Record` viene salvato con `structure_id`, `structure_branch_id` e un `body` dinamico
- La UI mostra chiaramente per quale nodo si sta salvando il record

---

Vuoi aggiungere anche la visualizzazione dei `parent_links`, breadcrumb o record duplicabili?



## 📘 Istruzioni: Scaffold per il modello `Role`

### ✅ Comando scaffold completo
```bash
rails g scaffold Role \
  user:references \
  record:references \
  name:string \
  description:text \
  parent_id:integer
```

Questo comando genera:
- Model `Role`
- Migration con:
  ```ruby
  t.references :user, null: false, foreign_key: true
  t.references :record, null: false, foreign_key: true
  t.string :name
  t.text :description
  t.integer :parent_id
  ```
- Controller completo con CRUD
- Views pronte per personalizzazione
- Rotte aggiornate

---

### ✅ Configurazione del model `Role`
```ruby
class Role < ApplicationRecord
  belongs_to :user, optional: true
  belongs_to :record, optional: true

  belongs_to :parent, class_name: 'Role', optional: true
  has_many :children, class_name: 'Role', foreign_key: 'parent_id', dependent: :destroy

  has_many :roletasks
  has_many :linked_records, through: :roletasks, source: :record
end
```

---

### ✅ Nuovo scaffold: `Roletask`
```bash
rails g scaffold Roletask \
  role:references \
  record:references \
  position:integer
```

Questo modello serve a:
- Collegare dinamicamente un ruolo a uno o più record
- Ordinare i compiti del ruolo con `position`

---

### ✅ Configurazione modello `Roletask`
```ruby
class Roletask < ApplicationRecord
  belongs_to :role
  belongs_to :record

  acts_as_list scope: :role  # se vuoi usare sortable/ordinamento
end
```

E nella migration:
```ruby
t.references :role, null: false, foreign_key: true
t.references :record, null: false, foreign_key: true
t.integer :position
```

---

### ✅ Uso dinamico
Puoi usare `role.linked_records` per ottenere tutti i `Record` collegati a un ruolo.
Oppure da un record: `record.roletasks` per vedere tutti i ruoli coinvolti.

---

### ✅ Possibili miglioramenti futuri
- Validazione di presenza per `name`:
  ```ruby
  validates :name, presence: true
  ```
- Scope per ruoli root:
  ```ruby
  scope :root_roles, -> { where(parent_id: nil) }
  ```
- Nested form per assegnare `Roletask` direttamente dal form di `Role`

---

Se vuoi, posso aiutarti ad:
- Visualizzare i ruoli ad albero
- Assegnare record a ruoli in un form dinamico drag-and-drop
- Costruire una vista "Kanban" per ruoli e record

------


## ⚙️ Flusso completo: Gestione Task e Ruoli in `Record#show` con Hotwire

---

## 🎯 Obiettivo

Nella pagina `Record#show`, se il record ha `task: true`, permettere di:

- ✅ Creare un nuovo `Task` associato a un `Role`
- ✅ Scegliere un `Role` esistente oppure crearne uno nuovo
- ✅ Usare `role.active_task_id` come `parent_id` del nuovo `Task`
- ✅ Gestire ordinamento con `acts_as_list`
- ✅ Restare sempre nella stessa pagina grazie a **Hotwire**

---

## ✅ Modelli coinvolti

```bash
rails g scaffold Record \
  user:references \
  structure:references \
  structure_branch_id:integer \
  name:string \
  description:text \
  body:jsonb \
  parent_id:integer \
  task:boolean \
  event:references \
  payable_role_id:integer \
  receivable_role_id:integer \
  pay:decimal \
  receive:decimal \
  completed_at:datetime

rails g scaffold Role \
  user:references \
  name:string \
  description:text \
  parent_id:integer \
  active_task_id:integer

rails g scaffold Task \
  role:references \
  record:references \
  position:integer \
  parent_id:integer
```

---

## ✅ In `RecordsController#show`

```ruby
def show
  @record = Record.find(params[:id])
  @roles = Role.all
  @task = Task.new(record: @record)
  @role = Role.new
end
```

---

## ✅ In `records/show.html.erb`

```erb
<% if @record.task? %>
  <%= turbo_frame_tag "task_section" do %>
    <%= render partial: "tasks/form", locals: { task: @task, roles: @roles } %>

    <p class="mt-2 text-sm text-gray-500">
      oppure
      <%= link_to "➕ Crea nuovo ruolo", new_role_path(turbo_return_to: request.fullpath), data: { turbo_frame: "task_section" }, class: "underline text-blue-600" %>
    </p>
  <% end %>
<% end %>
```

---

## ✅ In `tasks/_form.html.erb`

```erb
<%= form_with model: task, url: tasks_path, data: { turbo_frame: "task_section" } do |f| %>
  <div class="mb-2">
    <%= f.label :role_id, "Assegna a ruolo" %>
    <%= f.collection_select :role_id, roles, :id, :name, { prompt: "Seleziona un ruolo" }, required: true %>
  </div>

  <%= f.hidden_field :record_id, value: task.record_id %>
  <%= f.hidden_field :parent_id %>
  <%= f.hidden_field :position %>

  <div class="mt-4 flex gap-2">
    <%= f.submit "💾 Salva Task", class: "btn btn--primary" %>
  </div>
<% end %>
```

---

## ✅ In `RolesController#new`

```ruby
def new
  @role = Current.user.roles.build
  respond_to do |format|
    format.turbo_stream
    format.html
  end
end
```

---

## ✅ In `roles/new.turbo_stream.erb`

```erb
<%= turbo_stream.replace "task_section" do %>
  <%= render partial: "roles/form", locals: { role: @role } %>
<% end %>
```

---

## ✅ In `roles/_form.html.erb` (con Annulla)

```erb
<%= form_with model: role, url: roles_path, data: { turbo_frame: "task_section" } do |f| %>
  <div class="mb-2">
    <%= f.label :name %>
    <%= f.text_field :name, required: true, class: "input" %>
  </div>

  <div class="mb-2">
    <%= f.label :description %>
    <%= f.text_area :description, class: "input" %>
  </div>

  <div class="mt-4 flex gap-2">
    <%= f.submit "💾 Crea Ruolo", class: "btn btn--primary" %>

    <%= link_to "❌ Annulla", request.params[:turbo_return_to] || records_path, 
        data: { turbo_frame: "task_section", turbo_action: "replace" }, 
        class: "btn btn--secondary" %>
  </div>
<% end %>
```

---

## ✅ In `RolesController#create`

```ruby
def create
  @role = Current.user.roles.build(role_params)

  if @role.save
    @record = Record.find_by(id: params[:turbo_return_to]&.match(/records\/(\d+)/)&.captures&.first)
    @task = Task.new(record: @record, role: @role)
    @roles = Role.all

    render turbo_stream: turbo_stream.replace(
      "task_section",
      partial: "tasks/form",
      locals: { task: @task, roles: @roles }
    )
  else
    render partial: "roles/form", status: :unprocessable_entity, locals: { role: @role }
  end
end
```

---

## ✅ In `TasksController#create`

```ruby
def create
  @task = Task.new(task_params)
  role = Role.find(@task.role_id)

  @task.parent_id ||= role.active_task_id

  if @task.save
    role.update(active_task_id: @task.id)
    redirect_to record_path(@task.record), notice: "Task creato correttamente."
  else
    @roles = Role.all
    render partial: "tasks/form", status: :unprocessable_entity, locals: { task: @task, roles: @roles }
  end
end

private

def task_params
  params.require(:task).permit(:record_id, :role_id, :parent_id, :position)
end
```

---


----


# 📋 Aggiunta Elenco dei Task in `Record#show`

## 🎯 Obiettivo

Mostrare nella vista `records/show` l’elenco dei Task associati al Record corrente, con possibilità di modificarli, eliminarli o aggiungerne di nuovi.

---

## ✅ 1. Associazioni nei modelli

### `record.rb`
```ruby
has_many :tasks
```

### `task.rb`
```ruby
belongs_to :record
belongs_to :role
```

### `role.rb`
```ruby
has_many :tasks
```

---

## ✅ 2. Controller – `RecordsController#show`

```ruby
def show
  @record = Record.find(params[:id])
  @roles = Role.all
  @task = Task.new(record: @record)
  @role = Role.new
  @tasks = @record.tasks.includes(:role)
end
```

---

## ✅ 3. Vista – `records/show.html.erb`

```erb
<h2 class="text-xl font-bold my-4">📝 Elenco dei Task</h2>

<% if @tasks.any? %>
  <ul class="space-y-2">
    <% @tasks.each do |task| %>
      <li class="p-2 border rounded">
        <strong>Ruolo:</strong> <%= task.role.name %><br>
        <strong>Posizione:</strong> <%= task.position %><br>
        <strong>Parent ID:</strong> <%= task.parent_id || 'Nessuno' %><br>
        <%= link_to '✏️ Modifica', edit_task_path(task), class: "text-blue-600 underline" %> |
        <%= link_to '🗑️ Elimina', task, method: :delete, data: { confirm: 'Sei sicuro?' }, class: "text-red-600 underline" %>
      </li>
    <% end %>
  </ul>
<% else %>
  <p class="text-gray-600">Nessun task associato a questo record.</p>
<% end %>
```

---

## ➕ 4. Link per aggiungere un nuovo Task

```erb
<%= link_to '➕ Aggiungi Nuovo Task', new_task_path(record_id: @record.id), class: "btn btn--primary mt-4" %>
```

---

## 📌 Note finali

- Il controller `TasksController#new` deve accettare `params[:record_id]` per precompilare il campo `record_id` nel form.
- Puoi aggiungere `acts_as_list` e alberi di Task se vuoi visualizzazioni più complesse.

---
Show task in record:


# 📋 Aggiunta Elenco dei Task in `Record#show`

## 🎯 Obiettivo

Mostrare nella vista `records/show` l’elenco dei Task associati al Record corrente, con possibilità di modificarli, eliminarli o aggiungerne di nuovi.

---

## ✅ 1. Associazioni nei modelli

### `record.rb`
```ruby
has_many :tasks
```

### `task.rb`
```ruby
belongs_to :record
belongs_to :role
```

### `role.rb`
```ruby
has_many :tasks
```

---

## ✅ 2. Controller – `RecordsController#show`

```ruby
def show
  @record = Record.find(params[:id])
  @roles = Role.all
  @task = Task.new(record: @record)
  @role = Role.new
  @tasks = @record.tasks.includes(:role)
end
```

---

## ✅ 3. Vista – `records/show.html.erb`

```erb
<h2 class="text-xl font-bold my-4">📝 Elenco dei Task</h2>

<% if @tasks.any? %>
  <ul class="space-y-2">
    <% @tasks.each do |task| %>
      <li class="p-2 border rounded">
        <strong>Ruolo:</strong> <%= task.role.name %><br>
        <strong>Posizione:</strong> <%= task.position %><br>
        <strong>Parent ID:</strong> <%= task.parent_id || 'Nessuno' %><br>
        <%= link_to '✏️ Modifica', edit_task_path(task), class: "text-blue-600 underline" %> |
        <%= link_to '🗑️ Elimina', task, method: :delete, data: { confirm: 'Sei sicuro?' }, class: "text-red-600 underline" %>
      </li>
    <% end %>
  </ul>
<% else %>
  <p class="text-gray-600">Nessun task associato a questo record.</p>
<% end %>
```

---

## ➕ 4. Link per aggiungere un nuovo Task

```erb
<%= link_to '➕ Aggiungi Nuovo Task', new_task_path(record_id: @record.id), class: "btn btn--primary mt-4" %>
```

---

## 📌 Note finali

- Il controller `TasksController#new` deve accettare `params[:record_id]` per precompilare il campo `record_id` nel form.
- Puoi aggiungere `acts_as_list` e alberi di Task se vuoi visualizzazioni più complesse.

---






-----

role_show_task


# 📋 Visualizzare i Task in `Role#show` (struttura ad albero con `acts_as_list`)

---

## 🎯 Obiettivo

Mostrare nella vista `roles/show` tutti i Task associati al ruolo corrente, ordinati per `position`, e visualizzati come albero gerarchico (padre → figli).

---

## ✅ 1. Modello `Task` (già fatto)

```ruby
class Task < ApplicationRecord
  belongs_to :role
  belongs_to :record
  delegate :user, to: :role

  belongs_to :parent, class_name: "Task", optional: true
  has_many :children, class_name: "Task", foreign_key: "parent_id"

  acts_as_list scope: [:role_id, :parent_id]
end
```

---

## ✅ 2. Controller `RolesController#show`

```ruby
def show
  @role = Role.find(params[:id])
  @tasks = @role.tasks.where(parent_id: nil).order(:position)
end
```

---

## ✅ 3. Partial ricorsivo – `app/views/tasks/_tree.html.erb`

```erb
<ul class="ml-4 border-l border-gray-300 pl-2">
  <% tasks.each do |task| %>
    <li class="mb-2">
      🔹 <%= link_to task.record.name || "Record ##{task.record_id}", record_path(task.record) %>
      (<strong>Posizione:</strong> <%= task.position %>)

      <% if task.children.any? %>
        <%= render partial: "tasks/tree", locals: { tasks: task.children.order(:position) } %>
      <% end %>
    </li>
  <% end %>
</ul>
```

---

## ✅ 4. Vista `roles/show.html.erb`

```erb
<h2 class="text-xl font-bold my-4">📌 Task associati a questo ruolo</h2>

<% if @tasks.any? %>
  <%= render partial: "tasks/tree", locals: { tasks: @tasks } %>
<% else %>
  <p class="text-gray-600">Nessun task associato.</p>
<% end %>
```

---

## ✨ Estensioni possibili

- Drag & drop dei task (con Stimulus + Sortable.js)
- Pulsanti per completare, duplicare o spostare task
- Visualizzazione con timeline o Gantt
- Dashboard per ciclo produttivo

---

## 🧠 Consiglio

Aggiungi sempre un `position` visibile in debug per gestire gli ordinamenti nei test iniziali!

---

engaged_model.md



# 🤝 Engagement Model – Coinvolgimento dinamico per Record, Task o Eventi


---

## 🎯 Obiettivo

Definire una struttura flessibile per gestire **inviti, proposte e prenotazioni** su `Task` o `Event`, con possibilità di estendersi in futuro a **1impegno** e **calendar.flowpulse** via API e JSON.

---

## ✅ Generazione del Model

```bash
rails g model Engagement \
  task:references \
  event:references \
  user:references \
  tipo:integer \
  status:integer \
  note:text \
  contatto:jsonb \
  call_api:string \
  dati:jsonb
```
```bash
rails g migration AddPlatformToStructure business_name:string platform_url:string space_osposition:integer 
```
---

## ✅ Dettagli dei campi

| Campo         | Tipo       | Descrizione |
|---------------|------------|-------------|
| task          | references | Task associato (opzionale) |
| event         | references | Evento associato (opzionale) |
| user          | references | Chi riceve/invia l'invito |
| tipo          | integer    | Enum: `invito`, `richiesta`, `prenotazione` |
| status        | integer    | Enum: `proposto`, `accettato`, `rifiutato` |
| note          | text       | Note interne |
| contatto      | jsonb      | Dati di contatto in forma libera (nome, email, telefono, link, ecc.) |
| call_api      | string     | API chiamata (es. `calendar.data_event`, `impegno.transazione`) |
| dati          | jsonb      | Payload completo per conferma o proposta, libero |

---

## ✅ Enum nel Model

```ruby
class Engagement < ApplicationRecord
  belongs_to :user
  belongs_to :task, optional: true
  belongs_to :event, optional: true

  enum tipo: { invito: 0, richiesta: 1, prenotazione: 2 }
  enum status: { proposto: 0, accettato: 1, rifiutato: 2 }

  after_save :crea_ticket_se_accettato

  def crea_ticket_se_accettato
    return unless saved_change_to_status? && accettato?

    # esempio di azione
    Ticket.create!(user: user, event: event, engagement: self)
  end
end
```

---

## ✅ Esempio di `contatto` JSONB

```json
{
  "nome": "Mario Rossi",
  "email": "mario@example.com",
  "telefono": "1234567890",
  "link": "https://wa.me/39123456789"
}
```

---

## ✅ Esempio di `dati` JSONB (per 1impegno / calendar)

```json
{
  "tipo": "proposta",
  "orario": "10:00",
  "data": "2025-04-10",
  "luogo": "Online",
  "note": "Confermare disponibilità",
  "transazione": {
    "importo": 30,
    "valuta": "EUR"
  }
}
```

---

## ✅ Indici consigliati

```ruby
add_index :engagements, :status
add_index :engagements, :tipo
add_index :engagements, [:user_id, :task_id]
add_index :engagements, :contatto, using: :gin
add_index :engagements, :dati, using: :gin
```

---

## 📡 Integrazione futura

- `call_api` sarà utilizzabile per chiamate automatiche verso:
  - `1impegno.it/api/...` per disponibilità, transazioni, luoghi
  - `calendar.flowpulse/...` per creare o accettare eventi
- `dati` conterrà tutto il payload utile, in ingresso e in uscita

---

## 📦 Pronto per API e automazioni

Questo schema ti permette già da ora:
- Proporre task o eventi
- Gestire inviti anche senza contatti registrati
- Integrare flussi esterni senza cambiare struttura

---




