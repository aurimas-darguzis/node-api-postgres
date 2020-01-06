# RESTful API with Node.js and PostgreSQL

- Fully functional API server running on an Express framework in Node.js
- The API handles the HTTP request methods that correspond to the PostgreSQL database that the API gets its data from.

## PostgreSQL

```bash
brew install postgresql
```

```bash
brew services start postgresql
```

Connect to the default postgres

```bash
psql postgres
```

- `\q` Exit psql connection
- `\c` Connect to a new database
- `\dt` List all tables
- `\du` List all roles
- `\list` List databases

### Create Postgres user

```bash
postgres=# CREATE ROLE me WITH LOGIN PASSWORD 'password';
```

```bash
postgres=# ALTER ROLE me CREATEDB;
```

run `\du` to list all roles and see the changes.
then `\q` to quit postgres as superuser
re-login with created user `me`:

```bash
psql -d postgres -U me
```

### Create a database

```bash
postgres=> CREATE DATABASE api;
```

Use the `\list` command to see the available databases.

Connect to the new `api` database with `me` using the `\c` (connect) command:

```bash
postgres=> \c api
You are now connected to database "api" as user "me"
api=>
```

### Create a table

```bash
api=>
CREATE TABLE users (
  ID SERIAL PRIMARY KEY,
  name VARCHAR(30),
  email VARCHAR(30)
);
```

Insert values into `users` table

```bash
INSERT INTO users (name, email)
  VALUES ('John', 'john@example.com'), ('Marrie', 'marrie@example.com');
```

Test if values are in the table

```bash
api=> SELECT * FROM users;
id |  name  |       email
----+--------+--------------------
  1 | John  | john@example.com
  2 | Marrie | marrie@example.com
```

## Connecting to the database from Node.js

```js
const Pool = require('pg').Pool;
const pool = new Pool({
  user: 'me',
  host: 'localhost',
  database: 'api',
  password: 'password',
  port: 5432
});
```

### GET all users

```js
const getUsers = (request, response) => {
  pool.query('SELECT * FROM users ORDER BY id ASC', (error, results) => {
    if (error) {
      throw error;
    }
    response.status(200).json(results.rows);
  });
};
```

### GET a single user by id

```js
const getUserById = (request, response) => {
  const id = parseInt(request.params.id);

  pool.query('SELECT * FROM users WHERE id = $1', [id], (error, results) => {
    if (error) {
      throw error;
    }
    response.status(200).json(results.rows);
  });
};
```

### POST a new user

```js
const createUser = (request, response) => {
  const { name, email } = request.body;

  pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2)',
    [name, email],
    (error, results) => {
      if (error) {
        throw error;
      }
      response.status(201).send(`User added with ID: ${result.insertId}`);
    }
  );
};
```

### PUT updated data in an existing user

```js
const updateUser = (request, response) => {
  const id = parseInt(request.params.id);
  const { name, email } = request.body;

  pool.query(
    'UPDATE users SET name = $1, email = $2 WHERE id = $3',
    [name, email, id],
    (error, results) => {
      if (error) {
        throw error;
      }
      response.status(200).send(`User modified with ID: ${id}`);
    }
  );
};
```

### DELETE a user

```js
const deleteUser = (request, response) => {
  const id = parseInt(request.params.id);

  pool.query('DELETE FROM users WHERE id = $1', [id], (error, results) => {
    if (error) {
      throw error;
    }
    response.status(200).send(`User deleted with ID: ${id}`);
  });
};
```

Export all functions from `queries.js`

```js
module.exports = {
  getUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser
};
```

## Complete setup

index.js

```js
const db = require('./queries');

app.get('/users', db.getUsers);
app.get('/users/:id', db.getUserById);
app.post('/users', db.createUser);
app.put('/users/:id', db.updateUser);
app.delete('/users/:id', db.deleteUser);
```

## Test

#### GET

Open your browser on `localhost:3000/users` to GET all users

#### POST

```bash
curl --data "name=Elly&email=elly@example.com"
http://localhost:3000/users
```

#### PUT

```bash
curl -X PUT -d "name=Kramer" -d "email=kramer@example.com"
http://localhost:3000/users/1
```

#### DELETE

```bash
curl -X "DELETE" http://localhost:3000/users/1
```
