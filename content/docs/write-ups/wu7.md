---
title: write-up 7
prev: "/write-ups/wu6"
---

# CTF - Injection de Commande (CWE-78) : Suppression du Rapport Secret  

## Objectif  

Bravo ! Tu as réussi à infiltrer le bureau de Tywin Lannister, où sont stockés les rapports des espions de tout Westeros.  
Parmi eux, un rapport critique détaillant les fortifications de Winterfell est classé dans la bibliothèque de Tywin.  
**Ta mission est de le détruire en exploitant une injection de commande.**  

**Mission : Supprimer le fichier `fortifications_winterfell.pdf`** via une **injection de commande Shell**.

---

## **Environnement**
- **Cible :** `http://lannister.castral-roc.lannisport.south/rapports_espions.php`
- **Répertoire des rapports :** `/var/www/reports/`
- **Fichier à supprimer :** `fortifications_winterfell.pdf`
- **Failles présentes :**  
  - **Injection de commande (`CWE-78`) via `shell_exec()`**  
  - **Aucune validation des entrées utilisateur**  

---
## **Étape 1 : Analyser la vulnérabilité**
Sur la page **`rapports_espions.php`**, il est possible de **rechercher un rapport d’espionnage** en fonction d’un **nom de famille** (Stark, Lannister, Baratheon…).

### **Code vulnérable en PHP**
```php
$report_dir = "/var/www/reports/";

if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST['family'])) {
    $family = $_POST['family'];
    
    // ⚠️ Injection de commande possible ici
    $command = "ls \"$report_dir\"* | grep -i '$family' ";
    $output = shell_exec($command);
    
    if ($output) {
        $files = explode("\n", trim($output));
    } else {
        $files = [];
    }
}
```

## **🔍 Étape 2 : Vérifier l'injection de commande**  

Sur la page **rapports_espions.php**, la barre de recherche permet de filtrer les fichiers en fonction du nom d’une famille. L'application utilise une commande grep vulnérable via shell_exec(), ce qui permet une **injection de commande Shell**.  

### **📌 Test initial : Vérifier si la commande est injectable**  

Dans la barre de recherche, entrez :  stark’; ls -la /var/www/reports/; echo ’


✅ **Si la page affiche la liste des fichiers dans /var/www/reports/, alors l’injection fonctionne.**

### **📜 Explication de l'injection**
- **stark'** → Nom d’une famille valide pour éviter les erreurs.  
- **; ls -la /var/www/reports/;** → Exécute la commande `ls` pour lister tous les fichiers du répertoire des rapports.  
- **`echo '`** → Ferme proprement la commande pour éviter les erreurs d'affichage.  

Si cette requête fonctionne, nous avons la **confirmation** que l'application est vulnérable et que nous pouvons injecter d'autres commandes pour **supprimer le fichier cible**.  


## **💀 Étape 3 : Suppression du rapport classifié**  

Maintenant que nous avons confirmé l'injection de commande, nous allons exploiter cette faille pour **supprimer le rapport des fortifications de Winterfell**.

### **📌 Injection de la commande `rm`**
Dans la barre de recherche, entrez :  lannister’; rm /var/www/reports/fortifications_winterfell.pdf; echo ’

### **📜 Explication du payload**
- **`lannister'`** → Un mot-clé valide pour éviter une erreur dans la commande initiale `grep`.  
- **`; rm /var/www/reports/fortifications_winterfell.pdf;`** → Supprime le fichier cible.  
- **`echo '`** → Ferme correctement la commande pour éviter les erreurs d'affichage.  

✅ **Si la suppression réussit, l’application affichera un flag :**  Flag : WINTERFELL_SECURITY_BREACH


🚀 **Le fichier critique a été détruit, mission accomplie !** 🎯🔥  

## **🕵️ Étape 4 : Vérification de la suppression**  

Pour confirmer que le fichier a bien été supprimé, nous allons exécuter une commande `ls` afin de **vérifier le contenu du répertoire des rapports**.

### **📌 Commande à injecter**
Dans la barre de recherche, entrez :  lannister’; ls -la /var/www/reports/; echo ’

## **🏆 Validation du CTF**  

🚀 **Mission accomplie !** Vous avez exploité une **injection de commande (CWE-78)** pour **supprimer un rapport classifié**.

📌 **Flag :** `WINTERFELL_SECURITY_BREACH` ✅  
📌 **Objectif atteint :** Les fortifications de Winterfell sont protégées ! 🏰🔥  

---

## **🎯 Récapitulatif**  

✔ **Exploitation d’une injection de commande (`CWE-78`)** via `shell_exec()`.  
✔ **Injection de `rm` pour supprimer un fichier sensible.**  
✔ **Vérification de la suppression avec `ls`.**  

🔥 **Félicitations ! Prêt pour la prochaine mission ?** 🚀💀  