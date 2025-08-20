---
title: Mobilità articolare semplificata
parent: Webapp app rails
layout: default
nav_order: 1
---

# Webapp Mobilità articolare semplificata
Quest'app fa parte del workflow utente dell'igiene posturale:

####  Piattaforma online

Workflow: utente_online 
- Brand: igiene posturale  
  - Landing Page
  - Profilazione
    - Patologie 
    - Stato Salute 
    - Funzionamento dei sistemi
      - [Valutazione Mobilità articolare semplificata](https://markpostura.it/profiles/markpostura/posts/test-mobilita-articolare-generale)
      - Igiene posturale
        - Programma 4 settimane
          - In autonomia
          - Con un professionista
            - In gruppo 
            - Singolo
          - Feedback 
          - Prosecuzione programma


# 1. Genera l'app
rails new evaluemobility --database=postgresql --css=tailwind --javascript=importmap


## Development
ruby# config/environments/development.rb
config.cache_store = :memory_store
config.active_job.queue_adapter = :solid_queue
config.action_cable.adapter = :solid_cable ?

## Production
ruby# config/environments/production.rb
config.cache_store = :solid_cache_store
config.active_job.queue_adapter = :solid_queue
config.action_cable.adapter = :solid_cable ?


# 2. Modifica il Gemfile (rimuovi redis, tieni solid_*)
cd evaluemobility

# 3. Bundle install
bundle install

# 4. Installa Solid gems
rails generate solid_cache:install
rails generate solid_queue:install  
rails generate solid_cable:install

# 5. Migra il database
rails db:migrate


# config/routes.rb
Rails.application.routes.draw do
  root "assessments#new"
  
  resources :assessments, only: [:new, :create, :show] do
    member do
      get :result
    end
  end
  
  get "up" => "rails/health#show", as: :rails_health_check
end


# Genera il controller per le valutazioni con le azioni necessarie
rails generate controller Assessments new create show result

rails generate controller Home index about contact

rails generate controller Pages privacy terms contact


# . Genera model per Assessment
rails generate model Assessment name:string age:integer email:string phone:string privacy_consent:boolean marketing_consent:boolean notes:text flexion_extension:integer arms_overhead:integer spine_rotation_right:integer spine_rotation_left:integer deep_squat:integer hands_behind_back_right:integer hands_behind_back_left:integer straight_leg_raise_right:integer straight_leg_raise_left:integer


# db/migrate/001_create_assessments.rb
class CreateAssessments < ActiveRecord::Migration[8.0]
  def change
    create_table :assessments do |t|
      t.string :name, null: false
      t.integer :age, null: false
      t.string :email
      t.string :phone
      t.boolean :privacy_consent, default: false
      t.boolean :marketing_consent, default: false
      t.text :notes
      
      # Exercise scores (0-3)
      t.integer :flexion_extension, null: false, default: 0
      t.integer :arms_overhead, null: false, default: 0
      t.integer :spine_rotation_right, null: false, default: 0
      t.integer :spine_rotation_left, null: false, default: 0
      t.integer :deep_squat, null: false, default: 0
      t.integer :hands_behind_back_right, null: false, default: 0
      t.integer :hands_behind_back_left, null: false, default: 0
      t.integer :straight_leg_raise_right, null: false, default: 0
      t.integer :straight_leg_raise_left, null: false, default: 0
      
      t.timestamps
    end
    
    add_index :assessments, :created_at
  end
end

# app/models/assessment.rb
class Assessment < ApplicationRecord
  validates :name, presence: true
  validates :age, presence: true, numericality: { greater_than: 0 }
  validates :email, format: { with: URI::MailTo::EMAIL_REGEXP }, allow_blank: true
  validates :phone, format: { with: /\A[\+]?[\d\s\-\(\)]{8,15}\z/ }, allow_blank: true
  validates :privacy_consent, acceptance: true
  
  # Scores for each exercise (0-3)
  validates :flexion_extension, presence: true, inclusion: { in: 0..3 }
  validates :arms_overhead, presence: true, inclusion: { in: 0..3 }
  validates :spine_rotation_right, presence: true, inclusion: { in: 0..3 }
  validates :spine_rotation_left, presence: true, inclusion: { in: 0..3 }
  validates :deep_squat, presence: true, inclusion: { in: 0..3 }
  validates :hands_behind_back_right, presence: true, inclusion: { in: 0..3 }
  validates :hands_behind_back_left, presence: true, inclusion: { in: 0..3 }
  validates :straight_leg_raise_right, presence: true, inclusion: { in: 0..3 }
  validates :straight_leg_raise_left, presence: true, inclusion: { in: 0..3 }
  
  def total_score
    flexion_extension + arms_overhead + spine_rotation_right + spine_rotation_left + 
    deep_squat + hands_behind_back_right + hands_behind_back_left + 
    straight_leg_raise_right + straight_leg_raise_left
  end
  
  def mobility_level
    case total_score
    when 0..9
      { level: "Limitata", color: "red", description: "Mobilità limitata → consigliata valutazione individuale" }
    when 10..18
      { level: "Media", color: "yellow", description: "Mobilità media → percorso di gruppo consigliato" }
    when 19..27
      { level: "Buona", color: "green", description: "Buona mobilità → accedi a percorsi avanzati o di mantenimento" }
    end
  end
  
  def color_class
    case mobility_level[:color]
    when "red"
      "bg-red-100 border-red-500 text-red-800"
    when "yellow"
      "bg-yellow-100 border-yellow-500 text-yellow-800"
    when "green"
      "bg-green-100 border-green-500 text-green-800"
    end
  end
end

# app/controllers/assessments_controller.rb
class AssessmentsController < ApplicationController
  before_action :set_assessment, only: [:show, :result]
  
  def new
    @assessment = Assessment.new
  end
  
  def create
    @assessment = Assessment.new(assessment_params)
    
    if @assessment.save
      redirect_to result_assessment_path(@assessment)
    else
      render :new, status: :unprocessable_entity
    end
  end
  
  def show
    redirect_to result_assessment_path(@assessment)
  end
  
  def result
    @mobility_data = @assessment.mobility_level
  end
  
  private
  
  def set_assessment
    @assessment = Assessment.find(params[:id])
  end
  
  def assessment_params
    params.require(:assessment).permit(
      :name, :age, :email, :phone, :privacy_consent, :marketing_consent, :notes,
      :flexion_extension, :arms_overhead, :spine_rotation_right, :spine_rotation_left,
      :deep_squat, :hands_behind_back_right, :hands_behind_back_left,
      :straight_leg_raise_right, :straight_leg_raise_left
    )
  end
end


# application.html.erb 

<body class="bg-gray-50 min-h-screen">
    <div class="container mx-auto px-4 py-8">
      <header class="text-center mb-8">
        <h1 class="text-4xl font-bold text-gray-800 mb-2">🧘♂️ Valutazione Mobilità Articolare</h1>
        <p class="text-gray-600">Metodo PosturaCorretta - Scheda di valutazione semplificata</p>
      </header>
      
      <main>
        <%= yield %>
      </main>
      
      <footer class="text-center mt-12 text-gray-500 text-sm">
        <p>&copy; 2024 IgienePosturale.it - Valutazione Mobilità Articolare</p>
      </footer>
    </div>
  </body>


# app/views/assessments/new.html.erb
<div class="max-w-4xl mx-auto bg-white rounded-lg shadow-lg p-6">
  <div class="mb-6">
    <h2 class="text-2xl font-semibold text-gray-800 mb-4">Test di Autovalutazione</h2>
    <p class="text-gray-600">
      Questo test ti aiuta a valutare la tua mobilità articolare generale in modo semplice e autonomo.
      Per ogni movimento, assegna un punteggio da 0 a 3 secondo i criteri riportati.
    </p>
    <div class="bg-yellow-100 border-l-4 border-yellow-500 p-4 mt-4">
      <p class="text-yellow-800">
        ⚠️ <strong>Importante:</strong> Esegui ogni movimento lentamente, senza forzare. 
        Se avverti dolore, interrompi il test.
      </p>
    </div>
  </div>

  <%= form_with model: @assessment, local: true, class: "space-y-6" do |form| %>
    <% if @assessment.errors.any? %>
      <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
        <h4 class="font-bold">Errori nel form:</h4>
        <ul class="list-disc list-inside">
          <% @assessment.errors.full_messages.each do |message| %>
            <li><%= message %></li>
          <% end %>
        </ul>
      </div>
    <% end %>

    <!-- Personal Info -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
      <div>
        <%= form.label :name, "Nome e Cognome *:", class: "block text-sm font-medium text-gray-700 mb-2" %>
        <%= form.text_field :name, required: true, class: "w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" %>
      </div>
      
      <div>
        <%= form.label :age, "Età *:", class: "block text-sm font-medium text-gray-700 mb-2" %>
        <%= form.number_field :age, required: true, min: 1, max: 120, class: "w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" %>
      </div>
      
      <div>
        <%= form.label :email, "Email:", class: "block text-sm font-medium text-gray-700 mb-2" %>
        <%= form.email_field :email, class: "w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500", placeholder: "esempio@email.com" %>
        <p class="text-xs text-gray-500 mt-1">Per ricevere i risultati e promemoria re-test</p>
      </div>
      
      <div>
        <%= form.label :phone, "Telefono:", class: "block text-sm font-medium text-gray-700 mb-2" %>
        <%= form.telephone_field :phone, class: "w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500", placeholder: "+39 123 456 7890" %>
        <p class="text-xs text-gray-500 mt-1">Per eventuali follow-up o chiarimenti</p>
      </div>
    </div>

    <!-- Privacy Consent -->
    <div class="bg-blue-50 border border-blue-200 rounded-lg p-4 mb-6">
      <h3 class="font-semibold text-blue-900 mb-3">📋 Consenso Privacy</h3>
      
      <div class="space-y-3">
        <label class="flex items-start space-x-3 cursor-pointer">
          <%= form.check_box :privacy_consent, required: true, class: "mt-1 text-blue-600" %>
          <span class="text-sm text-blue-800">
            <strong>Accetto*</strong> il trattamento dei miei dati personali per la valutazione posturale e l'invio dei risultati, 
            come da <a href="#" class="underline">informativa privacy</a>.
          </span>
        </label>
        
        <label class="flex items-start space-x-3 cursor-pointer">
          <%= form.check_box :marketing_consent, class: "mt-1 text-blue-600" %>
          <span class="text-sm text-blue-800">
            Accetto di ricevere comunicazioni su servizi di postura e benessere 
            (newsletter, promemoria, offerte personalizzate).
          </span>
        </label>
      </div>
    </div>

    <!-- Exercise 1: Flexion Extension -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">1. Flesso-estensione in avanti (da in piedi)</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "sopra ginocchia"],
            ["1", "tibie"], 
            ["2", "caviglie"],
            ["3", "mani a terra"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :flexion_extension, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 2: Arms Overhead -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">2. Braccia sopra la testa (in piedi)</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "non oltre spalle"],
            ["1", "con compensi"],
            ["2", "vicino orecchie"],
            ["3", "allineate senza compensi"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :arms_overhead, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 3A: Spine Rotation Right -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">3A. Rotazione globale del rachide (collo + tronco) - Destra</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 g[48;46;193;1472;2702tap-6">
        <div class="space-y-2">
          <% [
            ["0", "limitata"],
            ["1", "ruoto 90°"],
            ["2", "quasi dietro 135°"],
            ["3", "ruoto completamente dietro 180°"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :spine_rotation_right, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 3B: Spine Rotation Left -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">3B. Rotazione globale del rachide (collo + tronco) - Sinistra</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "limitata"],
            ["1", "ruoto 90°"],
            ["2", "quasi dietro 135°"],
            ["3", "ruoto completamente dietro 180°"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :spine_rotation_left, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 4: Deep Squat -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">4. Squat profondo (a corpo libero)</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "non scende"],
            ["1", "instabile"],
            ["2", "quasi fondo"],
            ["3", "fondo stabile"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :deep_squat, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 5A: Hands Behind Back Right -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">5A. Mani dietro la schiena (destra sopra)</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", ">20 cm"],
            ["1", "10–20 cm"],
            ["2", "5–10 cm"],
            ["3", "si toccano"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :hands_behind_back_right, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 5B: Hands Behind Back Left -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">5B. Mani dietro la schiena (sinistra sopra)</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", ">20 cm"],
            ["1", "10–20 cm"],
            ["2", "5–10 cm"],
            ["3", "si toccano"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :hands_behind_back_left, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 6A: Straight Leg Raise Right -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">6A. Sollevamento gamba tesa (supino) - Destra</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "<45°"],
            ["1", "45–75°"],
            ["2", "75–89°"],
            ["3", "≥90° senza dolore"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :straight_leg_raise_right, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Exercise 6B: Straight Leg Raise Left -->
    <div class="border border-gray-200 rounded-lg p-6">
      <h3 class="text-lg font-semibold mb-4">6B. Sollevamento gamba tesa (supino) - Sinistra</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-2">
          <% [
            ["0", "<45°"],
            ["1", "45–75°"],
            ["2", "75–89°"],
            ["3", "≥90° senza dolore"]
          ].each do |value, description| %>
            <label class="flex items-center space-x-2 cursor-pointer">
              <%= form.radio_button :straight_leg_raise_left, value, class: "text-blue-600" %>
              <span class="text-sm"><strong><%= value %>:</strong> <%= description %></span>
            </label>
          <% end %>
        </div>
        <div class="text-center">
          <div class="w-32 h-32 bg-gray-200 rounded-lg flex items-center justify-center mx-auto">
            <span class="text-gray-500">📸 Immagine esercizio</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Notes -->
    <div>
      <%= form.label :notes, "Note aggiuntive:", class: "block text-sm font-medium text-gray-700 mb-2" %>
      <%= form.text_area :notes, rows: 3, class: "w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500", placeholder: "Eventuali osservazioni o difficoltà riscontrate..." %>
    </div>

    <!-- Submit -->
    <div class="text-center">
      <%= form.submit "Calcola Risultato", class: "bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-8 rounded-lg transition duration-200" %>
    </div>
  <% end %>
</div>


 app/views/assessments/result.html.erb
<div class="max-w-3xl mx-auto space-y-6">
  <!-- Results Card -->
  <div class="bg-white rounded-lg shadow-lg p-8">
    <div class="text-center mb-8">
      <h2 class="text-3xl font-bold text-gray-800 mb-2">Risultati del Test</h2>
      <div class="space-y-1">
        <p class="text-gray-600">Valutazione per: <strong><%= @assessment.name %></strong> (<%= @assessment.age %> anni)</p>
        <% if @assessment.email.present? %>
          <p class="text-sm text-gray-500">📧 <%= @assessment.email %></p>
        <% end %>
        <% if @assessment.phone.present? %>
          <p class="text-sm text-gray-500">📱 <%= @assessment.phone %></p>
        <% end %>
        <p class="text-sm text-gray-500 mt-2">Test completato il <%= @assessment.created_at.strftime("%d/%m/%Y alle %H:%M") %></p>
      </div>
    </div>

    <!-- Score Display -->
    <div class="text-center mb-8">
      <div class="inline-block">
        <div class="text-6xl font-bold text-blue-600 mb-2"><%= @assessment.total_score %></div>
        <div class="text-lg text-gray-600">su 27 punti</div>
      </div>
    </div>

    <!-- Mobility Level -->
    <div class="<%= @assessment.color_class %> border-2 rounded-lg p-6 mb-8">
      <div class="text-center">
        <div class="text-2xl font-bold mb-2">
          <% case @mobility_data[:color] %>
          <% when "red" %>
            🔴 Mobilità <%= @mobility_data[:level] %>
          <% when "yellow" %>
            🟡 Mobilità <%= @mobility_data[:level] %>
          <% when "green" %>
            🟢 Mobilità <%= @mobility_data[:level] %>
          <% end %>
        </div>
        <p class="text-lg"><%= @mobility_data[:description] %></p>
      </div>
    </div>

    <!-- Detailed Scores -->
    <div class="bg-gray-50 rounded-lg p-6 mb-8">
      <h3 class="text-xl font-semibold mb-4 text-center">Dettaglio Punteggi</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-sm">
        <div class="space-y-2">
          <div class="flex justify-between">
            <span>1. Flesso-estensione in avanti:</span>
            <span class="font-semibold"><%= @assessment.flexion_extension %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>2. Braccia sopra la testa:</span>
            <span class="font-semibold"><%= @assessment.arms_overhead %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>3A. Rotazione rachide (dx):</span>
            <span class="font-semibold"><%= @assessment.spine_rotation_right %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>3B. Rotazione rachide (sx):</span>
            <span class="font-semibold"><%= @assessment.spine_rotation_left %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>4. Squat profondo:</span>
            <span class="font-semibold"><%= @assessment.deep_squat %>/3</span>
          </div>
        </div>
        <div class="space-y-2">
          <div class="flex justify-between">
            <span>5A. Mani dietro schiena (dx sopra):</span>
            <span class="font-semibold"><%= @assessment.hands_behind_back_right %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>5B. Mani dietro schiena (sx sopra):</span>
            <span class="font-semibold"><%= @assessment.hands_behind_back_left %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>6A. Sollevamento gamba (dx):</span>
            <span class="font-semibold"><%= @assessment.straight_leg_raise_right %>/3</span>
          </div>
          <div class="flex justify-between">
            <span>6B. Sollevamento gamba (sx):</span>
            <span class="font-semibold"><%= @assessment.straight_leg_raise_left %>/3</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Notes -->
    <% if @assessment.notes.present? %>
      <div class="bg-blue-50 rounded-lg p-4 mb-6">
        <h4 class="font-semibold text-blue-900 mb-2">Note:</h4>
        <p class="text-blue-800"><%= simple_format(@assessment.notes) %></p>
      </div>
    <% end %>

    <!-- Recommendations -->
    <div class="bg-white border border-gray-200 rounded-lg p-6">
      <h3 class="text-xl font-semibold mb-4">📌 Raccomandazioni</h3>
      <div class="space-y-3 text-gray-700">
        <p>• Ripeti il test ogni 4 settimane per monitorare i tuoi miglioramenti</p>
        <p>• Se avverti dolore durante gli esercizi, consulta un professionista</p>
        <% case @mobility_data[:color] %>
        <% when "red" %>
          <div class="bg-red-50 border-l-4 border-red-500 p-4 mt-4">
            <p class="text-red-800">
              <strong>Consiglio:</strong> Ti consigliamo una valutazione individuale con un professionista qualificato 
              per sviluppare un piano di miglioramento personalizzato.
            </p>
          </div>
        <% when "yellow" %>
          <div class="bg-yellow-50 border-l-4 border-yellow-500 p-4 mt-4">
            <p class="text-yellow-800">
              <strong>Consiglio:</strong> Un percorso di gruppo potrebbe aiutarti a migliorare la tua mobilità articolare. 
              Considera di partecipare a corsi di ginnastica posturale o stretching.
            </p>
          </div>
        <% when "green" %>
          <div class="bg-green-50 border-l-4 border-green-500 p-4 mt-4">
            <p class="text-green-800">
              <strong>Complimenti!</strong> Hai una buona mobilità articolare. 
              Puoi accedere a percorsi avanzati o di mantenimento per preservare questo livello.
            </p>
          </div>
        <% end %>
      </div>
    </div>

    <!-- Actions -->
    <div class="flex flex-col sm:flex-row gap-4 justify-center mt-8">
      <%= link_to "Nuovo Test", root_path, class: "bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg text-center transition duration-200" %>
      <button onclick="window.print()" class="bg-gray-600 hover:bg-gray-700 text-white font-bold py-3 px-6 rounded-lg transition duration-200">
        🖨️ Stampa Risultati
      </button>
      <button onclick="shareResults()" class="bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg transition duration-200">
        📤 Condividi
      </button>
    </div>
  </div>

  <!-- Historical Progress (Future Feature) -->
  <div class="bg-white rounded-lg shadow-lg p-6">
    <h3 class="text-xl font-semibold mb-4">📈 Progressi nel Tempo</h3>
    <div class="text-center text-gray-500 py-8">
      <p>Funzionalità in arrivo: qui vedrai l'evoluzione dei tuoi punteggi nel tempo</p>
      <div class="mt-4 bg-gray-100 rounded-lg p-4">
        <div class="w-full h-32 bg-gray-200 rounded flex items-center justify-center">
          <span class="text-gray-400">Grafico dei progressi</span>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
  function shareResults() {
    const text = `Ho completato il test di valutazione mobilità articolare!\n\nRisultato: <%= @assessment.total_score %>/27 punti\nLivello: <%= @mobility_data[:level] %>\n\n<%= @mobility_data[:description] %>\n\nTest su: valutazione.igieneposturale.it`;
    
    if (navigator.share) {
      navigator.share({
        title: 'I miei risultati del test di mobilità articolare',
        text: text,
        url: window.location.href
      });
    } else {
      navigator.clipboard.writeText(text).then(() => {
        alert('Risultati copiati negli appunti!');
      });
    }
  }
</script>
# config/application.rb
require_relative "boot"

require "rails/all"

Bundler.require(*Rails.groups)

module MobilityAssessment
  class Application < Rails::Application
    config.load_defaults 8.0
    config.autoload_lib(ignore: %w[assets tasks])
    
    # Time zone
    config.time_zone = 'Rome'
    
    # Locale
    config.i18n.default_locale = :it
  end
end

# app/assets/stylesheets/application.tailwind.css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  }
}

@layer components {
  .btn-primary {
    @apply bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition duration-200;
  }
  
  .card {
    @apply bg-white rounded-lg shadow-lg p-6;
  }
  
  .form-input {
    @apply w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500;
  }
}

@media print {
  .no-print {
    display: none;
  }
  
  body {
    background: white;
  }
  
  .shadow-lg {
    box-shadow: none;
    border: 1px solid #e5e7eb;
  }
}

# prossimi miglioramenti


# config/environments/production.rb
require "active_support/core_ext/integer/time"

Rails.application.configure do
  config.enable_reloading = false
  config.eager_load = true
  config.consider_all_requests_local = false
  config.action_controller.perform_caching = true
  
  config.public_file_server.enabled = true
  config.public_file_server.headers = { "cache-control" => "public, max-age=#{1.year.to_i}" }
  
  config.cache_store = :solid_cache_store
  config.active_job.queue_adapter = :solid_queue
  config.action_cable.adapter = :solid_cable
  
  config.force_ssl = true
  config.ssl_options = { redirect: { exclude: ->(request) { request.path.start_with?("/up") } } }
  
  config.assume_ssl = true
  config.log_level = ENV.fetch("RAILS_LOG_LEVEL", "info")
  config.log_tags = [:request_id]
  
  config.action_mailer.perform_caching = false
  config.action_mailer.default_url_options = { 
    host: ENV.fetch("APP_HOST") { "valutazione.igieneposturale.it" },
    protocol: "https"
  }
  
  config.i18n.fallbacks = true
  config.active_support.report_deprecations = false
  config.active_record.dump_schema_after_migration = false
end 


# config/database.yml
production:
  primary:
    <<: *default
    database: <%= ENV.fetch("DB_PRIMARY") %>
    username: <%= ENV.fetch("DB_PRIMARY_USER") %>
    password: <%= ENV.fetch("DB_PRIMARY_PASS") %>
    host:     <%= ENV.fetch("DB_PRIMARY_HOST") %>

  queue:
    <<: *default
    database: <%= ENV.fetch("DB_QUEUE") %>
    username: <%= ENV.fetch("DB_QUEUE_USER") %>
    password: <%= ENV.fetch("DB_QUEUE_PASS") %>
    host:     <%= ENV.fetch("DB_QUEUE_HOST") %>

  cache:
    <<: *default
    database: <%= ENV.fetch("DB_CACHE") %>
    username: <%= ENV.fetch("DB_CACHE_USER") %>
    password: <%= ENV.fetch("DB_CACHE_PASS") %>
    host:     <%= ENV.fetch("DB_CACHE_HOST") %>

  cable:
    <<: *default
    database: <%= ENV.fetch("DB_CABLE") %>
    username: <%= ENV.fetch("DB_CABLE_USER") %>
    password: <%= ENV.fetch("DB_CABLE_PASS") %>
    host:     <%= ENV.fetch("DB_CABLE_HOST") %>
Poi nei initializer generati dai generator (o che puoi creare tu) indichi quale connessione usare:

ruby
Copia
Modifica
# config/initializers/solid_queue.rb
SolidQueue.configure do |c|
  c.database = :queue  # usa la chiave 'queue' di database.yml per l'env corrente
end

# config/initializers/solid_cache.rb
SolidCache.configure do |c|
  c.database = :cache
end

# config/initializers/solid_cable.rb
SolidCable.configure do |c|
  c.database = :cable
end

rails generate controller Admin::Dashboard index assessments users




```bash
# Setup per produzione
RAILS_ENV=production rails db:create
RAILS_ENV=production rails db:migrate
RAILS_ENV=production rails assets:precompile

# Variabili ambiente richieste:
export DATABASE_NAME=mobility_assessment_production
export DATABASE_USERNAME=your_db_user
export DATABASE_PASSWORD=your_db_password
export DATABASE_HOST=your_db_host
export APP_HOST=valutazione.igieneposturale.it
export RAILS_MASTER_KEY=your_master_key
```

## Struttura del Test

### Esercizi Valutati
1. **Flesso-estensione in avanti** (da in piedi)
2. **Braccia sopra la testa** (in piedi)
3. **Rotazione globale del rachide** (destra e sinistra)
4. **Squat profondo** (a corpo libero)
5. **Mani dietro la schiena** (destra e sinistra sopra)
6. **Sollevamento gamba tesa** (supino, destra e sinistra)

### Sistema di Punteggio
- Ogni esercizio: 0-3 punti
- Totale massimo: 27 punti
- Classificazione:
  - 🔴 **0-9 punti**: Mobilità limitata
  - 🟡 **10-18 punti**: Mobilità media  
  - 🟢 **19-27 punti**: Buona mobilità

## Tecnologie Utilizzate

- **Rails 8.0**: Framework principale
- **TailwindCSS**: Styling e responsive design
- **PostgreSQL**: Database
- **Turbo & Stimulus**: Interattività frontend
- **Solid Cache/Queue/Cable**: Performance e background jobs

## TODO per Miglioramenti Futuri

- [ ] Upload e gestione immagini per ogni esercizio
- [ ] Sistema di autenticazione utenti
- [ ] Storico progressi e grafici
- [ ] Export PDF dei risultati
- [ ] API per integrazione con altre app
- [ ] Sistema di promemoria per test periodici
- [ ] Integrazione video esplicativi per ogni esercizio

## Licenza

Sviluppato per IgienePosturale.it - 2024
