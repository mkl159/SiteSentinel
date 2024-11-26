# SiteSentinel

Un bot Telegram multilingue (FR/EN) pour surveiller vos sites web et recevoir des alertes en cas d'erreurs HTTP spécifiques ou de changements de mots-clés détectés. Chaque utilisateur peut gérer ses propres alertes de manière indépendante.

## Fonctionnalités

- **Surveillance des sites web** : Surveillez les sites web pour des erreurs HTTP spécifiques ou la présence/absence de mots-clés.
- **Notifications en temps réel** : Recevez des alertes instantanées via Telegram en cas d'anomalies détectées.
- **Multilingue** : Choisissez entre le français et l'anglais comme langue préférée.
- **Multiutilisateur** : Chaque utilisateur peut gérer ses propres sites web et alertes de manière indépendante.
- **Interface conviviale** : Interaction facile avec le bot via des commandes simples.

## Prérequis

- **Python 3.7** ou supérieur.
- Un compte Telegram.
- Un bot Telegram (instructions ci-dessous).

## Installation

### 1. Cloner le dépôt

```bash
git clone https://github.com/votre-utilisateur/SiteSentinel.git
cd SiteSentinel
```

### 2. Installer les dépendances

Assurez-vous d'avoir **pip** installé. Ensuite, exécutez :

```bash
pip install -r requirements.txt
```

### 3. Créer un bot Telegram

1. **Ouvrez Telegram** et recherchez **[@BotFather](https://t.me/BotFather)**.
2. Démarrez une conversation et envoyez la commande `/start`.
3. Envoyez la commande `/newbot` pour créer un nouveau bot.
4. Suivez les instructions pour donner un **nom** et un **nom d'utilisateur** à votre bot.
   - Le nom d'utilisateur doit se terminer par `bot` (par exemple, `SiteSentinelBot`).
5. Après la création, **BotFather** vous fournira un **token** d'accès HTTP API.
   - **Gardez ce token en lieu sûr**, vous en aurez besoin pour configurer votre bot.

### 4. Configurer le bot

Lors de la première exécution du script, vous serez invité à entrer le token de votre bot :

```bash
python SiteSentinel.py
```

Entrez le token que vous avez reçu de **BotFather**.

Le token sera enregistré dans le fichier `data.json` pour les exécutions futures.

## Utilisation

### 1. Démarrer le bot

Exécutez le script :

```bash
python BotWeb.py
```

### 2. Interagir avec le bot

- Sur Telegram, recherchez votre bot en utilisant le **nom d'utilisateur** que vous avez défini (par exemple, `@SiteSentinelBot`).
- Démarrez une conversation en cliquant sur **Démarrer** ou en envoyant la commande `/start`.

### 3. Commandes disponibles

- `/start` : Démarrer le bot et afficher un message de bienvenue.
- `/help` : Afficher l'aide et la liste des commandes disponibles.
- `/addsite` : Ajouter un site web à surveiller.
- `/listsites` : Lister les sites que vous surveillez.
- `/removesite` : Supprimer un site de la surveillance.
- `/langue` : Choisir votre langue préférée (FR/EN).
- `/support` : Contacter le support technique.

### 4. Ajouter un site à surveiller

1. Envoyez la commande `/addsite`.
2. **Entrez l'URL** du site que vous souhaitez surveiller.
   - Si vous oubliez d'inclure `http://` ou `https://`, le bot l'ajoutera automatiquement.
3. **Choisissez le type de surveillance** :
   - **Erreurs spécifiques** : Surveiller des codes d'erreur HTTP spécifiques (par exemple, 404, 500).
   - **Mots clés** : Surveiller la présence ou l'absence d'un mot clé spécifique dans le code source du site.
4. Si vous choisissez **Erreurs spécifiques** :
   - **Spécifiez les codes d'erreur HTTP** à surveiller (séparés par des virgules).
   - Exemples d'erreurs courantes :
     - `404` : Page non trouvée
     - `500` : Erreur interne du serveur
     - `503` : Service indisponible
5. Si vous choisissez **Mots clés** :
   - **Entrez le mot clé** à surveiller.
   - **Choisissez l'option de notification** :
     - **Mot détecté** : Être notifié lorsque le mot clé est détecté.
     - **Mot non détecté** : Être notifié lorsque le mot clé n'est plus détecté.
6. **Indiquez la fréquence de vérification** (en secondes).
7. **Confirmez les paramètres** pour commencer la surveillance.

### 5. Lister les sites surveillés

- Envoyez la commande `/listsites` pour afficher la liste des sites que vous surveillez.

### 6. Supprimer un site de la surveillance

- Envoyez la commande `/removesite` et suivez les instructions pour supprimer un site de la surveillance.

## Contribution

Contributions are welcome! Feel free to:

- **Fork** the repository
- **Create** issues to report bugs or suggest improvements
- **Submit** pull requests to share your changes

## Ressources additionnelles

- Découvrez la vidéo explicative sur YouTube : [Short SiteSentinel](https://youtube.com/shorts/Tc08WS2iAqk?si=-AnmXAUljfQAMCMc)

**Have fun with SiteSentinel!** 🚀
