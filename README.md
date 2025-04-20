# Gims-transaction-const express = require('express');
const cors = require('cors');
const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

const adressesBancaires = [
  { pays: 'France', type: 'IBAN', adresse: 'FR76 1234 5678 9012 3456 7890 123' },
  { pays: 'Brésil', type: 'PIX', adresse: 'contact@pix.com.br' },
  { pays: 'USA', type: 'Routing/Account', adresse: '123456789 / 000123456789' },
];

const transactions = [];
const utilisateurs = [];

app.get('/api/adresses', (req, res) => {
  res.json(adressesBancaires);
});

app.post('/api/envoyer', (req, res) => {
  const { nom, email, montant, moyenPaiement } = req.body;
  if (!nom || !email || !montant || !moyenPaiement) {
    return res.status(400).json({ message: 'Champs manquants' });
  }
  const nouvelleTransaction = {
    id: transactions.length + 1,
    nom,
    email,
    montant,
    moyenPaiement,
    date: new Date()
  };
  transactions.push(nouvelleTransaction);
  res.status(200).json({ message: 'Transaction enregistrée', transaction: nouvelleTransaction });
});

app.post('/api/inscription', (req, res) => {
  const { nom, email, motDePasse } = req.body;
  if (!nom || !email || !motDePasse) {
    return res.status(400).json({ message: 'Champs manquants' });
  }
  const existe = utilisateurs.find(u => u.email === email);
  if (existe) return res.status(409).json({ message: 'Utilisateur déjà existant' });
  const nouvelUtilisateur = { nom, email, motDePasse };
  utilisateurs.push(nouvelUtilisateur);
  res.status(201).json({ message: 'Utilisateur créé', utilisateur: nouvelUtilisateur });
});

app.post('/api/connexion', (req, res) => {
  const { email, motDePasse } = req.body;
  const utilisateur = utilisateurs.find(u => u.email === email && u.motDePasse === motDePasse);
  if (!utilisateur) return res.status(401).json({ message: 'Identifiants incorrects' });
  res.status(200).json({ message: 'Connexion réussie', utilisateur });
});

app.get('/', (req, res) => {
  res.send("Serveur Gims Transaction actif.");
});

app.listen(PORT, () => {
  console.log(`Serveur backend lancé sur http://localhost:${PORT}`);
});{
  "name": "gims-transaction",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.18.2"
  }
}<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gims Transaction</title>
</head>
<body>
  <h1>Envoyer et recevoir de l'argent</h1>

  <h2>Connexion utilisateur</h2>
  <form id="form-connexion">
    <input type="email" id="login-email" placeholder="Email" required><br>
    <input type="password" id="login-mdp" placeholder="Mot de passe" required><br>
    <button type="submit">Se connecter</button>
  </form>
  <p id="connexion-resultat"></p>

  <h2>Adresses bancaires disponibles :</h2>
  <ul id="adresses"></ul>

  <h2>Formulaire d'envoi d'argent</h2>
  <form id="formulaire-envoi">
    <input type="text" id="nom" placeholder="Votre nom" required><br>
    <input type="email" id="email" placeholder="Votre email" required><br>
    <input type="number" id="montant" placeholder="Montant" required><br>
    <select id="moyenPaiement" required>
      <option value="">Choisir un moyen de paiement</option>
      <option value="Carte Bancaire">Carte Bancaire</option>
      <option value="PayPal">PayPal</option>
      <option value="Crypto">Crypto</option>
      <option value="Mobile Money">Mobile Money</option>
    </select><br>
    <button type="submit">Envoyer</button>
  </form>
  <p id="confirmation"></p>

  <script>
    fetch('https://gims-transaction-backend.onrender.com/api/adresses')
      .then(res => res.json())
      .then(data => {
        const ul = document.getElementById('adresses');
        data.forEach(item => {
          const li = document.createElement('li');
          li.textContent = `${item.pays} - ${item.type} : ${item.adresse}`;
          ul.appendChild(li);
        });
      });

    document.getElementById('formulaire-envoi').addEventListener('submit', async (e) => {
      e.preventDefault();
      const nom = document.getElementById('nom').value;
      const email = document.getElementById('email').value;
      const montant = document.getElementById('montant').value;
      const moyenPaiement = document.getElementById('moyenPaiement').value;

      const res = await fetch('https://gims-transaction-backend.onrender.com/api/envoyer', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ nom, email, montant, moyenPaiement })
      });

      const result = await res.json();
      document.getElementById('confirmation').textContent = result.message;
      e.target.reset();
    });

    document.getElementById('form-connexion').addEventListener('submit', async (e) => {
      e.preventDefault();
      const email = document.getElementById('login-email').value;
      const motDePasse = document.getElementById('login-mdp').value;

      const res = await fetch('https://gims-transaction-backend.onrender.com/api/connexion', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, motDePasse })
      });

      const result = await res.json();
      document.getElementById('connexion-resultat').textContent = result.message;
      if (res.status === 200) {
        alert('Bienvenue ' + result.utilisateur.nom);
      }
    });
  </script>
</body>
</html>
