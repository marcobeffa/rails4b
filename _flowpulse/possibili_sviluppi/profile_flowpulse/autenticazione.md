---
layout: default
title: Jwt
parent: Profile 
grand_parent: Possibili Sviluppi 
nav_fold: true
has_children: true
nav_order: 1
---


Ti serve un Single Sign-On (SSO) basato su JWT che permetta di condividere l’autenticazione tra i vari sottodomini o servizi.

L'idea è quella di validare il JWT nelle app come note.flowpulse.net verificando che non solo il t

[Articolo autenticazione](https://a-chacon.com/en/on%20rails/2024/10/16/poc-using-rails-8-auth-system-in-api-only.html)
Vuoi che profile.flowpulse.net funzioni come il centro di autenticazione principale, e che le altre app (come note.flowpulse.net) verifichino sia il token di autenticazione che la sessione attiva per determinare se l'utente è autorizzato. Questo è un approccio che combina sessioni e token JWT, e ti consente di centralizzare la gestione dell'autenticazione in profile.flowpulse.net.

Ecco un piano d'azione dettagliato per implementarlo:

1. Flusso di autenticazione
Autenticazione su profile.flowpulse.net:

Quando l'utente si autentica su profile.flowpulse.net, il server Rails crea una sessione tradizionale (salvata nei cookie) e genera un token JWT che viene restituito come risposta all'utente.
Accesso su altre app (es. note.flowpulse.net):

Ogni volta che un'app esterna (note.flowpulse.net, event.flowpulse.net, ecc.) riceve una richiesta, verifica:
La presenza del token JWT nell'header Authorization: Bearer <token>.
Se la sessione è attiva su profile.flowpulse.net (per esempio tramite un cookie di sessione che viene inviato automaticamente dal browser).
Verifica del token JWT:

note.flowpulse.net riceve il token JWT nell'header della richiesta. Deve verificarne la validità (compreso l'expiry) e decodificarlo per recuperare l'ID dell'utente.
Verifica della sessione attiva:

La sessione deve essere attiva su profile.flowpulse.net, quindi note.flowpulse.net invia una richiesta a profile.flowpulse.net per verificare che l'utente abbia una sessione attiva.
Se la sessione è valida, viene restituito un risultato positivo.
Se la sessione non è attiva o il token è scaduto, la richiesta viene rifiutata.
2. Architettura
a. Generazione del token su profile.flowpulse.net
Quando l'utente si autentica, crea una sessione e un token JWT. Puoi farlo nel tuo SessionsController come già impostato:

ruby
Copia
Modifica
class SessionsController < ApplicationController
  def create
    user = User.authenticate_by(email: params[:email], password: params[:password])
    
    if user
      # Crea la sessione tradizionale
      start_new_session_for user
      
      # Crea il token JWT
      token = generate_jwt(user)
      
      render json: { token: token }, status: :ok
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end

  private

  def generate_jwt(user)
    JWT.encode({ user_id: user.id, exp: 24.hours.from_now.to_i }, Rails.application.secret_key_base)
  end
end
b. Verifica del token su note.flowpulse.net
Su note.flowpulse.net, crea un before_action per verificare il token e anche la sessione.

ruby
Copia
Modifica
class ApplicationController < ActionController::API
  before_action :authenticate_request

  private

  def authenticate_request
    token = request.headers['Authorization']&.split(' ')&.last
    if token.present?
      begin
        decoded_token = JWT.decode(token, Rails.application.secret_key_base).first
        @current_user = User.find(decoded_token['user_id'])
        
        # Verifica se la sessione è attiva su profile.flowpulse.net
        if session_active_on_profile?
          return true
        else
          render json: { error: 'Session expired or not active' }, status: :unauthorized
        end
      rescue JWT::DecodeError => e
        render json: { error: 'Not Authorized', message: e.message }, status: :unauthorized
      end
    else
      render json: { error: 'Token missing' }, status: :unauthorized
    end
  end

  def session_active_on_profile?
    # Chiedi a profile.flowpulse.net se la sessione è attiva (potresti fare una richiesta API o simile)
    response = HTTParty.get("https://profile.flowpulse.net/check_session", headers: { 'Authorization' => "Bearer #{request.headers['Authorization']}" })
    
    response.code == 200 # Verifica che la risposta sia positiva
  end
end
c. Verifica della sessione su profile.flowpulse.net
Aggiungi un endpoint su profile.flowpulse.net che verifica se la sessione è ancora attiva.

ruby
Copia
Modifica
class SessionsController < ApplicationController
  before_action :authenticate_user!

  def check_session
    # Se l'utente è autenticato su profile.flowpulse.net, ritorna successo
    render json: { success: true }, status: :ok
  rescue => e
    render json: { error: 'Session expired or invalid' }, status: :unauthorized
  end
end
3. Flusso completo
L'utente si autentica su profile.flowpulse.net.

Viene creata la sessione.
Viene generato il token JWT e restituito all'utente.
Quando l'utente accede a note.flowpulse.net:

Invia una richiesta con l'header Authorization: Bearer <token>.
note.flowpulse.net verifica il token (è valido e non scaduto).
note.flowpulse.net controlla anche se la sessione è attiva su profile.flowpulse.net (via richiesta API).
Se entrambe le verifiche sono positive, l'utente è autorizzato ad accedere alle risorse di note.flowpulse.net.
Se il token è scaduto o la sessione non è attiva, l'accesso viene negato.

4. Punti chiave
Centralizzazione dell'autenticazione: profile.flowpulse.net è il centro di autenticazione. Le altre app (note.flowpulse.net, event.flowpulse.net) sono "client" che chiedono conferma al server principale.
Verifica sia del token che della sessione: Ogni app esterna verifica prima il token, poi contatta profile.flowpulse.net per validare che la sessione dell'utente sia ancora attiva.
Conclusioni
Questo approccio ti consente di gestire una sessione centralizzata e un'autenticazione basata su token per diverse app. Se preferisci, puoi anche usare un sistema completamente JWT, ma questo approccio combinato ti offre maggiore flessibilità, dato che puoi sfruttare le sessioni per la gestione della persistenza e della scadenza delle credenziali.