---
layout: default
title: Note 
parent: Web app rails

nav_fold: true
has_children: true
nav_order: 3
---


## FlowPulse Note

Voglio costruire una semplice app per i contentui in rails:
- con https://csszero.lazaronixon.com/lookbook/pages/overview 
- rails g authentication 
- rails g Content user:references title description body:text data_pubblicazione:datetime visibillity:integer published:boolean stato:integer 

Vorrei impostrare il layout.
https://csszero.lazaronixon.com/lookbook/inspect/layouts/sidebar_layout_2 


come editor usare easy-markdown-editor per il campo body

note.rb
enum :visibility, { privato: 0, iscritti: 1, pubblico: 2 }
enum :stato, { bozza: 0, revisione: 1, pubblicato: 2 }



- rails g controller Pages home profili
- rails g controller Dashboard private group public published future


## Create app

```sh
rails new flowpulse_note -j importmap 
cd flowpulse_note

```

## FlowPulse Note



```sh

bundle add css-zero

bin/rails generate css_zero:install

bin/rails generate css_zero:add --help

bin/rails g authentication

User.create(email_address: "mario@mario.it", password: "123456")


rails g controller Pages home utenti

rails g controller Dashboard private group public published future
```
## Crea contenuti

```sh
rails g scaffold Content user:references title description body:text data_pubblicazione:datetime visibility:integer published:boolean stato:integer
```

content.rb

```rb

enum :visibility, { privato: 0, iscritti: 1, pubblico: 2 }

enum :stato, { bozza: 0, revisione: 1, pubblicato: 2 }

```


## SideBar
https://csszero.lazaronixon.com/lookbook/inspect/layouts/sidebar_layout_2
```sh
bin/rails generate css_zero:add dialog

bin/rails generate css_zero:add layout
```


- scaricare e copiare gli assets/image

da aggiungere ai controller dove l'utente non si deve autenticare.
```rb
allow_unauthenticated_access only: %i[ new create ]
```

Sidebar menù:
```html
<ul>
  <li>
    <%= link_to dashboard_private_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Private</span>
    <% end %>
  </li>
  <li>
    <%= link_to dashboard_group_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Gruppo</span>
    <% end %>
  </li>
  <li>
    <%= link_to dashboard_public_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Pubbliche</span>
    <% end %>
  </li>
  <br>
  <hr>
  <br>
  <li>
    <%= link_to dashboard_published_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Pubblicate</span>
    <% end %>
  </li>
  <li>
    <%= link_to dashboard_future_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Future</span>
    <% end %>
  </li>
  <li>
    <%= link_to contents_path, class: "btn sidebar-menu__button" do %>
      <%= image_tag "frame.svg", size: 16, aria: { hidden: true } %>
      <span class="overflow-ellipsis">Tutte</span>
    <% end %>
  </li>
</ul>
```
## Hotwire spark
(Installa la gemma hotwire spark)[https://github.com/hotwired/spark]

## content index
content.html.erb
```sh
bin/rails generate css_zero:add table
```


```html
<table class="table">
  <caption>
    A list of your recent invoices.
  </caption>
  <thead>
    <tr>
      <th>Invoice</th>
      <th>Status</th>
      <th>Method</th>
      <th class="text-end">Amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>INV001</th>
      <td>Paid</td>
      <td>Credit Card</td>
      <td class="text-end">$250.00</td>
    </tr>
   
  </tbody>
  <tfoot>
      <tr>
        <td colspan="3">Total</td>
        <td class="text-end">$2,500.00</td>
      </tr>
  </tfoot>
</table>
```




## Easy markdown 
https://github.com/Ionaru/easy-markdown-editor
https://chatgpt.com/share/67c9a7cc-9620-8002-89e2-0834e13e2828




- app/assets/stylesheets/easymde.min.css
<link rel="stylesheet" href="https://unpkg.com/easymde/dist/easymde.min.css">

- vendor/javascripts/easymde.min.js
<script src="https://unpkg.com/easymde/dist/easymde.min.js"></script>


/app/javascript/application.js
```js
import "@hotwired/turbo-rails"
import "controllers"
import "easymde"
```
app/javascript/controllers/application.js
aggiungi:
```js
const easyMDE = new EasyMDE({
  element: document.getElementById("editor"),
  sideBySide: true,      // Abilita la modalità affiancata
  previewRender: function(plainText) {
    // Puoi personalizzare il rendering se necessario
    return EasyMDE.prototype.markdown(plainText);
  },
});
```

## Redcarpet

gem 'redcarpet'

content_helper.rb
```rb
module ContentsHelper
  def rendered_body(body)
    renderer = Redcarpet::Render::HTML.new(hard_wrap: true, filter_html: true)
    markdown = Redcarpet::Markdown.new(renderer,
                                       autolink: true,
                                       tables: true,
                                       fenced_code_blocks: true)
    markdown.render(body).html_safe
  end
end

```




text.css
```css
/* Stili base per il contenuto Markdown */
.markdown-body,
.editor-preview-active-side,
.editor-preview-active {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
  font-size: 1rem;
  line-height: 1.6;
  color: #24292e;
  background-color: #fff;
  padding: 1rem;
}

/* Headings */
.markdown-body h1, .editor-preview-active-side h1, .editor-preview-active h1,
.markdown-body h2, .editor-preview-active-side h2, .editor-preview-active h2,
.markdown-body h3, .editor-preview-active-side h3, .editor-preview-active h3,
.markdown-body h4, .editor-preview-active-side h4, .editor-preview-active h4,
.markdown-body h5, .editor-preview-active-side h5, .editor-preview-active h5,
.markdown-body h6, .editor-preview-active-side h6, .editor-preview-active h6 {
  font-weight: 600;
  line-height: 1.25;
  margin-top: 1.2em;
  margin-bottom: 0.6em;
}

.markdown-body h1, .editor-preview-active-side h1, .editor-preview-active h1 {
  font-size: 2.25em;
  border-bottom: 1px solid #eaecef;
  padding-bottom: 0.3em;
}

.markdown-body h2, .editor-preview-active-side h2, .editor-preview-active h2 {
  font-size: 1.75em;
  border-bottom: 1px solid #eaecef;
  padding-bottom: 0.3em;
}

.markdown-body h3, .editor-preview-active-side h3, .editor-preview-active h3 {
  font-size: 1.5em;
}

.markdown-body h4, .editor-preview-active-side h4, .editor-preview-active h4 {
  font-size: 1.25em;
}

.markdown-body h5, .editor-preview-active-side h5, .editor-preview-active h5 {
  font-size: 1em;
}

.markdown-body h6, .editor-preview-active-side h6, .editor-preview-active h6 {
  font-size: 0.875em;
  color: #6a737d;
}

/* Paragrafi */
.markdown-body p, .editor-preview-active-side p, .editor-preview-active p {
  margin: 0 0 1em;
}

/* Blockquote */
.markdown-body blockquote, .editor-preview-active-side blockquote, .editor-preview-active blockquote {
  margin: 0 0 1em;
  padding: 0.5em 1em;
  color: #6a737d;
  border-left: 0.25em solid #dfe2e5;
  background-color: #f6f8fa;
}

/* Liste */
.markdown-body ul, .editor-preview-active-side ul, .editor-preview-active ul,
.markdown-body ol, .editor-preview-active-side ol, .editor-preview-active ol {
  padding-left: 2em;
  margin: 0 0 1em;
}

.markdown-body li, .editor-preview-active-side li, .editor-preview-active li {
  margin-bottom: 0.5em;
}

/* Codice inline */
.markdown-body code, .editor-preview-active-side code, .editor-preview-active code {
  padding: 0.2em 0.4em;
  margin: 0;
  font-size: 85%;
  background-color: rgba(27, 31, 35, 0.05);
  border-radius: 3px;
  font-family: "Courier New", monospace;
}

/* Blocchi di codice */
.markdown-body pre, .editor-preview-active-side pre, .editor-preview-active pre {
  display: block;
  padding: 1em;
  margin: 0 0 1em;
  font-size: 85%;
  line-height: 1.45;
  background-color: #f6f8fa;
  border-radius: 3px;
  overflow: auto;
}

.markdown-body pre code, .editor-preview-active-side pre code, .editor-preview-active pre code {
  padding: 0;
  margin: 0;
  font-size: 100%;
  border: none;
  background: none;
}

/* Tabelle */
.markdown-body table, .editor-preview-active-side table, .editor-preview-active table {
  width: 100%;
  margin-bottom: 1em;
  border-collapse: collapse;
}

.markdown-body table th, .editor-preview-active-side table th, .editor-preview-active table th,
.markdown-body table td, .editor-preview-active-side table td, .editor-preview-active table td {
  padding: 0.6em 1em;
  border: 1px solid #dfe2e5;
}

.markdown-body table th, .editor-preview-active-side table th, .editor-preview-active table th {
  background-color: #f6f8fa;
  font-weight: 600;
}

/* Immagini */
.markdown-body img, .editor-preview-active-side img, .editor-preview-active img {
  max-width: 100%;
  height: auto;
}

/* Link */
.markdown-body a, .editor-preview-active-side a, .editor-preview-active a {
  color: #0366d6;
  text-decoration: none;
}

.markdown-body a:hover, .editor-preview-active-side a:hover, .editor-preview-active a:hover {
  text-decoration: underline;
}

/* Evidenziazione */
.markdown-body strong, .editor-preview-active-side strong, .editor-preview-active strong {
  font-weight: bold;
}

.markdown-body em, .editor-preview-active-side em, .editor-preview-active em {
  font-style: italic;
}

/* Separatori */
.markdown-body hr, .editor-preview-active-side hr, .editor-preview-active hr {
  border: 0;
  height: 1px;
  background: #dfe2e5;
  margin: 1.5em 0;
}
```


## Modello content.rb


```rb
class Content < ApplicationRecord
  belongs_to :user
  before_save :set_publication_date
  after_find :check_and_update_publication_status


  # Enum per visibilità
  enum :visibility, { privato: 0, iscritti: 1, pubblico: 2 }

  # Enum per stato
  enum :stato, { bozza: 0, revisione: 1, corretto: 2 }

  # Validazioni
  validates :title, presence: true
  validates :body, presence: true
  validates :visibility, inclusion: { in: visibilities.keys }
  validates :stato, inclusion: { in: statos.keys }

  # Scopes utili
  scope :pubblici, -> { where(visibility: :pubblico) }
  scope :in_revisione, -> { where(stato: :revisione) }

  scope :futuri, -> { where(stato: :revisione) }
  scope :pubblicati, -> { where(stato: :pubblicato) }

  # Metodo helper per verificare lo stato
  def bozza?
    stato == "bozza"
  end

  def pubblicato?
    stato == "pubblicato"
  end

  def revisione?
    stato == "revisione"
  end

  private

  def set_publication_date
    if published? && data_pubblicazione.nil?
      self.data_pubblicazione = Time.current
    end
  end
  # Controllo automatico quando il record viene caricato
  def check_and_update_publication_status
    if data_pubblicazione.present? && data_pubblicazione <= Time.current && !pubblicato?
      update_column(:published, true) # Salva senza callback per evitare loop
    end
  end
end

```


form.html.erb
```html
  <%= form_with(model: content, html: { contents: true }) do |form| %>
  <% if content.errors.any? %>
    <div class="alert alert--negative flex flex-col gap-half mbe-4" role="alert">
      <h2 class="font-medium leading-none"><%= pluralize(content.errors.count, "error") %> prohibited this content from being saved:</h2>
      <ul class="text-sm mis-3" style="list-style: disc">
        <% content.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  <div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :user_id, class: "text-sm font-medium leading-none" %>
    <%= form.text_field :user_id, class: "input" %>
  </div>
  <div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :title, class: "text-sm font-medium leading-none" %>
    <%= form.text_field :title, class: "input" %>
  </div>
  <div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :description, class: "text-sm font-medium leading-none" %>
    <%= form.text_field :description, class: "input" %>
  </div>
  <div class="mbe-4">
    <%= form.label :body, class: "text-sm font-medium leading-none" %>
    <%= form.textarea :body, class: "input", id:"editor" %>
  </div>
  <div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :visibility, "Visibilità", class: "text-sm font-medium leading-none" %>
    <%= form.select :visibility, Content.visibilities.keys.map { |v| [v.humanize, v] }, {}, class: "input"  %>
  </div>
  <div class="flex flex-col items-start gap-half mbe-4">
    <%= form.label :stato, "Stato", class: "text-sm font-medium leading-none" %>
    <%= form.select :stato, Content.statos.keys.map { |s| [s.humanize, s] }, {}, class: "input",  onchange: "this.form.submit();"  %>
  </div>
  <% if @content.stato == "corretto" %>
    <div class="flex flex-col items-start gap-half mbe-4">
      <%= form.label :published, class: "text-sm font-medium leading-none" %>
      <%= form.checkbox :published, class: "checkbox",  onchange: "this.form.submit();"  %>
    </div>
    <div class="flex flex-col items-start gap-half mbe-4">
      <%= form.label :data_pubblicazione, class: "text-sm font-medium leading-none" %>
      <%= form.datetime_field :data_pubblicazione, class: "input" %>
    </div>
  <% end %>
  <div class="inline-flex items-center mbs-2 mie-1">
    <%= form.submit class: "btn btn--primary" %>
  </div>
<% end %>
```


## Selezionare le note in base ai campi

https://csszero.lazaronixon.com/lookbook/inspect/table/data_table

_nav_content.html.erb
```html

```

gem 'ransack'


## Pagy.rb 

https://ddnexus.github.io/pagy/playground/

contets/pagy.html.erb
```html
<div class="flex items-center">
      <div class="text-sm text-subtle show@md">Total of <%= @pagy.count %> record(s).</div>
      <div class="flex items-center mis-auto justify-end" style="column-gap: 1rem">
        <%= form_with url: pagy_url_for(@pagy, 1), class: "flex items-center gap show@md", method: :get, data: { controller: "form", action: "change->form#submit" } do |form| %>
          <%= form.label :limit, "Rows per page", class: "text-sm font-medium" %>
          <%= form.select :limit, [10, 20, 30, 40, 50], { selected: @pagy.limit }, { class: "input", style: "--input-inline-size: 70px" } %>
        <% end %>
        <div class="text-sm font-medium"><%= "Page #{@pagy.page} of #{@pagy.pages}" %></div>
        <nav class="flex items-center gap shrink-0" style="--btn-padding: .5rem;" aria-label="Pagination">
          <%= link_to pagy_url_for(@pagy, 1), class: "btn", aria: { disabled: @pagy.prev.nil? }.compact_blank do %>
            <%= image_tag "chevrons-left.svg", size: 16, aria: { hidden: true } %>
            <span class="sr-only">Go to first page</span>
          <% end %>
          <%= link_to pagy_url_for(@pagy, @pagy.prev || @pagy.page), class: "btn", aria: { disabled: @pagy.prev.nil? }.compact_blank do %>
            <%= image_tag "chevron-left.svg", size: 16, aria: { hidden: true } %>
            <span class="sr-only">Go to previous page</span>
          <% end %>
          <%= link_to pagy_url_for(@pagy, @pagy.next || @pagy.page), class: "btn", aria: { disabled: @pagy.next.nil? }.compact_blank do %>
            <%= image_tag "chevron-right.svg", size: 16, aria: { hidden: true } %>
            <span class="sr-only">Go to next page</span>
          <% end %>
          <%= link_to pagy_url_for(@pagy, @pagy.last), class: "btn", aria: { disabled: @pagy.next.nil? }.compact_blank do %>
            <%= image_tag "chevrons-right.svg", size: 16, aria: { hidden: true } %>
            <span class="sr-only">Go to last page</span>
          <% end %>
        </nav>
      </div>
    </div>
  <% end %>
</div>

```


## timezone Sarebbe da mettere insieme alla posizione dell'utente e alla lingua:

✅ Salva tutte le date in UTC nel database
✅ Rileva il fuso orario dinamicamente con JavaScript
✅ Aggiorna il timezone dell'utente nella sessione e/o database
✅ Converte le date dinamicamente quando vengono visualizzate

🚀 Ora il tuo sistema supporta automaticamente gli utenti che cambiano fuso orario mentre viaggiano! 🌍

per ora però impostiamo il fuso orario su europe roma

config/application.rb:

```rb
config.time_zone = "Rome"
config.active_record.default_timezone = :utc  # Salva in UTC nel DB

```


```rb
time_utc = Time.current.utc  # Ottiene la data attuale in UTC
```




## data 

Form per la data
```html
<% Time.current.strftime("%d-%m-%y") %>
<%= Time.current.in_time_zone.strftime("%Z")%>
<%= Time.zone %>
<%= form_with url: request.path, method: :get, local: true do |form| %>
  <%= form.datetime_field :data, class: "input",value: Time.current.strftime("%Y-%m-%dT%H:%M"), style: "max-inline-size: 180px" %>
  <%= form.submit "Vai", class: "btn" %>
<% end%>
<%= form_with url: request.path, method: :get, local: true do |form| %>
  <%= form.date_field :data, class: "input",value: Time.current.strftime("%Y-%m-%d"), style: "max-inline-size: 180px" %>
  <%= form.submit "Vai", class: "btn" %>
<% end%>
<div class="group" role="group" aria-label="Basic example">
  <%= link_to "Tutti", new_content_path, class: "btn btn--secondary" %>
  <%= link_to "Passati", new_content_path, class: "btn btn--secondary" %>
  <%= link_to "Futuri", new_content_path, class: "btn btn--secondary" %>
  <%= link_to "New content", new_content_path, class: "btn btn--primary" %>
</div>

```

## Routes rails

routes.rb
```rb
namespace :dashboard do
  get "all/:data/:visibility/:stato", to: "dashboard#all", as: "dashboard_all"
  get "past/:data/:visibility/:stato", to: "dashboard#past", as: "dashboard_past"
  get "future/:data/:visibility/:stato", to: "dashboard#future", as: "dashboard_future"
end
```

dashboard_helper.rb
```rb
module DashboardHelper
  def dashboard_path_for(type, params = {})
    routes = Rails.application.routes.url_helpers

    # Definiamo i valori predefiniti per data, stato e visibility
    default_params = {
      data: params[:data] || Date.today.strftime("%Y-%m-%d"),
      visibility: params[:visibility] || "all",
      stato: params[:stato] || "active"
    }

    # Scegliamo il path corretto in base al tipo richiesto
    base_path = case type
    when :all then routes.all_path(default_params[:data], default_params[:visibility], default_params[:stato])
    when :past then routes.past_path(default_params[:data], default_params[:visibility], default_params[:stato])
    when :future then routes.future_path(default_params[:data], default_params[:visibility], default_params[:stato])
    else routes.all_path(default_params[:data], default_params[:visibility], default_params[:stato]) # Default
    end

    base_path
  end
end

```
_nav_dashboard.html.erb
```html

<%= link_to "Tutti", dashboard_path_for(:all), class: "btn btn--secondary" %>
<%= link_to "Tutti", dashboard_path_for(:all, data: "2025-03-12", visibility: "private", stato: "archived"), class: "btn btn--secondary" %>

<div class="group" role="group" aria-label="Navigazione eventi">
  <%= link_to "Tutti", dashboard_path_for(:all), class: "btn btn--secondary" %>
  <%= link_to "Passati", dashboard_path_for(:past), class: "btn btn--secondary" %>
  <%= link_to "Futuri", dashboard_path_for(:future), class: "btn btn--secondary" %>
  <%= link_to "New content", new_content_path, class: "btn btn--primary" %>
</div>
```

## Dashboard all past futures

_dashboard_index.html.erb
```html


```


## Friendly id

gem 'friendly_id', '~> 5.5.0'
```sh
bundle install

rails g migration AddSlugToContent slug:uniq
```

content.rb
```rb

extend FriendlyId
friendly_id :title, use: :slugged

before_save :set_slug

def set_slug
  self.slug = title.to_s.downcase
                 .gsub(/[^a-z0-9\s]/, '')  # Rimuove caratteri speciali
                 .gsub(/\s+/, '-')          # Sostituisce spazi con trattini
                 .gsub(/-+/, '-')           # Evita più trattini consecutivi
                 .gsub(/^-|-$/, '')         # Rimuove eventuali trattini iniziali o finali
end

```



## aggiungere username a user

rails g migration AddUsernameToUsers username:uniq

validazione username unico

validates :username, presence: true, uniqueness: true,
                       format: { with: /\A[a-z0-9\_]+\z/, message: "può contenere solo lettere minuscole, numeri e trattini bassi" },
                       length: { minimum: 3, maximum: 20 }

https://blog.corsego.com/rails-8-authentication-registration

## Show public

get "utente/:username/contents/:slug/", to: "pages#content", as: "public_content"


----- 

## Utenti ?


## Posts controller ? 

vedere se vale la pena fare la ricerca con rainsack 

e mettere come link lo stato e il resto

Vedere la vista.

Mettere in production l'applicazione.


---- ethos pathos logos
--- hook parole -tempo recitazione tono e ritmo


## contente stato contenuti 
- Dario vignali 
- sephirot 
- Il discorso perfetto

[
  "Ideazione",       // Generazione di idee
  "Catalogazione",   // Etichettatura e organizzazione iniziale
  "Digestione",      // Assimilazione e approfondimento
  "Riordino",        // Creazione di un flusso logico
  "Parole Giuste",   // Refinement e ottimizzazione del linguaggio
  "Scrittura Slide", // Creazione di materiali
  "Memorizzare",     // Assimilazione attiva
  "Interpretare",    // Personalizzazione e adattamento
  "Pratica"          // Applicazione concreta nel mondo reale
]

[
  "Ideazione",        // Generazione di idee
  "Catalogazione",    // Organizzazione iniziale e archiviazione
  "Digestione",       // Approfondimento e assimilazione
  "Riordino",         // Creazione di un flusso logico
  "Parole Giuste",    // Refinement del linguaggio
  "Scrittura Storia", // Produzione del contenuto testuale
  "Scrittura Discorso", // Adattamento per la comunicazione orale
  "Memorizzare",      // Assimilazione attiva
  "Interpretare",     // Personalizzazione e adattamento
  "Pratica",          // Applicazione concreta nel mondo reale
  "Presentazione/Video" // Registrazione o esposizione dal vivo
]



## lingua

Passaggi per tradurre la data in italiano:
Assicurati che il file delle lingue sia configurato
Modifica (o crea) il file delle traduzioni in config/locales/it.yml:

yaml
Copia
it:
  date:
    formats:
      default: "%d-%m-%Y"
      long: "%A %d %B %Y - %H:%M"  # Es. "Giovedì 13 Marzo 2025"
    day_names: [Domenica, Lunedì, Martedì, Mercoledì, Giovedì, Venerdì, Sabato]
    abbr_day_names: [Dom, Lun, Mar, Mer, Gio, Ven, Sab]
    month_names: [~, Gennaio, Febbraio, Marzo, Aprile, Maggio, Giugno, Luglio, Agosto, Settembre, Ottobre, Novembre, Dicembre]
    abbr_month_names: [~, Gen, Feb, Mar, Apr, Mag, Giu, Lug, Ago, Set, Ott, Nov, Dic]

  time:
    formats:
      default: "%H:%M"
      long: "%A %d %B %Y - %H:%M" # Es. "Giovedì 13 Marzo 2025"

    
    
ruby
Copia
Modifica
config.i18n.default_locale = :it
Usa il metodo I18n.l per la traduzione
Modifica il tuo codice in questo modo:

erb
Copia
Modifica
<%= I18n.l(@post.publication_date, format: :long) if @post.publication_date %>


## Prossimamente content status log
Opzione 2: Creare un modello ContentStatusLog per registrare gli stati nel tempo (più scalabile)
Questa opzione ti permette di registrare ogni cambiamento di stato come un evento, senza dover aggiungere nuove colonne.

Tabella content_status_logs
ruby
Copia
Modifica
create_table :content_status_logs do |t|
  t.references :content, foreign_key: true # Associa il log a un contenuto
  t.integer :stato, null: false            # Stato del contenuto al momento del cambio
  t.datetime :changed_at, default: -> { 'CURRENT_TIMESTAMP' } # Data del cambio
end
Modello ContentStatusLog
ruby
Copia
Modifica
class ContentStatusLog < ApplicationRecord
  belongs_to :content
  enum stato: Content.statos # Usa la stessa enum del modello Content
end
Modello Content (aggiungere relazione e callback)
ruby
Copia
Modifica
class Content < ApplicationRecord
  has_many :content_status_logs

  enum :stato, {
    ideazione: 0, catalogazione: 1, riordino: 2, scrittura_storia: 3, scrittura_discorso: 4,
    revisione: 5, finalizzata: 6, memorizzare: 7, interpretare: 8, pratica: 9,
    registrazione_video: 10, editing: 11, titoli_descrizione: 12, pubblicazione: 13, concluso: 14
  }

  before_update :log_status_change, if: :will_save_change_to_stato?

  private

  def log_status_change
    content_status_logs.create(stato: stato, changed_at: Time.current)
  end
end
👎 Richiede una tabella in più, ma 👍 è molto più flessibile:

Puoi vedere storico degli stati.
Non devi cambiare la struttura del database ogni volta che aggiungi un nuovo stato.
Puoi calcolare il tempo medio in ogni stato.
Conclusione: Quale scegliere?
Se vuoi semplicità e non ti serve uno storico dettagliato → Opzione 1 (campi diretti per ogni stato).
Se vuoi flessibilità e tracciabilità storica → Opzione 2 (tabella content_status_logs).
Per un sistema scalabile, ti consiglio la Opzione 2, perché ti permette di gestire meglio i cambi di stato nel tempo senza modificare la struttura del database ogni volta. 🚀

Cosa ne pensi? Ti serve aiuto per implementarla? 😊





## Bug

in registration edit quando c'è un errore:
field_with_errors 

## Dash
- Acquisto Posso passarlo in dash 
- Mercato
- Proposal
- Delega vs Responsabilità
- Adozione attività locale 

## test

- Gemma	Funzione principale
- Rubocop (Consigliato)	Linting e correzione automatica per Ruby/Rails
- StandardRB	Più semplice di Rubocop, meno configurazione
- Fasterer	Analizza il codice per migliorarne le prestazioni
- Reek	Analizza il codice "puzzolente" per migliorare la qualità

bundle exec rubocop --auto-correct


## scarica il pdf dalle pagine

https://chatgpt.com/share/67d31b6b-fcc8-8002-af8a-423d080ac32b

## download delle note in csv

## Aggiungere un admin per vedere gli utenti

## Una struttura ad albero 
## I gruppi e 
## gli accessi alle note
## Stanze con accesso invito e stanze singole e commenti
## Stanza e invito messaggi 
## Vista singola

## Schede di esercizi
## Programmi delle attività
## Verfica step del programma
## Feedback


## Turbo frame ecc..


### Pwa

https://chatgpt.com/share/67d31c20-732c-8002-ae7b-6d84b94d9431

https://alicia-paz.medium.com/make-your-rails-app-work-offline-part-1-pwa-setup-3abff8666194


## Lo stato  se la visibilità è privata scompare.

bundle exec rubocop --auto-correct   




CI / scan_ruby (push) In progress - This check has started...
Details

CI / lint (push) In progress - This check has started...
Details

CI / test (push) In progress - This check has started...
Details

CI / scan_js (push) Successful in 12s


# config/initializers/field_with_errors.rb

```rb
# config/initializers/field_with_errors.rb
ActionView::Base.field_error_proc = Proc.new do |html_tag, _instance|
  html_tag.html_safe
end
```



---------------------------
# API Rails: Isolamento e Versioning con V1

## 1️⃣ Generare il Controller API

Usa il comando per creare un controller API con namespace `api/v1`:

```bash
rails g controller api/v1/profiles/posts --skip-template-engine --skip-assets --skip-helper
```

Questo creerà il file:

```
📂 app/controllers/api/v1/profiles/posts_controller.rb
```

---

## 2️⃣ Configurare il Controller API

Apri il file generato e implementa `index` e `show`:

```ruby
module Api
  module V1
    class Profiles::PostsController < ApplicationController
      before_action :set_profile

      # Lista di tutti i post pubblici (solo note)
      def index
        @posts = @profile.contents.published.public_visibility.notes_only.order(publication_date: :desc)
        render json: @posts
      end

      # Singolo post (controlla che sia pubblico e sia una nota)
      def show
        @post = @profile.contents.published.public_visibility.notes_only.find_by(id: params[:id])

        if @post
          render json: @post
        else
          render json: { error: "Post not found or not public" }, status: :not_found
        end
      end

      private

      def set_profile
        @profile = Profile.find_by(username_id: params[:username_id])
        render json: { error: "Profile not found" }, status: :not_found unless @profile
      end
    end
  end
end
```

---

## 3️⃣ Configurare le Rotte API

Apri `config/routes.rb` e aggiungi le rotte per il namespace API:

```ruby
namespace :api do
  namespace :v1 do
    resources :profiles, param: :username_id, only: [] do
      resources :posts, only: [:index, :show], module: :profiles
    end
  end
end
```

---

## 4️⃣ Esempi di chiamate API

### 📌 Ottenere tutte le note pubbliche di un profilo
```http
GET /api/v1/profiles/:username_id/posts
```
**Esempio:** `/api/v1/profiles/123/posts`

---

### 📌 Ottenere una singola nota pubblica per ID
```http
GET /api/v1/profiles/:username_id/posts/:id
```
**Esempio:** `/api/v1/profiles/123/posts/456`

---

## 5️⃣ (Opzionale) Creare un Namespace API per Futuro Versioning

Se in futuro vuoi rilasciare **v2**, puoi semplicemente copiare il controller in `api/v2/`, modificare eventuali logiche, e aggiungere le nuove rotte senza impattare la versione `v1`.

---

## 💡 Vantaggi di questa Struttura
✅ Separazione pulita tra API e altre logiche dell'app.  
✅ Facile da scalare con nuove versioni (`v2`, `v3`, ecc.).  
✅ Organizzazione chiara con namespace e moduli.  

Se hai bisogno di aggiungere autenticazione (JWT, token, ecc.) o ottimizzare la risposta con **Jbuilder** o **Fast JSON API**, fammelo sapere! 🚀


rails g migration AddImageToContent image:text

rails g migration AddVideoUrlToContents video_url:text


```rb
def video_provider
  case video_url
  when /youtu\.?be/ then 'youtube'
  when /vimeo\.com/ then 'vimeo'
  when /wistia/ then 'wistia'
  when /mux/ then 'mux'
  else 'unknown'
  end
end

def video_id
  case video_provider
  when 'youtube'
    video_url[/((?<==)v=|(?<=be\/))[^&\n]+/]
  when 'vimeo'
    video_url[%r{vimeo\.com/(\d+)}, 1]
  when 'wistia'
    video_url[%r{wistia\.com/medias/([^?\/]+)}, 1]
  when 'mux'
    video_url[%r{stream\.mux\.com/([^?\/]+)}, 1]
  end
end

def embed_url
  return nil unless video_id
  case video_provider
  when 'youtube'
    "https://www.youtube.com/embed/#{video_id}"
  when 'vimeo'
    "https://player.vimeo.com/video/#{video_id}"
  when 'wistia'
    "https://fast.wistia.net/embed/iframe/#{video_id}"
  when 'mux'
    "https://stream.mux.com/#{video_id}.m3u8" # o .mp4 a seconda del player
  end
end
```
