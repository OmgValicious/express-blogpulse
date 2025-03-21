# Express BlogPulse

To practice 1:M associations, we'll be adding comment functionality to an existing blog application.

#### Backstory: BlogPulse

Congrats! You have been hired by BlogPulse, an up-and-coming blog marketed as a local version of [Buzzfeed](https://www.buzzfeed.com/). Through BlogPulse, anyone can sign up to become a contributor, and contributors can create articles relating to issues and events in the Puget Sound area. However, we need you to add comment functionality to this site.

## Getting Started

We'll be using an existing application that includes two models, several routes, and several views.

* Fork and clone this repository
* Run `npm install` to install dependencies
* Setup your database (this app already has two existing models)
  * Run `createdb blogpulse_dev` to create the database
  * Run `sequelize db:migrate` to run migrations
  * Run `sequelize db:seed:all` to populate the database with 2 authors and 2 articles
* Use `npx nodemon` (or just `nodemon` if you installed it globally) to start your application

#### Read the Code

After setup, **STOP**. You're using an existing application, so make sure to read the code and ensure what the application does. Here is some information about the current setup.

#### Routes

| Method | Path | Purpose |
| ------ | -------------- | -------------------------------- |
| GET | `/` | home page that lists all articles |
| GET | `/authors` | authors page that lists all authors |
| POST | `/authors` | creates a new author, then redirects back to `GET /authors` |
| GET | `/authors/new` | page that has a form for creating a new author |
| GET | `/authors/:id` | page that shows a specific author and their articles |
| POST | `/articles` | creates a new article, then redirects back to `GET /` |
| GET | `/articles/new` | page that has a form for creating a new article |
| GET | `/articles/:id` | page that shows a specific article and the author |

#### Models
  
  * `author`
    * Attributes: `firstName`, `lastName`, `bio`
    * Associations: Has many articles
  * `article`
    * Attributes: `title`, `content`, `authorId`
    * Associations: Belongs to one author

sequelize model:create --name comment --attributes name:string,content:text,articleId:integer
sequelize model:create --name user --attributes firstName:string,lastName:string,age:integer,email:string



## User Stories

* As a user, I want to comment on an article in order to express my opinions.
* As a user, I want to view comments on an article in order to see my community's opinions about an article.

## Requirements

#### Part 1: Create a Comment model

In order to add comments, create a Sequelize model to store comments. It's recommended that you name this model `comment`. It will store three attributes: the name of the person creating the comment (as a string), the content of the comment (as text), and the article that the comment belongs to (as an integer)

Once this model has been created, **add the associations between comments the articles**. This may look similar to how Authors and Articles are related in the existing code in this app. Note the associate section in the models for both `article` and `author`.

**models/article.js**

```js
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class article extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
      models.article.belongsTo(models.author)
    }
  }
  article.init({
    title: DataTypes.STRING,
    content: DataTypes.TEXT,
    authorId: DataTypes.INTEGER
  }, {
    sequelize,
    modelName: 'article',
  });
  return article;
};
```

**models/author.js**

```js
'use strict';
const {
  Model
} = require('sequelize');
module.exports = (sequelize, DataTypes) => {
  class author extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method automatically.
     */
    static associate(models) {
      // define association here
      models.author.hasMany(models.article)
    }
    // instance mothods
    getFullName(){
      return this.firstName + ' ' + this.lastName
    }
  }
  author.init({
    firstName: DataTypes.STRING,
    lastName: DataTypes.STRING,
    bio: DataTypes.TEXT
  }, {
    sequelize,
    modelName: 'author',
  });
  return author;
};
```

Go ahead and associate your new comments model and the existing article model in a similar fashion. This is a one to many relationship. One article can have many comments, but each comment belongs to a single article.

### Create a comment

Now, run the migration for the model and test the model's functionality. This can be done in a separate file. An example:

**dbTest.js**

```js
const db = require('./models')

db.comment.create({
  name: 'Paul Allen',
  content: 'This is really neat! Thanks for posting.',
  articleId: 1
})
.then(comment => {
  console.log(comment.get())
})


const createComment = async () => {
  try {
    const newComment = await db.comment.create({
      name: 'Ada Lovelace',
      content: 'So excited for this!',
      articleId: 2
    })
    console.log(newComment)
  } catch (err) {
    console.log(err)
  }
}

createComment()
```

Be sure to also test querying comments off of articles, which should verify that the association exists. Here's an example, once you've created a comment:

```js
const db = require('./models')

db.article.findOne({
  where: { id: 1 },
  include: [db.comment]
}).then(article => {
  // by using eager loading, the article model should have a comments key
  console.log(article.comments)
})

const readArticles = async () => {
  try {
    const article = await db.article.findOne({
      where: { id: 2},
      include: [db.author]
    })
    console.log(article)
  } catch (err) {
    console.log(err)
  }
}

readArticles()
```

#### Part 2: Integrate the model with the app

Now that the model has been created, you'll want to add the ability to create and view comments to the rest of the application. Here is an approach that can be taken:

* Add the ability to view comments on `GET /articles/:id`.
  * See the example above on how to include the comments, then use EJS to render each comment's information on the page. Make sure you have a comment in the database you can use to verify this functionality.
* On the same page (`GET /articles/:id`), create a form to submit a new comment. Note that we don't *necessarily* need to render a form on a separate page - most sites have a comment form on the same page.
  * Include the necessary attributes, `name` and `content`. Feel free to look at the forms for authors and articles as examples.
  * Create a new route to receive this form data. This will be the action for your form. You could either make a separate comments controller at `POST /comments`, (especially good if you plan on having more comments-related routes in the future) or you could define the route in the articles controller since comments are related to articles `POST /articles/:id/comments`. This implementation detail is up to you. Note how we're passing the article id in each case - in the form body in the first example vs a param (part of the URL) in the second one.
    * Test the route by using your form 
    * Once you've verified the route is working, redirect back to the article that was commented on for a completely smooth user experience.
* Verify functionality by creating more authors, articles, and comments. Pay attention to the user experience, and make sure the user can navigate between articles, authors, and comments.

#### Part 3: Styling

When finished with the above, style the application appropriately with CSS. Use other media and blog sites as examples.

## Bonuses

* Add the ability to edit articles
* Instead of redirecting to `/authors/new` to create a new author, have the form appear using a Bootstrap modal.
* Add the ability to create rich text articles and comments using Markdown.
  * Front-end Bootstrap Markdown editor
  * A markdown parser for your EJS templates

## Deliverables

Here's an example screenshot of the article page, complete with comments. Your finished deliverable will differ and include the desired functionality.

![Example Comments](./example-comments.jpg)

---

## Licensing
1. All content is licensed under a CC-BY-NC-SA 4.0 license.
2. All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.
