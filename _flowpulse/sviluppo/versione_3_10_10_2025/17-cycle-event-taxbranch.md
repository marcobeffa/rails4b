---
title: Cycle event taxbranch
parent: Versione 3 10 ottobre 2025 # Versione 3 10 ottobre 2025
grand_parent: Sviluppo 
nav_order: 17
---


# Cycle event taxbranch
bin/rails g scaffold Eventdate date_start:datetime date_end:datetime taxbranch:references lead:references cycle:integer status:integer description:text meta:jsonb --skip-test-unit --skip-system-test