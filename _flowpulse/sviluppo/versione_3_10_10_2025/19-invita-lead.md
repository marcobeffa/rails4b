---
title: Invita lead
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 19
---


# Invita lead

ma per ora non posso tenerlo così? tolgo solo il form... e mi generi il link per il referral e non puoi invitare più di 7 persone? e l'approve lo fa il tutor? e se non sei approved il link referral non funziona?


Sì, puoi tenerlo così e semplificare: togli il form, mostri solo il tuo link referral, metti limite 7 invitati, approva il Tutor, e fai sì che il link funzioni solo se l’invitante è “approved” e ha ancora slot disponibili.

Ti lascio tutto pronto (migrazioni + model + controller + view). Copia/incolla e vai.

1) Migrazione (limite inviti sul referrer)

rails g migration AddInvitesToUsers invites_count:integer invites_limit:integer referrer_id:integer
# db/migrate/20251110_add_invites_to_users.rb
class AddInvitesToUsers < ActiveRecord::Migration[7.2]
  def change
    add_column :users, :invites_count, :integer, default: 0, null: false
    add_column :users, :invites_limit, :integer, default: 7, null: false
    add_column :users, :referrer_id, :bigint
    add_index  :users, :referrer_id
  end
end


Se referrer_id è già sul modello User, rimuovi quella riga.

2) User model (token referral, gating, contatore)
# app/models/user.rb
class User < ApplicationRecord
  belongs_to :referrer, class_name: "User", optional: true

  # Sei “approved” per invitare? (adatta alla tua logica: ruolo Tutor/TeamManager o Lead approvato)
  def approved_referrer?
    has_role?(:tutor) || has_role?(:team_manager) || lead&.approved?
  rescue
    false
  end

  def can_invite?
    approved_referrer? && invites_count < invites_limit
  end

  def remaining_invites
    [invites_limit - invites_count, 0].max
  end

  # Link referral firmato a scadenza (rilancia quando vuoi)
  def referral_token(expires_in: 14.days)
    return nil unless approved_referrer? && can_invite?
    signed_id(purpose: :referral, expires_in: expires_in)
  end

  def referral_url
    tok = referral_token
    tok ? Rails.application.routes.url_helpers.new_registration_url(ref: tok) : nil
  end

  # Conta un invito andato a buon fine
  def count_successful_invite!
    increment!(:invites_count)
  end
end

3) RegistrationsController (accetta ref firmato, applica limiti, fallback legacy)

Manteniamo compatibilità col tuo referral_code (username) ma preferiamo ref firmato.

# app/controllers/registrations_controller.rb
class RegistrationsController < ApplicationController
  allow_unauthenticated_access only: %i[new create]
  layout "posts", only: %i[new create]

  def new
    @referrer = locate_referrer(params[:ref], params.dig(:lead, :referral_code))
    @user = User.new
    if params[:ref].present? && @referrer.nil?
      flash.now[:alert] = "Link invito non valido, scaduto o referrer non abilitato."
    elsif @referrer && !@referrer.can_invite?
      flash.now[:alert] = "Questo invito non è più disponibile."
      @referrer = nil
    end
  end

  def create
    # preferisci token firmato se presente
    referrer = locate_referrer(params[:ref], signup_params[:referral_code])

    email    = signup_params[:email].to_s.strip
    password = signup_params[:password]

    provisional_username = generate_username_from(email)

    # Se il referrer non è valido/abilitato, ignoralo
    referrer = nil unless referrer&.can_invite?

    flow = LeadSignupFlow.new.call!(
      lead_params: { email: email, username: provisional_username },
      password: password,
      referral_lead_id: nil,      # non più da username, usiamo referrer se c'è
      auto_approve: false         # il Tutor approva
    )

    # collega il referrer (se valido)
    if referrer
      flow.user.update!(referrer_id: referrer.id)
    end

    start_new_session_for(flow.user)

    # conteggia solo se c'è referrer valido
    referrer&.count_successful_invite!

    redirect_to after_authentication_url, notice: "Benvenuto! Account creato."
  rescue ActiveRecord::RecordInvalid => e
    @lead = Lead.new(signup_params)
    flash.now[:alert] = e.record.errors.full_messages.to_sentence
    render "sessions/new", status: :unprocessable_entity
  end

  def edit
    @user = Current.user
    @lead = @user.lead || Lead.find_by(email: @user.email_address) || Lead.new(email: @user.email_address, username: default_username(@user))
  end

  def update
    @user = Current.user
    @lead = @user.lead || Lead.find_by(email: @user.email_address) || Lead.new(email: @user.email_address, username: default_username(@user))

    ActiveRecord::Base.transaction do
      @user.update!(user_params)
      if @lead.new_record?
        @lead.assign_attributes(lead_params)
        @lead.user_id ||= @user.id if @lead.respond_to?(:user_id)
        @lead.save!
      else
        @lead.update!(lead_params)
      end
    end

    redirect_to after_authentication_url, notice: "Profilo aggiornato!"
  rescue ActiveRecord::RecordInvalid => e
    flash.now[:alert] = e.record.errors.full_messages.to_sentence
    render :edit, status: :unprocessable_entity
  end

  private

  def locate_referrer(signed_ref, legacy_ref_code)
    # 1) token firmato
    if signed_ref.present?
      user = User.find_signed(signed_ref, purpose: :referral) rescue nil
      return (user if user&.approved_referrer?)
    end

    # 2) fallback legacy: username nel Lead (come prima)
    if legacy_ref_code.present?
      lead = Lead.find_by(username: legacy_ref_code)
      user = lead&.user
      return (user if user&.approved_referrer?)
    end

    nil
  end

  def user_params
    params.require(:user).permit(:name, :surname, :phone, :email_address, :password, :password_confirmation)
  end

  def lead_params
    params.fetch(:lead, {}).permit(:username, :phone, :notes)
  end

  def default_username(user)
    user.email_address.to_s.split("@").first.to_s.parameterize.presence || "utente"
  end

  def signup_params
    params.require(:lead).permit(:email, :password, :referral_code)
  end

  def generate_username_from(email)
    base = email.to_s.split("@").first.to_s.parameterize.presence || "user"
    uname = base
    i = 1
    while Lead.exists?(username: uname)
      i += 1
      uname = "#{base}-#{i}"
      break if i > 1000
    end
    uname
  end
end

4) Solo il Tutor approva

Se usi già l’approve sui Lead, tieni questo guard-rail:

# app/controllers/account/leads_controller.rb
class Account::LeadsController < ApplicationController
  before_action :require_tutor!

  def approve
    lead = Lead.find(params[:id])
    # ... la tua logica di approve (crea user o setta stato) ...
    redirect_to account_leads_path, notice: "Lead approvato."
  end

  private

  def require_tutor!
    unless Current.user&.has_role?(:tutor) || Current.user&.has_role?(:team_manager)
      redirect_to account_leads_path, alert: "Non autorizzato."
    end
  end
end

5) View “I miei invitati” (niente form; mostra link + quota)

Sostituisci il blocco del form con questo:

<% content_for :title, "I miei invitati" %>

<div class="mx-auto max-w-3xl">
  <h1 class="text-2xl font-bold mb-4">I miei invitati</h1>

  <% if flash[:notice] %>
    <p class="py-2 px-3 bg-green-50 mb-5 text-green-600 font-medium rounded-lg inline-block"><%= flash[:notice] %></p>
  <% end %>
  <% if flash[:alert] %>
    <p class="py-2 px-3 bg-red-50 mb-5 text-red-600 font-medium rounded-lg inline-block"><%= flash[:alert] %></p>
  <% end %>

  <div class="mb-6 rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
    <h2 class="text-lg font-semibold mb-3">Il tuo link referral</h2>

    <% if Current.user.approved_referrer? %>
      <% if Current.user.can_invite? %>
        <% link = Current.user.referral_url %>
        <div class="flex items-center gap-2">
          <input type="text" readonly value="<%= link %>"
                 class="w-full rounded-md border border-slate-300 px-3 py-2">
          <button data-action="click->clipboard#copy"
                  data-clipboard-text-value="<%= link %>"
                  class="rounded-md bg-blue-600 hover:bg-blue-500 text-white px-4 py-2.5 font-medium">
            Copia
          </button>
        </div>
        <p class="mt-2 text-xs text-slate-500">
          Inviti rimasti: <%= Current.user.remaining_invites %> su <%= Current.user.invites_limit %>.
          Il link scade tra 14 giorni.
        </p>
      <% else %>
        <p class="text-sm text-slate-600">Hai raggiunto il limite massimo di inviti ( <%= Current.user.invites_limit %> ).</p>
      <% end %>
    <% else %>
      <p class="text-sm text-slate-600">Il referral è attivo solo per utenti approvati. Contatta un Tutor per l’abilitazione.</p>
    <% end %>
  </div>

  <div class="rounded-xl border border-slate-200 bg-white p-4 shadow-sm">
    <h2 class="text-lg font-semibold mb-3">Invitati (referral)</h2>
    <!-- qui mantieni la tua tabella @invited_leads come già ce l'hai -->
    <table class="w-full text-sm">
      <thead class="text-left text-slate-500">
        <tr>
          <th class="py-2">Email</th>
          <th class="py-2">Username</th>
          <th class="py-2">Nome</th>
          <th class="py-2">Cognome</th>
          <th class="py-2"></th>
        </tr>
      </thead>
      <tbody class="divide-y divide-slate-100">
        <% @invited_leads.each do |lead| %>
          <tr>
            <td class="py-2"><%= lead.email %></td>
            <td class="py-2"><%= lead.username %></td>
            <td class="py-2"><%= lead.name %></td>
            <td class="py-2"><%= lead.surname %></td>
            <td class="py-2 text-right">
              <% if Current.user.has_role?(:tutor) || Current.user.has_role?(:team_manager) %>
                <%= button_to "Approva", approve_account_lead_path(lead), method: :post,
                      form: { data: { turbo_confirm: "Approvare questo invitato?" } },
                      class: "text-blue-600 hover:underline mr-3" %>
              <% end %>
              <%= button_to "Rimuovi", account_lead_path(lead), method: :delete,
                    form: { data: { turbo_confirm: "Rimuovere questo invitato?" } },
                    class: "text-red-600 hover:underline" %>
            </td>
          </tr>
        <% end %>
      </tbody>
    </table>
  </div>
</div>

6) Risposte ai tuoi punti (sintesi)

“Per ora posso tenerlo così?” Sì: togli il form; usa link referral firmato; password nel form resta ok.

“Generi il link e max 7 inviti?” Sì: invites_limit=7, can_invite? e count_successful_invite!.

“Approve lo fa il Tutor?” Sì: require_tutor! su approve.

“Se non sei approved il link non funziona?” Esatto: approved_referrer? viene controllato in locate_referrer e nella view; il link non si genera / non viene accettato.




----

