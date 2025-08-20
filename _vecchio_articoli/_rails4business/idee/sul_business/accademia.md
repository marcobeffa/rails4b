---
layout: default
title: Accademia
parent: Web app rails
grand_parent: Strumenti 
nav_fold: true
has_children: true
nav_order: 5
---


## Accademia 

User
└── Business accademia
      └── Funnel
            └── Services
                └── Level
                      └── Task       
Rete 
  └── Business accademia
        └── Funnel
              └── Services
                   └── Level
     └── profiles 
          ├── tasks
          │     └── invites
          │            └── transactions
          ├──  contacts
          └── locations

```rb
class User < ApplicationRecord
  has_many :user_progresses
  has_many :programs, through: :user_progresses
end

class Program < ApplicationRecord
  has_many :levels
  enum mode: { self_paced: 0, supervised: 1 } # Modalità del programma
end

class Sheet < ApplicationRecord
  belongs_to :program
  has_many :tasks
end

class Task < ApplicationRecord
  belongs_to :level
  has_many :user_tasks
end

class UserProgress < ApplicationRecord
  belongs_to :user
  belongs_to :program
  belongs_to :level
  belongs_to :professional, optional: true # Se supervised, assegnato a un professionista

  enum mode: { self_paced: 0, supervised: 1 }

  def check_level_progression
    if mode == "self_paced"
      advance_level if all_tasks_completed?
    elsif mode == "supervised"
      advance_level if all_tasks_approved?
    end
  end

  def all_tasks_completed?
    level.tasks.all? { |task| user.user_tasks.find_by(task: task)&.completed? }
  end

  def all_tasks_approved?
    level.tasks.all? { |task| user.task_approvals.find_by(task: task)&.approved? }
  end

  def advance_level
    next_level = program.levels.where("order > ?", level.order).first
    update(level: next_level) if next_level
  end
end

class UserTask < ApplicationRecord
  belongs_to :user
  belongs_to :task
  enum status: { pending: 0, completed: 1 }
end

class Professional < ApplicationRecord
  has_many :user_progresses
end

class TaskApproval < ApplicationRecord
  belongs_to :user
  belongs_to :task
  belongs_to :professional

  enum status: { pending: 0, approved: 1, rejected: 2 }
end

```