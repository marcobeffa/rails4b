---
layout: default
title: Note
parent: Flowpulse 
grand_parent: Idee
nav_fold: true
has_children: true
nav_order: 1
---

https://github.com/Ionaru/easy-markdown-editor
https://chatgpt.com/share/67c9a7cc-9620-8002-89e2-0834e13e2828
## FlowPulse Note

```sh
rails new note
cd note
```

## DaisyUy template


```sh
rails app:template LOCATION='https://railsbytes.com/script/XLEs4P'
```

tag:references child_tag_id:integer 
```sh
rails g scaffold Post user:references title description body:text visibility:integer data_pubblicazione:datetime stato:integer 
```
enum :visibility, { privato: 0, iscritti: 1, pubblico: 2 }

enum :stato, { bozza: 0, revisione: 1, pubblicato: 2 }

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>EasyMDE demo</title>
    <!---EasyMDE-->
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/easymde/dist/easymde.min.css"
    />
    <script src="https://cdn.jsdelivr.net/npm/easymde/dist/easymde.min.js"></script>
  </head>

  <body>
    <%= form_with model: @post do |form| %>
      <%= form.text_area :content, id: "my-text-area" %>
    <% end %>

    <script>
      const easymde = new EasyMDE({
        element: document.getElementById('my-text-area'),
      });
    </script>
  </body>
</html>
```

```html
gem "redcarpet"

```

  <article class="prose lg:prose-lg mx-auto my-8 p-6 bg-white rounded-lg shadow">
    <%= link_to 'Modifica', edit_note_path(edit_body: true), class: "text-white bg-red-600 hover:bg-red-700 focus:ring-4 focus:outline-none focus:ring-red-300 font-medium rounded-lg text-sm px-4 py-2" %>
    <div>
      <%= @note.rendered_content  if  @note.content %>
    </div>
  </article>




note.rb
```rb

  def rendered_content
    # renderer = Redcarpet::Render::HTML.new
    # markdown = Redcarpet::Markdown.new(renderer, extensions = {})
    # markdown.render(body).html_safe

    renderer = Redcarpet::Render::HTML.new(hard_wrap: true, filter_html: true)
    markdown = Redcarpet::Markdown.new(renderer,
                                       autolink: true,
                                       tables: true,
                                       fenced_code_blocks: true)
    markdown.render(content).html_safe
  end
 

```