pip install python-telegram-bot
from telegram import Update
from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, CallbackContext
import logging

# Configurer le logging pour surveiller les erreurs et les activités
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logger = logging.getLogger(__name__)

# Instancier ton algorithme ici
systeme = SystemeAchatEtoiles(id_fondateur="fondateur_123")

# Fonction pour démarrer le bot
def start(update: Update, context: CallbackContext):
    update.message.reply_text('Bienvenue dans le bot de gestion des étoiles!')

# Fonction pour acheter des étoiles
def acheter_etoiles(update: Update, context: CallbackContext):
    try:
        user_id = update.message.chat_id
        etoiles_achetees = int(context.args[0])  # Récupère la quantité d'étoiles achetées
        systeme.acheter_etoiles(user_id, etoiles_achetees)
        update.message.reply_text(f"Vous avez acheté {etoiles_achetees} étoiles !")
    except (IndexError, ValueError):
        update.message.reply_text('Usage: /acheter_etoiles <nombre>')

# Fonction pour redistribuer les étoiles
def redistribuer_cagnotte(update: Update, context: CallbackContext):
    systeme.redistribuer_cagnotte()
    update.message.reply_text("La cagnotte a été redistribuée.")

# Fonction pour afficher l'état actuel
def afficher_etat(update: Update, context: CallbackContext):
    systeme.afficher_etat()
    update.message.reply_text(f"Cagnotte actuelle : {systeme.cagnotte_etoiles} étoiles")

# Gestion des erreurs
def error(update: Update, context: CallbackContext):
    """Log les erreurs causées par les mises à jour."""
    logger.warning(f'Update "{update}" a causé une erreur "{context.error}"')

def main():
    # Insère ton Token API ici
    TOKEN = '7846731127:AAGhl9XRLYUeld2-IqL2iXprnXBVIqhwK5U'

    # Crée un Updater pour gérer le bot
    updater = Updater(TOKEN, use_context=True)

    # Récupère le dispatcher pour enregistrer les gestionnaires
    dp = updater.dispatcher

    # Enregistre les commandes
    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("acheter_etoiles", acheter_etoiles))
    dp.add_handler(CommandHandler("redistribuer_cagnotte", redistribuer_cagnotte))
    dp.add_handler(CommandHandler("afficher_etat", afficher_etat))

    # Log toutes les erreurs
    dp.add_error_handler(error)

    # Démarre le bot
    updater.start_polling()

    # Arrête le bot proprement en cas d'interruption
    updater.idle()

if __name__ == '__main__':
    main()
class SystemeAchatEtoiles:
    def __init__(self, id_fondateur):
        self.id_fondateur = id_fondateur
        self.cagnotte_etoiles = 0
        self.membres = {}
        self.contributeurs = []

    # Fonction d'achat d'étoiles
    def acheter_etoiles(self, id_utilisateur, etoiles_achetees):
        # Allocation des étoiles
        etoiles_utilisateur = etoiles_achetees * 0.50  # 50% pour l'utilisateur
        etoiles_cagnotte = etoiles_achetees * 0.50  # 50% pour la cagnotte
        self.cagnotte_etoiles += etoiles_cagnotte

        # Enregistrement des étoiles de l'utilisateur
        if id_utilisateur in self.membres:
            self.membres[id_utilisateur] += etoiles_utilisateur
        else:
            self.membres[id_utilisateur] = etoiles_utilisateur

        # Ajouter l'utilisateur aux contributeurs
        if id_utilisateur not in self.contributeurs:
            self.contributeurs.append(id_utilisateur)

        print(f"Utilisateur {id_utilisateur} a acheté {etoiles_achetees} étoiles.")

    # Fonction de redistribution de la cagnotte
    def redistribuer_cagnotte(self):
        # Trier les contributeurs selon les étoiles possédées
        top_contributeurs = sorted(self.membres.items(), key=lambda x: x[1], reverse=True)[:3]

        # 70% pour le fondateur
        etoiles_fondateur = self.cagnotte_etoiles * 0.70
        print(f"Le fondateur ({self.id_fondateur}) reçoit {etoiles_fondateur} étoiles.")

        # 30% pour les 3 plus grands contributeurs (10% chacun)
        etoiles_contributeurs = self.cagnotte_etoiles * 0.30
        for contributeur, _ in top_contributeurs:
            etoiles_partage = etoiles_contributeurs / 3
            print(f"Le contributeur {contributeur} reçoit {etoiles_partage} étoiles.")

        # Réinitialiser la cagnotte après redistribution
        self.cagnotte_etoiles = 0

    # Fonction pour afficher l'état actuel
    def afficher_etat(self):
        print(f"Cagnotte actuelle : {self.cagnotte_etoiles} étoiles")
        print(f"Membres : {self.membres}")

# Exemple d'utilisation
if __name__ == "__main__":
    # Créer un système d'achat d'étoiles avec un fondateur
    systeme = SystemeAchatEtoiles(id_fondateur="fondateur_123")

    # Simuler des achats d'étoiles par différents utilisateurs
    systeme.acheter_etoiles("user_1", 100)
    systeme.acheter_etoiles("user_2", 200)
    systeme.acheter_etoiles("user_3", 150)
    systeme.acheter_etoiles("user_4", 50)

    # Afficher l'état actuel
    systeme.afficher_etat()

    # Redistribuer la cagnotte
    systeme.redistribuer_cagnotte()

    # Afficher l'état après redistribution
    systeme.afficher_etat()
