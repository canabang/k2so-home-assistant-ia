# 🤖 Guide d'utilisation : Script IA K-2SO

Ce script centralise la personnalité de **K-2SO** (Star Wars) pour toutes vos notifications Home Assistant. Il utilise l'IA pour générer des messages sarcastiques et factuels, avec un système de secours (fallback) intégré.

## 📜 Genèse du projet
À force de jouer avec l'IA, je me retrouvais avec de nombreuses automatisations où je devais redéfinir à chaque fois le prompt, le contexte et les règles de personnalité de K-2SO. C'était devenu une véritable galère à maintenir dès que je voulais changer un petit détail. 

En packagant tout ça dans un script unique :
1. **Maintenance simplifiée** : On change le prompt à un seul endroit.
2. **Code propre** : Mes automatisations font 3 lignes au lieu de 50.
3. **Robustesse** : J'ai pu ajouter une gestion d'erreur et un fallback global.

## 🧠 Le choix du cerveau (Cloud vs Local)
Ce script est agnostique : il fonctionne aussi bien avec des solutions Cloud (**Gemini**, **OpenAI**) qu'avec du Local LLM.

Cependant, j'ai migré d'une solution Cloud vers du **100% Local** via **Ollama** et le modèle **Llama 3.2** :
- **Confidentialité** : Aucune donnée de votre maison (températures, présence, usages) ne quitte votre réseau.
- **Accessibilité** : Llama 3.2 tourne de façon fluide même sur une **petite carte graphique** (ex: NVIDIA GTX 1050 Ti 4GB).
- **Latence réelle** : Comptez environ **2.5 secondes** pour la génération complète sur ce type de matériel (ce qui reste très acceptable pour du local).

## ⚙️ Configuration Infrastructure (Ollama)
Pour faire tourner efficacement l'IA sur un matériel modeste, voici les paramètres recommandés pour votre serveur **Ollama** :

### Variables d'environnement
Afin d'éviter les surcharges de VRAM et garantir une réponse stable :
```bash
OLLAMA_MAX_LOADED_MODELS=1
OLLAMA_NUM_PARALLEL=1
```

### Paramètre de modèle
Utilisez `keep_alive: -1` dans l'intégration Home Assistant (ou via l'API) pour que le modèle reste chargé en mémoire vidéo, supprimant ainsi le temps de chargement à chaque requête.

## 🛠️ Le Script Central
Le script est situé dans `scripts.yaml` sous l'ID `k_2so_generateur_de_message`.

### Paramètres (Champs)
| Champ | Description | Exemple |
| :--- | :--- | :--- |
| `mission` | L'action ou l'événement | `cafe`, `batterie`, `volets` |
| `details` | Données brutes à intégrer | `15%`, `Frigo, 6 minutes`, `{{ variable }}` |
| `consigne` | Ordre impératif (priorité absolue) | `IMPÉRATIF : MAX 10 MOTS. Sarcastique.` |

---

## 🚀 Exemples d'utilisation

### 1. Machine à Laver (Fin de cycle)
Utilise une consigne pour ajouter une pression sur l'odeur du linge.
```yaml
action: script.k_2so_generateur_de_message
data:
  mission: machine à laver finie
  consigne: "Mentionne subtilement que le linge va finir par sentir mauvais si Gaël ne se bouge pas."
response_variable: generated_message
```

### 2. Portes du Frigo (Alerte dynamique)
Utilise du code Jinja pour changer la consigne selon la durée d'ouverture.
```yaml
action: script.k_2so_generateur_de_message
data:
  mission: porte_frigo_ouverte
  details: "{{ portes_info | map(attribute='nom') | join(', ') }}, {{ duree_max }} minutes"
  consigne: >
    {% if duree_max > 5 %} 
      Sois beaucoup plus alarmiste et sarcastique sur le gaspillage énergétique. 
    {% else %} 
      Sois simplement direct et moqueur sur l'oubli. 
    {% endif %}
response_variable: generated_message
```

### 3. État de la Batterie
Transmet simplement le pourcentage dans les détails.
```yaml
action: script.k_2so_generateur_de_message
data:
  mission: batterie_faible
  details: "{{ states('sensor.tricordeur_14_battery_level') }}%"
response_variable: generated_message
```

---

## 💡 Maîtriser la concision (Voice Assistant)
Le script est conçu pour être loquace par défaut (idéal pour Discord). Cependant, pour une utilisation **vocale** (Alexa/Google), la `consigne` est traitée comme une **priorité absolue**.

Pour forcer K-2SO à être bref, utilisez des mots-clés impératifs :
- `"IMPÉRATIF : MAXIMUM 10 MOTS. Sarcastique."`
- `"ORDRE : SOIS TRÈS BREF. Style militaire."`
- `"STRICTEMENT 5 MOTS MAX."`

## 🛡️ Sécurité (Fallback)
Le script contient un dictionnaire de messages prédéfinis. Si l'IA rencontre une erreur ou est indisponible, il renverra automatiquement un message cohérent basé sur la `mission` fournie.
