---
layout: default
title: Rails8-Tailwind4-Flowbite
parent: Web app rails

nav_fold: true
has_children: true
nav_order: 1
---


# 🚀 Rails 8 con TailwindCSS 4 e Flowbite


---

## **1️⃣ Creazione del progetto Rails 8**

```sh
rails new myapp -d postgresql
cd myapp
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
