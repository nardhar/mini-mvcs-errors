# Mini MVCS Errors

Small Utility library for handling and transforming errors in a rest app style, but it could be used in any other setting.

Extracted first from Mini MVCS Library and from Node Backend Skel for individual use without any other dependencies.

## Features

- Throw Errors that follow a simple convention whenever you want
- Transform Errors to keep flow of the application

## Installation

Using npm:

```
npm install --save mini-mvcs-errors
```

## Usage

This library provides one main Error Abstract Class (but extensible) and two other frequently used error classes

### ApiError

The base class which other classes extend from

### NotFoundError

The error to use when searching for a resource and it is not found

#### constructor params

- **resourceName** (String): Name of the resource
- **filter** (Object): Filters used to find the resource

Example:

```javascript
const searchBookWrapper = (bookId) => {
  return methodForSearchingBook(bookId)
  .then((result) => {
    // the method returns null when a resource is not found
    if (!result) {
      throw new NotFoundError('Book', { id: bookId });
    }
    return result
  });
};
```

### ValidationError

The error to use when executing an operation that could not be completed because the user made an error

#### constructor params

- **resourceName** (String): Name of the resource or executing operation
- **fieldErrors** (Array<FieldError>): A list of the errors that the user made

Example:

```javascript
const createBookWrapper = (book) => {
  if (!book.name) {
    throw new ValidationError('Book', [
      new FieldError('name', 'null'),
    ]);
  }
  return methodForCreatingBook(book)
};
```

### FieldError

Although not a proper Error (can not be throwed) it is used for having a unique format for the body of the ValidationError

### Transforming

Both AuthorizationError and NotFoundError provide an static method that allows them to be catched and transformed to some successful result or another error

Example:

```javascript
// the creation of the Book should only return if a book is created successfully or throw a ValidationError if something is wrong (e.g.: the author relationship is not found)
const createBookWrapper = (book) => {
  return database.searchAuthorById(book.authorId)
  // it throws a NotFoundError when the authorId does not exists in the database
  .catch(NotFoundError.handle((notFoundErrorInstance) => {
    // the notFoundErrorInstance is provided for using in our code
    // we handle the NotFoundError by throwing a ValidationError
    throw new ValidationError('Book', [
      new FieldError('author', 'invalid', [book.authorId]);
    ]);
  }))
  // the author exists
  .then(() => {
    return database.createBook(book)
  });
};
```

the NotFoundError.handle is a shortcut for:

```javascript
.catch((error) => {
  if (error instanceof NotFoundError) {
    // ... callback
    return callbackPassedInHandle();
  }
  throw error;
});
```

Sometimes you just want to return a simple value to the callback of the .handle method, in that case you just need to pass it as the only param

Example:

```javascript
const createBookWrapper = (book) => {
  return database.searchAuthorById(book.authorId)
  // it throws a NotFoundError when the authorId does not exists in the database
  .catch(NotFoundError.handle({ id: 1 })) // default author to be used
  // using the sent author if found or the default author
  .then((author) => {
    // modifying the book to be created
    return database.createBook({
      ...book,
      authorId: author.id,
    })
  });
};
```

There are some other common uses for the .handle method that I will put here in the future

## License

[MIT](https://github.com/nardhar/mini-mvcs-errors/blob/master/LICENSE)
