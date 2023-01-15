# Livrable 2 - Conseil technique avancé à un/une apprenant(e)

-> c'est quoi `fetch` ?

On va commencer par le commencement : déjà, qu'est-ce que ça veut dire en anglais ?

> fetch : aller chercher, récupérer, rapporter, extraire.

Tu l'auras compris, `fetch`, c'est une méthode qui te permet de récupérer des choses... C'est cool, mais quoi? Quand? Comment? Et où?
On va répondre à tout ça, mais d'abord, techniquement, `fetch` c'est une API. Et le plus souvent, on s'en sert pour communiquer avec d'autres API... Donc il va falloir que je t'explique ce que c'est qu'une API.

## C'est quoi une API ?

En version longue ça donne : "Application Programming Interface" -> une interface de programmation d'application.
Je pourrais me lancer dans des explications infernales, mais je vais faire très simple avec un exemple :

> une API, c'est comme une prise électrique...

1. Le fournisseur d'électricité distribue un service (l'électricité) à travers un réseau (le réseau électrique)
   **_comme_** une application va distribuer ses services (des outils pour manipuler les données qu'elle contrôle par exemple) à travers un réseau (internet par exemple)
2. La lampe consomme l'électricité chez toi par le biais de la prise électrique **_comme_** un dév ou un programme consomme les données/services d'une application par le biais d'une API !

-> Fournisseur d'électricité = Application distributrice
-> Prise électrique = API
-> Lampe = dév ou programme qui consomme les données/services de l'API

L'intérêt de la prise, comme de l'API, c'est qu'on peut contrôler les échanges entre le fournisseur et le consommateur grâce à elle : il y a un point d'échange, et des façons bien déterminées et prévisibles d'échanger l'électricité/les données.

## Fetch est une API javascript

Elle va te permettre d'envoyer une question sous une forme précise, qui sera reconnue par le destinataire, et te renvoyer une réponse sous une forme précise, qu'il y ait une erreur ou pas, et dont tu pourras te servir facilement.

Et c'est généralement une API que tu utilises pour consommer d'autres APIs... Oulalah le mal de tête, je sais 😅 Ce sera plus clair avec des exemples.

## Quand est-ce que tu veux utiliser fetch déjà?

Quand tu veux récupérer des données que tu n'es pas sûr de pouvoir récupérer.
-> Par exemple, quand on demande des données à une autre application, on n'est pas du tout sûr que l'autre soit disposé à nous répondre. Il pourrait y avoir une erreur dans son code, un problème de réseau, ou notre question pourrait être mal comprise (mal formulée), ou simplement, on pourrait ne pas avoir le droit de savoir...

C'est pour ça que fetch fonctionne sur le principe des promesses... What?! Mais...qu'est-ce que c'est que ce truc nouveau? Attends attends pas de panique c'est plutôt simple et tu en as déjà utilisé dans ton exercice!

## Les promesses de fetch

Quand tu poses une question avec fetch, elle créée une promesse (Promise) qui pourra être résolue ou rejetée. Alors tu peux décider que ça s'appelle une promesse parce que fetch te PROMET une réponse, ou tu peux décider simplement que comme un humain, fetch peut très bien tenir ses promesses ou les trahir 😂

Dans tous les cas, l'important, c'est que toi tu es obligé d'ATTENDRE d'avoir le résultat de cette promesse pour pouvoir utiliser ce qu'il y a à l'intérieur (soit la réponse, soit une erreur en général). C'est pour ça qu'il y a une syntaxe particulière à utiliser quand on appelle fetch. Et c'est pour ça aussi que c'est généralement du code qu'on exécute dans l'ombre, sans bloquer toute une application pendant qu'on attend bêtement le résultat de la promesse.

## Les différentes façons d'utiliser fetch, par l'exemple

On parlait d'attendre ? Tu te souviens des "await" et "async" qui sont un peu partout dans ton code ? C'est la première façon de se servir de fetch! Tu vois tu sais déjà le faire!

Maintenant, on va changer un peu ton code pour le deck pour utiliser fetch 😉 Plus besoin de recharger la page quand on ajoute une carte au deck!

### On va construire une nouvelle route post cette fois, pour changer:

```js
//router.js
const apiController = require("./controllers/apiController");
router.post("/api/deck/add/:id", apiController.addCardToDeck);
```

POST, c'est la méthode qu'on doit utiliser pour accèder à cette route du coup. Pas GET, comme les autres.
(en général c'est comme ça qu'on envoie des données sensibles et des données de formulaires)

### On créé un nouveau fichier apiController avec dedans la méthode addCardToDeck

```js
//apiController.js
const dataMapper = require("../dataMapper.js");
const apiController = {
	addCardToDeck: async (req, res, next) => {
		try {
			//On récupère l'id comme d'habitude
			const id = parseInt(req.params.id);

			//Ça c'est ton code de cardController > addCard
			const card = await dataMapper.getOneCard(id);
			if (card) {
				// Rechercher si la carte est déjà dans le deck, si non l'ajouter, si oui ne rien faire
				const findCardInDeck = req.session.cards.find(
					(card) => card.id === id
				);

				//ça c'est pour dire au client qui va recevoir la réponse que c'est du json
				res.setHeader("Content-Type", "application/json");

				if (!findCardInDeck && req.session.cards.length < 5) {
					req.session.cards.push(card);
					//ici, on va renvoyer un message pour dire que la carte a bien été ajouté
					//on utilise JSON stringify pour que le client puisse lire ça sous forme d'objet json
					res.send(
						JSON.stringify({
							message: `La carte ${card.name} a bien été ajouté à ton deck!`,
						})
					);
				} else if (req.session.cards.length >= 5) {
					//même chose, on notifie l'utilisateur que son deck est plein
					res.send(
						JSON.stringify({
							message: `Ton deck est déjà plein... Tu ne peux pas ajouter d'autres cartes.`,
						})
					);
				} else if (findCardInDeck) {
					//même chose, cette fois il saura pourquoi la carte n'a pas été ajoutée!
					res.send(
						JSON.stringify({
							message: `La carte ${card.name} est déjà dans ton deck... Tu ne peux pas l'ajouter 2 fois!`,
						})
					);
				}
			}
		} catch (error) {
			console.error(error);
		}
	},
};

module.exports = apiController;
```

On a juste c/c ton code bien cool de cardController dedans! Le but, c'est de faire des trucs en front sans rechargement de page. Tu vas voir c'est plus fluide!

### On ajoute la bonne méthode pour appeler cette route en front, et c'est là que les choses sérieuses arrivent...

1. on modifie le template : plus de lien, juste un onclick sur un élément p. Si on laissait un lien, la page se rechargerait.

```html
<%- include('partials/header') %>
<h1><%= title %></h1>

<div class="cardsContainer">
	<% if(cards.length){ %> <% cards.forEach( (card) => { %>

	<div class="card">
		<img
			src="/visuals/<%= card.visual_name %>"
			alt="<%= card.name %> illustration"
		/>
		<p class="cardName"><%= card.name %></p>
		<p class="link--addCard" onclick="addCardToDeck(<%=card.id %>)">
			[ + ]
		</p>
	</div>

	<% }) %> <% } else { %> Empty <% } %>
</div>
<script>
	//c'est là qu'on va mettre les choses sérieuses ;-)
</script>
<%- include('partials/footer') %>
```

Tu remarqueras qu'on appelle une méthode addCardToDeck dans onclick, et qu'on lui passe l'id de la carte...

2. Maintenant va falloir écrire addCardToDeck! il y a 2 possibilités (principalement... en réalité on peut encore faire différemment)

**_PREMIERE METHODE : ASYNC/AWAIT_**

```html
<script>
	const addCardToDeck = async (cardId) => {
		//on utilise await pour attendre le résultat de la promise
		const result = await fetch(`/api/deck/add/${cardId}`, {
			//on a déclaré la route comme post, donc on précise cette méthode
			// si on ne précisait rien, fetch enverrait la requête en GET
			method: "POST",
		});
		let message = "";

		//si la requête a renvoyé une erreur http (ex:500) fetch renverrait ok = false
		if (!result.ok) message = "Il y a eu une erreur http...";

		//comme on a décidé de renvoyer les infos en json , on est obligé d'attendre encore que la méthode json() lise le flux pour avoir accès aux données
		const json = await result.json();

		//ça y est, on peut récupérer notre message!
		message = json.message;

		//parce qu'on aime bien enquiquiner l'utilisateur...
		alert(message);
	};
</script>
```

**_DEUXIEME METHODE : THEN/CATCH_**

```html
<script>
	const addCardToDeck = (cardId) => {
		let message = "";
		//on utilise then et catch cette fois donc pas besoin d'async/await
		fetch(`/api/deck/add/${cardId}`, {
			method: "POST",
		})
			.then((res) => {
				/* une fois que la promesse est résolue, s'il n'y a pas d'erreur, on peut récupérer la réponse dans une clause then
      -> là, on renvoie la promesse de l'objet json telle quelle - donc sans await - pour le prochain then */
				return res.json();
			})
			.then((json) => {
				/* une fois que la promesse de json() est résolue, on peut enfin récupérer notre message
      et le mettre dans la variable message pour l'afficher plus tard */
				message = json?.message || "Quelque chose s'est mal passé...";
			})
			.catch((error) => {
				/* si on arrive là, c'est qu'il y a une une erreur http, ou une erreur dans les then
      -> c'est la même clause catch que tu as l'habitude d'utiliser déjà */
				message = "Il y a eu une erreur http...";
			})
			.finally(() => {
				/* finally, c'est le truc qui s'exécute dans tous les cas à la fin, qu'il y ait eu erreur ou pas
      -> du coup nous on en profite pour lancer notre alert et enquiquiner l'utilisateur ;-) */
				alert(message);
			});
	};
</script>
```

Tu préfères laquelle des 2 ? Moi j'ai l'impression de mieux maîtriser ce qu'il se passe quand j'utilise la première méthode, mais c'est vraiment une question de préférence.

## Pour conclure

`fetch` n'a longtemps existé que côté client, mais elle existe désormais sur les versions récentes de node sur le serveur et s'utilise de la même manière.

Si tu veux une alternative, il existe une bibliothèque très populaire qui fait à peu près la même chose que `fetch`: [AXIOS](https://axios-http.com/fr/docs/intro).

Si tu veux en savoir plus sur fetch ou t'entraîner :
Voilà un cours avec exos plutôt cool, mais [en anglais](https://codeyourfuture.github.io/syllabus-london/js-core-3/week-2/lesson.html) - va falloir s'habituer à le lire si ce n'est pas déjà fait.
