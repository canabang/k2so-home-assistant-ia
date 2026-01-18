# 🧪 Protocole de Test : Migration Ollama (Distant) -> Home Assistant

Ce document détaille les étapes pour valider la communication entre votre Home Assistant et votre serveur Ollama (NAS), ainsi que la réactivité du modèle `llama3.2`.

## 📍 Pré-requis
*   **Serveur Ollama (NAS)** : Allumé, modèle `llama3.2` chargé.
*   **Home Assistant** : Intégration configurée vers l'IP du NAS.

---

## 1. Test de Connexion Basique (Ping)
*Objectif : Vérifier que HA "voit" bien le serveur Ollama.*

1.  Allez dans **Outils de développement** > **Services** (ou **Actions** selon la version).
2.  Recherchez le service `ai_task.generate_data` (ou le service spécifique à votre intégration Ollama si différent, ex: `ollama.generate`).
3.  Passez en mode **YAML** et lancez ce test minimaliste :
    ```yaml
    action: ai_task.generate_data
    data:
      task_name: "Test Ping"
      instructions: "Réponds juste par le mot 'Pong'."
    ```
4.  **Résultat attendu** : Une coche verte apparaît rapidement.
5.  **Vérification** : Regardez la réponse dans l'interface (si affichée) ou passez à l'étape 2 pour voir le contenu.

---

## 2. Test du Script K-2SO (Intégration Complète)
*Objectif : Vérifier que le script centralisé fonctionne avec le nouveau moteur.*

1.  Allez dans **Outils de développement** > **Services**.
2.  Recherchez le script : `script.k_2so_generateur_de_message`.
3.  Utilisez ce payload de test :
    ```yaml
    action: script.k_2so_generateur_de_message
    data:
      mission: "test_connexion_ollama"
      details: "Latence réseau inconnue"
      consigne: "Confirme que tu es opérationnel sur le nouveau serveur NAS."
    response_variable: reponse_test
    ```
4.  **Important** : Cliquez sur **"Exécuter"**.
5.  **Analyse** : Comme c'est un script qui renvoie une variable, le résultat ne s'affiche pas directement ici facilement. L'idéal est de regarder les **Traces**.

---

## 3. Analyse de la Latence (Traces)
*Objectif : Mesurer si la GTX 1050 Ti répond assez vite pour du vocal.*

1.  Allez dans **Paramètres** > **Automatisations et scènes** > **Scripts**.
2.  Trouvez "K-2SO - Générateur de Message" et cliquez sur l'icône **Historique/Traces** (l'horloge entourée d'une flèche).
3.  Sélectionnez votre exécution de test (la plus récente).
4.  Regardez le **Diagramme** (Timeline) :
    *   Cliquez sur le bloc `ai_task.generate_data`.
    *   Regardez le champ `Changed variables` à droite.
    *   **Vérifiez le temps d'exécution** :
        *   🟢 **< 1 seconde** : Parfait (Le modèle est bien en VRAM).
        *   🟡 **1 à 3 secondes** : Acceptable (Latence réseau + petit délai).
        *   🔴 **> 5 secondes** : Problème (Le modèle déborde sur le CPU ou le "Keep Alive" n'a pas fonctionné).

---

## 4. Test du "Keep Alive" (Maintien en mémoire)
*Objectif : Vérifier que le réglage `-1` fonctionne et que le modèle ne se décharge pas.*

1.  Attendez **10 minutes** sans rien demander à l'IA.
2.  Relancez le test de l'étape 2.
3.  Vérifiez la latence via les Traces.
    *   Si c'est **immédiat** : Le `Keep Alive: -1` fonctionne parfaitement.
    *   Si vous avez un délai de **3-4 secondes** (temps de chargement des poids dans le GPU) : Le modèle s'était déchargé.

---

## 5. Test de Résilience (Coupure Réseau)
*Objectif : Valider que vos automatisations ne planteront pas si le NAS redémarre.*

1.  **Action** : Coupez Ollama (ou débranchez le câble réseau du NAS virtuellement/physiquement, ou arrêtez le conteneur Docker).
2.  Relancez le test de l'étape 2.
3.  **Résultat attendu** :
    *   Le script ne doit **PAS** se mettre en erreur rouge.
    *   La variable `generated_message` doit contenir le message de fallback (défini dans le script K-2SO).
    *   *Exemple de fallback* : "Notification système : test_connexion_ollama".

---

## ✅ Checklist de Validation

- [ ] Connexion HA -> NAS OK
- [ ] Réponse K-2SO sur script de test OK
- [ ] Latence < 2s (Indicateur VRAM OK)
- [ ] Réponse rapide même après 10min d'inactivité (Keep Alive OK)
- [ ] Fallback fonctionnel en cas de coupure du service
