# 🛠️ Guide d'installation pour HACF (Provisoire)

Ce document t'explique étape par étape comment partager ton projet et comment l'installer proprement.

## 1. La Genèse (À mettre dans ton post)
"À force de jouer avec l'IA, je me retrouvais avec de nombreuses automatisations où je devais redéfinir à chaque fois le prompt et le contexte. C'était sympa au début, mais c'est vite devenu une galère à maintenir. Comme j'adore les scripts, j'ai tout centralisé pour simplifier mon code et gagner en robustesse."

## 2. Le Script YAML (Copier-Coller pour ton post)
Voici la version la plus claire possible, avec des commentaires expliquant chaque étape pour la communauté.

```yaml
# ID unique du script pour vos automatisations
k_2so_generateur_de_message:
  alias: K-2SO - Générateur de Message
  description: 'Génère un message sarcastique style K-2SO via l''IA'
  
  # Configuration des entrées via l'UI de Home Assistant
  fields:
    mission:
      name: mission
      description: 'Le contexte (ex: cafe, volets, batterie)'
      required: true
      selector:
        text: {}
    details:
      name: 'détails'
      description: 'Données brutes (ex: 15%, nom de la porte)'
      required: false
      selector:
        text: {}
    consigne:
      name: 'consigne spécifique'
      description: 'Nuance pour l''IA (ex: sois plus insistant)'
      required: false
      selector:
        text: {}

  sequence:
    # ÉTAPE 1 : Appel au service IA
    - action: ai_task.generate_data
      continue_on_error: true # On continue même si l'IA plante
      data:
        task_name: 'K-2SO Persona - {{ mission }}'
        instructions: >
          # TON ET PERSONNALITÉ
          Tu es K-2SO de Star Wars. Ton ton est factuel, direct et sarcastique.
          Pas d'émoji, phrases courtes pour le TTS.

          # CONTEXTE DE LA MISSION
          L'action : {{ mission }} 
          {% if details is defined and details != '' %} INFOS : {{ details }} {% endif %}
          {% if consigne is defined and consigne != '' %} NOTE : {{ consigne }} {% endif %}

          Génère la phrase adaptée.
      response_variable: raw_ai_response

    # ÉTAPE 2 : Préparation de la réponse (IA ou Secours)
    - variables:
        # Messages de secours si l'IA est hors-ligne
        fallback_msg: >
          {% set msgs = {
            'cafe_pret': "Le café est prêt. Je suppose que vous en avez besoin.",
            'batterie_faible': "Niveau de batterie faible (" ~ details ~ ").",
            'porte_frigo_ouverte': "Alerte : porte de réfrigérateur ouverte."
          } %} {{ msgs.get(mission, "Notification : " ~ mission) }}
        
        # Choix final de la variable
        generated_message:
          data: >
            {% if raw_ai_response is defined and raw_ai_response.data is defined and raw_ai_response.data | length > 0 %}
              {{ raw_ai_response.data }}
            {% else %}
              {{ fallback_msg }}
            {% endif %}

    # ÉTAPE 3 : On renvoie le message à l'automatisation qui l'a appelé
    - stop: Message généré
      response_variable: generated_message
```

## 3. Comment utiliser ce script dans son auto ?
C'est très simple, on appelle le script et on récupère la réponse dans une variable.

```yaml
- action: script.k_2so_generateur_de_message
  data:
    mission: "machine à laver"
    consigne: "Dis-lui que ça va sentir le poney mort s'il attend trop."
  response_variable: generated_message

- action: notify.alexa_media
  data:
    message: "{{ generated_message.data }}"
    target: media_player.salon
```

---
**Astuce HACF** : N'hésite pas à dire que ça marche avec OpenAI, Google Gemini ou même Ollama en local !
