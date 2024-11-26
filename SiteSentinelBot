import asyncio
import json
import logging
import os
import sys
from datetime import datetime
import ipaddress
import urllib.parse

from telegram import (
    BotCommand,
    InlineKeyboardButton,
    InlineKeyboardMarkup,
    Update,
)
from telegram.constants import ParseMode
from telegram.ext import (
    Application,
    CallbackQueryHandler,
    CommandHandler,
    ContextTypes,
    ConversationHandler,
    MessageHandler,
    filters,
)
from telegram.error import TelegramError
import aiohttp

# Configuration du logging
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)
logger = logging.getLogger(__name__)

# Chemin du fichier JSON pour le stockage des données
DATA_FILE = "data.json"


def load_data():
    """Charge les données depuis le fichier JSON."""
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as f:
            return json.load(f)
    else:
        return {"sites": [], "bot_token": "", "users": {}}


def save_data():
    """Sauvegarde les données dans le fichier JSON."""
    with open(DATA_FILE, "w") as f:
        json.dump(data, f, indent=4)


# Chargement des données
data = load_data()


def get_user_language(user_id):
    """Récupère la langue préférée de l'utilisateur."""
    user_id = str(user_id)
    if "users" in data and user_id in data["users"]:
        return data["users"][user_id].get("language", "FR")
    else:
        return "FR"  # Français par défaut


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Commande /start pour accueillir l'utilisateur."""
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = (
            f"👋 Bonjour {update.effective_user.first_name} ! Bienvenue sur le bot de surveillance de sites web.\n"
            "Utilisez /help pour voir les commandes disponibles."
        )
    else:
        message = (
            f"👋 Hello {update.effective_user.first_name}! Welcome to the website monitoring bot.\n"
            "Use /help to see the available commands."
        )
    await update.message.reply_text(message)


async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Commande /help pour afficher l'aide."""
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        help_text = (
            "Voici les commandes disponibles :\n"
            "/addsite - Ajouter un site à surveiller\n"
            "/listsites - Lister les sites surveillés\n"
            "/removesite - Supprimer un site de la surveillance\n"
            "/langue - Choisir votre langue préférée\n"
            "/support - Contacter le support technique\n"
            "/help - Afficher cette aide"
        )
    else:
        help_text = (
            "Here are the available commands:\n"
            "/addsite - Add a website to monitor\n"
            "/listsites - List monitored websites\n"
            "/removesite - Remove a website from monitoring\n"
            "/language - Choose your preferred language\n"
            "/support - Contact technical support\n"
            "/help - Display this help message"
        )
    await update.message.reply_text(help_text)


async def support_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Commande /support pour contacter le support."""
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = "Si vous avez besoin d'aide, veuillez contacter le support à l'adresse email support@example.com."
    else:
        message = "If you need assistance, please contact support at support@example.com."
    await update.message.reply_text(message)


# États de la conversation pour /addsite
(
    ADD_SITE_URL,
    ADD_SITE_TYPE,
    ADD_SITE_ERRORS,
    ADD_SITE_KEYWORDS,
    ADD_SITE_KEYWORD_OPTION,
    ADD_SITE_FREQUENCY,
    ADD_SITE_CONFIRMATION,
) = range(7)


async def add_site(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Débute le processus d'ajout d'un site."""
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = "Veuillez entrer l'URL du site que vous souhaitez surveiller."
    else:
        message = "Please enter the URL of the website you want to monitor."
    await update.message.reply_text(message)
    return ADD_SITE_URL  # Passe à l'étape suivante


async def add_site_url(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre l'URL fournie et demande le type de surveillance."""
    url = update.message.text.strip()
    language = get_user_language(update.effective_user.id)

    # Vérification de l'URL
    try:
        # Vérifier si l'entrée est une adresse IP valide
        ipaddress.ip_address(url)
        # C'est une adresse IP valide, ajouter "http://" devant
        url = "http://" + url
    except ValueError:
        # Ce n'est pas une adresse IP, vérifier si l'URL commence par http:// ou https://
        parsed_url = urllib.parse.urlparse(url)
        if not parsed_url.scheme:
            # Le protocole est manquant, ajouter "http://"
            url = "http://" + url
            parsed_url = urllib.parse.urlparse(url)
            if language == "FR":
                message = f"L'URL ne contenait pas de protocole. Nous avons ajouté 'http://' devant : {url}"
            else:
                message = f"The URL did not contain a protocol. We have added 'http://' in front: {url}"
            await update.message.reply_text(message)

    context.user_data["new_site"] = {"url": url, "user_id": update.effective_user.id}
    keyboard = [
        [
            InlineKeyboardButton("Erreurs spécifiques" if language == "FR" else "Specific errors", callback_data="errors"),
            InlineKeyboardButton("Mots clés" if language == "FR" else "Keywords", callback_data="keywords"),
        ]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    if language == "FR":
        message = "Choisissez le type de surveillance :"
    else:
        message = "Choose the type of monitoring:"
    await update.message.reply_text(message, reply_markup=reply_markup)
    return ADD_SITE_TYPE  # Passe à l'étape suivante


async def add_site_type(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre le type de surveillance choisi."""
    query = update.callback_query
    await query.answer()
    choice = query.data
    context.user_data["new_site"]["type"] = choice
    language = get_user_language(update.effective_user.id)
    if choice == "errors":
        if language == "FR":
            message = (
                "Veuillez spécifier les codes d'erreur HTTP à surveiller (séparés par des virgules, ex: 404,500).\n"
                "Voici quelques erreurs courantes :\n"
                "- 404 : Page non trouvée\n"
                "- 500 : Erreur interne du serveur\n"
                "- 503 : Service indisponible\n"
                "- 403 : Accès interdit\n"
                "- 401 : Non autorisé"
            )
        else:
            message = (
                "Please specify the HTTP error codes to monitor (separated by commas, e.g., 404,500).\n"
                "Here are some common errors:\n"
                "- 404: Page not found\n"
                "- 500: Internal server error\n"
                "- 503: Service unavailable\n"
                "- 403: Forbidden\n"
                "- 401: Unauthorized"
            )
        await query.edit_message_text(message)
        return ADD_SITE_ERRORS
    else:
        if language == "FR":
            message = "Veuillez entrer le mot clé à surveiller dans le code source."
        else:
            message = "Please enter the keyword to monitor in the source code."
        await query.edit_message_text(message)
        return ADD_SITE_KEYWORDS


async def add_site_errors(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre les erreurs HTTP à surveiller."""
    errors = update.message.text.split(",")
    context.user_data["new_site"]["errors"] = [error.strip() for error in errors]
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = "À quelle fréquence souhaitez-vous vérifier le site (en secondes) ?"
    else:
        message = "How often would you like to check the site (in seconds)?"
    await update.message.reply_text(message)
    return ADD_SITE_FREQUENCY


async def add_site_keywords(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre le mot clé à surveiller."""
    keyword = update.message.text
    context.user_data["new_site"]["keyword"] = keyword
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        keyboard = [
            [
                InlineKeyboardButton("Mot détecté", callback_data="detected"),
                InlineKeyboardButton("Mot non détecté", callback_data="not_detected"),
            ]
        ]
        message = "Choisissez l'option de notification :"
    else:
        keyboard = [
            [
                InlineKeyboardButton("Keyword detected", callback_data="detected"),
                InlineKeyboardButton("Keyword not detected", callback_data="not_detected"),
            ]
        ]
        message = "Choose the notification option:"
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text(message, reply_markup=reply_markup)
    return ADD_SITE_KEYWORD_OPTION


async def add_site_keyword_option(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre l'option de notification pour le mot clé."""
    query = update.callback_query
    await query.answer()
    option = query.data
    context.user_data["new_site"]["keyword_option"] = option
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = "À quelle fréquence souhaitez-vous vérifier le site (en secondes) ?"
    else:
        message = "How often would you like to check the site (in seconds)?"
    await query.edit_message_text(message)
    return ADD_SITE_FREQUENCY


async def add_site_frequency(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre la fréquence de vérification."""
    language = get_user_language(update.effective_user.id)
    try:
        frequency = int(update.message.text)
        context.user_data["new_site"]["frequency"] = frequency
        # Confirmation des paramètres
        new_site = context.user_data["new_site"]
        if language == "FR":
            summary = f"Veuillez confirmer les paramètres de surveillance :\n\n"
            summary += f"🔗 URL : {new_site['url']}\n"
            if new_site["type"] == "errors":
                summary += f"🚨 Surveillance des erreurs HTTP : {', '.join(new_site['errors'])}\n"
            else:
                summary += f"🔍 Mot clé : {new_site['keyword']}\n"
                option = "détecté" if new_site["keyword_option"] == "detected" else "non détecté"
                summary += f"🛎️ Notification lorsque le mot est {option}\n"
            summary += f"⏱️ Fréquence : {new_site['frequency']} secondes\n"
            keyboard = [
                [
                    InlineKeyboardButton("Confirmer", callback_data="confirm"),
                    InlineKeyboardButton("Annuler", callback_data="cancel"),
                ]
            ]
        else:
            summary = f"Please confirm the monitoring settings:\n\n"
            summary += f"🔗 URL: {new_site['url']}\n"
            if new_site["type"] == "errors":
                summary += f"🚨 Monitoring HTTP errors: {', '.join(new_site['errors'])}\n"
            else:
                summary += f"🔍 Keyword: {new_site['keyword']}\n"
                option = "detected" if new_site["keyword_option"] == "detected" else "not detected"
                summary += f"🛎️ Notify when the keyword is {option}\n"
            summary += f"⏱️ Frequency: {new_site['frequency']} seconds\n"
            keyboard = [
                [
                    InlineKeyboardButton("Confirm", callback_data="confirm"),
                    InlineKeyboardButton("Cancel", callback_data="cancel"),
                ]
            ]
        reply_markup = InlineKeyboardMarkup(keyboard)
        await update.message.reply_text(summary, reply_markup=reply_markup)
        return ADD_SITE_CONFIRMATION
    except ValueError:
        if language == "FR":
            message = "Veuillez entrer un nombre valide pour la fréquence en secondes."
        else:
            message = "Please enter a valid number for the frequency in seconds."
        await update.message.reply_text(message)
        return ADD_SITE_FREQUENCY


async def add_site_confirmation(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Confirme et enregistre le nouveau site à surveiller."""
    query = update.callback_query
    await query.answer()
    language = get_user_language(update.effective_user.id)
    if query.data == "confirm":
        new_site = context.user_data["new_site"]
        new_site["last_checked"] = None
        new_site["status"] = "unknown"
        data["sites"].append(new_site)
        save_data()
        if language == "FR":
            message = "✅ Le site a été ajouté à la surveillance avec succès."
        else:
            message = "✅ The website has been successfully added to monitoring."
        await query.edit_message_text(message)
        # Démarrage de la tâche de surveillance
        context.application.create_task(monitor_site(context, new_site))
    else:
        if language == "FR":
            message = "❌ L'ajout du site a été annulé."
        else:
            message = "❌ The addition of the site has been canceled."
        await query.edit_message_text(message)
    context.user_data.clear()
    return ConversationHandler.END


async def list_sites(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Affiche la liste des sites surveillés."""
    language = get_user_language(update.effective_user.id)
    user_sites = [site for site in data["sites"] if site["user_id"] == update.effective_user.id]
    if not user_sites:
        if language == "FR":
            message = "Aucun site n'est actuellement surveillé."
        else:
            message = "No websites are currently being monitored."
        await update.message.reply_text(message)
        return
    if language == "FR":
        message = "📄 Liste des sites surveillés :\n"
    else:
        message = "📄 List of monitored websites:\n"
    for idx, site in enumerate(user_sites, start=1):
        message += f"\n{idx}. 🔗 URL : {site['url']}\n"
        if site["type"] == "errors":
            if language == "FR":
                message += f"   🚨 Surveillance des erreurs : {', '.join(site['errors'])}\n"
            else:
                message += f"   🚨 Monitoring errors: {', '.join(site['errors'])}\n"
        else:
            if language == "FR":
                message += f"   🔍 Mot clé : {site['keyword']}\n"
                option = "détecté" if site["keyword_option"] == "detected" else "non détecté"
                message += f"   🛎️ Notification lorsque le mot est {option}\n"
            else:
                message += f"   🔍 Keyword: {site['keyword']}\n"
                option = "detected" if site["keyword_option"] == "detected" else "not detected"
                message += f"   🛎️ Notify when keyword is {option}\n"
        if language == "FR":
            message += f"   ⏱️ Fréquence : {site['frequency']} secondes\n"
        else:
            message += f"   ⏱️ Frequency: {site['frequency']} seconds\n"
    await update.message.reply_text(message)


# États de la conversation pour /removesite
(REMOVE_SITE_SELECTION, REMOVE_SITE_CONFIRMATION) = range(2)


async def remove_site(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Débute le processus de suppression d'un site."""
    language = get_user_language(update.effective_user.id)
    user_sites = [site for site in data["sites"] if site["user_id"] == update.effective_user.id]
    if not user_sites:
        if language == "FR":
            message = "Aucun site n'est actuellement surveillé."
        else:
            message = "No websites are currently being monitored."
        await update.message.reply_text(message)
        return ConversationHandler.END
    keyboard = []
    for idx, site in enumerate(user_sites, start=1):
        keyboard.append(
            [InlineKeyboardButton(f"{idx}. {site['url']}", callback_data=str(idx - 1))]
        )
    reply_markup = InlineKeyboardMarkup(keyboard)
    if language == "FR":
        message = "Sélectionnez le site à supprimer :"
    else:
        message = "Select the website to remove:"
    await update.message.reply_text(message, reply_markup=reply_markup)
    context.user_data["user_sites"] = user_sites
    return REMOVE_SITE_SELECTION


async def remove_site_selection(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Affiche les détails du site sélectionné pour confirmation."""
    query = update.callback_query
    await query.answer()
    site_idx = int(query.data)
    user_sites = context.user_data["user_sites"]
    site = user_sites[site_idx]
    context.user_data["remove_site"] = site
    language = get_user_language(update.effective_user.id)
    if language == "FR":
        message = f"Vous avez sélectionné le site suivant :\n\n"
        message += f"🔗 URL : {site['url']}\n"
        if site["type"] == "errors":
            message += f"🚨 Surveillance des erreurs : {', '.join(site['errors'])}\n"
        else:
            message += f"🔍 Mot clé : {site['keyword']}\n"
            option = "détecté" if site["keyword_option"] == "detected" else "non détecté"
            message += f"🛎️ Notification lorsque le mot est {option}\n"
        message += f"⏱️ Fréquence : {site['frequency']} secondes\n\n"
        message += "Êtes-vous sûr de vouloir supprimer ce site ?"
        keyboard = [
            [
                InlineKeyboardButton("Oui", callback_data="yes"),
                InlineKeyboardButton("Non", callback_data="no"),
            ]
        ]
    else:
        message = f"You have selected the following website:\n\n"
        message += f"🔗 URL: {site['url']}\n"
        if site["type"] == "errors":
            message += f"🚨 Monitoring errors: {', '.join(site['errors'])}\n"
        else:
            message += f"🔍 Keyword: {site['keyword']}\n"
            option = "detected" if site["keyword_option"] == "detected" else "not detected"
            message += f"🛎️ Notify when keyword is {option}\n"
        message += f"⏱️ Frequency: {site['frequency']} seconds\n\n"
        message += "Are you sure you want to remove this website?"
        keyboard = [
            [
                InlineKeyboardButton("Yes", callback_data="yes"),
                InlineKeyboardButton("No", callback_data="no"),
            ]
        ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_text(message, reply_markup=reply_markup)
    return REMOVE_SITE_CONFIRMATION


async def remove_site_confirmation(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Confirme et supprime le site sélectionné."""
    query = update.callback_query
    await query.answer()
    language = get_user_language(update.effective_user.id)
    if query.data == "yes":
        site = context.user_data["remove_site"]
        data["sites"].remove(site)
        save_data()
        if language == "FR":
            message = f"✅ La surveillance du site {site['url']} a été supprimée avec succès."
        else:
            message = f"✅ Monitoring of the site {site['url']} has been successfully removed."
        await query.edit_message_text(message)
    else:
        if language == "FR":
            message = "❌ La suppression du site a été annulée."
        else:
            message = "❌ The removal of the site has been canceled."
        await query.edit_message_text(message)
    context.user_data.clear()
    return ConversationHandler.END


async def monitor_site(context: ContextTypes.DEFAULT_TYPE, site):
    """Tâche asynchrone pour surveiller un site spécifique."""
    while True:
        try:
            async with aiohttp.ClientSession() as session:
                async with session.get(site["url"]) as response:
                    content = await response.text()
                    status_code = response.status
        except Exception as e:
            logger.error(f"Erreur lors de la vérification du site {site['url']}: {e}")
            status_code = None
            content = ""

        now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        site["last_checked"] = now
        save_data()

        user_id = site["user_id"]
        language = get_user_language(user_id)
        if site["type"] == "errors":
            if status_code and str(status_code) in site["errors"]:
                if site["status"] != "error":
                    site["status"] = "error"
                    if language == "FR":
                        message = (
                            f"🚨 *Anomalie détectée*\n\n"
                            f"🔗 URL : {site['url']}\n"
                            f"❗ Erreur HTTP {status_code}\n"
                            f"⏰ Horodatage : {now}"
                        )
                    else:
                        message = (
                            f"🚨 *Anomaly detected*\n\n"
                            f"🔗 URL: {site['url']}\n"
                            f"❗ HTTP Error {status_code}\n"
                            f"⏰ Timestamp: {now}"
                        )
                    await context.bot.send_message(
                        chat_id=user_id, text=message, parse_mode=ParseMode.MARKDOWN
                    )
            else:
                if site["status"] == "error":
                    site["status"] = "ok"
                    if language == "FR":
                        message = (
                            f"✅ *Retour à la normale*\n\n"
                            f"🔗 URL : {site['url']}\n"
                            f"🟢 Le site est de nouveau accessible.\n"
                            f"⏰ Horodatage : {now}"
                        )
                    else:
                        message = (
                            f"✅ *Back to normal*\n\n"
                            f"🔗 URL: {site['url']}\n"
                            f"🟢 The site is accessible again.\n"
                            f"⏰ Timestamp: {now}"
                        )
                    await context.bot.send_message(
                        chat_id=user_id, text=message, parse_mode=ParseMode.MARKDOWN
                    )
        else:
            keyword_present = site["keyword"] in content
            should_notify = False
            if site["keyword_option"] == "detected" and keyword_present and site["status"] != "keyword_detected":
                site["status"] = "keyword_detected"
                should_notify = True
            elif site["keyword_option"] == "not_detected" and not keyword_present and site["status"] != "keyword_missing":
                site["status"] = "keyword_missing"
                should_notify = True
            elif (site["keyword_option"] == "detected" and not keyword_present and site["status"] == "keyword_detected") or (
                site["keyword_option"] == "not_detected" and keyword_present and site["status"] == "keyword_missing"
            ):
                site["status"] = "ok"
                should_notify = True

            if should_notify:
                if site["status"] in ["keyword_detected", "keyword_missing"]:
                    if language == "FR":
                        state = "détecté" if site["status"] == "keyword_detected" else "non détecté"
                        message = (
                            f"🚨 *Anomalie détectée*\n\n"
                            f"🔗 URL : {site['url']}\n"
                            f"❗ Le mot clé '{site['keyword']}' est {state}.\n"
                            f"⏰ Horodatage : {now}"
                        )
                    else:
                        state_en = "detected" if site["status"] == "keyword_detected" else "not detected"
                        message = (
                            f"🚨 *Anomaly detected*\n\n"
                            f"🔗 URL: {site['url']}\n"
                            f"❗ The keyword '{site['keyword']}' is {state_en}.\n"
                            f"⏰ Timestamp: {now}"
                        )
                else:
                    if language == "FR":
                        message = (
                            f"✅ *Retour à la normale*\n\n"
                            f"🔗 URL : {site['url']}\n"
                            f"🟢 Le statut du mot clé est revenu à la normale.\n"
                            f"⏰ Horodatage : {now}"
                        )
                    else:
                        message = (
                            f"✅ *Back to normal*\n\n"
                            f"🔗 URL: {site['url']}\n"
                            f"🟢 The keyword status has returned to normal.\n"
                            f"⏰ Timestamp: {now}"
                        )
                await context.bot.send_message(
                    chat_id=user_id, text=message, parse_mode=ParseMode.MARKDOWN
                )

        await asyncio.sleep(site["frequency"])


async def set_language(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Commande /langue pour que l'utilisateur choisisse sa langue préférée."""
    keyboard = [
        [
            InlineKeyboardButton("Français", callback_data="FR"),
            InlineKeyboardButton("English", callback_data="EN"),
        ]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    await update.message.reply_text(
        "Choisissez votre langue préférée / Choose your preferred language:",
        reply_markup=reply_markup,
    )
    return 0  # État pour la sélection de la langue


async def language_selection(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Enregistre la langue choisie par l'utilisateur."""
    query = update.callback_query
    await query.answer()
    language = query.data
    user_id = str(update.effective_user.id)
    if "users" not in data:
        data["users"] = {}
    data["users"][user_id] = {"language": language}
    save_data()
    if language == "FR":
        message = "Votre langue préférée a été définie sur Français."
    else:
        message = "Your preferred language has been set to English."
    await query.edit_message_text(message)
    return ConversationHandler.END


async def post_init(application: Application):
    """Fonction asynchrone exécutée après l'initialisation de l'application."""
    # Définition des commandes du bot
    commands = [
        BotCommand("start", "Démarrer le bot / Start the bot"),
        BotCommand("help", "Obtenir de l'aide / Get help"),
        BotCommand("addsite", "Ajouter un site / Add a site"),
        BotCommand("listsites", "Lister les sites / List sites"),
        BotCommand("removesite", "Supprimer un site / Remove a site"),
        BotCommand("langue", "Choisir la langue / Choose language"),
        BotCommand("support", "Contacter le support / Contact support"),
    ]
    await application.bot.set_my_commands(commands)

    # Démarrage des tâches de surveillance pour les sites déjà enregistrés
    for site in data["sites"]:
        application.create_task(monitor_site(application, site))


def main():
    """Démarre le bot."""
    try:
        # Vérification du token du bot
        if not data.get("bot_token"):
            token = input("Veuillez entrer le token de votre bot Telegram : ").strip()
            data["bot_token"] = token
            save_data()
        else:
            token = data["bot_token"]

        application = Application.builder().token(token).post_init(post_init).build()

        # Gestionnaires de commandes
        application.add_handler(CommandHandler("start", start))
        application.add_handler(CommandHandler("help", help_command))
        application.add_handler(CommandHandler("support", support_command))
        application.add_handler(CommandHandler("listsites", list_sites))

        # Gestionnaire pour /addsite
        add_site_handler = ConversationHandler(
            entry_points=[CommandHandler("addsite", add_site)],
            states={
                ADD_SITE_URL: [MessageHandler(filters.TEXT & ~filters.COMMAND, add_site_url)],
                ADD_SITE_TYPE: [CallbackQueryHandler(add_site_type)],
                ADD_SITE_ERRORS: [MessageHandler(filters.TEXT & ~filters.COMMAND, add_site_errors)],
                ADD_SITE_KEYWORDS: [MessageHandler(filters.TEXT & ~filters.COMMAND, add_site_keywords)],
                ADD_SITE_KEYWORD_OPTION: [CallbackQueryHandler(add_site_keyword_option)],
                ADD_SITE_FREQUENCY: [MessageHandler(filters.TEXT & ~filters.COMMAND, add_site_frequency)],
                ADD_SITE_CONFIRMATION: [CallbackQueryHandler(add_site_confirmation)],
            },
            fallbacks=[CommandHandler("cancel", lambda update, context: ConversationHandler.END)],
        )
        application.add_handler(add_site_handler)

        # Gestionnaire pour /removesite
        remove_site_handler = ConversationHandler(
            entry_points=[CommandHandler("removesite", remove_site)],
            states={
                REMOVE_SITE_SELECTION: [CallbackQueryHandler(remove_site_selection)],
                REMOVE_SITE_CONFIRMATION: [CallbackQueryHandler(remove_site_confirmation)],
            },
            fallbacks=[CommandHandler("cancel", lambda update, context: ConversationHandler.END)],
        )
        application.add_handler(remove_site_handler)

        # Gestionnaire pour /langue
        language_handler = ConversationHandler(
            entry_points=[CommandHandler("langue", set_language)],
            states={
                0: [CallbackQueryHandler(language_selection)],
            },
            fallbacks=[CommandHandler("cancel", lambda update, context: ConversationHandler.END)],
        )
        application.add_handler(language_handler)

        # Démarrage du bot
        application.run_polling()
    except TelegramError as e:
        logger.error(f"Erreur Telegram : {e}")
    except Exception as e:
        logger.error(f"Erreur inattendue : {e}")
        sys.exit(1)


if __name__ == "__main__":
    main()
