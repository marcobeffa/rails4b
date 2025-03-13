---
layout: default
title: Ricerca semantica vettoriale
parent: Sul business
grand_parent: Idee
nav_fold: true
has_children: true
nav_order: 6
---
<!--Perfetto ora mi fai un altro file md dove spieghi in modo semplice i passaggi e cosa andremo a fare nell'app? come se fosse un testo divulgativo per non addetti ai lavori e per far capire i passaggi-->
# Ricerca semantica vettoriale

## **Obiettivo**
L'app che stiamo creando permette di cercare informazioni in modo più intelligente. Anziché basarsi solo sulle parole esatte che scriviamo, il sistema capisce il significato delle frasi e trova i risultati più pertinenti.

## **Perché è diversa da una ricerca classica?**
Nella ricerca tradizionale, se cerchiamo "mal di schiena", il sistema trova solo le pagine che contengono esattamente queste parole. Con la nostra app, invece, il sistema capisce che "mal di schiena" è simile a "esercizi per la schiena" e può suggerire informazioni utili anche se le parole non corrispondono esattamente.

## **Come funziona?**
Ecco i passaggi principali:

1. **Salviamo le informazioni** – Ogni contenuto che vogliamo rendere ricercabile viene convertito in una rappresentazione numerica chiamata "vettore".
2. **Interpretiamo la domanda** – Quando un utente fa una domanda, il sistema trasforma anche questa in un vettore.
3. **Cerchiamo la risposta migliore** – Confrontiamo il vettore della domanda con quelli delle informazioni salvate e troviamo quelle più simili.
4. **Mostriamo i risultati** – L'utente vede i contenuti più pertinenti, anche se le parole non coincidono perfettamente.

## **Cosa useremo per realizzarla?**
- **PostgreSQL con pgvector**: Un database avanzato che permette di gestire i vettori.
- **OpenAI**: Per generare i vettori delle parole e delle frasi.
- **Rails (Ruby on Rails)**: Il framework che ci permette di costruire l'app.
- **DaisyUI**: Per creare un'interfaccia grafica moderna e intuitiva.

## **Cosa potremo fare con questa tecnologia?**
- Creare un motore di ricerca per un blog o un archivio di documenti.
- Offrire risposte personalizzate ai clienti in una chat.
- Organizzare meglio le informazioni in un sito o in un’app.

Questa tecnologia ci permette di migliorare l'esperienza degli utenti, aiutandoli a trovare rapidamente quello di cui hanno bisogno. 🚀


# Implementazione di Ricerca Vettoriale in Rails con pgvector e DaisyUI

Questa guida mostra come creare un'app Rails che utilizza `pgvector` per la ricerca vettoriale e un'interfaccia con **DaisyUI**.

## **1. Configurare il database**
Assicurati di avere **PostgreSQL** con l'estensione `pgvector` installata:

```sh
psql -d tua_app_db -c "CREATE EXTENSION vector;"
```

Aggiungi la gemma `pgvector` al `Gemfile`:

```ruby
gem 'pgvector', '~> 0.5.0'
```

Esegui `bundle install` per installarla.

---

## **2. Creare il modello per salvare le risorse vettoriali**
Genera un modello `Resource` per salvare i dati:

```sh
rails generate model Resource title:string description:text embedding:vector
```

Esegui la migrazione:

```sh
rails db:migrate
```

---

## **3. Ottenere l'embedding da OpenAI**
Nel modello `Resource`, aggiungi un metodo per generare il vettore:

```ruby
require 'net/http'
require 'uri'
require 'json'

class Resource < ApplicationRecord
  before_create :generate_embedding

  def generate_embedding
    self.embedding = fetch_openai_embedding(title + " " + description)
  end

  def fetch_openai_embedding(text)
    uri = URI.parse("https://api.openai.com/v1/embeddings")
    request = Net::HTTP::Post.new(uri)
    request["Authorization"] = "Bearer #{ENV['OPENAI_API_KEY']}"
    request["Content-Type"] = "application/json"
    request.body = { input: text, model: "text-embedding-ada-002" }.to_json

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) { |http| http.request(request) }
    JSON.parse(response.body)["data"][0]["embedding"]
  end
end
```

**Nota:** Imposta `OPENAI_API_KEY` nel tuo `.env`.

---

## **4. Cercare una risorsa simile alla query**
Quando un utente fa una domanda, calcoliamo il vettore della query e cerchiamo nel database la risorsa più vicina.

```ruby
class ResourcesController < ApplicationController
  def search
    query = params[:query]
    query_embedding = fetch_openai_embedding(query)

    closest_resources = Resource.order(Arel.sql("embedding <=> '#{query_embedding}'::vector")).limit(5)

    render json: closest_resources
  end

  private

  def fetch_openai_embedding(text)
    uri = URI.parse("https://api.openai.com/v1/embeddings")
    request = Net::HTTP::Post.new(uri)
    request["Authorization"] = "Bearer #{ENV['OPENAI_API_KEY']}"
    request["Content-Type"] = "application/json"
    request.body = { input: text, model: "text-embedding-ada-002" }.to_json

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) { |http| http.request(request) }
    JSON.parse(response.body)["data"][0]["embedding"]
  end
end
```

Aggiungi la route nel file `config/routes.rb`:

```ruby
get 'resources/search', to: 'resources#search'
```

---

## **5. Testare l'API**
Esegui il server Rails:

```sh
rails s
```

Poi prova la ricerca con una query:

```sh
curl "http://localhost:3000/resources/search?query=esercizi per la schiena"
```

---

## **6. Interfaccia con DaisyUI**

Ecco un componente **React + DaisyUI** per cercare risorse.

```jsx
import { useState } from "react";

export default function SearchApp() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);

  const searchResources = async () => {
    if (query.trim() === "") return;

    const response = await fetch(`/resources/search?query=${query}`);
    const data = await response.json();
    setResults(data);
  };

  return (
    <div className="flex flex-col items-center p-6">
      <h1 className="text-2xl font-bold mb-4">Ricerca Risorse</h1>
      <div className="flex w-full max-w-lg">
        <input
          type="text"
          className="input input-bordered w-full"
          placeholder="Cerca..."
          value={query}
          onChange={(e) => setQuery(e.target.value)}
        />
        <button className="btn btn-primary ml-2" onClick={searchResources}>
          Cerca
        </button>
      </div>
      <div className="mt-6 w-full max-w-lg">
        {results.length > 0 ? (
          <ul className="menu bg-base-200 p-4 rounded-box">
            {results.map((resource) => (
              <li key={resource.id} className="p-2 border-b">
                <h2 className="font-semibold">{resource.title}</h2>
                <p className="text-sm text-gray-600">{resource.description}</p>
              </li>
            ))}
          </ul>
        ) : (
          <p className="text-gray-500">Nessun risultato trovato.</p>
        )}
      </div>
    </div>
  );
}
```

Salva questo file come `app/javascript/components/SearchApp.jsx` e usa **Hotwire** per renderizzarlo in Rails.

---

## **Risultato atteso**
Se hai risorse come:
- "Esercizi per la schiena" → `[0.12, -0.98, 0.75]`
- "Dieta per dimagrire" → `[0.89, 0.45, -0.32]`

E cerchi `"mal di schiena"`, il sistema troverà la risorsa più simile basandosi sulla distanza tra vettori.

---

## **Prossimi passi**
- Integrare la ricerca in una chatbot.
- Usare **Hotwire Turbo** per aggiornamenti in tempo reale.
- Implementare un'interfaccia più avanzata con filtri.

🚀 Vuoi che ti aiuti a integrare il componente React in Rails?
