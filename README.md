# graphql-apollo-server

Implement a **GraphQL server with TypeScript** based on Prisma, [apollo-server](https://www.apollographql.com/docs/apollo-server/) and [GraphQL Nexus](https://graphql-nexus.com/).

## Getting Started

### 1. Prisma CLI

Install Prisma CLI to run the server

```bash
npm i -g prisma
```

### 2. Install

Clone the repository:

```bash
git clone https://github.com/flavioespinoza/graphql-apollo-server.git
```

Install Node dependencies:

```bash
cd graphql-apollo-server
npm i
```

### 3. Docker Compose

Ensure you have Docker installed on your machine. If not, you can get it from [here](https://store.docker.com/search?offering=community&type=edition).

Review `docker-compose.yml` for MySQL

```yaml
version: '3'
services:
  prisma:
    image: prismagraphql/prisma:1.34
    restart: always
    ports:
    - "4466:4466"
    environment:
      PRISMA_CONFIG: |
        port: 4466
        databases:
          default:
            connector: mysql
            host: mysql
            port: 3306
            user: root
            password: prisma
            migrations: true
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: prisma
    volumes:
      - mysql:/var/lib/mysql
volumes:
  mysql:
```


Run docker to create database
```bash
docker-compose up -d
```

### 4. Endpoint

Create `prisma/prisma.yml` which sets the enpoint on your localhost (default 4466):

```bash
touch prisma/prisma.yml
```

Copy the code below and paste into prisma.yml

```yaml
# Specifies the HTTP endpoint of your Prisma API.
endpoint: 'http://localhost:4466'

# Defines your models, each model is mapped to the database as a table.
datamodel: datamodel.prisma

# Specifies the language and directory for the generated Prisma client.
generate:
  - generator: typescript-client
    output: ../src/generated/prisma-client/

# Ensures Prisma client is re-generated after a datamodel change.
hooks:
  post-deploy:
    - prisma generate
    - npx nexus-prisma-generate --client ./src/generated/prisma-client --output ./src/generated/nexus-prisma # Runs the codegen tool from nexus-prisma.

# Seeds initial data into the database by running a script.
seed:
  run: yarn ts-node ./prisma/seed.ts
```

### 5. GraphQL Server
Start the graphql server

```bash
npm start
```

Navigate to [http://localhost:4000](http://localhost:4000) in your browser to explore the API of your GraphQL server in a [GraphQL Playground](https://github.com/prisma/graphql-playground).


### 6. Data
For this example, you'll use a free _demo database_ (AWS Aurora) hosted in Prisma Cloud.

Open a new Terminal shell and deploy data to the database
```bash
prisma deploy
```

### 6. Using the GraphQL API

The schema that specifies the API operations of your GraphQL server is defined in [`./src/schema.graphql`](./src/schema.graphql). Below are a number of operations that you can send to the API using the GraphQL Playground.

Feel free to adjust any operation by adding or removing fields. The GraphQL Playground helps you with its auto-completion and query validation features.

## Query

#### Retrieve all published posts and their authors

```graphql
query {
  feed {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

## Mutation

####Create a new user

```graphql
mutation {
  signupUser(
    name: "Flavio"
    email: "flavio@webshield.io"
  ) {
    id
  }
}
```

#### Create a new draft

```graphql
mutation {
  createDraft(
    title: "See what Webshield and our partners are doing."
    content: "https://webshield.io/partners"
    authorEmail: "flavio@webshield.io"
  ) {
    id
    published
  }
}
```

This will return an id that you will use to publish

```json
{
  "data": {
    "publish": {
      "id": "cjwgekemk004x0894zsfvjple",
      "published": true
    }
  }
}
```


#### Publish an existing draft

Use id returned from previous 

```graphql
mutation {
  publish(id: "__POST_ID__") {
    id
    published
  }
}
```

#### Search for posts with a specific title or content

```graphql
{
  filterPosts(searchString: "graphql") {
    id
    title
    content
    published 
    author {
      id
      name
      email
    }
  }
}
```

#### Retrieve a single post

Use id returned from previous 

```graphql
{
  post(id: "__POST_ID__") {
    id
    title
    content
    published
    author {
      id
      name
      email
    }
  }
}
```

#### Delete a post

Use id returned from previous 

```graphql
mutation {
  deletePost(id: "__POST_ID__") {
    id
  }
}
```

### 6. Changing the GraphQL schema

To make changes to the GraphQL schema, you need to manipulate the `Query` and `Mutation` types that are defined in [`index.ts`](./src/index.ts). 

Note that the [`start`](./package.json#L6) script also starts a development server that automatically updates your schema every time you save a file. This way, the auto-generated [GraphQL schema](./src/generated/schema.graphql) updates whenever you make changes in to the `Query` or `Mutation` types inside your TypeScript code.

## Prisma Resources

- [Use Prisma with an existing database](https://www.prisma.io/docs/-t003/)
- [Explore the Prisma client API](https://www.prisma.io/client/client-typescript)
- [Learn more about the GraphQL schema](https://www.prisma.io/blog/graphql-server-basics-the-schema-ac5e2950214e/)