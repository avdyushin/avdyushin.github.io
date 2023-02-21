---
title: Rust and SQLite
date: "2022-08-23 19:37:52 +0200"
tags: [rust, sqlite]
mermaid: true
---

In this article I'll describe my walk-through to create tiny Rust library and CLI tool to work with Bible texts.
The Bible is represented as relative database using SQL syntax. The structure is described in a corresponding section.

Before we start, let's check `rustc` version installed:

```console
$ rustc --version
rustc 1.51.0 (2fd73fabe 2021-03-23)
```

Hm, a bit old, so it's time for the update. Updating to the latest version is _easy_:

```console
$ rustup update
info: syncing channel updates for 'stable-x86_64-apple-darwin'
info: latest update on 2022-08-11, rust version 1.63.0 (4b91a6ea7 2022-08-08)
info: downloading component 'rust-src'
info: downloading component 'cargo'
info: downloading component 'clippy'
info: downloading component 'rust-docs'
[...]
$ rustc --version
rustc 1.63.0 (4b91a6ea7 2022-08-08)
```

Now it's way better. Next is to create a library project.
This library will work with models and preparing SQL statments to fetch verses from the database.

## Creating new library

Using `cargo` we can create new binary or library project and also manage our dependencies.

```console
$ cargo new --lib bible-sql-rs
     Created library `bible-sql-rs` package
$ ls -la bible-sql-rs
total 16
drwxr-xr-x   9 avdyushin  staff   288B Aug 23 19:45 .git
-rw-r--r--   1 avdyushin  staff    20B Aug 23 19:45 .gitignore
-rw-r--r--   1 avdyushin  staff   181B Aug 23 19:45 Cargo.toml
drwxr-xr-x   3 avdyushin  staff    96B Aug 23 19:45 src
```

We have git reporistory created automatically for us.
Main library file located at `src/lib.rs`{: .filepath }.
Let's look what's inside:

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}
```
{: file="src/lib.rs" }

Not too much. Here is only simple public function and unit test. Let's run tests for our empty project.

## Running tests

We can run tests to see if everything works correctly:

```console
$ cargo test
   Compiling bible-sql-rs v0.1.0 (/[...]/bible-sql-rs)
    Finished test [unoptimized + debuginfo] target(s) in 1.46s
     Running unittests src/lib.rs (target/debug/deps/bible_sql_rs-9de449324a76a3a1)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests bible-sql-rs

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

## Adding dependencies

In our `Cargo.toml`{: .filepath } (`[dependencies]` section):

```toml
[package]
name = "bible-sql-rs"
version = "0.1.0"
edition = "2022"

[dependencies]
# Parse Bible references from plain text
bible-reference-rs = { version = "0.1.3" }
# Command line argument parser
clap = { version = "3.2.17" }
# SQLite wrapper
rusqlite = { version = "0.28.0" }
```
{: file="Cargo.toml" }

Let's build our library to see if all dependencies are valid:

```console
$ cargo build
    Updating crates.io index
    [...]
    Finished dev [unoptimized + debuginfo] target(s) in 1m 06s
```

All has to be green.

## Test resources

This is important to test our models and statements if they're valid.

Now we need some SQL files to build our test database which will be used for our unit tests.

First create `res/test`{: .filepath } directory where all data needed for tests will be located:

```console
$ mkdir -p res/test
```
We also need to exclude this directory from library publishing process. In `Cargo.toml`{: .filepath }:

```toml
[package]
[...]
exclude = ["res/test/**"]
[...]
```
{: file="Cargo.toml"}

Next step is to generate database to work with and run our tests over it.

## Creating SQLite database for tests

Schema is a pretty simple:

```sql
CREATE TABLE kjv_bible_books (
    id   SMALLINT NOT NULL PRIMARY KEY, -- book id
    idx  SMALLINT NOT NULL,             -- book index
    book VARCHAR(40) NOT NULL,          -- book name
    alt  VARCHAR(20) NOT NULL,          -- book alternative name
    abbr VARCHAR(20) NOT NULL,          -- book abbreviation
    UNIQUE (book, alt, abbr),
    UNIQUE (idx),
    UNIQUE (id)
);
```

`kjv_bible_book` table row contains `id` as primary key, book name `book`, alternative book name `alt`, and book abbreviation `abbr`.

`idx` is used to sort books as it's vary in different tranlations.

```sql
CREATE TABLE kjv_bible (
    book_id SMALLINT NOT NULL, -- book id
    chapter SMALLINT NOT NULL, -- chater number inside book
    verse   SMALLINT NOT NULL, -- verse number inside chapter
    text    TEXT NOT NULL,     -- verse text
    PRIMARY KEY (book_id, chapter, verse)
);
```

`kjv_bible` table contains `book_id`, `chapter` number, `verse` number and verses's text itself. Primary key for this table will be `(book_id, chapter, verse)` as it's unique per verse.

The Bible version (translation) is defined in the prefixes of the tables. In example above it's `kjv_`.
All tables related to the same version should have same prefix to be able switch between different translations.
> Thinks of daily reader plans or verse of the day —  they all are based on the version.
{: .prompt-info }

Having only this two tables we are be able to navigate over Bible and fetch any verse(s).

I have ready to use SQL script files with KJV version of the Bible.
Here is shell script to fetch data files and put them into `res/test`{: .filepath } directory:

```shell
#!/bin/sh

files=(
    "kjv_bible_books.sql"
    "kjv_bible_verses_table.sql" 
    "kjv_bible_verses_data.sql"
)

base_url=https://raw.githubusercontent.com/avdyushin/bible-docker-postgres/master/data/
index=10 #start prefix
output=res/test

for file in ${files[@]}; do
    src=$base_url$file
    dst=$index-${file//_/-}
    curl $src -o $output/$dst
    index=$((index + 10))
done
```

Now `res/test`{: .filepath } should contain `*.sql` files:

```console
$ ls -l res/test
total 19192
-rw-r--r--  1 avdyushin  staff   2.8K Aug 25 16:41 10-kjv-bible-books.sql
-rw-r--r--  1 avdyushin  staff   329B Aug 25 16:41 20-kjv-bible-verses-table.sql
-rw-r--r--  1 avdyushin  staff   4.4M Aug 25 16:41 30-kjv-bible-verses-data.sql
```

File are prefixed with numbers so the can be applied with the given order (e.g. table created first and then data inserted).

Now we can create database named `kjv.db`{: .filepath }:

```console
$ cd res/test
$ find . -type f | sort | xargs cat | sqlite3 kjv.db
```
> Using `sort` we execute files in order.
{: .prompt-info }

And check if it's correctly created:

```console
$ sqlite3 kjv.db
SQLite version 3.37.0 2021-12-09 01:34:53
Enter ".help" for usage hints.
sqlite> .tables
kjv_bible kjv_bible_books
sqlite> select count(*) from kjv_bible_books;
66
```

It's two table and 66 books in the books table as expected.

## Connecting to SQLite from library

*[CLI]: Command Line Interface
*[KJV]: King James Version
*[SQL]: Structured Query Language
