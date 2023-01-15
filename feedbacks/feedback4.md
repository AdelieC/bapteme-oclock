## Apprenant 4

<br>

> Tu as essayé mais je soupçonne que tu n'aies pas compris tout ce que tu essayais de faire puisqu'il manque des morceaux un peu partout. La prochaine fois, n'hésite pas à faire appel à moi! Il n'y a pas de question idiote, ce qui est idiot quand on veut apprendre, c'est de ne pas demander 😉

<br>

-   [Étape 1](#étape-1---détail-dune-carte)
-   [Étape 2](#étape-2---recherche)
-   [Étape 3](#étape-3---construire-un-deck)

### Étape 1 - Détail d'une carte

#### **_dataMapper > getCard_**

Mmh... C'est presque bon. Il y a juste 2 erreurs :

1. tu ne passes pas de valeur dans ta requête qui demande une valeur avec `$1`
2. `queryResult` ou `resultQuery`? il faut choisir 😉

Ta méthode marchera beaucoup mieux comme ça :

```js
...
getCard: async (id) => {
    const queryResult = await client.query({
        text:  `SELECT * FROM card WHERE card.id = $1;`,
        values: [id]
    });

    if (queryResult.rowCount === 1) {
        return queryResult.rows[0]
    }
    return null;
  },
...
```

#### **_mainController > cardPage_**

Là, on a deux autres soucis : tu ne récupères pas l'id dans les param de ta requète, et tu ne passes rien du tout à ta view.
On corrige ça :

```js
cardPage: async (req, res) => {
    try {
        //on récup l'id que tu devrais avoir passé dans ton lien
        const id = req.params.id
        //on la passe à getCard
        const card = await dataMapper.getCard(id);
        //on rajoute card, ton resultat, dans l'objet que tu envoies à ta view
        res.render("card", {card});
    } catch (error) {
        console.error(error);
        res.status(500).send(
            `An error occured with the database :\n${error.message}`
        );
    }
	},
```

Ensuite, si jamais on te demande une carte qui n'existe pas, il se passe quoi pour l'instant?
-> Ta view renvoie une erreur en t'expliquant qu'elle ne comprend pas pourquoi tu lui demandes d'aller chercher des trucs dans une card qui n'existe pas...

On pourrait lui dire de ne rien afficher s'il n'y a pas de card, mais c'est quand-même mieux de dire à l'utilisateur qu'il essaie d'aller sur une page qui n'existe pas, non?

On fait ça comme ça :

```js
cardPage: async (req, res) => {
    try {
        const id = req.params.id
        const card = await dataMapper.getCard(id);
        if(card) {
            //si card n'est ni null, ni undefined, ni false, ni 0, on affiche la view et on lui passe card
            res.render("card", {card});
        } else {
            //ça va afficher la 404 par défaut si tu n'en as pas créée et c'est une bonne pratique de renvoyer le bon statut http pour la bonne erreur
            response.status(404).send(`Card with id ${cardId} not found`);
        }
    } catch (error) {
        console.error(error);
        res.status(500).send(
            `An error occured with the database :\n${error.message}`
        );
    }
	},
```

Mais... ça ne marche pas?
Ah! il y a un souci avec ta liste de cartes 😉 Ton lien pour aller vers le détail d'une carte est vide. Correction rapide :

```
//cardList.ejs

<a href="/card/<%= card.id %>"> ... </a>
```

Et ça ne marche toujours pas, pourquoi?
La réponse est dans ton router.
Il faut que tu expliques au router que tu vas passer un param `id` à cette route :

```js
//router.js
router.get("/card/:id", mainController.cardPage);
```

#### **_views > card_**

Bon là il y a un nouveau souci : j'imagine que tu as laissé ton c/c de cardList 😅
On enlève donc la boucle -> on ne lui passe pas un tableau de carte, mais une seule carte. Et soit on lui passe un `title` depuis le controller, soit on change le `title` :

```html
<%- include('header') %>
<!-- Je suis flemmarde donc je mets le titre en dur... -->
<h1>Détails de la carte</h1>

<div class="cardsContainer">
	<div class="card">
		<a href="#">
			<img
				src="/visuals/<%= card.visual_name %>"
				alt="<%= card.name %> illustration"
			/>
			<p class="cardName"><%= card.name %></p>
		</a>
		<a class="link--addCard" title="Ajouter au deck" href="#">[ + ]</a>
	</div>
</div>

<%- include('footer') %>
```

Je vais te laisser ajouter les autres détails de la carte si tu en a envie 😉

### Étape 2 - Recherche

Je vois un petit console.log et un searchController, c'est un début. Si tu ne savais pas comment faire, demande moi de l'aide pour réessayer, n'hésite pas!

### Étape 3 - Construire un deck

Je vois que tu n'as pas eu le temps de faire ça. Si c'est parce que tu n'as pas compris que tu n'as pas fait, n'hésite pas à réessayer en me demandant des explications 😉 On est là pour vous aider!
