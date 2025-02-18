<img src="res/logo.png" style="display:block; margin-left:auto; margin-right:auto; width:500px; " />

[![CirlceCI](https://img.shields.io/circleci/build/github/Jomy10/simple_tables)](https://app.circleci.com/pipelines/github/Jomy10/simple_tables?branch=master)
![Language](https://img.shields.io/badge/lang-Rust-B7410E)
[![Licenses](https://img.shields.io/crates/l/simple_tables)](#license)
[![Crates.io](https://img.shields.io/crates/v/simple_tables)](https://crates.io/crates/simple_tables)
[![Docs.rs](https://img.shields.io/docsrs/simple_tables)](https://docs.rs/simple_tables/latest/simple_tables/)

Simple Tables is a rust crate for easily creating table structures. This is made easy using macros.

This crate is actively maintained and everything is well documented.

## Table of Contents

- [Overview](#overview)
  - [Creating tables](#creating-tables)
  - [Functions](#functions)
    - [ToString](#tostring)
    - [Creating a new table instance](#creating-a-new-table-instance)
    - [Get rows](#get-rows)
    - [Get columns](#get-columns)
    - [Inserting rows](#inserting-rows)
    - [Removing rows](#removing-rows)
    - [Column and row count](#column-and-row-count)
  - [Tables with UID's](#tables-with-uids)
    - [Getting a row based on the uid](#getting-a-row-based-on-the-uid)
- [Adding derive attributes](#adding-derive-attributes)
- [Installing](#installing)
- [Contributing](#contributing)
- [Documentation](#documentation)
- [Use cases](#use-cases)
- [Questions](#questions)
- [License](#license)

## Overview
### Creating tables
To create a table, you use the macros `table_row` to indicate the structure of a row and `table` to use a struct as a 
table for that row.

**Example**
```rust
use simple_tables::macros::{table_row, table};

#[table_row]
struct MyTableRow {
  id: u32,
  name: String,
  email: String,
  address: String
}

#[table(rows = MyTableRow)]
struct MyTable {}
```

These macros will implement the `TableRow` and `Table` trait respectively. You could also implement these manually.

**NOTE**: If you use **IntelliJ**, I highly encourage you to enable the `org.rust.cargo.evaluate.build.scripts` and `org.rust.macros.proc`
experimental features. You can accomplish this by pressing `⇧⌘A` (macOs) or `⌃⇧A` (Linux/Windows) and searching
for `Experimental Features`, then enabling the two before-mentioned features.

I don't know if rust-analyzer has this feature enabled by default, so I don't know what the state is in other IDE's.

### Functions
The traits `TableRow` and `Table` define a collection a functions, most of them with default implementations. Using the
`table_row` and `table` macros will implement these traits and their respective functions for the struct you are targetting.
The macros also provide some additional functions.

#### ToString
The macros also implement the `ToString` trait for tables, so you can use the `to_string` function.

**Example**
```rust
use simple_tables::Table;

let rows: Vec<MyTableRow> = vec![
  MyTableRow{ id: 0, name: "David Bowie".to_string(), email: "david@bowie.com".to_string(), address: "England".to_string()},
  MyTableRow{ id: 1, name: "David Gilmour".to_string(), email: "david@gilmour.com".to_string(), address: "England".to_string()},
  MyTableRow{ id: 2, name: "Opeth".to_string(), email: "info@opeth.com".to_string(), address: "Sweden".to_string()},
  MyTableRow{ id: 3, name: "The Beatles".to_string(), email: "info@beatles.com".to_string(), address: "England".to_string()}
];

let table = MyTable::from_vec(&rows);
let s = table.to_string();
println!("{}", s);
```

The output will be:
```
+----+---------------+-------------------+---------+
| id | name          | email             | address |
+====+===============+===================+=========+
| 0  | David Bowie   | david@bowie.com   | England |
+----+---------------+-------------------+---------+
| 1  | David Gilmour | david@gilmour.com | England |
+----+---------------+-------------------+---------+
| 2  | Opeth         | info@opeth.com    | Sweden  |
+----+---------------+-------------------+---------+
| 3  | The Beatles   | info@beatles.com  | England |
+----+---------------+-------------------+---------+
```

#### Creating a new table instance
```rust
let empty_table = MyTable::new();
let populated_table = MyTable::from_vec(&vec);
```

#### Get rows
You can get the rows using one of the getters:
- `table.get_rows()`
- `table.get_rows_mut()`
- `table.get_row_at(index)`

#### Get columns
You can get all values of a column using the `get_column` function. This function takes in a closure to get the value
of each entry.

**Example**
```rust
#[table_row]
struct MyTableRow2 {
  id: u32,
  name: String
}

#[table(rows = TableRow)]
struct MyTable2 {}

let vec: Vec<MyTableRow2> = vec![
  MyTableRow2{id: 1, name: String::from("Metallica")}, 
  MyTableRow2{id: 2, name: String::from("Slipknot")}
];
let table2 = MyTable2::from_vec(&vec);
let ids: Vec<u32> = table2.get_column(|row| row.id); // The function takes in a closure
assert_eq!(vec![1,2], ids);
```

#### Inserting rows

**Example**
```rust
let row = MyTableRow { id: 4, name: "Pink Floyd", email: "pink@floyd.com", address: "England"};
// Appending the row to the end of the table
table.push(row);
// Inserting a row at the top of the table
table.insert_top(row);
// Inserting a row in the second position 
table.insert(2, row);
```

#### Removing rows
You can use the `rm_row_at(index)` method to remove the row with the specified index, or use the `rm_row(id)` to remove
a row for a table with a [uid](#tables-with-uids).

**Examples**
```rust
let row = MyTableRow { id: 4, name: "Pink Floyd", email: "pink@floyd.com", address: "England"};
// assume table was empty before this
table.push(row);
table.rm_row_at(0);
assert_eq!(vec![], table.get_rows());
```

And for a table with a uid:
```rust
let row = MyTableRow { id: 4, name: "Pink Floyd", email: "pink@floyd.com", address: "England"};
// assume table was empty before this
table.push(row);
table.rm_row(4);
assert_eq!(vec![], table.get_rows());
```

#### Column and row count
You can get the amount of columns using the `column_count()` function on your table. You can also get the amount of rows
using the `row_count()`.

### Tables with UID's
We can specify a table with a unique identifier. The following example shows how to do this:

```rust
use simple_tables::IdTable;

#[table_row]
struct MyTableRow {
  id: u32,
  name: String
}

#[table(rows = MyTableRow)]
struct MyTable {}

// When you need a table with rows containing uid's, you will have to manually implement the 
// `get_id_from_row` function
// The `IdTable` takes in 2 type parameters, one for the id's type and one for the rows's type
impl IdTable<u32, MyTableRow> for MyTable {
  // This function will simply return the id field from the row
  fn get_id_from_row(row: &MyTableRow) -> u32 { 
    row.id 
  }
}
```

**NOTE**: If your IDE complains saying `the trait bound MyTable: Table<MyTableRow> is not satisfied`, you can simply 
ignore this. The `Table` trait is implemented in the `table` macro, but your IDE just doesn't know this.

To get rid of these warnings, you will have to enable the `org.rust.cargo.evaluate.build.scripts` and `org.rust.macros.proc`
experimental features. In IntelliJ, you can accomplish this by pressing `⇧⌘A` (macOs) or `⌃⇧A` (Linux/Windows) and searching
for `Experimental Features`, then enabling the two before-mentioned features.

#### Getting a row based on the uid

You can get a row that matches the uid using the `get_row()` function.

**Example**
```rust
let vec = vec![ 
  MyTableRow { id: 1, name: "Jimmy Page".to_string() }, 
  MyTableRow { id: 2, name: "Slayer".to_string() }, 
  MyTableRow { id: 3, name: "MGMT".to_string() } 
];

let table = MyTable::from_vec(&vec);
let table_row = table.get_row(2).unwrap();

assert_eq!(vec[1], table_row.clone());
```

You can also get a row mutably using its uid using the `get_row_mut(id)` method.

**Example**
```rust
let vec = vec![
    MyTableRow { id: 1, name: "Swedish House Mafia".to_string() },
    MyTableRow { id: 2, name: "Pink Floyd".to_string() },
    MyTableRow { id: 3, name: "Nick Cave & The Bad Seeds".to_string() }
];
let vec_unedited = vec![
    MyTableRow { id: 1, name: "Swedish House Mafia".to_string() },
    MyTableRow { id: 2, name: "Pink Floyd".to_string() },
    MyTableRow { id: 3, name: "Nick Cave".to_string() }
];

let table = MyTable::from_vec(&vec);
let mut table2 = MyTable::from_vec(&vec_unedited);
let row = table2.get_row_mut(3).unwrap();
row.name =  format!("{} {}", row.name, "& The Bad Seeds");
assert_eq!(table2.get_rows(), table.get_rows());
```

You can get a row's index using `get_row_index(id)`.

You can remove a row with a uid using `rm_row(id)`.

## Adding derive attributes
You can add derive attributes to your table, but you should put them beneath the `#[table]`.

**Example**
```rust
#[table(MyTableRow)
#[derive(PartialEq)]
struct MyTable {}
```

## Installing
Simply add the crate to your `cargo.toml`.

```toml
[dependencies]
simple_tables = "0.3.0"
```

You can see the crate on [crates.io](https://crates.io/crates/simple_tables)

## Contributing
Contributions and suggestions are always welcome. 

If you know how to improve the crate just let me know.

You can also claim one of the issues in the issues tab.

When you contribute, make sure your commit passes the tests. You can run the tests by going into the tables directory
`cd tables` and then running the tests with `cargo test`. I also have a CircleCI set up for this
repository that will test all commits. If you add a new feature, make sure to also write a test for this feature.

## Documentation
The documentation can be found [here](https://jomy10.github.io/simple_tables/simple_tables/index.html).

It is also available on [docs.rs](https://docs.rs/simple_tables/latest/simple_tables/), here you can also find 
documentation of previous versions.

## Use cases
This project started because I was working on an application that retrieved data from an SQL database. After making a
debug representation of my table, I thought that it might be a good idea to make this more generic, and so I started
working on this crate.

Here's a snippet if you are interested:
```rust
use mysql::PooledConn;
use mysql::prelude::Queryable;
use simple_tables::{IdTable, Table};
use simple_tables::macros::*;

#[table_row]
pub struct DatabaseEntry {
    id: u32,
    name: String,
    year: u32,
    ban_id: u32,
    ban: String,
    emails: Emails
}

#[table(rows = DatabaseEntry)]
pub struct Database {}

impl IdTable<u32, DatabaseEntry> for Database {
  fn get_id_from_row(row: &DatabaseEntry) -> u32 {
    row.id
  }
}

impl Database {
  pub fn fetch(conn: &mut PooledConn) -> Database {
    let mut database: Database = Database::new();

    let query = "\
        SELECT Users.User_Id as id, Users.Name as name, Years.Year_Id as year, Bannen.Ban_Id as ban_id, Bannen.Naam as ban, Emails.Email as email FROM Users \
        INNER JOIN Years ON Users.Year_Id = Years.Year_Id \
        INNER JOIN Bannen ON Years.Ban_Id = Bannen.Ban_Id \
        LEFT JOIN Emails ON Emails.Email_Id = Users.User_Id\
        ";

    conn.query_iter(query)
            .unwrap()
            .for_each(|row| {
              let (user_id, name, years_id, ban_id, ban_naam, email):
                      (u32, String, u32, u32, String, Option<String>)
                      = mysql::from_row(row.unwrap());
              if let Some(email) = email {
                if let Some(entry) = database.get_row_mut(user_id) {
                  entry.emails.push(email);
                } else {
                  // Create new entry
                  database.push(
                    DatabaseEntry {
                      id: user_id,
                      name,
                      year: years_id,
                      ban_id,
                      ban: ban_naam,
                      emails: Emails { emails: Some(vec![email]) }
                    }
                  );
                }
              } else {
                // Create new entry
                database.push(
                  DatabaseEntry {
                    id: user_id,
                    name,
                    year: years_id,
                    ban_id,
                    ban: ban_naam,
                    emails: if email.is_some() { Emails { emails: Some(vec![email.unwrap()]) } } else { Emails { emails: None } }
                  }
                );
              }

            });

    database
  }
}

#[derive(Clone, Debug)]
pub struct Emails {
  emails: Option<Vec<String>>
}

impl Emails {
  fn push(&mut self, s: String) {
    if let Some(ref mut v) = self.emails {
      v.push(s);
    }
  }
}

impl ToString for Emails {
  fn to_string(&self) -> String {
    let mut s = String::new();
    if self.emails.is_some() {
      let len = self.emails.as_ref().unwrap().len();
      for (i, email) in self.emails.as_ref().unwrap().iter().enumerate() {
        if i != len - 1 {
          s.push_str(format!("{}{}", email, ", ").as_str());
        } else {
          s.push_str(email);
        }
      }
    } else {
      s.push_str("None");
    }
    s
  }
}
```

If you end up building something with this crate, let me know!

## Questions?
Feel free to open an issue or contact me directly with any questions.

## License
This crate is licensed under the **MIT License**. Details in the [LICENSE](LICENSE) file.

Alternatively, this crate can also be licensed under the [Apache 2.0-license](ALT_LICENSE).