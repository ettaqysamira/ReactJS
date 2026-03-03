# Réponses aux questions du TP TaskFlow

## Partie 1 : Initialisation du projet

**Q1 : Ouvrez `index.html`. Que contient le `<body>` ? Lien avec le CSR ?**
Le `<body>` du fichier `index.html` contient uniquement une balise `<div id="root"></div>` vide et un lien vers le script principal `<script type="module" src="/src/main.tsx"></script>`. 
Cela est caractéristique du **CSR (Client-Side Rendering)**. Le serveur envoie un HTML presque vide, et c'est le navigateur web qui se charge de télécharger puis d'exécuter le Javascript. React va ensuite générer dynamiquement l'ensemble des balises HTML et les injecter dans ce conteneur `#root`.

## Partie 2 : Backend avec json-server

**Q2 : Quelle différence entre des données en dur dans le code et une API REST ?**
- **Données en dur** : Les données sont écrites directement dans les fichiers JS/TS. Elles ne peuvent pas être modifiées par l'utilisateur (sans recompiler/déployer l'application) et sont perdues au rechargement si l'application utilise un état local simple.
- **API REST** : Les données sont stockées sur un serveur distant (ou local ici avec `json-server`). L'application web interagit avec via des requêtes HTTP (GET, POST, PUT, DELETE). Cela permet la persistance, le partage entre plusieurs utilisateurs et la mise à jour dynamique.

## Partie 3 : Composants avec Props

**Q3 : Pourquoi className au lieu de class en JSX ?**
Parce que JSX est transpilé en JavaScript, et que `class` est un mot-clé réservé en JavaScript (utilisé pour déclarer des classes orientées objet). L'équipe React a donc choisi `className` (qui correspond d'ailleurs à la propriété native du DOM HTMLInputElement.className) pour éviter tout conflit de syntaxe.

**Q4 : Pourquoi key={p.id} est obligatoire dans .map() ? Que se passe-t-il avec l’index ?**
L'attribut `key` sert d'identifiant unique à React pour suivre chaque élément d'une liste virtuelle. Lors du "re-render", cela permet à React de savoir exactement quel élément a été ajouté, modifié ou supprimé sans devoir tout recalculer (réconciliation). 
Utiliser l'index du tableau (`key={index}`) est une mauvaise pratique lorsque la liste peut changer (tri, suppression, insertion au milieu), car l'index d'un élément va changer, ce qui peut causer des bugs d'affichage ou des pertes d'état local des composants internes de la liste. Il faut utiliser une vraie clé unique pérenne (comme `p.id`).

## Partie 4 : State, useEffect & Fetch

**Q1 : Combien de fois le useEffect s’exécute-t-il ? Pourquoi ?**
Le `useEffect` s'exécute **une seule fois** (au montage du composant principal `App`), car on a passé un tableau de dépendances vide `[]` en deuxième argument. (Attention : En mode développement (`npm run dev`) avec le StrictMode de React 18, React execute le useEffect une seconde fois pour simuler le démontage et reméontage immédiat, afin d'aider à détecter les bugs dans les effets de nettoyage).

**Q2 : Arrêtez json-server (Ctrl+C) et rechargez. Que se passe-t-il ?**
La requête `fetch` échoue car le port 4000 ne répond plus. Le bloc `catch` capture l'erreur et affiche `Erreur: TypeError: Failed to fetch` dans la console. Les états `projects` et `columns` restent vides par défaut (donc l'interface se charge vide sans faire planter React), et l'état de chargement s'arrête (`finally`).

**Q3 : Ouvrez Network (F12). Voyez-vous les requêtes vers localhost:4000 ? Code HTTP ?**
Oui, on observe deux requêtes HTTP GET vers `/projects` et `/columns`. Le code HTTP renvoyé par le Mock Server `json-server` est **200 OK**.

**Q4 : Les nouvelles données s’affichent ? Décrivez le cycle complet.**
Oui, les nouvelles données s'affichent après rechargement de React. 
Cycle: Recharge de React -> Composant `App` se monte -> Déclenche `useEffect` -> Fait les `fetch` HTTP -> Les promesses se résolvent avec les nouvelles valeurs -> On appelle les mutateurs `setProjects()` et `setColumns()` -> Modification du "State" -> Provoque le "Re-render" de `App` -> Envoie des nouvelles 'props' aux composants `Sidebar` et `MainContent` -> Affichage des nouvelles données sur l'écran.

**Q5 : Dessinez le flux : json-server → fetch → useState → useEffect → composants → props.**
Le flux complet :
1. **json-server** : Héberge les données et expose l'API sur `localhost:4000`
2. **useEffect** : S'active au premier montage de l'App. Il orchestre l'appel.
3. **fetch** : Interroge json-server et télécharge les données JSON de manière asynchrone.
4. **useState** : Reçoit les données téléchargées via `setProjects` et `setColumns`, modifiant l'état local du composant `App`.
5. **composants** : Le changement d'état déclenche un re-rendu du parent (`App`).
6. **props** : Le nouveau flux de données descend du parent `App` vers les enfants `Sidebar` et `MainContent` en tant que `props` qui vont alors l'afficher.
