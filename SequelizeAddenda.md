Sequelize Docs Addenda
======================

I hear a lot of shade thrown at the [sequelize docs](http://docs.sequelizejs.com/en/v3/) and IMHO I think only *some* of it justified.   Maybe because I have a good grasp on how to do the primary things like setting up a *basic* connection and defining a model, that I find these parts well-written enough. I do find the docs substantially lacking in the more advanced topics and options, and today I will share what I've learned specifically about associations and the methods that are born out of them.

First a little bit of background and clarity on terms though.  A **model** is the [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping "ORM Wikipedia") term for what must of us visually think of as a table.  In the land of relational databases, tables are often also called **relations**.  All of these terms for the same thing can get kind of confusing, so I like to think of it this way: **Table** is the layperson word for what we are talking about.  We couldn't elevate ourselves if we didn't have jargon, right? **Relation** is what tables are called in a stricly relational database model. To reiterate, working solely on the backend, a relation = a table.  Once we enter frontend world (the whole point of Sequelize is to have a way to talk to our database without having to know SQL), we are creating objects which *represent* our tables.  However, when we do javascript-y things with these objects, Sequelize translate them into actual queries or actions on the backend. (Thanks, Sequelize!) Since these objects, again, *represent* our tables/relations, but are not actually tables, we call them **models**.  And we've come full circle.  From now on, when I use the term model, I will be refering to an object which represents a table, not an a tall anorexic person who makes their money strutting around fiercely in ridiculous get-ups.

When we first create models, they are independent -- that is they have nothing to do with one another.  Models like this have some limited use, but the whole point of *relational* databases is that information in one data table often has some kind of *relationship* with information in another data table.  For example we might have a model full of information about books.  The book title, ISBN ([International Standard Book Number](https://en.wikipedia.org/wiki/International_Standard_Book_Number "ISBN Wikipedia") author, etc. etc. would be stored in this model. *Create a visual of this!* But this model would have a relationship with another model that stores information about authors, for example their name, date of birth, country, etc.  The relationship between these models is called an **association**.

Back to the sequelize docs, here's where they start getting kind of crappy.  First of all, this [section of the docs](http://docs.sequelizejs.com/en/v3/docs/associations/) is called "Relations/Assocations", which we know is using the terminology incorrectly.  A relation is a table, goddamnit, NOT a relationship between tables.  From there, they split up the associations based on types, which is a very reasonable thing to do.  Problem is, there are kind of redundant associations, (for example, 'belongsTo' is the same thing as 'hasOne', just switching with party gets the foreign key) which makes the whole thing more bloated than it needs to be.  More on this in a second.  When I first skimmed this page (and I have read it several times since then), I steeled myself up for learning *a lot* of new content and then felt as though I didn't understand the docs when it seemed I had not actually gotten that much more information.  So let me try to break it down in a succinct way:

There are basically *two* kinds of assocations:  One-to-one and one-to-many.  Though many-to-many associations may exist in the real world, it is not useful to try to initiate this kind of association between models.  More on that later.  


### One-To-One Associations

Let's talk concrete examples here.  Because we live in a still largely monogenistic (that is the correct spelling of the word, [I looked it up](http://www.dictionary.com/browse/monoganistic?s=t)) and I'm gonna go all heteronormative on you, let's say we have a model called husbands and we have a model called wives.  Each model stores basic information about a specific person. Each wife has one husband and vice versa. (I'm sorry queer and/or polygamous communities, I'll come up with a better example soon!)

```
var Wife = this.sequelize.define('wife', {/* attributes */})
  , Husband  = this.sequelize.define('husband', {/* attributes */});
```

In order to create a one-to-one association between exactly two models we can use either the `.belongsTo()` method or the `.hasOne()` method.  From what I can tell, these are **functionally the same**.  The only difference is which model gets the foreign key.

So if we do:

```
Husband.belongsTo(Wife);
```

Husband, the first model (oh snap, we're getting into some Biblical shit here), or more explicitly the model that the function is being invoked on, is the **source** model and Wife, the model being passed as an argument, is the **target** model. When using  `.belongsTo()` the source model gets the foreign key.  This means that the Husband model will now have an extra column called 'wifeId' which lists the id of the wife, from the Wife Model.  If we did:

```
Husband.hasOne(Wife);
```
it's flip-flopped.  In other words, the target model (still Wife) gets the foreign key.  Specifically, there will be a new column in the Wife model called 'husbandId'.  BTW, the default casing is camelCase (hence husbandId) but if you add the `{underscored: true}` attribute to the model definition, the foreign key (and presumably other created entities) will be in snake_case.

```
var Husband = this.sequelize.define('husband', {/* attributes */}, {underscored: true}),
  Wife = this.sequelize.define('player', {/* attributes */})
  
Husband.belongsTo(Wife);
```
yields a foreign key on the Husband model called 'wife_id'.  There are some other configuration options such as 'as' which allow you to name your foreign keys differently, and 'targetKey' which allows you to create a foreign key with something other than the primary key, but the docs are pretty clear on that, so no need to discuss.  The point I am trying to make here is that whether you use `.hasOne()` or `.belongsTo()` the results are the same, just which table gets the foreign key is different.   So `Husband.hasOne(Wife);` creates the exact same results as `Wife.belongsTo(Husband);`.  That is the Wife model will get a husbandId column.  (Yes, yes, typing that sentence did leave a dirty taste on my hands.) The sequelize docs claim "In Sequelize 1:1 relationship can be set using HasOne and BelongsTo. They are suitable for different scenarios. Lets study this difference using an example." To me this implied that there were truly different scenarios.  Maybe I'm just not understanding, but it seems to me there aren't really different 'scenarios' just which model you want to have the foreign key.

Other than creating foreign keys or join-tables, associations create built-in methods, which remember, are what this blogpost is actually supposed to be about.  I think the documentation on this is kind of sucky because there isn't a specifically named method created -- the names of the methods depend not only on your model names, but also the direction (i.e. `.hasOne` or `.belongsTo`) of the association.  Luckily it's actually pretty simple.  Let's stick with the `Husband.hasOne(Wife);` association and the methods wrought from this.  There are only two new methods for a one-to-one association.  After creating this association, Husband.prototype now has new methods called `getWife` and `setWife`. An important part of this that the documentation glosses over is that these methods need to be called with an actual wife instance! Example below:

```
Insert awesome example here:
```
### One-To-Many Associations





Stuff I still need to clarify for myself:
=========================================

#### I either don't understand this paragraph from the docs or disagree with it: 

```
Even though it is called a HasOne association, for most 1:1 relations you usually want the BelongsTo association since BelongsTo will add the foreignKey on the source where hasOne will add on the target. 
```

#### Why add the `Game.belongsTo(Team);` at the end?
```
// Example from Sequelize docs:

Project.hasOne(User, { as: 'Initiator' })
// Now you will get Project#getInitiator and Project#setInitiator


// If you need to join a table twice you can double join the same table
Team.hasOne(Game, {as: 'HomeTeam', foreignKey : 'homeTeamId'});  
//My note: Game will get a homeTeamId as a foreign key and Team(?) will now have 'getHomeTeam' 'setHomeTeam' methods?
Team.hasOne(Game, {as: 'AwayTeam', foreignKey : 'awayTeamId'});   
//My note: Game will get a awayTeamId as a foreign key and Team(?) will now have 'getAwayTeam' 'setAwayTeam' methods?
Game.belongsTo(Team);

```









