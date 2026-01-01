# 🤖 Guide d'utilisation : Script IA K-2SO

Ce script centralise la personnalité de **K-2SO** (Star Wars) pour toutes vos notifications Home Assistant. Il utilise l'IA pour générer des messages sarcastiques et factuels, avec un système de secours (fallback) intégré.

## 🛠️ Le Script Central
Le script est situé dans `scripts.yaml` sous l'ID `k_2so_generateur_de_message`.

### Paramètres (Champs)
| Champ | Description | Exemple |
| :--- | :--- | :--- |
| `mission` | L'action ou l'événement | `cafe`, `batterie`, `volets` |
| `details` | Données brutes à intégrer | `15%`, `Frigo, 6 minutes`, `{{ variable }}` |
| `consigne` | Nuance spécifique pour l'IA | `sois très alarmiste`, `insulte son orgueil` |

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

## 💡 Astuces pour la créativité
Pour que l'IA soit plus créative, n'hésitez pas à remplir le champ `consigne` avec des ordres comme :
- *"Fais une référence à l'Empire."*
- *"Sois particulièrement condescendant sur la mémoire de l'utilisateur."*
- *"Dis-le comme si c'était la fin du monde."*

## 🛡️ Sécurité (Fallback)
Le script contient un dictionnaire de messages prédéfinis. Si l'IA rencontre une erreur ou est indisponible, il renverra automatiquement un message cohérent basé sur la `mission` fournie.
