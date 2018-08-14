# mongo-crud
- a small database that has basic CRUD functionality and uses mongo/mongoose and mocha for testing

[Getting Started](#getting-started)<br />
[Setup](#setup)<br />
[Creating the model](#create-the-model)<br />
[Creating the Connection](#create-the-test-connection)<br />
[Creating a DB item](#create-and-test-an-item-to-the-db)<br />
[Finding an Item in the DB](#find-an-item-in-the-db)<br />
[Deleting an Item in the DB](#deleting-items-in-the-db)<br />
[Updating the Item in the DB](#updating-items-in-the-db)<br />
[Validation](#validation)<br />

## Getting started 
- clone the repo
- install the node modules
- `$ npm run test`

## Setup
- create the script to run the test
    - `"test": "mocha"` 
    -  `"test": "nodemon --exec 'mocha -R min'"` (hot reloading)

## Create the Model
- import mongoose and the schema object
    ```javascript
    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;
    ```
- create the Schema
    ```javascript
    const UserSchema = new Schema({
        name: String
    })
    ```
- create the model 
    ```javascript
    const User = mongoose.model('user', UserSchema);
    ```
- export:
    ```javascript
    module.exports = User;
    ```

## Create the Test Connection
- import mongoose and state that you want to use ES6 promises
    ```javascript
    const mongoose = require('mongoose');
    mongoose.Promise = global.Promise;
    ```
- create and connect to the db
    ```javascript
    before((done) => {
        mongoose.connect('mongodb://localhost/users_test');
        mongoose.connection
            .once('open', () =>  done() )
            .on('error', () => (error) => {
                console.warn('Warning', error);
            })
    })
    ```

- connect to the collection and drop before each test
    ```javascript
    beforeEach((done) => {
        mongoose.connection.collections.users.drop(() => {
            // ready to run the next test! 
            done();
        });
    });
    ```
## Create and Test an Item to the DB
- setup: import the assert lib and model
    ```javascript
    const assert = require('assert');
    const User = require('../src/user');
    ```
- create and save a book to the db
    ```javascript
    describe('Creating records', () => {
        it('saves a book', (done) => {
            const book1 = new Book({ title: 'Oliver Twist'})

            book1.save()
                .then(() => {
                    assert(!book1.isNew); 
                    done();
                })
        })
    });
    ```
## Find an Item in the db
- setup: import the model and assert lib; create a `beforeEach()` function that creates and saves a new model 
    ```javascript
    const assert = require('assert');
    const User = require('../src/user');

    describe('Reading books out of the database', () => {
        let book1;

        beforeEach((done) => {
            book1 = new Book({ title: 'Oliver Twist'});
            book1.save()
                .then(() => done());
        })
    })
    ```
- use `.find()` to find multiple books (returns an array)
    ```javascript
        it('finds all users with a name of joe', (done) => {
            Book.find({ title: 'Oliver Twist' })
                .then((books) => {
                    assert((books[0]._id.toString() === book1._id.toString()));
                    done();
                })
        })
    ```
- user `.findOne()` to find a particular book (returns just one)
    ```javascript
        it ('find a book with a particular id', (done) => {
            Book.findOne({ _id: joe._id })
                .then((book) => {
                    assert(book.title === 'Oliver Twist');
                    done();
                })
        })
    ```
## Deleting items in the db
- setup: import the model and assert lib; create a `beforeEach()` function that creates and saves a new model
    ```javascript
    const assert = require('assert');
    const Book = require('../src/book');

    describe('Deleting a book', () => {
        let book1;

        beforeEach((done) => {
            book1 = new Book({title: 'Oliver Twist'});
            book1.save()
                .then(() => done())
        })
    })
    ```
- create a helper function that finds and asserts a book
    ```javascript
        function assertTitle(operation, done) {
            operation 
                .then(() => Book.findOne({ title: 'Oliver Twist'}))
                .then((book) => {
                    assert(book === null)
                    done();
                })
        }
    ```
- remove a model instance
    ```javascript
        it('model instance remove', (done) => {
            assertTitle(book1.remove(), done);
        })
    ```
- class method remove multiple based on criteria
    ```javascript
        it('class method remove', (done) => {
            assertTitle(Book.remove({ title: 'Oliver Twist'}), done);
        })
    ```
- class method remove one based on criteria
    ```javascript
        it('class method findAndRemove', (done) => {
            assertTitle(Book.findOneAndRemove({ title: 'Oliver Twist'}), done);
        })
    ```
- class method remove one based on the ID
    ```javascript
        it('class method findByIdAndRemove', (done) => {
            assertTitle(Book.findByIdAndRemove(book1._id), done)
        })
    ```
## Updating items in the db
- setup: import the model/assert lib; create a `beforeEach()` function that creates and saves the model
    ```javascript
    const assert = require('assert');
    const Book = require('../src/book');

    describe('Updating records', () => {
        let book1;

        beforeEach((done) => {
            book1 = new Book({ title: 'Oliver Twist'})
            book1.save()
                .then(() => done())
        })
    })
    ```
- create a helper function that finds all the books and asserts the book length is still one and properly updated to the right title
    ```javascript
        function assertTitle(operation, done) {
            operation   
                .then(() => Book.find({}))
                .then((books) => {
                    assert(books.length === 1)
                    assert(books[0].title === 'The Great Gatsby')
                    done();
                })
        }
    ```
- create an instance method using `set()` and `save()`
    ```javascript
        it('instance type using set n save', (done) => {
            book1.set('title', 'The Great Gatsby');
            assertTitle(book1.save(), done)
        })
    ```
- create an instance method using `update()`
    ```javascript
        it('A model instance can update', (done) => {
            assertTitle(book1.update({ title: 'The Great Gatsby'}), done)
        })
    ```
- create a class method using `update()`
    ```javascript
        it('A model class can update', (done) => {
            assertTitle(
                Book.update({ title: 'Oliver Twist'}, {title: 'The Great Gatsby'}),
                done    
            )
        })
    ```
- create a class method using `findOneAndUpdate()`
    ```javascript
        it('A model class can update one record', (done) => {
            assertTitle(
                Book.findOneAndUpdate({ title: 'Oliver Twist'}, { title: 'The Great Gatsby'}),
                done
            )
        })
    ```
- crete a class method that usies `findByIdAndUpdate()`
    ```javascript
        it('A model class can find a record with an Id and update', (done) => {
            assertTitle(
                Book.findByIdAndUpdate(book1._id, {title: 'The Great Gatsby'}),
                done
            )
        })
    ```

## mongo-validation 
- a project that creates a user database and makes sure the user enters valid information.

- setup
    - in `user.js`: create the schema and export the module.
        ```javascript
        const mongoose = require('mongoose');
        const Schema = mongoose.Schema;

        const UserScema = new Schema({
            name: String,
            phoneNumber: Number,
            email: String
        })

        const User = mongoose.model('user', UserScema);

        module.exports = User;
        ```
    - create the tests
        - for test creation, [see this repo](https://github.com/mlizchap/mongo-crud)


- making a feild required
    ```javascript
    const UserScema = new Schema({
        name: {
            type: String,
            required: [true, "Name is required"]
        },
    ```

- making rules for a specific field
    ```javascript
        phoneNumber: {
            type: String,
            validate: {
                validator: function(v) {
                return /\d{3}-\d{3}-\d{4}/.test(v);
                },
                message: 'Not a valid phone number!'
            }
        },
    ```

### Validation Testing
- test to make sure value exists if required
    ```javascript
        it('requires a user name', () => {
            const user = new User({ name: undefined})
            const validationResult = user.validateSync();
            const { message } = validationResult.errors.name;
            assert(message === 'Name is required');
        })
    ```

- test to make sure validation works properly
    ```javascript
        it('validates a phone number', () => {
            const user = new User({
                name: 'Jane',
                phoneNumber: 123-456
            })
            const validationResult = user.validateSync();        
            const { message } = validationResult.errors.phoneNumber;
            assert(message === 'Not a valid phone number!')
        })
    ```
- test to make sure invalid entries do not get saved
    ```javascript
        it('disallows invalid phone numbers from being saved', (done) => {
            const user = new User({
                name: 'Jane',
                phoneNumber: 123-456
            })
            user.save()
                .catch((validationResult) => {
                    const {message} = validationResult.errors.phoneNumber
                    assert(message === 'Not a valid phone number!');
                    done();
                })
        })
    ```