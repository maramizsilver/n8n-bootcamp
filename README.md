# Healthy Life AI Assistant

Healthy Life AI Assistant est un système d'automatisation n8n qui répond à des questions de nutrition et de recettes en combinant une base de connaissances personnelle (documents importés depuis Google Drive) avec une recherche web en temps réel, le tout accessible via un chat et l'envoi d'emails.

## Ce que fait le projet

- Ingère des documents personnels (fiches nutritionnelles, recueils de recettes) depuis Google Drive et les indexe dans une base vectorielle Supabase.
- Répond aux questions via un AI Agent qui interroge en priorité les documents personnels de l'utilisateur dans Supabase.
- Complète les réponses avec une recherche web via Firecrawl lorsque l'information doit être récente ou n'est pas couverte par la base de connaissances.
- Scrape le contenu complet des pages web pertinentes trouvées par la recherche, pour aller au-delà du simple extrait.
- Envoie un email via Gmail à la demande de l'utilisateur (par exemple pour partager une recette ou un résumé nutritionnel).

## Workflows

Le fichier exporté `healthy-life n8n project.json` regroupe deux workflows :

| Workflow | Objectif |
|---|---|
| Ingestion des documents | Télécharge les fichiers depuis Google Drive (fiche nutritionnelle PDF, recueil de recettes), extrait le texte, le découpe en morceaux, génère les embeddings avec Google Gemini (`gemini-embedding-2-preview`) et les insère dans la table `documents` de Supabase. À exécuter pour initialiser ou rafraîchir la base de connaissances. |
| Chat Agent | Fournit l'expérience de chat de l'assistant. L'AI Agent (Google Gemini) interroge d'abord Supabase pour les documents personnels, puis utilise Firecrawl Search/Scrape pour des sources externes, et peut envoyer un email via Gmail sur demande explicite de l'utilisateur. |

Documents utilisés pour peupler la base de connaissances :

- `Food-Fact-Sheet-Healthy-Eating.pdf` — fiche nutritionnelle de la British Dietetic Association (Eatwell Guide, portions recommandées, graisses saturées/insaturées).
- `Recipe_collection.txt` — recueil de recettes saines (bol de quinoa, saumon au four, soupe de lentilles, etc.) avec conseils généraux d'alimentation saine.

## Prérequis

Vous avez besoin d'une instance n8n (Cloud ou self-hosted) et de comptes ou identifiants pour :

- Google Drive et Gmail (via OAuth)
- Supabase avec une table `documents` compatible vecteurs (pgvector)
- Google Gemini pour le chat et les embeddings
- Firecrawl pour la recherche web et le scraping

Le fichier exporté contient des références d'identifiants n8n et des identifiants de ressources personnelles (fichier Drive, table Supabase, adresse email). Ces éléments doivent être remappés vers vos propres identifiants et ressources après import.

## Import et exécution

1. Clonez ce dépôt pour récupérer `healthy-life n8n project.json`.
2. Dans n8n : **Workflows > Import from File**, puis sélectionnez le fichier JSON. Les deux sous-workflows (ingestion et chat agent) s'affichent avec tous leurs nœuds.
3. Reconnectez les identifiants : chaque nœud importé (Google Drive, Supabase Vector Store, Gemini, Gmail, HTTP Request Firecrawl) référence un credential qui n'existe pas chez vous — ouvrez chaque nœud et sélectionnez ou créez votre propre credential.
4. Remplacez l'ID des fichiers Google Drive (PDF + txt) par vos propres fichiers, vérifiez le nom de la table Supabase, et mettez à jour l'adresse email de destination dans le nœud Gmail.
5. Exécutez manuellement le workflow d'ingestion pour peupler Supabase (téléchargement, découpage, embeddings, insertion).
6. Testez le Chat Agent via le nœud **When chat message received** (bouton "Open Chat") pour vérifier qu'il interroge bien Supabase puis Firecrawl si besoin.

## Sécurité

Ne committez pas de clés API, de secrets OAuth ou de tokens n8n. Conservez-les dans un fichier d'environnement local ignoré ou dans le gestionnaire d'identifiants de n8n. Faites tourner (rotate) tout identifiant qui aurait été exposé.

## Contexte du projet

Ce dépôt contient les artefacts d'un projet n8n combinant agents IA, RAG (Retrieval-Augmented Generation), services externes, APIs et web crawling dans un seul workflow d'assistant nutrition et recettes.