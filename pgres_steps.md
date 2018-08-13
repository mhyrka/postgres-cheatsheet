### First Time CLI Installs

- `brew update` //first time install
- `brew install postgres` //first time install
- `brew services start postgresql` //first time install
- `brew services list` //first time install
- `psql --list` //list all postgres databases
- `npm install knex -g` //first time install

### Create Database and Set Up Directory

- `createdb name_of_database` //creates your database
- `take directory-name`
- `git init` //creates empty git repo
- `npm init`
- `npm install --save express knex pg morgan cors body-parser nodemon` //installs everything you will need
- go into your package.json and edit scripts
  `"start": "node index.js"`

### Setting Up knexfile.js

- `knex init`
- Make your file look like this:

```
module.exports = {
  development: {
    client: 'pg',
    connection: 'postgres://localhost/name_of_db'
  },
  production: {
    client: 'pg',
    connection: process.env.DATABASE_URL
  }
}
```

### Create Migration File

- `knex migrate:make name_of_table`
- edit file to look like such

```
exports.up = function(knex, Promise) {
  return knex.schema.createTable('table_name', table => {
    table.increments('id')
    table.text('name')
    table.boolean('yes or no')
    table.float('number')
  })
}
exports.down = function(knex, Promise) {
  return knex.schema.dropTableIfExists('table_name')
}
```

##### CLI Commands

- `knex migrate:latest` //migrate latest creates columns for table
- `psql database_name` //open postgres to see database lists
- `database_name=# \dt` //show data tables
- `database_name=# select * from snacks;`
- exit psql by typing `\q`

### seed data to table

- `knex seed:make 00_name` //why this format?
  edit seed file to look like this

```
exports.seed = function(knex, Promise) {
  return knex('name_of_table').del()
    .then(function () {
      return knex('name_of_table').insert([
        {DATA OBJECT}
        {DATA OBJECT}
        {DATA OBJECT}
      ])
    })
}
```

- `knex seed:run` //this seeds the data to the table in migration file

### Create app.js file

- Add boilerplate

```
const express = require("express")
const bodyParser = require("body-parser")
const cors = require('cors')
const morgan = require("morgan")
const app = express()
app.use(morgan('dev'))
app.use(bodyParser.urlencoded({
  extended: false
}))
app.use(bodyParser.json())
app.use(cors({
  origin: true,
  credentials: true
}))
app.use((req, res, next) => {
  const err = new Error("Not Found")
  err.status = 404
  next(err)
})
app.use((err, req, res, next) => {
  res
    .status(err.status || 500)
    .json({
      message: err.message,
      error: req.app.get("env") === "development" ? err.stack : {}
    })
})
module.exports = app
```

### Create index.js File

- Edit file to look like this:

```
const app = require("./app");
const port = parseInt(process.env.PORT || 3000)
app.listen(port)
  .on('error', console.error.bind(console))
  .on('listening', console.log.bind(console, 'Listening on ' + port))
```

### Connect server to Database

- create connection.js file
- insert following code:

```
const environment = process.env.NODE_ENV || 'development'
const config = require('./knexfile')
const environmentConfig = config[environment]
const knex = require('knex')
const connection = knex(environmentConfig)

module.exports = connection
```

### Queries

- Create "queries" folder
- Add "queries.js" file

```
const database = require("../connection")
module.exports = {
  list() {
    return database('name_of_table')
  },
  create(user) {
    return database("name_of_table").insert(user, '*').then(record => record[0])
  },
  update(id, user) {
    return database('name_of_table').where('id', id).update(user, '*').then(user => user[0])
  },
  delete(id) {
    return database('name_of_table').where('id', id).delete()
  },
  read(id) {
    return database('name_of_table').where('id', id).first()
  }
}
```

### Routes

- create "routes" folder
- create "routes.js" file

```
const express = require('express')
const router = express.Router()
const queries = require('../queries/queries.js')
router.get("/", (request, response, next) => {
    queries.list()
    .then(company => {
        response.json({company})
    })
    .catch(next)
})
router.get("/:id", (request, response) => {
    queries.read(request.params.id)
        .then(company => response.json(company))
})
router.delete('/:id', (request, response) => {
    queries.delete(request.params.id).then(() => {
        queries.list()
        .then(company => response.json({company}))
    })
})
router.post('/', (request, response, next) => {
    queries.create(request.body)
    .then(()=> {
        queries.list().then(company => {
            response.status(201).json({company})
        })
    }).catch(next)
})
router.put('/:id', (request, response) => {
    queries.update(request.params.id, request.body)
        .then(company => {
            response.json({company: company[0]})
        })
        .catch(console.error)
})
module.exports = router
```

> Note: not all the above code works properly right now, but it's a good starting point

### Edit app.js to include routes

```
const route = require("./routes/routes")
app.use("/", route)
```

### Test Your Code

- in CLI, run `nodemon`
- CLI should say "Listening on 3000"
- go to localhost:3000 and see if your data is displaying
- go to localhost:3000/1 to see if you can view data by index

### Deploy to Heroku

- `heroku create name_your_database`
- `git add .`
- `git commit -m "heroku create"`
- `git push heroku master`
- `heroku addons:create heroku-postgresql:hobby-dev`
- `heroku run knex migrate:latest`
- `heroku run knex seed:run`
- `heroku pg:psql`

### Additional CLI

- `knex migrate:rollback` //deletes tables from recent migrations, allows to make new tables
