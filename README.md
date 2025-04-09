# HUMINT
Prototype HUMINT RED TEAMING
Super, on va commencer avec l'**ajout de l’avatar IA réaliste** et le **stockage local des images**. Cela va donner à chaque profil un aspect beaucoup plus réaliste et visuellement convaincant, tout en conservant la simplicité du script.

### **Étapes pour ajouter l'avatar IA et le stockage local des images**

1. **Récupérer un avatar généré depuis `https://thispersondoesnotexist.com/`**
2. **Télécharger et stocker l’image dans un dossier local `/static/avatars/`**
3. **Mettre à jour le profil pour inclure l’avatar localement stocké**

---

### **1. Installation des dépendances nécessaires**

Pour télécharger l'image depuis une URL et la stocker localement, on aura besoin de `requests`. Si ce n’est pas encore installé :

```bash
pip install requests
```

---

### **2. Code modifié pour récupérer et stocker l'avatar IA**

Ajout dans **`generateur_profils.py`** pour télécharger l’image et la sauvegarder dans un dossier `static/avatars/`.

```python
import requests
import os
from faker import Faker
import random
import json

fake = Faker('fr_FR')

# Dossier pour stocker les avatars
AVATAR_FOLDER = "static/avatars"
if not os.path.exists(AVATAR_FOLDER):
    os.makedirs(AVATAR_FOLDER)

INTERETS_POSSIBLES = [
    "cybersécurité", "jeux vidéo", "randonnée", "lecture", "photographie",
    "musique", "voyages", "programmation", "crypto", "hacking éthique"
]

def telecharger_avatar():
    # URL de l'avatar généré automatiquement
    url = "https://thispersondoesnotexist.com/image"
    response = requests.get(url)
    
    # Nom unique pour l'image (éviter les collisions)
    nom_fichier = f"{random.randint(1000, 9999)}.jpg"
    chemin_fichier = os.path.join(AVATAR_FOLDER, nom_fichier)
    
    # Sauvegarde de l'image dans le dossier local
    with open(chemin_fichier, 'wb') as f:
        f.write(response.content)
    
    return f"/static/avatars/{nom_fichier}"

def generer_faux_profil():
    prenom = fake.first_name()
    nom = fake.last_name()
    age = random.randint(18, 60)
    genre = random.choice(["Homme", "Femme", "Non-binaire"])
    ville = fake.city()
    pays = "France"
    profession = fake.job()
    interets = random.sample(INTERETS_POSSIBLES, k=random.randint(2, 4))

    username = f"{prenom.lower()}.{nom.lower()}{random.randint(1, 99)}"

    reseaux_sociaux = {
        "Facebook": f"https://facebook.com/{username}",
        "LinkedIn": f"https://linkedin.com/in/{username}",
        "Instagram": f"@{prenom.lower()}{random.randint(10,99)}"
    }

    # Téléchargement et attribution de l'avatar
    photo_profil = telecharger_avatar()

    return {
        "prenom": prenom,
        "nom": nom,
        "age": age,
        "genre": genre,
        "ville": ville,
        "pays": pays,
        "profession": profession,
        "interets": interets,
        "reseaux_sociaux": reseaux_sociaux,
        "photo_profil": photo_profil
    }

def generer_profils_en_lot(nombre=10, fichier_sortie="profils.json"):
    profils = [generer_faux_profil() for _ in range(nombre)]
    
    with open(fichier_sortie, "w", encoding="utf-8") as f:
        json.dump(profils, f, indent=4, ensure_ascii=False)
    
    print(f"{nombre} profils générés et sauvegardés dans {fichier_sortie}")

if __name__ == "__main__":
    nombre = int(input("Combien de profils veux-tu générer ? "))
    nom_fichier = input("Nom du fichier de sortie (.json) : ") or "profils.json"
    generer_profils_en_lot(nombre, nom_fichier)
```

---

### **3. Mise à jour de l’interface web pour afficher l’avatar local**

Tu n’as pas besoin de changer beaucoup de choses dans l’interface Flask, juste quelques ajustements pour afficher les images stockées localement.

Modifie le fichier **`templates/resultats.html`** pour afficher l’avatar à partir du chemin relatif :

```html
<!doctype html>
<html lang="fr">
<head>
    <meta charset="utf-8">
    <title>Résultats</title>
</head>
<body>
    <h1>Profils générés</h1>
    {% for profil in profils %}
        <div style="margin-bottom:20px;">
            <strong>{{ profil.prenom }} {{ profil.nom }}</strong> ({{ profil.age }} ans, {{ profil.genre }})<br>
            {{ profil.profession }} à {{ profil.ville }}, {{ profil.pays }}<br>
            Intérêts : {{ profil.interets | join(", ") }}<br>
            Facebook : {{ profil.reseaux_sociaux.Facebook }}<br>
            LinkedIn : {{ profil.reseaux_sociaux.LinkedIn }}<br>
            Instagram : {{ profil.reseaux_sociaux.Instagram }}<br>
            <img src="{{ profil.photo_profil }}" width="150">
        </div>
    {% endfor %}
    <a href="/">Retour</a>
    <a href="/telecharger">Télécharger en JSON</a>
</body>
</html>
```

---

### **4. Tester le tout**

- Lance l’application Flask et génère des profils.
- Les avatars seront automatiquement téléchargés et stockés localement dans le dossier `/static/avatars/`.
- L'interface web affichera les images des profils générés.

---

### **Prochaines étapes**

- Si ça fonctionne bien, on pourrait automatiser encore plus, en générant des avatars plus variés (âge, sexe, origine).
- Ajouter un bouton de téléchargement des images individuelles dans l’interface pour rendre ça encore plus fluide.
- Intégration avec des bases de données pour gérer un grand nombre de profils, ou génération de lots très facilement.

---

Ça te va pour cette première étape d'amélioration ? On peut continuer à ajouter des fonctionnalités à la demande.
Pour la suite, voici quelques suggestions d'améliorations pour aller encore plus loin dans l’outil HUMINT/OSINT. Nous pourrions ajouter plusieurs fonctionnalités pour rendre ton générateur de faux profils plus puissant et réaliste, tout en intégrant des aspects plus avancés de l’automatisation et de l’analyse :
1. Intégration d’une base de données (SQLite ou PostgreSQL)

Si tu souhaites gérer une quantité plus grande de profils, les stocker durablement et effectuer des requêtes avancées, il serait intéressant d'intégrer une base de données.
Pourquoi une base de données ?

    Gestion des profils à grande échelle : Tu pourras ajouter, modifier et supprimer des profils facilement.

    Requêtes complexes : Par exemple, chercher des profils par âge, profession, ville, ou même des interactions.

    Persistante : Contrairement au stockage en fichiers JSON, tout est centralisé et peut être consulté à tout moment.

Tu souhaites que je t’aide à configurer cette base de données et l’intégrer à l’outil ?
2. Automatisation d’interactions avec GPT (ex : simulateur de conversation HUMINT)
Pourquoi ?

    Test Red Teaming : Tu peux simuler des conversations avec des profils fictifs et les utiliser pour automatiser des scénarios de phishing ou social engineering.

    Collecte de données en profondeur : L’intégration d’un chatbot permettrait à tes faux profils d’avoir une personnalité cohérente et de répondre de manière réaliste aux questions.

Si tu veux, on peut configurer une fonction où GPT générera des réponses adaptées à chaque profil en fonction de son historique, âge, profession, et autres paramètres.
3. Interface avancée avec gestion des profils (Dashboard, filtres, recherche)

Une interface plus aboutie permettrait de gérer facilement des centaines de profils, avec des fonctionnalités comme :

    Affichage en liste ou tableau

    Filtrage par critère : Âge, profession, etc.

    Recherche rapide : Recherche par mot-clé dans les profils.

    Téléchargement en masse : Export de profils en CSV ou JSON pour l’analyse.

Une interface type dashboard avec gestion d’utilisateurs et filtrage avancé serait idéale si tu veux rendre l'outil encore plus flexible et performant.
4. Ajout de scénarios ou de "missions" HUMINT
Pourquoi ?

    Personnalisation de l’outil : Tu pourrais créer des profils en fonction de scénarios spécifiques (par exemple, un profil "employé d'une entreprise de cybersécurité" ou un "cible pour une mission de test de phishing").

    Simulations complexes : Automatiser des scénarios où les profils interagissent entre eux selon un contexte donné (ex : plusieurs profils sur un réseau social en simulant des interactions, publications, etc.).

Prochaine étape ?

Voici les options à explorer selon tes besoins :

    Intégration d’une base de données (SQLite, PostgreSQL)

    Automatisation des interactions avec GPT (chatbot pour chaque profil)

    Interface avancée avec dashboard, filtres et recherche

    Ajout de scénarios HUMINT ou missions automatisées


from flask import Flask, render_template, request, session, redirect, url_for
from chat_module import repondre_au_message
import json
import os

app = Flask(__name__)
app.secret_key = "supersecret"  # à remplacer par un vrai secret

# Chargement des profils
with open("profils.json", "r", encoding="utf-8") as f:
    profils = json.load(f)

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        profil_id = request.form["profil_id"]
        session["profil_id"] = profil_id
        session["chat"] = []  # réinitialise la conversation
        return redirect(url_for("chat"))
    return render_template("index.html", profils=profils)

@app.route("/chat", methods=["GET", "POST"])
def chat():
    if "profil_id" not in session:
        return redirect(url_for("index"))

    profil = profils[int(session["profil_id"])]
    historique = session.get("chat", [])

    if request.method == "POST":
        message = request.form["message"]
        historique.append({"role": "user", "content": message})
        reponse = repondre_au_message(profil, message, historique)
        historique.append({"role": "assistant", "content": reponse})
        session["chat"] = historique

    return render_template("chat.html", profil=profil, historique=historique)

if __name__ == "__main__":
    app.run(debug=True)
