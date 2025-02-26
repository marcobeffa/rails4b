---
layout: default
title: Card Profile

parent: Profile 
grand_parent: Possibili Sviluppi
nav_fold: true
has_children: true
nav_order: 1
---


----
  <div class="card">
    <div class="card-header">
      <img class="profile-img" src="https://via.placeholder.com/150" alt="Profile Image">
    </div>
    <div class="card-body">
      <h3 class="name">John Doe</h3>
      <p class="bio">Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nullam non urna nec leo auctor.</p>
    </div>
    <div class="card-footer">
      <button class="btn">Contact</button>
    </div>

  </div>
----
<pre>

style.css
/* Importa CSSZero (assicurati di avere il file CSSZero collegato correttamente nel tuo progetto) */
@import url('https://cdn.jsdelivr.net/gh/lazaronixon/css-zero@latest/css-zero.min.css');
/* Stili per la card */
.card {
  max-width: 320px;
  margin: 20px auto;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  background-color: #fff;
}

.card-header {
  position: relative;
}

.profile-img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-bottom: 2px solid #eee;
}

.card-body {
  padding: 15px;
  text-align: center;
}

.name {
  font-size: 1.5rem;
  margin: 10px 0;
}

.bio {
  font-size: 1rem;
  color: #666;
}

.card-footer {
  padding: 10px;
  text-align: center;
}

.btn {
  background-color: #007bff;
  color: #fff;
  padding: 10px 20px;
  font-size: 1rem;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn:hover {
  background-color: #0056b3;
}

/* Stili per le informazioni del profilo */
.profile-info {
  padding: 20px;
}

.info-section {
  margin-bottom: 15px;
}

.info-section h4 {
  margin: 0;
  font-size: 1.2rem;
  color: #333;
}

.info-section p {
  margin: 5px 0;
  color: #666;
}

</pre>
