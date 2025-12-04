---
layout: default
title: Upgrade Sortablejs nested list
parent: Branch Tree
grand_parent:  Web app rails
nav_fold: true
has_children: true
nav_order: 3
---

Possibili opzioni:

# 📌 MIGLIORAMENTO: Drag & Drop posizione rami (Branch Tree)

## ✅ Stato attuale
- Il sistema aggiorna la posizione dei rami con una richiesta `GET` e redirect:
  ```
  /branches/1/updatenav_order?parent_id=2&nav_order=1&id=5
  ```
- Funziona bene in fase di sviluppo e test.
- UX non ottimale: ricarica l’intera pagina, più consumo di banda e tempo.

---

## 🎯 Obiettivo futuro
Aggiornare il sistema per:
- Evitare reload pagina.
- Inviare la modifica in background con `POST`.
- Risparmiare risorse server/client.
- UX più moderna e reattiva.

---

## 🔧 Opzione 1 – `fetch()` con `POST` asincrono

### ✅ Frontend (JavaScript)

```js
fetch('/branches/updatenav_order', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCSRFToken()
  },
  body: JSON.stringify({
    id: itemId,
    parent_id: parentId,
    nav_order: newnav_order
  })
}).then(response => {
  if (response.ok) window.location.reload(); // oppure aggiorna il DOM
});
```

**Helper per CSRF token:**

```js
function getCSRFToken() {
  return document.querySelector('meta[name="csrf-token"]').getAttribute('content');
}
```

---

### ✅ Backend (Rails)

**Controller:**
```ruby
def updatenav_order
  branch = Branch.find(params[:id])
  branch.update(parent_id: params[:parent_id], nav_order: params[:nav_order])

  respond_to do |format|
    format.html { redirect_to branches_path, notice: "Posizione aggiornata" }
    format.json { render json: { status: "ok" } }
  end
end
```

**Rotta:**
```ruby
post 'branches/updatenav_order', to: 'branches#updatenav_order'
```

---

## 🚀 Opzione 2 – Turbo Stream (Hotwire Rails)

**Requisiti:**
- Hotwire installato (Turbo, Stimulus)

**Controller:**
```ruby
format.turbo_stream
```

**Partial:**
`app/views/branches/updatenav_order.turbo_stream.erb`

**Form:**
```erb
<%= form_with url: updatenav_order_branches_path, method: :post, data: { turbo: true } do |f| %>
  ...
<% end %>
```

---

## 📅 Quando fare lo switch
- Quando aumentano gli utenti
- Quando serve maggiore fluidità UX
- Quando misuri problemi di performance (CPU, traffico, slow UX)

---
