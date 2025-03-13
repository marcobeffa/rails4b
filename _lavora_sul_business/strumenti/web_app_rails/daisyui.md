---
layout: default
title: Rails8-Tailwind4-DaisyUi
parent: Web app rails
grand_parent: Strumenti 
nav_fold: true
has_children: true
nav_order: 2
---


rails app:template LOCATION='https://railsbytes.com/script/XLEs4P'

# 🚀  Rails 8 - Tailwindcss 4 - DaisyUi

```sh
rails new note
cd note
```

```sh
./bin/bundle add tailwindcss-ruby
./bin/bundle add tailwindcss-rails
./bin/rails tailwindcss:install

```


```sh
curl -sLo app/assets/tailwind/daisyui.js https://esm.run/daisyui@5/index.js

```


```css
@import "tailwindcss";
@plugin "./daisyui.js";
```


```sh
rails g controller Pages home

rails g authentication
User.create(email_address: "mario@mario.it", password: "123456")

```

config/routes.rb
```rb
Rails.application.routes.draw do
  root to: 'pages#home'
end
```


```html
app/views/pages/home.html.erb
<button class="btn btn-primary">Hello daisyUI!</button>
```

