# Object Relational Mapping

## Objectives

1. Explain the concept of an ORM and why we build them.
2. Describe the code that will map your Ruby objects to a database. 

## What is ORM?

Object Relational Mapping (ORM) is the technique of accessing a relational database using an object-oriented programming language. Object Relational Mapping is a way for our Ruby programs to manage database data by "mapping" database tables to classes and instances of classes to rows in those tables.

There is no special programming magic to an ORM––it is simply a manner in which we implement the code that connects our Ruby program to our database. For example, you may have seen code that connects your Ruby program to a given database:

```ruby
database_connection = SQLite3::Database.new('db/my_database.db')

database_connection.execute("Some SQL statement")
```

An ORM is really just a concept. It is a design pattern, a conventional way for us to organize our programs when we want those programs to connect to a database. The convention is this:

**When "mapping" our program to a database, we equate classes with database tables and instances of those classes with table rows.**

You may also see this referred to as "wrapping" a database, because we are writing Ruby code that "wraps" or handles SQL. 

## Why Use ORM?

There are a number of reasons why we use the ORM pattern. Two good ones are:

* Cutting down on repetitious code. 
* Implementing conventional patterns that are organized and sensical. 

### Cutting Down on Repetition

Let's take a look at some of the common code we might use to interact our Ruby program with our database.

Let's say we have a program that helps a veterinary office keep track of the pets it treats and those pets' owners. Such a program would have an `Owners` class and `Cats` class (among classes to represent other pets). Our program surely needs to connect to a database so that the veterinary office can persist information about its pets and owners. 

Our program would create a connection to the database:

```ruby
database_connection = SQLite3::Database.new('db/pets.db')
```

We would create an owners table and a cats table:

```ruby
database_connection.execute("CREATE TABLE IF NOT EXISTS cats(id INTEGER PRIMARY KEY, name TEXT, breed TEXT, age INTEGER)")

database_connection.execute("CREATE TABLE IF NOT EXISTS owners(id INTEGER PRIMARY KEY, name TEXT)")
```

Then, we would need to regularly insert new cats and owners into these tables:

```ruby
database_connection.execute("INSERT INTO cats (name, breed, age) VALUES ('Maru', 'scottish fold', 3)")

database_connection.execute("INSERT INTO cats (name, breed, age) VALUES ('Hana', 'tortoiseshell', 1)")
```

Notice that in the lines of code above, there is a lot of repetition. In fact, the only difference between the two lines in which we insert data into the database are the actual values. 

The repetition would also occur for other SQL statements we might want to execute against our database. Any `SELECT` queries, for example, would repeat the call to the `database_connection.execute` method and differ only in the specifics of what data we are selecting from which table. 

As programers, you might remember, we are lazy. We don't like to repeat ourselves if we can avoid it. Repetition qualifies as a "code smell". Instead of repeating the same, or similar, code any time we want to perform common actions against our database, we can write a series of methods to abstract that behavior. 

For example, we can write a `#save` method on our `Cats` class that handles the common action of `INSERT`ing data into the database. 

```ruby
class Cat

  @@all = []
  
  def initialize(name, breed, age)
    @name = name
    @breed = breed
    @age = age
    @@all << self
  end
  
  def self.all
    @@all
  end

  def self.save(name, breed, age, database_connection)
    db.execute("INSERT INTO cats (name, breed, age) VALUES (?, ?, ?)", name, breed, age)
  end
end
```

Now let's create some new cats and save them to the database:

```ruby
database_connection = SQLite3::Database.new('db/pets.db')

Cat.new("Maru", "scottish fold", 3)
Cat.new("Hana", "tortoiseshell", 1)

Cat.all.each do |cat|
  Cat.save(cat.name, cat.breed, cat.age, database_connection)
end
```

Here we establish the connection to our database, create two new cats and then iterate over our collection of cat instances stored in the `Cat#all` method. Inside this iteration, we use the `Cat#save` method, giving it arguments of the data specific to each cat to `INSERT` those cat records into the cats table. 

Now, thanks to our `Cats#save` method, we have some re-usable code––code that we can easily use again and again to "save" or `INSERT`, cat records into the database. 

This is just one example of the types of methods we will learn to build as we create our ORM. *Don't worry too much about the code shown above.* We'll learn more about how and why we define our ORM methods later on. This is just a preview of the kind of method we will write to tell our classes how to talk to our database.

### Logical Design 

Another important reason to implement the ORM pattern is that it just makes sense. Telling our Ruby program to communicate with our database is confusing enough without each individual developer having to make their own, individual decision about *how* our program should talk to our database. 

Instead, we follow the convention: classes are mapped to or equated with tables and instances of a class are equated to table rows. 

If we have a `Cat` class, we have a cats table. Cat instances get stored as rows in the cats table. 

Further, we don't have to make our own, potentially confusing or non-sensical decision about what kinds of methods we will build to help our classes communicate with our database. Just like the `#save` method we previewed above, we will learn to build a series of common, conventional methods that our programs can rely on again and again to communicate with our database. 

<a href='https://learn.co/lessons/ruby-orm' data-visibility='hidden'>View this lesson on Learn.co</a>
