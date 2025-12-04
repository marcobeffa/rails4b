---
title: Domains
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 09
---


------
ci sarebbero gate posts oppure al posto dei gate
--> blog e corsi 

-------

# Domains
Alla taxonomy  dovrei aggiungere i - domains perchè così quando c'è un dominio posso dirottarlo direttamente  - userei il domains come porta d'accesso ai rami taxbranches
 perciò:  
rails g scaffold Domain 
primary ( il dominio principale ) solo il nome es posturacorretta.org 
manage_language:jsonb  (ex: quando ci saranno le lingue { posturacorretta.org: it, goodposture.com: eng } )
taxbranch:references (il text branch di ingresso)
active:boolean

e poi 
posturacorretta.org/taxbranches/ in base alla request host parte poi dalla taxbranches che corrisponde al dominio

che ne dici? 

----
posturacorretta.org/taxbranches/ in base alla request host parte poi dalla taxbranches che corrisponde al dominio
poi   posts per creare - blog contenuti - corsi online - meglio creare dei nuovi modelli? 

---


✅ Un unico albero Taxbranch
✅ Ogni dominio (Domain) punta a un taxbranch root
✅ Current.domain e Current.taxbranch impostati da request.host
✅ I domini gestibili solo dal superadmin
✅ Funziona con Hatchbox + Redis cache + multi-domain HTTPS

1️⃣ Generatore base
bin/rails g scaffold Domain \
  host:string:uniq \
  language:string \
  title:string \
  description:text \
  favicon_url:string \
  square_logo_url:string \
  horizontal_logo_url:string \
  provider:string \
  taxbranch:references

bin/rails db:migrate


2️⃣ Modello Domain
# app/models/domain.rb
class Domain < ApplicationRecord
  belongs_to :taxbranch

  validates :host, presence: true, uniqueness: { case_sensitive: false }
  validates :language, presence: true
  validate  :taxbranch_must_be_root

  before_validation :normalize_host!

  private

  def normalize_host!
    return if host.blank?
    h = host.to_s.strip.downcase
    h = h.sub(/\Ahttps?:\/\//, "")
    h = h.sub(/\Awww\./, "")
    h = h.split(":").first
    self.host = h
  end

  def taxbranch_must_be_root
    errors.add(:taxbranch, "deve essere un nodo radice") if taxbranch&.ancestry.present?
  end
end


3️⃣ Current + Resolver
# app/models/current.rb
class Current < ActiveSupport::CurrentAttributes
  attribute :session, :domain, :taxbranch
  delegate :user, to: :session, allow_nil: true
end

# app/services/domain_resolver.rb
class DomainResolver
  TTL = 10.minutes

  class << self
    def resolve(host)
      key = "domain:#{normalize(host)}"
      Rails.cache.fetch(key, expires_in: TTL) do
        Domain.includes(:taxbranch).find_by(host: normalize(host))
      end
    end

    def normalize(host)
      host.to_s.downcase.sub(/\Ahttps?:\/\//, "").sub(/\Awww\./, "").split(":").first
    end
  end
end


4️⃣ ApplicationController
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :set_current_domain

  private

  def set_current_domain
    dom = DomainResolver.resolve(request.host)
    if dom
      Current.domain    = dom
      Current.taxbranch = dom.taxbranch
    else
      # opzionale: fallback, redirect o 404
      # redirect_to("https://posturacorretta.org", allow_other_host: true)
    end
  end
end

5️⃣ Accesso riservato ai superadmin
🧩 Concern
# app/controllers/concerns/require_superadmin.rb
module RequireSuperadmin
  extend ActiveSupport::Concern

  included do
    before_action :require_superadmin!
  end

  private

  def require_superadmin!
    unless Current.user&.superadmin?
      redirect_to root_path, alert: "Non autorizzato."
    end
  end
end


6️⃣ Controller “Domain” dentro Superadmin

Spostiamo il CRUD in un namespace protetto:

mkdir -p app/controllers/superadmin
mv app/controllers/domains_controller.rb app/controllers/superadmin/domains_controller.rb

🔧 Modifica controller
# app/controllers/superadmin/domains_controller.rb
module Superadmin
  class DomainsController < ApplicationController
    include RequireSuperadmin
    before_action :set_domain, only: %i[show edit update destroy]

    def index
      @domains = Domain.includes(:taxbranch).order(:host)
    end

    def show; end
    def new
      @domain = Domain.new
    end

    def edit; end

    def create
      @domain = Domain.new(domain_params)
      if @domain.save
        redirect_to superadmin_domains_path, notice: "Dominio creato."
      else
        render :new, status: :unprocessable_entity
      end
    end

    def update
      if @domain.update(domain_params)
        redirect_to superadmin_domains_path, notice: "Dominio aggiornato."
      else
        render :edit, status: :unprocessable_entity
      end
    end

    def destroy
      @domain.destroy
      redirect_to superadmin_domains_path, notice: "Dominio eliminato."
    end

    private

    def set_domain
      @domain = Domain.find(params[:id])
    end

    def domain_params
      params.require(:domain).permit(:host, :language, :title, :description,
                                     :favicon_url, :square_logo_url, :horizontal_logo_url,
                                     :provider, :taxbranch_id)
    end
  end
end

7️⃣ Routing
# config/routes.rb
namespace :superadmin do
  resources :domains
  resources :taxbranches
  root "domains#index"
end

8️⃣ Vista Tailwind semplice
<!-- app/views/superadmin/domains/index.html.erb -->
<h1 class="text-2xl font-bold mb-4">Domini configurati</h1>

<%= link_to "➕ Nuovo dominio", new_superadmin_domain_path,
      class: "px-3 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700" %>

<table class="mt-4 w-full border text-sm">
  <thead class="bg-slate-100">
    <tr>
      <th class="p-2 text-left">Host</th>
      <th class="p-2">Lingua</th>
      <th class="p-2">Taxbranch</th>
      <th class="p-2">Titolo</th>
      <th class="p-2"></th>
    </tr>
  </thead>
  <tbody>
    <% @domains.each do |d| %>
      <tr class="border-t">
        <td class="p-2 font-medium"><%= d.host %></td>
        <td class="p-2"><%= d.language %></td>
        <td class="p-2"><%= d.taxbranch.slug_label %></td>
        <td class="p-2"><%= truncate(d.title, length: 40) %></td>
        <td class="p-2 text-right">
          <%= link_to "✏️", edit_superadmin_domain_path(d), class: "text-blue-600" %> |
          <%= link_to "🗑️", superadmin_domain_path(d),
                data: { turbo_confirm: "Eliminare questo dominio?", turbo_method: :delete },
                class: "text-red-600" %>
        </td>
      </tr>
    <% end %>
  </tbody>
</table>

9️⃣ Hatchbox: configurazione produzione

Aggiungi i domini alla sezione “Domains” di Hatchbox

Attiva Let’s Encrypt

In config/environments/production.rb:

config.hosts += %w[
  posturacorretta.org
  igieneposturale.it
]
config.force_ssl = true
config.cache_store = :redis_cache_store, { url: ENV["REDIS_URL"] }


Nel deploy script Hatchbox, dopo db:migrate aggiungi:

bundle exec rake domain:warm

🔟 Warmup cache task
# lib/tasks/domain_cache.rake
namespace :domain do
  desc "Precarica la cache domini"
  task warm: :environment do
    Domain.find_each { |d| DomainResolver.resolve(d.host) }
    puts "Cache domini precaricata."
  end
end


✅ In sintesi
Componente	Ruolo
Domain	definisce host, lingua, branding, taxbranch radice
DomainResolver	risolve da request.host (cache TTL 10m)
Current.domain	disponibile ovunque
superadmin/domains_controller.rb	CRUD riservato
RequireSuperadmin	blocca accesso ai non admin
Hatchbox	gestisce domini HTTPS e deploy
domain:warm	precarica la cache


bin/rails solid_cache:install
bin/rails db:migrate

bin/rails solid_cache:prune

bundle exec rails solid_cache:prune 
oppure aggiungi una rake task cron settimanale in Hatchbox:
-- (tenere pulita la cache )
------


  # app/controllers/pages_controller.rb
class PagesController < ApplicationController
  def home
    @taxbranch = Current.taxbranch || Taxbranch.where(ancestry: nil).ordered.first
    return render file: Rails.root.join("public/404.html"), status: :not_found, layout: false unless @taxbranch

    # figli diretti da mostrare in home
    @children = @taxbranch.children.ordered

    # Esempio: cache per host + taxbranch
    # cache_key = ["home", Current.domain&.host, @taxbranch.cache_key_with_version]
    # Rails.cache.fetch(cache_key, expires_in: 10.minutes) { @children.to_a }
  end
end



# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  before_action :set_current_domain
  before_action :set_locale_from_domain

  private

  def set_current_domain
    dom = DomainResolver.resolve(request.host)
    if dom
      Current.domain    = dom
      Current.taxbranch = dom.taxbranch
    end
  end

  def set_locale_from_domain
    I18n.locale = Current.domain&.language.presence || I18n.default_locale
  end
end


# app/helpers/domain_helper.rb
module DomainHelper
  def domain_title(default: "PosturaCorretta")
    Current.domain&.title.presence || default
  end

  def domain_description(default: "")
    Current.domain&.description.presence || default
  end

  def domain_favicon_url
    Current.domain&.favicon_url
  end

  def domain_logo_square
    Current.domain&.square_logo_url
  end

  def domain_logo_horizontal
    Current.domain&.horizontal_logo_url
  end
end
E nelle layout:

erb
Copia codice
<!-- app/views/layouts/application.html.erb (head) -->
<title><%= domain_title %></title>
<% if domain_description.present? %>
  <meta name="description" content="<%= domain_description %>">
<% end %>
<% if domain_favicon_url.present? %>
  <link rel="icon" href="<%= domain_favicon_url %>">
<% end %>






🗓️ Obiettivi 


🧠 1. Routine e organizzazione personale

Rimettere giù tutte le routine quotidiane e settimanali:
Sistemare tutto il calendario e chiudere i cerchi
Definire giorni e slot precisi 

🩺 Salute
💼 Lavoro
📚 Formazione
🌿 Tempo libero


🌐 2. Servizi e contenuti

Avanzare con la parte online:
🔸 Accesso ai contenuti (blog youtube instagram) e corsi online
🔸 Servizi online e dal vivo pronti
🔸 Revisione obiettivi, calendario e piano produzione contenuti

💰 3. Contabilità

Sistemare tutta la parte contabile
Preparare documenti e note per il commercialista (altrimenti mi sgrida 😅)


📅 4. Calendario Novembre–Dicembre

- Presentazioni ai professionisti: 
🔹 Materiale igiene posturale
🔹 Elenco professionisti da contattare
🔹 Appuntamenti di presentazione 


🚗 5. Spostamenti e sopralluoghi

Pianificare giri tra Costa Valle Imagna e Torre de Busi
→ valutare punti di appoggio o collaborazioni 
→ Vedere per una base operativa che faccia anche da casa
→ Vedere  punti di appoggio (bar agriturismi professionisti salute benessere)

🎓 6. Corsi e materiali PosturaCorretta

Completare:

🧍‍♂️ PosturaCorretta alla scrivania

👁️ Il tuo punto di vista sulla salute

🧘 Igiene Posturale (corso online autonomia + bozza insegnanti)

🧩 7. Taxonomia

Sistemare e completare la struttura logica di accesso ai servizi e alle informazioni

💌 8. Posticipare appuntamenti
   Cantù centro fecondità (almeno da novembre → gennaio a ) 
   perchè prima non riesco...



🏡 9.Ponna Festeggiare
Andare a trovare Monica a Ponna e darvi i baci! 😘


p.s. se vuoi aiutarmi o ci sono aiutanti ❤️