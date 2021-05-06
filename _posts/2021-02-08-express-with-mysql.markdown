---
layout: post
title: Express with MySQL
date: 2021-02-15 20:35:46 -0800
categories: node express mysql
---

Have you ever wondered how you can hook up a MySQL DB to express? In this article I am going to explain in great detail how to hook up Express with an MySQL database.

This ariticle will go about showing you the file structures that you will need as well as to the bare minimum of what you need to get everything working. For my project I used
React for my front end and I also used node.js alongside express. 

From the client side:

#### App.js

{% highlight jsx %}

import { Route, Router, Switch } from "react-router";
import { history } from './browserHistory';
import { HomeView } from "./views/HomeView";
import './App.scss';

export const App = () => {
    return (
        <div className="App">
            <Router history={history}>
                <Switch>
                    <Route exact path="/">
                        <HomeView/>
                    </Route>
                </Switch>
            </Router>
        </div>
    );
}
{% endhighlight %}

Wow, Look at that , pretty easy right?  With the client set up, lets move on to the server side and serve up some code for it.

Your server will have four folders: Config, Controllers, Models, and Routes. 
For this project I will use two models, a User and Movie model. Feel free to rename them to whatever you want for your project.

From the server side:

### config folder:

#### database.js

{% highlight javascript %}

const { defineUserModel } = require("../models/user");
const { defineMovieModel } = require("../models/movie")

const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('sqlite::memory:');

console.log("initializing database.");

const User = defineUserModel(sequelize);
const Movie = defineMovieModel(sequelize);

sequelize
    .sync({ force: true })
    .then(() => {
        console.log(`Database & tables created!`);
        const { insertTestData } = require("../models/insert_data");
        insertTestData(User, Movie);
    });

module.exports = {
    User,
    Movie
};

{% endhighlight %}

### controllers folder

#### movie.js

{% highlight javascript %}
const { Movie } = require('../config/database');

const createMovie = (req, res) => {
    Movie.create(req.body)
        .then(movie => {
            console.log(movie);
            res.json(movie);
        });
};

const findAllMovies = (req, res) => {
    Movie.findAll()
        .then(movies => res.json(movies))
        .catch(err => res.status(400).json(err));
};

const findMovie = (req, res) => {
    Movie.findByPk(req.params.id)
        .then(movie => res.json(movie))
        .catch(err => res.status(400).json(err));
};

const updateMovie = (req, res) => {
    Movie.findByPk(req.params.id)
        .then(movie => {
            movie.update(req.body)
                .then(updatedMovie => res.json(updatedMovie))
                .catch(err => res.status(400).json(err));

        })
        .catch(err => res.status(400).json(err));
};

const deleteMovie = (req, res) => {
    Movie.findByPk(req.params.id)
        .then(movie => {
            movie.destroy()
                .then(deletedMovie => res.json(deletedMovie))
                .catch(err => res.status(400).json(err));
        })
        .catch(err => res.status(400).json(err));
};

module.exports = {
    createMovie,
    findAllMovies,
    findMovie,
    updateMovie,
    deleteMovie
};

{% endhighlight %}

#### user.js

{% highlight javascript %}
const { User } = require('../config/database');

const createUser = (req, res) => {
    User.create(req.body)
        .then(user => {
            console.log(user);
            res.json(user);
    });
};

const findAllUsers = (req, res) => {
    User.findAll()
        .then(users => res.json(users))
        .catch(err => res.status(400).json(err));
};

const findUser = (req, res) => {
    User.findByPk(req.params.id)
        .then(user => res.json(user))
        .catch(err => res.status(400).json(err));
};

const updateUser = (req, res) => {
    User.findByPk(req.params.id)
        .then(user => {
            user.update(req.body)
                .then(updatedUser => res.json(updatedUser))
                .catch(err => res.status(400).json(err));
        })
        .catch(err => res.status(400).json(err));
};

const deleteUser = (req, res) =>{
    User.findByPk(req.params.id)
        .then(user => {
            user.destroy()
                .then(deletedUser => res.json(deletedUser))
                .catch(err => res.status(400).json(err));
        })
        .catch(err => res.status(400).json(err));
};


module.exports = {
    createUser,
    findAllUsers,
    findUser,
    updateUser,
    deleteUser
};

{% endhighlight %}

Next up we are going to take a look at a very special  file, our insert_data.js file. The entire point of this file is to insert dummy data into our DB to make
sure that our MySQL and Express are properly connected.

### models folder:

#### insert_data.js

{% highlight javascript %}
const insertTestData = async (User, Movie) => {
    console.log("inserting test data")

    const jane = await User.create({
        id: 1,
        name: 'jane doe',
        numberOfReviews: 0,
        bio: 'i am jane doe'
    });
    console.log(jane.toJSON());

    const aliens = await Movie.create({
        id: 1,
        title: 'aliens',
        reviewedBy: 1
    });
    console.log(aliens.toJSON());
};

module.exports = {
    insertTestData
};


{% endhighlight %}


#### movie.js

{% highlight javascript %}
const { DataTypes } = require('sequelize');
    const defineMovieModel = (sequelize) => {

    console.log("initializing movie model.");

    return sequelize.define('movie',  {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },
        title: DataTypes.STRING,
        reviewedBy: DataTypes.NUMBER
    });

};

module.exports = {
    defineMovieModel
}

{% endhighlight %}


#### user.js

{% highlight javascript %}
const { DataTypes } = require('sequelize');

const defineUserModel = (sequelize) => {

    console.log("initializing user model.");

    return sequelize.define('user',  {
        id: {
            type: DataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },
        name: DataTypes.STRING,
        numberOfReviews: DataTypes.INTEGER,
        bio: DataTypes.STRING
    });

};

module.exports = {
    defineUserModel
};

{% endhighlight %}

### routes folder

#### routes.js

{% highlight javascript %}

const user = require("../controllers/user");
const movie = require("../controllers/movie");

module.exports = (app) => {
    app.get("/users", user.findAllUsers);
    app.post("/users", user.createUser);
    app.get("/users/:id", user.findUser);
    app.put("/users/:id", user.updateUser);
    app.delete("/users/:id", user.deleteUser);

    app.get("/movies", movie.findAllMovies); 
    app.post("/movies", movie.createMovie); 
    app.get("/movies/:id", movie.findMovie); 
    app.put("/movies/:id", movie.updateMovie); 
    app.delete("/movies/:id", movie.deleteMovie); 
}

{% endhighlight %}

For your dependencies  you are going to need cors,express,mysql,mysql2,sequelize, and sqlite3.

#### server.js

{% highlight javascript %}
const express = require("express");
const cors = require("cors");

const app = express();
const PORT = 7777;

app.use(cors());
app.use(express.json());
app.use(express.urlencoded({extended:true}));

require('./config/database'); // may need to change this
require('./routes/routes')(app);

// import the data
// don't import data in the /config/database file because we don't
// want to insert data every time we import data database models.
require("./models/insert_data");

app.listen(PORT, () => console.log(`your server is running on port ${PORT}`));

//npm run start for 1st terminal
//nodemon server.js for 2nd terminal
{% endhighlight %}

As a final, we are going to be running node.js for your project so do be sure to download node as well for your project. Be sure to
download nodemon(not pokemon or digimon) for your project.

We will need two terminals. The first is for our client. 
#### client terminal: 

`npm run start`

The second terminal is for our server.
#### server terminal: 

`nodemon server.js`

And there you have it, this should get you on your way to hooking up express and MySQL. 
