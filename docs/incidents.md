# Incidents de production

[Retour à l'accueil](../README.md)

Du 7 au 19 octobre 2026, le système a tourné sur des projets réels, et il a cassé plusieurs fois. Voici les
incidents les plus instructifs, avec la cause trouvée dans les journaux des sessions et le correctif. Chaque
correctif est passé par un agent critique avant d'être appliqué.

Sur 27 lancements de l'orchestrateur entre le 7 et le 19 octobre, 4 ont fini en échec, tous lancés avant le 9 octobre
à midi. Trois ont la même cause (incident 1). Le quatrième, le 7 octobre, venait d'une commande refusée par la session :
depuis, au premier refus, la session s'arrête et rend un échec explicite au lieu de réessayer, et le dépôt où
démarrent les sessions déclare la liste exacte des commandes dont elles ont besoin.

## 1. Sessions figées en attendant une notification

*8 et 9 octobre 2026, orchestrateur.*

**Symptôme.** Trois lancements sont restés muets : rien de poussé, pas de compte rendu. Le verrou de quatre heures
les a passés en échec, et le projet en cours n'a pas avancé.

**Cause.** Les journaux des trois sessions montraient le même enchaînement. L'orchestrateur lançait les agents du
pipeline en arrière-plan. Une fois le rapport du critique reçu, il écrivait qu'il attendait le décompte des appels
d'outils de cet agent pour clore l'étape, et terminait son tour. La notification de fin de tâche n'arrivait jamais ;
la session restait inactive.

**Correctif.** Deux changements, dans les consignes de l'orchestrateur et dans la procédure du pipeline :
les agents se lancent au premier plan et leur résultat s'attend dans le même tour ; une session ne termine jamais son
tour pour attendre une notification. Le décompte des appels d'outils est devenu facultatif dans le code : il ne sert
qu'à l'historique et ne justifie jamais d'attendre.

**Vérification.** Le lancement suivant, le 9 octobre, a duré 69 minutes sans se figer et a fait avancer un projet
jusqu'à une décision. Jusqu'au 19 octobre, les 13 lancements suivants, celui-ci compris, se sont terminés
normalement.

## 2. Agents copiés en cours de session, jamais chargés

*6 et 7 octobre 2026, mise en place des sessions cloud.*

**Symptôme.** Les agents du pipeline, copiés au début d'une session cloud, n'étaient pas reconnus par l'outil de
lancement d'agents.

**Cause.** Un test dédié l'a montré : la liste des agents d'une session est figée à son démarrage. En revanche, une
procédure (skill) copiée dans le dossier de l'utilisateur est reconnue tout de suite. Un deuxième test a révélé que
les sessions ne démarraient pas dans le dépôt que la documentation indiquait.

**Correctif.** Les 38 agents sont versionnés dans le dépôt où démarrent réellement les sessions. Le script de
préparation ne copie plus que les procédures. Un test vérifie que les copies sont à jour par rapport aux sources. Si
un agent manque malgré tout, les consignes donnent un repli : lire son fichier et lancer un agent général avec le
même modèle.

**Vérification.** Le 7 octobre, une session de test a listé les 38 agents chargés.

## 3. Une décision réservée à l'humain bloquait un projet

*7 octobre 2026, pipeline.*

**Symptôme.** Pendant le premier test de bout en bout, un projet s'est arrêté en élaboration, en attente d'une
décision réservée à l'humain. La règle était pourtant de ne pas le solliciter pendant cette phase, sauf projet
« prioritaire ».

**Cause.** Une étape avait posé une décision bloquante d'un type que le code ne savait pas confier à l'IA. Par
défaut, elle revenait à l'humain.

**Correctif.** Toute décision bloquante qui a au moins deux options et n'engage pas un envoi à des tiers est confiée
au décideur, puis relue par son critique. Si le décideur juge qu'elle dépend d'un fait que seul l'humain connaît,
elle devient une question d'information.

**Vérification.** Le lendemain, la même décision est passée par le décideur. Le critique a contesté son premier choix :
il contredisait un mot de la note d'origine et s'appuyait sur des chiffres sans source. La décision a donc été
renvoyée en question à l'humain. C'est le comportement voulu.

## 4. Des questions illisibles sur un téléphone

*8 octobre 2026, orchestrateur.*

**Symptôme.** Une question arrivée sur Telegram faisait 1 768 caractères, avec le vocabulaire interne du pipeline et
des identifiants de besoins. Il a fallu la réexpliquer avant d'y répondre.

**Cause.** Les consignes disaient quoi demander. Elles ne disaient rien de la façon de l'écrire pour quelqu'un qui lit
sur un téléphone.

**Correctif.** Un format imposé : la question en une phrase, dans les mots de l'utilisateur ; un éclairage d'une ou deux phrases,
250 caractères au plus ;
les options en lignes `a)`, `b)`, `c)` ; une recommandation en une ligne ; 700 caractères au plus par question ; ni
titre, ni identifiant, ni explication du fonctionnement interne. Les consignes donnent un exemple complet.

**Vérification.** La question suivante tenait en deux phrases.

## 5. Les consignes internes visibles dans un dépôt public

*9 octobre 2026, publication d'un projet.*

**Symptôme.** Un dépôt où des sessions cloud avaient travaillé venait d'être passé en public. Son historique git
contenait le brief donné aux sessions et leurs notes de passage, qui ne s'adressaient pas à un lecteur extérieur.

**Cause.** Un dépôt de travail garde tout dans son historique. Supprimer les fichiers dans un dernier commit n'y change
rien.

**Correctif.** Le dépôt est repassé en privé le jour même. Règle depuis : un dépôt de travail n'est jamais publié. La
version publique est un nouveau dépôt, sans l'historique de travail, construit à partir d'une copie nettoyée, avec un
auteur sans adresse personnelle, passée au scanner de secrets et relue avant publication.

## Ce qu'il faut en retenir

- Il faut partir du principe qu'une session cloud peut ne jamais rendre compte : verrou qui expire, délai, relance
  limitée.
- Une hypothèse sur la plateforme (agents chargés, dépôt de départ) se vérifie par un test réel avant d'être écrite
  dans la documentation.
- Lire les journaux des sessions avant de corriger. Les trois figements avaient la même phrase juste avant de
  s'arrêter.
