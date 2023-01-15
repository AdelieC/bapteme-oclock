## Apprenant 1

<br>

> Excellent 👏 ! Tu as fait mieux et plus que la correction, tu as expérimenté en back comme en front et cherché à partitionner ton code. Si tu continues comme ça, tu progresseras trèèèès vite et tu iras loin!

<br/>

-   [Étape 1](#étape-1---détail-dune-carte)
-   [Étape 2](#étape-2---recherche)
-   [Étape 3](#étape-3---construire-un-deck)

<br />

### Étape 1 - Détail d'une carte

#### **_dataMapper > getOneCard_**

Rien à dire sur la requête, et ça marche bien 👍

Concernant ceci :

```
if (result.rowCount === 1){
    return result.rows[0];
}
```

...c'est toujours une bonne initiative de vérifier qu'on reçoit bien ce qu'on a demandé 👌 Cependant, une bonne pratique dans n'importe quel langage : lorsqu'une fonction retourne une valeur dans un cas, il vaut mieux qu'elle retourne une valeur dans TOUS les cas, et c'est encore mieux si elle peut toujours être du même type. Comme ça, la personne qui se servira de cette fonction n'aura pas de surprise à l'avenir. Là, tu peux par exemple retourner `null` ou `undefined` dans un `else`.
Ou mieux et plus court, juste :

```
return result?.rows?.[0]
```

-> le `?.` signifie que l'on ne va voir la suite de l'expression que si celle qui précède n'est pas falsy. Si result, alors result.rows, et si result.rows, alors result.rows[0]. Si une des étapes est nullable, cette expression renverra `undefined`. Le mieux, c'est que tu ne risques pas de lancer une erreur comme ça, même si `result` est `undefined` ou `null`

#### **_cardController > renderOneCard_**

Bravo d'avoir pensé à t'assurer de bien envoyer un nombre à dataMapper avec `Number()`. Il vaut mieux utiliser `parseInt` comme dans la correction parce que `Number()` convertit le type de ce que tu lui passes, et que tu peux te retrouver avec `null` transformé en 0, `true`en 1, `''` en 0 etc... Si tu veux t'assurer de récupérer `NaN` si id n'est ou ne contient pas de nombre, utilise `parseInt`.
Si tu veux creuser, tu peux aller lire [cet article](https://thisthat.dev/number-constructor-vs-parse-int/).

Bravo aussi d'avoir pensé à gérer les erreurs avec un `try catch`, d'afficher l'erreur avec la bonne méthode de `console` et de renvoyer le bon code de statut http 👍 Le choix d'utiliser `next()` pour la 404 est discutable. Cette fonction envoie vers le prochain middleware, or dans le cas d'un router plus complexe, tu n'es pas toujours certain qu'il n'y ait pas d'autres middleware qui se soit glissé entre ta route actuelle et ton handler 404. Le mieux c'est au moins de passer une erreur 404 en paramètre de `next()`.

#### **_views > card > card-item_**

Tu as rangé tes views dans des dossiers et sous-dossiers! Chapeau 🎩 Rien à dire, tu as fait mieux que la correction. Tu as même utilisé un h1!
Tu peux éventuellement faire plus court et raccourcir ça :

```js
<% if(card.element){ %>
    <%= card.element %>
<% } else { %>
    empty
<% } %>
```

...en ça :

```js
<%= card?.element || 'empty' %>
```

(les || signifient que on affiche card.element s'il n'est pas falsy (undefined, null, false, '', NaN, ou 0), OU ALORS on affiche 'empty')

### Étape 2 - Recherche

#### **_Par élément_**

Rien à dire d'autre que bravo! Tu as trouvé l'astuce. Utilisé la méthode get pour le formulaire, envoyé une valeur cohérente pour chaque élément (ou absence de). Je vois que tu as hésité à utiliser une seconde méthode pour la recherche des cartes sans éléments. Je trouve que tu as fait le bon choix en utilisant la même.
Seul petit détail minuscule qui me chatouille les yeux :

```js
res.render("main/cardList", {
	cards: searchedResults,
	title:
		"Résultat de la recherche : " +
		(searchedElement === "null" ? " sans élément" : +searchedElement),
});
```

-> ce `+ searchedElement`. Tu peux enlever ce + en trop.

#### **_Par niveau_**

C'est parfait! Même remarque que plus haut sur `Number()`.

#### **_Par valeurs_**

Tu utilises `Number()` sur searchedValue puis vérifies ceci 😉 ?

```js
if(searchedDirection != '' && searchedValue != '')
```

Pour plus de sûreté, tu peux remplacer ta vérification par :

```js
if(searchedDirection && searchedValue !== NaN && searchValue >= 0)
```

-> si searchedDirection n'est pas falsy (null, undefined, O, '', false, NaN) ET searchedValue n'est pas [not a number] ET searchedValue est supérieure ou égale à 0

Ensuite, pour ta solution ingénieuse de passer uniquement la valeur que tu cherches dans ta query, j'adhère! Je ne comprends juste pas pourquoi tu prends la peine de remplacer un single quote par 2 single quotes dans ton nom de colonne ? Un truc m'échappe. Sur ma machine ça fonctionne parfaitement sans ça.

Bon...dans la pratique,

1. il faudrait plus de vérifications sur le nom de ta colonne, pour éviter de faire des requêtes erronées pour rien et générer des erreurs évitables,
2. jamais on ne ferait passer une variable venant directement du client telle quelle dans une requète sql. C'est très dangereux.
3. ta solution ne renvoie pas la même chose que la correction, qui cherche la valeur SUPÉRIEURE ou ÉGALE, pas juste égale. Je ne suis pas certaine de qui de vous 2 a respecté la consigne cela dit.

Quoi qu'il en soit, j'admire ta flemme géniale 👏

#### **_Par nom_**

Top d'avoir pensé à l'insensibilité à la casse et à la recherche partielle!!

### Étape 3 - Construire un deck

#### **_Activer les sessions_**

-   Tu as créé un middleware que tu as rangé dans le bon dossier pour initialiser ton deck s'il n'existe pas encore en session 👏 c'est parfait!
    En revanche, je suis d'accord avec la correction pour dire que ce n'est pas le rôle du router d'instantier le deck. Il devrait être initialisé avant le router puisqu'il est déjà nécessaire aux fonctionnalités de la page d'accueil.

Je te conseille donc de déplacer l'import de ton middleware et son utilisation dans index.js :

```js
...
app.use(
	session({
		secret: process.env.SESSION_SECRET,
		resave: false,
		saveUninitialized: false,
		cookie: {},
	})
);
app.use(middleware.cardSession);
...
```

saveUnititialize devrait être false : cela signifie que si la session n'était jamais utilisée, elle ne serait pas conservée vide. Dans notre cas cela dit, c'est impossible puisqu'on l'utilise juste après 😅 . Mais dans la pratique, moins on laisse de choses inutiles en mémoire, mieux c'est.
Rien à dire d'autre sur la session, en dehors de ce détail c'est top!

#### **_Ajouter une carte au deck_**

Je te félicite encore d'avoir déplacé l'initialisation du deck dans un middleware (cf. ton code commenté 😉)
Pour le code de addCard et removeCard, tout est nickel 👌

Pssst : `!(card.id === cardId)`c'est la même chose que `card.id !== cardId`
(Je chipote, mais c'est plus habituel de le formuler comme ça)

#### **_Page pour visualiser le deck_**

Ton controller : tu ne peux pas ne pas avoir de deck, surtout si tu l'as initialisé avant le router 😉 Du coup, tu peux juste écrire `res.render("card/deck", { cards: req.session.cards });`

Et ta view...elle est par-faite 👌
