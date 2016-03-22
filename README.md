# A GraphQL proxy that uses graphql-js

MVP for a GraphQL proxy. Initially nothing more than a GraphQL server.

This GraphQL proxy is built on
* express
* graphql-js
* graphql-shorthand-parser


## How to use:

Apollo-proxy takes two inputs:
1. A GraphQL schema in shorthand format
1. One or more javascript code files that define resolve functions

An example of a valid GraphQL shorthand input:
```
type Book {
  id: ID!
  title: String!
  genre: Genre
  author: Author
}

interface Person {
  firstName: String!
  lastName: String
  middleName: String
}

type Author: Person {
  id: ID!
  firstName: String!
  lastName: String
  middleName: String
  booksPublished: [Book]
}

enum Genre {
  FICTION
  NONFICTION
}

type RootQuery {
  author(id: ID!): Author,
  book(title: String!): Book
}

type RootMutation {
  addBook(title: String!, genre: Genre!, author_id: ID): Book
}
```

The corresponding file that defines how fields are resolved. If a field does not
have a resolve function, it is assumed to be the standard resolve function.
```
import DB from "DB.js";

export const {
  Book: {
    author: (book, _, _) => DB.Authors.get(book.author_id),

    // fields without resolve function just return book.<fieldName>
  },

  Author: {
    booksPublished: (author, _, _) => DB.Books.list( {author_id: author.id}),
  },

  RootQuery: {
    author: (_, { id }, _) => DB.Authors.get(id),
    book: (_, { title }, _) => DB.Books.getByTitle(title),
  },

  RootMutation: {
    addBook: (_, { title, genre, author_id }, _) => {
      return DB.createBook( {title: title, genre: genre, author_id: author_id});
    },
  },
}
```

Providing a schema in this format has the advantage of staying relatively close
to graphql-js while at the same time separating the type system from the
resolve rules for clarity.

It is still possible to dynamically generate the resolve functions, as they just
have to be exported from the file that Apollo Proxy imports for the schema.


It is also possible to not use the shorthand with resolve definitions and simply
upload a schema for graphql-js in the standard format.
