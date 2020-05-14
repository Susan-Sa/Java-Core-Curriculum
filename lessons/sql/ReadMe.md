# Intro to Databases, SQLite, and SQLiteOpenHelper

## Objectives

- Engineers will become familiar with what a relational database is and when to use one
- Engineers will see how common clauses in SQL are composed and executed like (CREATE, INSERT, SELECT, UPDATE, DELETE)
- Engineers will gain exposure to SQLite examples and JDBC, two abstractions to interact with SQL.


## Resources

- [What is SQL?](https://www.youtube.com/watch?v=27axs9dO7AE)
- [Glossary of commonly used SQL commands](https://www.codecademy.com/articles/sql-commands)
- [SQLite Online Emulator](http://kripken.github.io/sql.js/GUI/)
- [Performing Database Operations in Java](https://www.geeksforgeeks.org/performing-database-operations-java-sql-create-insert-update-delete-select/)
- [What is JDBC](http://tutorials.jenkov.com/jdbc/index.html)

## Vocabulary
|Term|Definition|
|:-:|:-|
|**Relational Database**|a database that organizes information into one or more *tables*|
|**Table**|a collection of data organized into *rows* and *columns*. Tables are sometimes referred to as *relations*
|**Column**|a named category for a set of data values of a particular type; in SQL they are referred to as *fields*
|**Row**|a single entry in a table; in SQL they are referred to as *records*
|**Schema**|a collection of database objects (tables) associated with one particular database username. This username is called the *schema owner*. You may have one or multiple schemas in a database|
|**SQL statement**|text that the database recognizes as a valid command|
|**clause**|the part of a SQL statement that performs a specific task. By convention, clauses are written in capital letters. Clauses are sometimes referred to as *commands*|
|**parameter**|a list of columns, data types, or values that are passed to a *clause* as an argument|
|**query**|a statement used to extract data from the database in a readable format according to the user's request|

# Lecture

## What is a database?

A database is simply a collection of information, that is organized so that it can be easily accessed, managed and updated by a user. 

The most effective databasse can be thought of as a collection of **associated** information, beyond a single key/value pair. An example of this can be visualized as a table (like a spreadsheet), with **columns** for value categories, and **rows** for individual entries. Let's say we wanted to track the users of a blogging site. We can put them in something like the table below:

|entry_order|username|password|
|:-:|:-|:-|
|1|dogperson123|ComplicatedPassword123!|
|2|catperson456|ComplicatedPassword456!|
|3|hamsterperson789|ComplicatedPassword789!|

To extend this metaphor, you might be asking "Which one is the key?" Well, let's say we wanted to find out the password for a certain user - dogperson123 - visually, we could look at the table, under the column `username`, go down until we see the user we wanted (dogperson123), then look to the right or left of that row, until we hit the column `password`, where we'd find the password associated with that particular user. In this sense, technically speaking, any value could be a key. This is both a good and a bad thing. Good, because it means you can search for associated data using multiple keys, but bad, since it means there's no real way to tell one entry from the other, especially if a username is identical:

|entry_order|username|password|
|:-:|:-|:-|
|1|dogperson123|ComplicatedPassword123!|
|2|catperson456|ComplicatedPassword456!|
|3|hamsterperson789|ComplicatedPassword789!|
|4|catperson456|SomeOtherPassword!|

We need to make one key unique - so that if we do find duplicate data, we can at least tell the difference between the two. If we make sure that the entry order number can **never be changed**, then at least we'd know for sure that we have mutiple entries for the same user - and we can either remove the duplicate entry, or change the username for them entirely. We can look at the `entry_order` as the **primary key** to associate with all the other data in our rows, making each row a unique entry.

If you wanted to, you could even create a table tracking the multiple blog posts each username might author - you could create a "relationship" between these two tables, so that whenever a user makes a post, you create a relationship between the user data, and the blog post data.

On a very high level, this is essentially how Relational SQL-based databases work.

## What is a Relational Database?

A **Relational Database** is a database that organizes information into one or more tables. MySQL and SQLite are examples of relational database management systems used for mobile and web projects.

All data stored in a relational database is of a certain **data type**. SQLite has the following data types:

|Type|Definition|
|:-:|:-|
|**INTEGER**|a positive or negative whole number|
|**TEXT**|a text string, stored using the database encoding (UTF-8, UTF-16BE or UTF-16LE). Wrap text strings in single quotes, e.g. **'Hello, world!'**|
|**REAL**|a decimal value, stored as an 8-byte IEEE floating point number|
|**BLOB**|a blob of data, stored exactly as it was input|
|**NULL**|the null value|

**IMPORTANT:** 

SQLite does not have a separate BOOLEAN data type class. Instead, BOOLEAN values are stored as INTEGERS `0` (false) and `1` (true).

SQLite also does not have a type set aside for storing dates and/or times. Instead, dates and times are stored as TEXT, REAL, or INTEGER values:

- **TEXT** as [ISO8601 strings](https://en.wikipedia.org/wiki/ISO_8601) ("YYYY-MM-DD HH:MM:SS.SSS")
- **REAL** as [Julian day](https://en.wikipedia.org/wiki/Julian_day) numbers, the number of days since noon in Greenwich on November 24, 4714 B.C. according to the [proleptic Gregorian calendar](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar)
- **INTEGER** as Unix Time, the number of seconds since 1970-01-01 00:00:00 UTC

A user can insert the current data and current time by using function like `DATE()` and `TIME()` - this is especially useful for timestamping your data. 

## Why use Relational Databases in Android Applications?

Relational databases are ideal for repeating or structured data (e.g. contacts, messages, users, tweets). You might want to use a Relational Database if:

- The data you wish to store is of a type, complexity or quantity that cannot be stored easily in SharedPreferences (e.g. a list)
- You want to cache data from a REST API locally for faster access or offline viewing - for example: received emails in your Gmail app are always available offline. This can be great for user experience, but it is important to ensure that cached data does not become stale

## What is SQL?

SQL (Structured Query Language) is a programming language designed to manipulate and manage data stored in relational databases.

|Term|Definition|
|:-:|:-|
|**SQL statement**|text that the database recognizes as a valid command|
|**clause**|the part of a SQL statement that performs a specific task. By convention, clauses are written in capital letters. Clauses are sometimes referred to as *commands*|
|**parameter**|a list of columns, data types, or values that are passed to a *clause* as an argument|
|**query**|a statement used to extract data from the database in a readable format according to the user's request|
|**field**|comparable to a **column**, a named category for a set of data values of a particular type|
|**record**|comparable to a **row**, or a single entry in a table|

The structure of SQL statements vary. The number of lines used doesn't matter - a statement can be written in a single line or split up across multiple lines for readability. However, statements always end in a semi-colon `;` - much like in other languages.

These statements constitute what are referred to as [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations, or the four basic functions of persistent storage:

* Create
* Read
* Update
* Delete

### `DROP TABLE` statements

The `DROP TABLE` statement is used to delete an existing table in a database:

```SQL
DROP TABLE table_name;
```

**IMPORTANT**: ***Be careful before dropping a table*** - deleting a table will result in loss of complete information stored in the table!

### `CREATE TABLE` statements

Create a new table in the database. Allows you to specify the name of the table and the name and data type of each field (column) in the table:

```SQL
CREATE TABLE table_name (column_1 datatype, column_2 datatype, column_3 datatype);
```

### `ALTER TABLE` statements

Add fields (columns) to an existing table in the database:

```SQL
ALTER TABLE table_name ADD column datatype;
```

### `INSERT INTO` statements

Add a new record (row of data) to a table:

```SQL
INSERT INTO table_name (column_1, column_2, column_3) VALUES (value_1, value_2, value_3);
```

### `SELECT` statements

When you compose `SELECT` statments, you are making **queries** - or accessing the database to get data specific to certain search requirements.

The character `*` can be used as a wildcard meaning *all*. This will return all of the records in a database table, including all of its fields/ field values:

```SQL
SELECT * FROM table_name;
```

You can fetch partial data just for a certain field from the database:

```SQL
SELECT column_name FROM table_name;
```

You can fetch partial data for a number of fields from the database:

```SQL
SELECT column_name01, column_name02 FROM table_name;
```

You can fetch all records that have fields containing certain values from the database:

```SQL
SELECT * FROM table_name WHERE column_name = some_value;
```

You can fetch all records, and order them based on the values of specific fields from the database:

```SQL
SELECT * FROM table_name ORDER BY column_name;
```

You can fetch all records, and order them based on the values of specific fields from the database, in reverse order:

```SQL
SELECT * FROM table_name ORDER BY column_name DESC;
```

You can fetch all records that have fields containing certain values of a certain length from the database:

```SQL
SELECT * FROM table_name WHERE LENGTH(column_name) = 4;
```

Honestly, this is simply scratching the surface as to the many ways a user can query a database table. People have spent entire careers as Database Administrators designing/optomizing useful queries for data analytics on massive tables/databases. If you'd like to know more, you can follow [this link](https://www.sqlite.org/lang_select.html) for more information.

### `UPDATE` statements

Edit records (rows) in a table:

```SQL
UPDATE table_name SET some_column = some_value WHERE some_column = some_value;
```

### `DELETE FROM` statements

Remove records (rows) from a table:

```SQL
DELETE FROM table_name WHERE some_column = some_value;
```

### Adding a `PRIMARY KEY`

Remember in the example we described at the beginning of the lecture - where you could have multiple entries with identical values, adding to inconsistent data? There is a way to add a `PRIMARY KEY` you your table, listing the order in which each record was placed into the table: 

```SQL
CREATE TABLE students(_id INTEGER PRIMARY KEY, last_name TEXT, first_name TEXT);
```

Now, you can add that `PRIMARY KEY` every time you insert a new record:

```SQL
INSERT INTO students(_id, last_name, first_name) VALUES(1, "Lin", "Lily");
```

However, SQL will make sure that every primary key is unique, so if you add the same primary key when you insert a new record. One way around this is to enter the keyword `NULL` when entering the `PRIMARY KEY` value:

```SQL
INSERT INTO students(_id, last_name, first_name) VALUES(NULL, "Smith", "Jordan");
```

However, it still requires you to enter a value. You can still add this `PRIMARY KEY` field, and have the number for each record increase automatically every time a new record is added, without having to use `NULL` every single time:

```SQL
CREATE TABLE students(_id INTEGER PRIMARY KEY AUTOINCREMENT, last_name TEXT, first_name TEXT);

INSERT INTO students(last_name, first_name) VALUES("Smith", "Jordan");
```

You might notice though that after deleting records, the `PRIMARY KEY` field may appear to be missing record numbers. That's because each primary key is unique, and should never be used again. This is a normal part of table operations.

## What is SQLite?

**SQLite** is a **relational database management system (RDBMS)** - a program that lets you create, update, and administer a relational database.

SQLite is a popular open source, compact RDBMS that uses SQL to access the database. It is able to store an entire database in a single file. One of the biggest advantages this provides is that all of the data can be stored locally without having to connect your database to a server.

SQLite is a popular choice for databases in phones (it is the default embedded database in both iOS and Android) and other electronic gadgets.

## How can we get better at SQLite?

![Try breaking stuff until you stop breaking stuff](https://i.pinimg.com/originals/f0/ec/70/f0ec70b24a69ea7b57ade4d639f95619.jpg)

One great way to experiment with SQLite is to mess around with a **sandbox**, or a safe way to mess around with a new language, without actually setting your house on fire.

A great resource is this [online emulator](http://kripken.github.io/sql.js/GUI/)!

## How do engineers use management systems like SQLite to create, update, and administer a relational database in modern Applications?

There are a number of ways to create a SQL database, add records to it, and retreive data from it. For the Android Platform There are libraries like Cupboard and Room, and and for more web and server side interactions JDBC is a popular tool which helps make your lives easier. Throughout the rest of this lesson we will go through code examples of SQL "helpers" and gain exposure to how SQL language is roped in with the Java language. <b>You do not have to code along with the code examples</b> following along and examing the code closely should be sufficient.

**SQLiteOpenHelper**.

### What is SQLiteOpenHelper?

**SQLiteOpenHelper** is an Abstract Class which must be subclassed. The abstract class has a number of abstract methods which need to be overridden. This class will allow you to create a database, create a number of tables, run queries, and ultimately run SQLite code directly in your apps, but allow you to create your own methods to expose to the rest of your app, so you won't have to constantly write SQLite code every time you want to run a simple operation.


Let's look at a DBC example that loads the JDBC driver, opens a database connection, creates a Statement, executes a query, and iterates the returned ResultSet:

```java
import java.sql.*;

public class JdbcExample {

    public static void main(String[] args) throws ClassNotFoundException {
        Class.forName("org.h2.Driver");

        String url      = "jdbc:h2:~/test";   //database specific url.
        String user     = "sa";
        String password = "";

        try(Connection connection = DriverManager.getConnection(url, user, password)) {
            try(Statement statement = connection.createStatement()){
                String sql = "select * from people";
                try(ResultSet result = statement.executeQuery(sql)){
                    while(result.next()) {
                        String name = result.getString("name");
                        long   age  = result.getLong  ("age");
                    }
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }

    }
}
```
Now lets looks at an example in Android using SQLite.

Let's say we want to create an app to list Dream Corps Graduates, and the companies they now work for.

First, we'll need to create a data model class that we'll want to add to the database:

```java
package com.bbah.sqliteexample.model;

public class Engineer {

    private String firstName;
    private String lastName;
    private String company;

    public Engineer(String firstName, String lastName, String company) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.company = company;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getCompany() {
        return company;
    }

    public void setCompany(String company) {
        this.company = company;
    }
}
```

Next, we'll create a class that inherits from the abstract class SQLiteOpenHelper, which overrides two methods - `onCreate(SQLiteDatabase sqLiteDatabase)`, and `onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1)`:

```java
package com.bbah.sqliteexample.database;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class EngineerDatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "engineers.db";
    private static final String TABLE_NAME = "engineers";
    private static final int SCHEMA_VERSION = 1;

    public EngineerDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, SCHEMA_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        sqLiteDatabase.execSQL(
                "CREATE TABLE " + TABLE_NAME +
                " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "last_name TEXT, first_name TEXT, company TEXT);");
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        /*  Important for when you update an entire database with a new version.
        *   We won't be exploring that in this lecture's example.
        */
    }
}
```

In our constructor, we call the super's (SQLiteOpenHelper) constructor, with certain arguments:

* The application's context (getApplicationContext())
* the database name (DATABASE_NAME, or "engineers.db")
* we pass "null" for a factory we don't need to supply, and
* the version of our database (SCHEMA_VERSION, or 1)

The `onCreate()` method is called automatically whenever the methods that actually GET a table are called - `getReadableDatabase()` or `getWritableDatabase()`. Inside the `onCreate()` you can we that we are simply calling raw SQL commands to create a new database table, based on the database created when we first called the constructor, IF the table doesn't already exist.

Essentially, the SQLite code we are executing is:

```SQL
CREATE TABLE students(_id INTEGER PRIMARY KEY AUTOINCREMENT, last_name TEXT, first_name TEXT, company TEXT);
```

Next, we can make methods that allow us to add students, and get a list of the students we already have in the database:

```java
package nyc.c4q.sqliteexample.database;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;
import java.util.List;

import com.bbah.sqliteexample.model.Engineers;

public class EngineersDatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "engineers.db";
    private static final String TABLE_NAME = "engineers";
    private static final int SCHEMA_VERSION = 1;

    public EngineersDatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, SCHEMA_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        sqLiteDatabase.execSQL(
                "CREATE TABLE " + TABLE_NAME +
                " (_id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                "last_name TEXT, first_name TEXT, company TEXT);");
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
        /*  Important for when you update an entire database with a new version.
        *   We won't be exploring that in this lecture's example.
        */
    }

    public void addEngineer(Engineer engineer) {
        Cursor cursor = getReadableDatabase().rawQuery(
                "SELECT * FROM " + TABLE_NAME + " WHERE first_name = '" + engineer.getFirstName() +
                "' AND last_name = '" + engineer.getLastName() + "' AND company = '" + engineer.getCompany() +
                        "';", null);
        if (cursor.getCount() == 0) {
            getWritableDatabase().execSQL("INSERT INTO " + TABLE_NAME +
                    "(last_name, first_name, company) VALUES('" +
                    engineer.getLastName() + "', '" +
                    engineer.getFirstName() + "', '" +
                    engineer.getCompany() + "');");
        }
        cursor.close();
    }

    public List<Engineer> getEngineerList() {
        List<engineer> engineer = new ArrayList<>();
        Cursor cursor = getReadableDatabase().rawQuery(
                "SELECT * FROM " + TABLE_NAME + ";", null);
        if(cursor != null) {
            if(cursor.moveToFirst()) {
                do {
                    Engineer engineer = new Engineer(
                            cursor.getString(cursor.getColumnIndex("last_name")),
                            cursor.getString(cursor.getColumnIndex("first_name")),
                            cursor.getString(cursor.getColumnIndex("company")));
                    engineer.add(engineer);
                } while (cursor.moveToNext());
            }
        }
        return engineerList;
    }
}
```

The `addEngineer()` method allows us to add a engineer, if it does not already exist in our database. First, we check to see if a record already exists with those values, by querying those values first.

We create a `Cursor` object, to store all of the values at any given row. Think of a `Cursor` as just that - a cursor that rests on a certain record in the database.

Once again, we are running raw SQL commands to query for records that match certain criteria. This code:

```java
Cursor cursor = getReadableDatabase().rawQuery(
                "SELECT * FROM " + TABLE_NAME + " WHERE first_name = '" + engineer.getFirstName() +
                "' AND last_name = '" + engineer.getLastName() + "' AND company = '" + engineer.getCompany() +
                        "';", null);
```

effectively runs the SQL code:

```SQL
SELECT * FROM engineer WHERE first_name = 'a engineers first name' AND last_name = 'a engineers last name' AND company = 'the company they work for;
```

Next, we confirm that the record count that returns from that query is zero (0). If it is, add the record - if not, don't. This way, we can avoid adding duplicates:

```java
if (cursor.getCount() == 0) {
            getWritableDatabase().execSQL("INSERT INTO " + TABLE_NAME +
                    "(last_name, first_name, company) VALUES('" +
                    engineer.getLastName() + "', '" +
                    engineer.getFirstName() + "', '" +
                    engineer.getCompany() + "');");
        }
```

As you can see, we run queries by calling the `getReadableDatabase()`, but insert records by calling `getWritableDatabase()`.

Finally, we should create a method that returns a list of engineers currently in the database table:

```java
public List<Engineer> getEngineersList() {
        List<Engineer> engineerList = new ArrayList<>();
        Cursor cursor = getReadableDatabase().rawQuery(
                "SELECT * FROM " + TABLE_NAME + ";", null);
        if(cursor != null) {
            if(cursor.moveToFirst()) {
                do {
                    Engineer engineer = new Engineer(
                            cursor.getString(cursor.getColumnIndex("last_name")),
                            cursor.getString(cursor.getColumnIndex("first_name")),
                            cursor.getString(cursor.getColumnIndex("company")));
                    engineerList.add(engineer);
                } while (cursor.moveToNext());
            }
        }
        return engineerList;
    }
```

After first getting a list of the records currently in the table, we want to grab the values from each of the records, create a new students object with these values, and add them to our list, so we can return it.

The methods `cursor.moveToFirst()` and `cursor.moveToNext()` are kind of magical, in that they both move the `Cursor` to a new position, but also return a boolean value as to whether they were successful at doing it or not. So what we're doing in the `getEngineerList()` method is essentially:

* Getting all the records
* checking to see if we have any records
* if we do, we check to see if the cursor is on the first record
* if it is, we create a new Engineer object based on the field values of that record
* and we keep moving to a new record and creating Engineer objects based on each new record, as long as there are records to go to



And in Logcat, we can see the results:

```
01-13 22:37:56.420 25565-25565/? I/art: Not late-enabling -Xcheck:jni (already on)
01-13 22:37:56.466 25565-25565/com.bbah.sqliteexample W/art: Before Android 4.1, method android.graphics.PorterDuffColorFilter android.support.graphics.drawable.VectorDrawableCompat.updateTintFilter(android.graphics.PorterDuffColorFilter, android.content.res.ColorStateList, android.graphics.PorterDuff$Mode) would have incorrectly overridden the package-private method in android.graphics.drawable.Drawable
01-13 22:37:56.550 25565-25565/com.bbah.sqliteexample D/Engineers?: Lily Lin - Spotify
01-13 22:37:56.550 25565-25565/com.bbah.sqliteexample D/Engineers?: Jordan Smith - LinkedIn
01-13 22:37:56.550 25565-25565/com.bbah.sqliteexample D/Engineers?: Rusi Li - Weight Watchers
01-13 22:37:56.550 25565-25565/com.bbah.sqliteexample D/Engineers?: Derek Santos - Uber
01-13 22:37:56.550 25565-25565/com.bbah.sqliteexample D/Engineers?: Danny Lui - Max2
01-13 22:37:56.555 25565-25581/com.bbah.sqliteexample D/OpenGLRenderer: Use EGL_SWAP_BEHAVIOR_PRESERVED: true
```

Alright! It prints the list! And, there are no dupicates in the list as well!


