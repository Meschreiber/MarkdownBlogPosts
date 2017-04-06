Sequelize Docs Addenda
======================

I hear a lot of shade thrown at the [sequelize docs](http://docs.sequelizejs.com/en/v3/) and IMHO I think only *some* of it justified.   Maybe only because I have a good grasp on how to do the primary things like setting up a *basic* connection and defining a model do I find these parts well-written enough. I do find the docs substantially lacking in the more advanced topics and options, and today I will share what I've learned specifically about associations and the methods that are born out of them.

First a little bit of background and clarity on terms though.  A **model** is the [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping "ORM Wikipedia") term for what must of us visually think of as a table.  In the land of **relational** databases, tables are often also called **relations**.  All of these terms for the same thing/almost the same thing can get kind of confusing, so I like to think of it this way: **Table** is the layperson word for what we are talking about.  We couldn't elevate ourselves if we didn't have jargon, right? **Relation** is what tables are called in a stricly relational database world. To reiterate, working solely on the backend, a relation is the same thing as a table.  Once we enter frontend world (the whole point of Sequelize is to have a way to talk to our database without having to know SQL), we are creating objects which *represent* our tables.  However, when we do javascript-y things with these objects, Sequelize translates them into actual queries or actions on the backend. (Thanks, Sequelize!) Since these objects, again, *represent* our tables/relations, but are not actually tables, we call them **models**.  And we've come full circle, but with a bit of nuance added in. Models are not actually the same thing as tables.  (Maybe they're like the Platonic forms of tables? No that's not right either.)  From now on, when I use the term model, I will be refering to an object which represents a table/relation, not an a tall anorexic person who makes their money strutting around real fierce in ridiculous get-ups.

When we first create models, they are independent -- that is they have nothing to do with one another.  Models like this have some use, but the whole point of *relational* databases is that information in one data table often has some kind of *relationship* with information in another data table.  For example we might have a model full of information about books.  The book title, ISBN ([International Standard Book Number](https://en.wikipedia.org/wiki/International_Standard_Book_Number "ISBN Wikipedia")), author, etc. etc. would be stored in this model. *Create a visual of this!* But this model would have a relationship with another model that stores information about authors, for example their name, date of birth, country, etc.  The relationship between these models is called an **association**.

Back to the sequelize docs, here's where they start getting kind of crappy, from my point of view.  First of all, this [section of the docs](http://docs.sequelizejs.com/en/v3/docs/associations/) is called "Relations/Assocations", which is incorrect terminology!  A relation is a table, goddammit, NOT a relationship between tables.  From there, they split up the associations based on types, which is a very reasonable thing to do.  Problem is, there are kind of redundant associations, (for example, 'belongsTo' is almost the same thing as 'hasOne', just switching with party gets the foreign key), which makes the whole thing more bloated than it needs to be.  When I first skimmed this page (and I have re-skimmed it several times since then), I steeled myself up for learning *a lot* of new content and then felt as though I didn't understand the docs when it seemed I had not actually gotten that much more information.  So let me try to break it down in a succinct way:

There are basically *two* kinds of assocation methods:  One-to-one and one-to-many.  Many-to-many associations are not explicitly created, but can be implicitly created by doubling up on one-to-many associations.  More on that later.  


### One-To-One Associations and Methods

Let's talk concrete examples here.  Because we live in a still largely monogenistic (that is the correct spelling of the word, [I looked it up](http://www.dictionary.com/browse/monoganistic?s=t)) and I'm gonna go all heteronormative on you, at least for now, let's say we have a model called Husband and we have a model called Wife.  Each model stores basic information about a specific person. Each wife has one husband and vice versa. (I'm sorry queer and/or polygamous communities, it'll get better soon!)

```
var Wife = this.sequelize.define('wife', {/* attributes */})
  , Husband  = this.sequelize.define('husband', {/* attributes */});
```

In order to create a one-to-one association between exactly two models we can use either the `.belongsTo()` method or the `.hasOne()` method.  From what I can tell, these are **functionally the same**.  The only difference is which model gets the foreign key.

So if we do:

```
var Husband = this.sequelize.define('husband', {/* attributes */}),
  Wife = this.sequelize.define('wife', {/* attributes */});
  
Husband.belongsTo(Wife);
```

Husband, the first model (oh snap, we're getting into some Biblical shit here), or more explicitly the model that the function is being invoked on, is the **source** model and Wife, the model being passed as an argument, is the **target** model. When using  `.belongsTo()` the source model gets the foreign key.  This means that the Husband model will now have an extra column called 'wifeId' which lists the id of the wife, from the Wife model.  If we did:

```
Husband.hasOne(Wife);
```
it's flip-flopped.  In other words, the target model (still Wife) gets the foreign key.  Specifically, there will be a new column in the Wife model called 'husbandId'.  BTW, the default casing is camelCase (hence husbandId) but if you add the `{underscored: true}` attribute to the model definition, the foreign key (and presumably other created entities) will be in snake_case.

So 
```
var Husband = this.sequelize.define('husband', {/* attributes */}, {underscored: true}),
  Wife = this.sequelize.define('wife', {/* attributes */});
  
Husband.belongsTo(Wife);
```

yields a foreign key on the Husband model called 'wife_id' instead of wifeId.  Let's just assume I have `{underscored: true}` for the rest of the post, since wifeId kind of looks like wifeld.  There are some other configuration options such as 'as' which allow you to name your foreign keys differently, and 'targetKey' which allows you to create a foreign key with something other than the primary key, but the docs are pretty clear on that, so no need to discuss.  

The point I am trying to make here is that whether you use `.hasOne()` or `.belongsTo()` the results are the same, just which table gets the foreign key is different.   So `Husband.hasOne(Wife);` creates the exact same results as `Wife.belongsTo(Husband);`.  That is the Wife model will get a husband_id column.  (Yes, yes, typing that sentence did leave a dirty taste on my hands.) The sequelize docs claim "In Sequelize 1:1 relationship can be set using HasOne and BelongsTo. They are suitable for different scenarios." To me this implied that there were truly different scenarios.  Maybe I'm just not understanding, but it seems to me there aren't really different 'scenarios' just which model you want to have the foreign key.

Other than creating foreign keys (or models of join-tables as we'll see in a second), associations create built-in methods, which remember, are what this blogpost is actually supposed to be about.  Part of the reason the documentation on this is kind of sucky because the methods always have different names depending not only on your model names, but also the direction (i.e. `.hasOne` or `.belongsTo`) of the association.  Luckily it's all actually pretty simple.  Let's stick with the `Husband.hasOne(Wife);` association and the methods wrought from this.  

There are only two new methods for a one-to-one association.  After creating the above association, Husband.prototype now has new methods called `.getWife()` and `.setWife()` which the documentation glosses over. The `.getWife()` method will return an instance of wife. **DOUBLE CHECK**  The `.setWife()` methods need to be called *with an actual wife instance not simply an object with the properties you want your new wife attribute to have.* Again, my hands felt dirty writing that. But also I was smirking. This means, you might need to create a wife instance first.  Example below:

```
Wife.create({name: 'Grace Hopper', occupation: 'Bad ass'})
.then(grace => Husband.setWife(grace))
.catch(console.err)
```

### One-To-Many Associations and Methods

Like in the one-to-one associations, there are actually two ways of creating a one-to-many association: `.hasMany()` and `.belongsToMany()`.  Let's get into some polygamous fun!

```
Wife.hasMany(Husband)
```

This creates a wife_id for each husband in the Husband model.  Presumably many husbands will have the same wife_id, why else would be creating this particular association?  Instances of Wife will now have `.getHusbands()` and `.setHusbands()` accessors.  `.getHusbands()` will return an array of husband instances and that you will probably want to `.then()` off. `.setHusbands()` needs to take an array of actual husband instances that will also what to `.then()` with a callback function without arguments to save them.  **IS ANYTHING RETURNED FROM a .set---()? into a then?** You can also pass in a `{where: 'hair === blond'}` as an argument to find only specific instances of blond husbands and `{attributes: ['name', 'occupation']}` to only get the names and jobs of your husbands.

Other than getting and setting, there are a few more methods that are born of this assocation, **adding** and **removing**.

If you want to get rid of a husband, there a couple of ways to do that:

```
// Notice the 's' that gets added to the model name below
Wife.setHusbands([]).then(function(husbands) {
  // you will get an empty array
})
 
// Maybe you just want to get rid of one:
Wife.removeHusband(husband1).then(function() {
  // he's gone
  
// But you want him back:
Wife.addHusband(husband1).then(function() {
// he's back!
```

Again, important to note here is that you need to pass an actual instance of husband into these  `.removeHusband()` and `.addHusband()` just like you needed to in the `.setWife()` method born of a one-to-one association.  On the flipside, if we do 

```
Wife.belongsToMany(Husband, {through: 'Relationships'});
// notice the 'through'.  It is NOT optional and defines the name of the newly created join-table. Erp, I mean join-model. I mean model of a join-table.
```

Let's say we want go totally polygamous though.  This is that implicit many-to-many association I was referencing earlier:

```
Wife.belongsToMany(Husband, {through: 'Relationships'});
Husband.belongsToMany(Wife, {through: 'Relationships'});
```

This creates that new model called Relationships with the equivalent foreign keys husband_id and wife_id.  Husband.prototype now was `.getWifes()` (notice the weirdness created by just adding 's' to the end of the model name unless fixed with the `as` config) `.setWifes()`, `.addWife()`, and `.addWifes()` and Wife.prototype now was `.getHusbands()`, `.setHusbands()`, `.addHusband()`, and `.addHusbands()`.

Ok, polygamous community is probably happier now than at the beginning of my poorly chosen example.  Queer community, you're gonna like this:  You can actually self reference with `belongsToMany`, and actually with `hasOne` too.

```
var Wife = sequelize.define('wife', { /* ... */})

//notice the 'self-reference'?
Wife.hasOne(Wife {as: partnerId})
// This will add the attribute partnerId to Wife
// Note, these are monogamous relationships

Husband.belongsToMany(Husband, { as: 'Partners', through: 'Relationships' })
// This will create the table 'Relationships' which stores the ids of the partners.
// Husbands can have many husbands.
// Not trying to say anything about the differences between gay men and lesbians here.
```

The last method I want to touch on is `.findAll()` which is actually a query method available to all Sequelize models.  I'm bringing it up because it can also be used to query even with associations:

```
Wife.findAll({
  include: [{
    model: Husband,
    through: {
      attributes: ['name'], // thought you're only search for one attribute, it still needs to be in an array
      where: {hair: blond}
    }
  }]
});
// In case it wasn't clear, this let's a girl find all of her blond husbands' names.
```

Believe it or not, this is just the beginning of the rabbit hole.  (Did you know you can define a model for a join-table BEFORE creating the association and then just set-up the association to use it instead of making a new one? Or that you can directly fill in an instance of a join-table model? And that there are different ways to create elements of different associations?)  Through writing this I realized that while the Sequelize docs might not be perfect, like good books, they do improve (and make more sense) with re-reading.  Unlike good books, I will probably not stay up too late reading/re-reading the Sequelize docs.  That said, I have a lot more re-reading (and possible writing) to do.  Next up might be association scopes which already confuse me three sentences in, ["Scopes can be placed both on the associated model (the target of the association), and on the through table for n:m relations."] (http://docs.sequelizejs.com/en/v3/docs/associations/#scopes) Huh? Oh, wait! I think I get it. (See, re-reading. It works.)  Stay tuned in for next time.


Stuff I still need to clarify for myself:
=========================================

##### I either don't understand this paragraph from the docs or disagree with it: 

```
Even though it is called a HasOne association, for most 1:1 relations you usually want the BelongsTo association since BelongsTo will add the foreignKey on the source where hasOne will add on the target. 
```

##### Why add the `Game.belongsTo(Team);` at the end?
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

##### Is the hashtag function ACTUALLY a thing? Is it only created from hasOne associations?

```
Task.hasOne(User, {as: "Author"})
Task#setAuthor(anAuthor)

Project.hasOne(User, { as: 'Initiator' })
// Now you will get Project#getInitiator and Project#setInitiator

// also possible:
Person.hasOne(Person, {as: 'Father', foreignKey: 'DadId'})
// this will add the attribute DadId to Person
 
// In both cases you will be able to do:
Person#setFather
Person#getFather
```








