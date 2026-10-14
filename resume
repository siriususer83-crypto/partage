Absolument. C'est un document très structuré et bien pensé. Avant de plonger dans le code, il est essentiel de bien comprendre les tenants et aboutissants. Voici une **analyse complète et détaillée** du cahier des charges, section par section, avec les points clés, les pièges à éviter et les questions à poser.

---

### Analyse Complète du Cahier des Charges : Assistant IA Sirius Digital

#### 1. Introduction (Pages 4-5)

*   **Contexte :** Sirius Digital édite MairiePlus (SaaS de gestion des contribuables). Le projet est de créer un **assistant IA indépendant**, hébergé sur `assistant.siriusdigital.tech`, capable de se brancher sur n'importe quel projet de l'entreprise.
*   **Vision :** Un service unique, modulable, avec un panneau d'administration centralisé. L'assistant est un **service à part**, pas un module du SaaS.
*   **Objectifs :**
    *   **Fiabilité :** Ne jamais inventer d'informations. Répondre uniquement à partir de sources publiques (données du SaaS, contenu publié).
    *   **Sécurité :** Protéger l'assistant public contre les abus (robots, spam) et garantir l'isolation des données entre communes.
    *   **Contrôle :** Pouvoir activer/couper l'assistant projet par projet.
    *   **Mesure :** Mettre en place un journal d'usage dès le premier jour pour préparer un futur modèle à crédits.
*   **Hors Périmètre (V1) :** Pas de RH, pas de facturation, pas d'actions modifiant les données, pas d'accès des contribuables à leurs propres données.
*   **Décisions Arrêtées :** Architecture séparée, compte assistant par commune, double vérification des droits, phasage en 3 versions.
*   **Stack Technique :**
    *   **Service Assistant :** Laravel (Blade).
    *   **Base de données :** Aucun moteur imposé. Le code doit être **portable** (MySQL, PostgreSQL, etc.). Utiliser les migrations Laravel et Eloquent.
    *   **Fournisseur IA :** Non choisi. L'application doit utiliser une **interface unique** (`FournisseurIA`) pour pouvoir en changer facilement.
    *   **Côté MairiePlus :** Laravel 10, Spatie Permissions (mode teams), Sanctum.

#### 2. Architecture Générale (Pages 6-8)

*   **Vue d'ensemble :** Deux mondes : le **Service Assistant** (à construire) et le **SaaS** (MairiePlus). L'utilisateur interagit avec le SaaS, qui appelle l'API de l'Assistant.
*   **Règle Absolue :** **Jamais de connexion directe à la base de données du SaaS.** Tout passe par l'API du SaaS.
*   **Fournisseur IA Interchangeable :** Utiliser une interface (`FournisseurIA`). Créer un **faux fournisseur** pour les tests (simuler des réponses, des lenteurs, des pannes).
*   **Compatibilité BDD :** Le code doit être portable. Éviter les spécificités d'un moteur (ex: `ILIKE` de PostgreSQL). Utiliser `LOWER` et `LIKE`, des colonnes texte plutôt que `ENUM`, le cast `array` pour le JSON, etc.
*   **Pannes et Lenteurs du Fournisseur IA :**
    *   **Délai maximum :** Configurer un timeout pour chaque appel.
    *   **Une seule nouvelle tentative :** Uniquement pour les erreurs temporaires (timeout, surcharge).
    *   **Message poli :** En cas d'échec, afficher un message générique ("L'assistant rencontre une difficulté...").
    *   **Disjoncteur :** Après plusieurs échecs, suspendre les appels pendant quelques minutes.
    *   **Fournisseur de secours :** Optionnel, à décider par la direction.
    *   **Coût :** Une erreur ne compte pas dans le quota. Une ligne d'usage avec le résultat "erreur" est écrite.
*   **Affichage Progressif (Streaming) :** La réponse doit s'afficher au fur et à mesure. Un widget léger (quelques dizaines de ko) doit être intégré au site public.

#### 3. Les Niveaux d'Accès et les Capacités (Pages 8-9)

*   **Pourquoi des niveaux ?** Pour réguler l'accès aux données : public (niveau 1), aide à l'usage (niveau 2), consultation de données (niveau 3).
*   **Les Niveaux :**
    *   **Niveau 1 (Public) :** Visiteur anonyme. Répond avec le contenu publié et les tarifs publics. **Aucune donnée métier.**
    *   **Niveau 2 (Aide à l'usage) :** Agent connecté. Explique comment utiliser le logiciel (documentation). **Aucune donnée métier.**
    *   **Niveau 3 (Données) :** Agent connecté avec permissions. Consulte les informations du SaaS en **lecture seule**. Les données vues dépendent des droits de l'agent.
*   **Les Capacités :** C'est une **liste blanche**. Ce qui n'est pas dans la liste est interdit.
    *   Exemples : `repondre_public` (niveau 1), `consulter_tarifs` (niveau 1), `aide_usage` (niveau 2), `consulter_situation_contribuable` (niveau 3, permission `situations.voir`).
*   **Le Plan d'un Projet :** Chaque projet est rattaché à un **plan** qui définit un ensemble de capacités. On peut aussi activer/désactiver une capacité pour un projet précis.
    *   Exemples de plans : `Découverte` (niveau 1), `Standard` (niveaux 1 et 2), `Complet` (niveaux 1, 2 et 3).

#### 4. Identité et Sécurité (Pages 10-12)

*   **Règle d'Or :** **L'identité vient du SaaS, jamais de la conversation.** Un visiteur sans jeton reste au niveau 1.
*   **Protection du Niveau 1 :**
    *   **Clé publique limitée :** La clé utilisée par le site public ne permet que les capacités de niveau 1.
    *   **Domaines autorisés :** Vérifier l'en-tête `Origin`.
    *   **Limiteur de débit :** Par IP et par session.
    *   **Protection anti-robot :** Test avant l'ouverture d'une discussion. Délivre une session courte.
    *   **Limites de taille :** Longueur des messages, nombre de messages, longueur des réponses.
    *   **Quotas par projet :** Dernier filet de sécurité.
    *   **Alerte :** Pic d'usage visible dans le panneau d'administration.
*   **Compte Assistant par Commune :** Un utilisateur technique "assistant" par commune, avec des permissions (Spatie teams). Jamais super-admin. Ne peut pas se connecter normalement. Son jeton peut être révoqué.
*   **Double Vérification des Droits :** L'API du SaaS vérifie à la fois les permissions du compte assistant **et** celles de la personne qui parle.
*   **Trajet d'une Question de Niveau 3 :** L'agent se connecte au SaaS -> Ouvre l'assistant -> Le SaaS génère un jeton de session court -> L'agent pose sa question -> L'Assistant appelle l'API du SaaS avec son jeton + le jeton de session -> L'API vérifie les droits et répond.
*   **Lecture Seule d'Abord :** Tous les outils sont en lecture seule.
*   **Injection de Prompt :** Ne pas compter sur le texte pour se protéger. Les règles de sécurité sont appliquées par le code et l'API du SaaS.
*   **Données Personnelles :** Vérifier le cadre légal togolais avant la V3. N'envoyer que le strict nécessaire au fournisseur IA. Ne jamais logger de secrets. Nettoyer les messages (masquer les données personnelles).
*   **Journalisation :** Chaque appel d'outil est enregistré (projet, outil, utilisateur, résultat, date). Les enregistrements ne se modifient pas.

#### 5. Fiabilité et Qualité des Réponses (Pages 12-14)

*   **Règle :** Répondre **uniquement** à partir de deux sources publiques : (1) les données structurées du SaaS (via outil prédéfini), (2) le contenu texte publié par la commune.
*   **Si l'information n'est pas trouvée :** Dire "Je ne sais pas" et orienter vers la mairie.
*   **Pour tout chiffre :** Reprendre la valeur exacte de la source.
*   **Application par le Code :** Pour un tarif, la valeur vient de l'outil `consulter_tarifs`. Pour le contenu texte, le service cherche les extraits pertinents et ne transmet que ceux-ci au fournisseur IA. Si aucun extrait n'est trouvé, le code répond "je ne sais pas" sans appeler le fournisseur.
*   **Les Tarifs :** Lus dans les données du SaaS via un outil prédéfini (`consulter_tarifs`). L'IA ne choisit que l'outil et ses paramètres. Une requête fixe par outil, écrite et testée côté SaaS. L'endpoint est public, en lecture seule, avec limiteur de débit.
*   **Citer la Source :** Chaque réponse factuelle indique sa source (fiche tarif, titre de la page, lien, date).
*   **Historique Court :** Contexte limité aux derniers messages (ex: 6-10). Durée de conservation limitée (ex: 30 jours). Anonymisation (identifiant de session aléatoire, IP jamais conservée en clair, nettoyage des données personnelles).
*   **Améliorer la Qualité :** Boutons "Réponse utile" / "Pas utile". Liste des questions sans réponse. Mesure de la part de réponses "sans réponse" et "pas utile".

#### 6. Le Panneau d'Administration (Pages 14-15)

*   **Accès :** Uniquement le personnel habilité de Sirius Digital.
*   **Écrans :**
    *   **Projets :** Liste, nom, type, plan, abonnement, état, usage du mois.
    *   **Fiche d'un projet :** Bouton Couper/Réactiver, plan, capacités, clés d'API, domaines autorisés, limites, quotas.
    *   **Plans :** Création/modification des plans et capacités.
    *   **Usage :** Consommation par projet, part de réponses "sans réponse"/"pas utile", erreurs récentes.
    *   **Journal :** Liste des appels et refus, avec filtres.
    *   **Conversations :** Historique court, réservé au personnel habilité.
*   **Couper l'Assistant :** Bouton "Couper" agit immédiatement pour un projet précis.
*   **Conditions pour qu'une Demande soit Acceptée :**
    1.  Le projet est actif (non coupé).
    2.  Le projet a un abonnement valide.
    3.  La capacité demandée est activée pour ce projet.
    4.  Le quota d'usage n'est pas dépassé.
    *   Si l'un des contrôles échoue, la demande est refusée et la raison exacte est écrite dans le journal.

#### 7. Abonnement, Usage et Crédits (Pages 15-16)

*   **Abonnement :** Statuts `essai`, `actif`, `suspendu`, `expiré`.
*   **Journal d'Usage :** **Point le plus important.** Chaque réponse écrit une ligne d'usage (date, projet, niveau, capacité, tokens entrée/sortie, résultat). Résultat : `accepté`, `refusé` ou `erreur`. Une ligne `erreur` ne compte pas dans le quota.
*   **Plus Tard : Les Crédits :** Ne pas coder les crédits en V1, mais ne rien faire qui les empêcherait. Le journal d'usage doit être complet.
*   **Les Quotas :** Limite d'usage par jour ou par mois. Message poli quand atteint.

#### 8. Modèle de Données (Pages 17-18)

*   **Tables :** `projets`, `plans`, `capacites`, `plan_capacites`, `projet_capacites`, `abonnements`, `cles_api`, `usages`, `conversations`, `messages`, `journal_activites`.
*   **Règles :** Migrations Laravel portables. Les conversations sont purgées automatiquement. Les lignes d'usage et le journal de sécurité ne se suppriment jamais. Les tarifs ne sont **pas** stockés dans la base de l'Assistant.

#### 9. Contrat d'API entre l'Assistant et le SaaS (Pages 18-19)

*   **Exemple 1 : Niveau 1, Tarifs Publics :**
    *   `GET /api/assistant/public/tarifs?taxe=marche`
    *   En-tête `X-Projet: <identifiant court>`
    *   Réponse 200 : `{ "commune": "...", "taxes": [ { "code": "marche", "libelle": "Taxe de marché", "montant": ..., "unite": "...", "en_vigueur_depuis": "AAAA-MM-JJ" } ] }`
    *   Réponses de refus : 404 (taxe inexistante), 429 (trop de demandes), 5xx (SaaS indisponible).
*   **Exemple 2 : Niveau 3, Situation d'un Contribuable :**
    *   `GET /api/assistant/contribuables/123/situation`
    *   En-têtes `Authorization: Bearer <jeton du compte assistant>` et `X-Session-Utilisateur: <jeton de session court de la personne>`
    *   Réponse 200 : `{ "contribuable": "...", "solde": ..., "statut": "..." }`
    *   Réponses de refus : 401 (jeton absent/expiré), 403 (droits insuffisants), 404 (contribuable inexistant dans CETTE commune).
*   **Remarque :** Un contribuable d'une autre commune doit renvoyer 404.

#### 10. Scénarios d'Exemple (Pages 19-20)

*   Tableau de scénarios de recette très détaillé. À utiliser comme base pour les tests.

#### 11. Phasage et Critères de Validation (Pages 21-23)

*   **Version 1 : Le Service et l'Assistant Public.**
    *   Objectif : Assistant sur le site public d'une commune pilote.
    *   À construire : Service, panneau d'admin, clé API publique, outil `consulter_tarifs`, protections niveau 1, règle "je ne sais pas", historique court, gestion des pannes, journal d'usage, widget léger.
    *   Recette : Liste de critères à valider (tarif exact, "je ne sais pas", refus poli, anonymisation, limiteur de débit, etc.).
*   **Version 2 : L'Aide à l'Usage pour les Agents.**
    *   Objectif : Aider les agents connectés à utiliser le logiciel.
    *   À construire : Compte assistant par commune, jeton de session court, capacité d'aide à l'usage.
    *   Recette : Aide cohérente avec le rôle, visiteur prétendant être agent reste au niveau 1, jeton expiré refusé, etc.
*   **Version 3 : Les Agents Consultent des Données (Lecture Seule).**
    *   Objectif : Consulter des données du SaaS, outil par outil, avec double vérification.
    *   Recette : Agent autorisé obtient la bonne information, agent non autorisé refusé, contribuable d'une autre commune renvoie 404, journalisation, etc.

#### 12. Règles de Travail pour le Stagiaire (Pages 23-24)

*   **À faire :**
    *   Commentaires en français.
    *   Traiter un seul sujet à la fois.
    *   Poser des questions si une règle manque.
    *   Ranger les secrets dans `.env`.
    *   Écrire un test pour chaque scénario de la section 10.
    *   Utiliser le faux fournisseur d'IA.
    *   Requêtes fixes et testées côté SaaS.
    *   Code indépendant du moteur de BDD.
*   **À ne jamais faire :**
    *   Connexion directe à la BDD du SaaS.
    *   Laisser l'IA écrire une requête.
    *   Se fier au texte du chat pour les droits.
    *   Donner le statut de super-admin au compte assistant.
    *   Stocker un jeton en clair.
    *   Écrire les règles de sécurité uniquement sous forme de consigne à l'IA.
    *   Supprimer/modifier les lignes d'usage ou de sécurité.
    *   Donner un tarif/montant/date qui ne vient pas d'une source publique.
    *   Recopier les tarifs dans la base de l'Assistant.
    *   Traiter la clé du site public comme un secret.
    *   Appeler le fournisseur d'IA avant les contrôles.
    *   Conserver du texte sans purge ou non nettoyé.
    *   Écrire du SQL non portable.

#### 13. Décisions de Détail et Points Restant à Définir (Pages 24-25)

*   **Décisions Arrêtées :** Laravel, sous-domaine, public = visiteurs anonymes, source des tarifs, fiabilité, protection niveau 1, conservation des conversations, pannes, accès au panneau, cadre légal, langue (français), BDD portable.
*   **Points à Définir par la Direction :**
    *   Fournisseur d'IA et budget.
    *   Quotas chiffrés.
    *   Valeurs chiffrées de protection et performance.
    *   Durée de conservation de l'historique (proposition : 30 jours).
    *   Service anti-robot.
    *   Règle de conversion en crédits.
    *   Nom du produit.
    *   Commune pilote.
    *   Manière d'accéder au contenu texte publié.
    *   Nom et chemin exacts de l'endpoint des tarifs côté MairiePlus. **À arrêter avant de coder la V1.**

#### 14. Livrables Attendus (Page 25)

*   Service Assistant déployé.
*   Panneau d'administration.
*   Widget léger et intégration.
*   Endpoint public de lecture des tarifs.
*   Documentation technique.
*   Jeu de tests.
*   Manuel d'utilisation du panneau.

#### 15. Glossaire (Pages 25-26)

*   Définitions des termes clés (Assistant IA, SaaS, Projet, Tenant, Capacité, Outil, Endpoint, Plan, Jeton d'API, Jeton de session, Token, Crédit, Injection de prompt, Double vérification, Lecture seule, Limiteur de débit, Anonymisation, Purge, Streaming, Disjoncteur, Moteur de BDD, Portable).

---

### Synthèse et Points d'Attention pour le Développement

1.  **Architecture Propre :** Service séparé, communication via API uniquement. Aucun accès direct à la BDD du SaaS.
2.  **Sécurité avant tout :** L'identité vient du SaaS. Double vérification des droits. Liste blanche des capacités. Protections contre les abus (débit, anti-robot, quotas).
3.  **Fiabilité :** Répondre uniquement à partir de sources publiques. "Je ne sais pas" est une réponse valide. Citer les sources. Ne jamais inventer de chiffres.
4.  **Portabilité :** Code indépendant du moteur de BDD. Utiliser les fonctionnalités de Laravel.
5.  **Testabilité :** Faux fournisseur d'IA pour les tests. Scénarios de la section 10 à couvrir.
6.  **Journalisation :** Le journal d'usage est crucial pour l'avenir (crédits). Ne jamais le modifier.
7.  **Respect des Règles de Travail :** Commentaires en français, un sujet à la fois, poser des questions, etc.
8.  **Points Bloquants avant la V1 :** Le nom et le chemin de l'endpoint des tarifs côté MairiePlus doivent être définis avec le directeur général.

---

### Prochaines Étapes

Maintenant que nous avons une vue d'ensemble, nous pouvons commencer à travailler de manière structurée. Voici ce que je te propose :

1.  **Clarifier les Points Bloquants :** Avant de coder la V1, il faut absolument obtenir les informations manquantes (notamment l'endpoint des tarifs). Tu dois poser ces questions à ton directeur général.
2.  **Planifier la V1 :** Découper le travail en tâches concrètes (création du projet Laravel, migrations, modèles, contrôleurs, interface, etc.).
3.  **Commencer par le Modèle de Données :** C'est la fondation. Nous pouvons créer les migrations et les modèles Eloquent en respectant les règles de portabilité.
4.  **Implémenter l'Interface `FournisseurIA` :** Commencer par le faux fournisseur pour les tests.
5.  **Développer l'Outil `consulter_tarifs` :** C'est le cœur de la V1.
6.  **Mettre en place les Protections :** Limiteur de débit, domaines autorisés, etc.
7.  **Construire le Panneau d'Administration :** Même une version simple pour commencer.

**Par où veux-tu commencer ?** Nous pouvons, par exemple, détailler le schéma de la base de données et écrire les migrations Laravel correspondantes. Ou bien, nous pouvons lister toutes les questions à poser à ton directeur général. Dis-moi ce qui te semble le plus utile pour avancer.