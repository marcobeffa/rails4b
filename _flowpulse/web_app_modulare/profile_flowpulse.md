---
layout: default
title: FlowPulse Profile
parent: Crea il tuo business con FlowPulse
---

## FlowPulse Profile: Gestione Utenti e Autenticazione

L'app **FlowPulse Profile** è il modulo fondamentale per la gestione degli utenti e dell'autenticazione nel sistema **FlowPulse**. Questo modulo è responsabile di registrare e autenticare gli utenti, consentendo l'accesso sicuro agli altri moduli dell'ecosistema **FlowPulse**. Utilizza il framework **Rails Authentication** insieme a **JWT** per generare token di accesso, e integra la gestione dei profili utente, consentendo la personalizzazione e il salvataggio dei dati relativi agli utenti.

**Caratteristiche principali:**

- **Gestione utenti**: Permette di registrare e autenticare gli utenti tramite email e password.
- **Autenticazione con JWT**: Utilizza JSON Web Tokens (JWT) per generare e gestire i token di accesso.
- **Profili utente**: Ogni utente ha un profilo personalizzato con informazioni come username, contatti e avatar.

### Mockup (da aggiungere il link alla foto)

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
rails g scaffold Profile user:references username:string contacts_public:text avatar_img_url:string
</pre>

6️⃣ **Aggiungere il gemma `jwt` per gestire i token di autenticazione:**

<pre>
gem 'jwt'
</pre>

7️⃣ **Creare il controller per l'autenticazione con JWT:**

<pre>
class AuthenticationController < ApplicationController
  def login
    user = User.find_by(email: params[:email])

    if user&.authenticate(params[:password])
      token = generate_jwt(user)
      render json: { token: token }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end

  private

  def generate_jwt(user)
    JWT.encode({ user_id: user.id, exp: 24.hours.from_now.to_i }, Rails.application.secret_key_base)
  end
end
</pre>

8️⃣ **Generare il controller API per la gestione dei profili utente:**

<pre>
rails generate controller Api::Profiles
</pre>

9️⃣ **Aggiungere i metodi CRUD per la gestione dei profili utente nel controller:**

<pre>
module Api
  class ProfilesController < ApplicationController
    before_action :authenticate_user!

    def index
      profiles = Profile.all
      render json: profiles
    end

    def show
      profile = Profile.find(params[:id])
      render json: profile
    end

    def create
      profile = Profile.new(profile_params)
      if profile.save
        render json: profile, status: :created
      else
        render json: profile.errors, status: :unprocessable_entity
      end
    end

    private

    def profile_params
      params.require(:profile).permit(:username, :contacts_public, :avatar_img_url)
    end
  end
end
</pre>

---

