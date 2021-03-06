db == database

to view databases that already exist on your local machine:

`show dbs`

collection == collection of documents; use name of collection you're
querying. Any time a collection is referenced that doesn't exist, it will be
created.

generic == db.collection.method()

  To view all collections in your current database:

  `show collections`

  If your database is books and your collection is orwell, you would do:

  `use books`
  `db.orwell.method`

  Here are some CRUD operations:

  To Read a document, simply do:

`db.collection.find() // Returns all documents in a collection,
  // unless parameters are specified. Will return all documents matching the
  // parameters specified.`

  Parameters look like JSON, for instance, if we want to return all books in
  the orwell collection, we would do:

  `db.orwell.find()`

  If we wanted to find only Animal Farm, it would look like:

  `db.orwell.find({"title": "Animal Farm"})`

  You may do this with any combination of parameters in JSON-style format. For
  instance, if there were two books named Animal Farm written by George Orwell
  (there aren't, just for the sake of illustration), we could narrow it down
  to one written in 1945:

  `db.orwell.find({
      "title": "Animal Farm",
      "year": 1945
      })`

Notice that numbers are represented without quotes, just like in JSON. This
of course applies to booleans as well. So, if we wanted to find all books
written by Orwell that we already read, we could do:

  `db.orwell.find({"read": true})`

We could also pass a second argument to the find method, called a
projection. A projection is quite simply the fields we're interested in
returning.

Again with Animal Farm, if we only want to return the year, it would look
like this:

  `db.orwell.find({
    "title": "Animal Farm"
    }, {
    "year": 1
  })`

Note that that any number other than 0, the boolean true, and any string
tells Mongo to return that field. 0, null, and false tell Mongo to exclude a
field and return the others. The id field must be explicitly excluded this
way, otherwise it will always be returned.


Create:

  `use db // creates database with name db if it doesn't already exist`

  `db.collection.insertOne() // Inserts a single document into a collection;
  // creates the collection if it doesn't exist`

  You may insert a document into a collection with any parameters
  whatsoever. Unlike in SQL, NoSQL platforms such as MongoDB don't require
  that you follow any pre-established schema or format. There will be no
  empty columns, or columns with default values, unless explicitly specified.

  So if we were to enter:

  `db.orwell.insertOne({
    "title": "Homage to Catalonia",
    "year": 1938,
    "year published in US": 1952,
    "read": false
  })`

Followed by:

  `db.orwell.insertOne({
    "title": "1984",
    "year": 1949,
    "read": true
  })`

The latter document would not have an empty "year published in US"
property like it would if we were using a predefined schema with an SQL
platform.

The only exception to this rule is with the _id field. By default, this
field will be a unique hexadecimal string. You may specify _id manually, but
the insertion will throw an error if the string you enter is not unique.

`db.collection.insertMany() // Inserts several documents into a collection`

  This looks like an array of JSON objects, like this:

  `db.orwell.insertMany([{
    "title": "Burmese Days",
    "year": 1934
    }, {
    "title": "The Road to Wigan Pier",
    "year": 1937
  }])`

The rules for inserting a single document apply the same way here.

Update:

`db.collection.updateOne() // update a single document`

  This method can take up to three arguments, but requires only one. They
  are filter, update, and options. The update argument is the only one
  that's required. There must be a filter argument, but using an empty
  object {} updates the first document returned.

  Recall that in a previous example, we marked Homage to Catalonia as not
  read. If we read that book and want our database to reflect that, this is
  what the command looks like:

  `db.orwell.updateOne({
    "title": "Homage to Catalonia"
    }, {
    $set: {
    "read": true
    }
  })`

Notice the property that begins with $. This is the update operator, which
tells Mongo which type of update to perform. The update operators are:

`$currentDate // Sets the value of a field to the current date. Useful
// for manually updating Date or Timestamp fields.`

`$inc // Increments the value of a numeric field be the amount you
// specify.`

`$min // Only updates a numeric field if the value you specify is less than the
// existing value of that field.`

`$max // Only updates a numeric field if the value you specify is greater than
// the existing value of that field.`

`$mul // Multiplies a numeric field by the amount you specify`

`$rename // Renames a field; does not change its value.`

`$set // Sets the value of a field in a document. Probably the most
// commonly used update operator.`

`$setOnInsert // Sets the value of a field if a new document is inserted.`

`$unset // Removes a field from a document.`

There are also update operators for use on arrays. They are:

`$ // Updates the first element matching a query condition.`

`$[] // Updates all elements that match the query condition.`

`$[<identifier>] // Updates all elements that match the arrayFilers
// condition in the documents matching the query condition.`

`$addToSet // Adds elements to an array if they do not exist in that set.`

`$pop // Removes the first or last item of an array.`

`$pull // Removes all array elements matching the specified query.`

`$pullAll // Removes all matching values from an array`

`$push // Add an item to an array regardless of whether it exists.`

These array operators can be modified with the following:

`$each // Makes $push or $addToSet append multiple items`

`$position // Specifies an array index to add elements when using $push`

`$slice // Limits size of updated arrays when using $push`

`$sort // $push reorders documents stored in an array`

Finally, there is the bitwise operator for performing AND, OR, XOR updates
of integer values. To be clear, in any kind of programming or mathematics,
   AND means both values are true, OR means one or the other value is true,
   and XOR means one value is true and the other is false.

   With all that said, there are still the optional parameters left to discuss.
   The first is upsert. By default, updateOne will not update a document that
   doesn't exist. However, if we set {upsert: true} after the update operation,
   a new document is created.

   Say we want to update Down and Out in Paris and London to reflect that we
   read it. If you've been following along, this book won't be in our
   collection, so just calling:

   `db.orwell.updateOne({
    "title": "Down and Out in Paris and London"
    }, {
    $set: {
      "read": true
      }
    })`

Won't do anything, because this book isn't in our collection. While we could
just as easily add it with insertOne(), we could also:

`db.orwell.updateOne({
    "title": "Down and Out in Paris and London"
  }, {
    $set: {
     "read": true
    }
  }, {
  upsert: true
})`

To accomplish the same thing. Of course, setting upsert to true does not
prevent us from performing an update on an existing document, in which case
it updates like normal.

Another optional argument, is Write Concern. Write concern is important for
administration of a MongoDB server set up, and is beyond the scope of this
tutorial.

We also have the option to include a Collation argument. This deals with how
the actual language is handled, and if used, we must at least specify a
locale. Any further explanation is beyond the scope of this tutorial.


Finally, new to version 3.6, is the ability to modify array elements based
on criteria we define ourselves. Let's say we want to write a paper based on
our extensive George Orwell reading. Let's also say that we want to keep
track of how many paragraphs per page we discuss a subject from Homage to
Catalonia. We could first update like this:

`db.orwell.updateOne({
    "title": "Homage to Catalonia"
  }, {
  $set: {
    "paragraphs": [0, 2, 3, 1, 0]
  }
})`

According to this update, we wrote 0 paragraphs about Homage to Catalonia
on page 1, 2 on page 2, 3 on page 3, 1 on page 4, and 0 on page 5.
Next, lets say that we decided that 3 paragraphs on page 3 wasn't enough,
  that we ended up writing 2 more paragraphs. We could update it like so,
  using an array filter:

  `db.orwell.updateOne({
    "title": "Homage to Catalonia"
    }, {
      $set: {
       "paragraphs.$[element]": 5
      }
    }, {
      arrayFilters : [{
        "element": {
          $eq: 3
        }
      }]
  })`

There's kind of a lot going on here, so let's look at it in more detail:

`db.orwell.updateOne({
  "title": "Homage to Catalonia"    // Filter, no different from before
  }, {
    $set: {
      "paragraphs.$[element]": 5    // $[element] is a placeholder. It
      // behaves similarly to a variable,
      // and we can name it whatever we want.
    }
  }, {
    arrayFilters : [{
      "element": { // This is where we define our placeholder.
        $eq: 3     // This particular operator looks for an index with a
        // value of 3. It is not the only $ operator we could use, but it
        // works here. If there was more than one index with a
        // value of 3, only the first with that value would be modified.
      }
    }]
})`

`db.collection.updateMany() // update several documents`

  This of course works similarly to updateOne(), but with some key
  differences. Instead of having to specify each element to update in an
  array, we can update each element that matches the filter. Recall of
  course that updateOne() only updates the first element matching that
  filter.

  Let's say we want to include more books in our paper. First, we can
  include some paragraphs about 1984 and Down and Out in Paris and London.
  First we update those two books with a paragraph list, one at a time:

  `db.orwell.updateOne({
    "title": "1984"
    }, {
    $set: {
      paragraphs: [1, 3, 6, 3, 0]
    }
  })`

`db.orwell.updateOne({
  "title": "Down and Out in Paris and London"
  }, {
    $set: {
      paragraphs: [0, 5, 2, 2, 1]
    }
})`

Next, let's say we decide that the paper is too long; we want to cut
paragraphs across the board. We will take any books with 5 or more
paragraphs on a single page and cut them down to 4 paragraphs. This is how
we do that:

`db.orwell.updateMany({
  paragraphs: {
    $gte: 5   // $gte operates on everything greater than or equal to
              // number specified, which is 5 in this case
  }
}, {
  $set: {
    "paragraphs.$[element]": 4
  }
}, {
  arrayFilters: [{
    "element": {
      $gte: 5
    }
  }]
})`

Notice that this looks a lot like the previous command to update a single
document. The difference is that instead of just updating the first array
element returned, it updates all array elements that match the filter. If
we wanted to do an updateMany on non-array integers, we would leave out
the arrayFilters argument.

`db.collection.replaceOne() // removes and replaces a single document;
  // without parameters, this is the first document returned
  This works exactly the same way as updateOne(), except for the fact that
  the document returned from the filter parameter is deleted, and only those
  fields defined for the new document exist. This is useful if you want to
  update a document but also remove unnecessary fields at the same time.`

  Delete:

`db.collection.deleteOne() // deletes a single document; without parameters,
  this is the first document returned`

  This method is very simple. The only argument it requires is a filter. So
  let's say we want to delete Animal Farm altogether. We do:

  `db.orwell.deleteOne({
      "title": "Animal Farm"
  })`

That's it. Of course, we can specify a write concern and collation if we
want, but that's still beyond the scope of this tutorial. Also, if there
were two books titled Animal Farm, because this is the deleteOne() method,
only the first one returned is actually deleted. If we wanted to delete
every possible Animal Farm document, we'd do deleteMany().

`db.collection.deleteMany() // deletes several documents`

  This looks exactly like the previous method. Let's say we decided to
  catalogue multiple print editions of Animal Farm, so maybe we have a few
  documents like this:

  `[{
    "title": "Animal Farm",
    "edition": "first"
  }, {
    "title": "Animal Farm",
    "edition": "US reissue"
  }, {
    "title": "Animal Farm",
    "edition": "Japanese translation"
  }]`

If we just did deleteOne(), only the first edition document would be
removed. deleteMany() gets rid of all of them.


Finally, if we want to drop the entire orwell collection, not just specific
documents, we'd do:

`db.orwell.drop()`

  This was just a cursory examination of MongoDB's CRUD operations. There is of
  course a great deal more, but this tutorial should be enough to get you
  started with it.

  Once you're used to these basics, the documentation can explain other, more
  specific functions you may or may not need. As of January, 10th, 2018, that
  documentation can be found here: https://docs.mongodb.com/manual/reference/method/js-collection/
