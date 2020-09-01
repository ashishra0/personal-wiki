+++
title = "Postgres full text search"
tags =["postgres"]
date = "2020-01-02"
+++

In this guide we will walk through how to implement a full text search
in Postgres to create a custom SQL function and expose it to the GraphQL
API.

# What is FTS?

Imagine we have a set of text documents stored in a database. These
documents could be an abstract of certain text article or the entire
article itself and we want to find out if certain words are present in
them or not.

The way FTS is implemented in Postgres is by getting a semantic vector
for all the words contained in the document. So, when we search for
words like \"jump\", we will match all instances of the word like
\"jumping\", \"jumped\". So in essence we will be searching just the
vector and not the document which is fast.

# Functions

PostgreSQL has two functions that do exactly what we intend to do:

- `to_tsvector`: This will create a list of tokens.
- `to_tsquery`: To query the vector for occurences of certain words.

**to_tsvector()**

Example: To create a vector for the sentence \"The quick brown fox
jumped over the lazy dog\"

```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog');
```

returns:

```bash
to_tsvector
-------------------------------------------------------
'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
```

Every word is normalized into a lexeme.

> Sometimes the word won\'t be normalized depending on the localization
> settings of your postgres installation. Default language is English but
> this can be changed by passing the language of you are working with as
> an argument.

```sql
SELECT to_tsvector('French', 'Le rapide renard brun sauta par dessus le chien paresseux');
```

It would return a vector normalized according to the French language.

```bash
to_tsvector
-------------------------------------------------------------------------
'brun':4 'chien':9 'dessus':7 'paress':10 'rapid':2 'renard':3 'saut':5
```

**to_tsquery()**

This function will accept a list of words that will be checked against
the normalized vector we created with `to_tsvector()`

Example:

```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog') @@ to_tsquery('fox');
```

```bash
?column?
----------
t
```

The `@@` operator is used to check if the `tsquery` matches `tsvector`.

**tsquery** also provides a set of operators such as:

- AND operator (&)

```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog') @@ to_tsquery('fox & dog');
```

- OR operator (\|)

```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog') @@ to_tsquery('fox | clown');
```

- NEGATION operator (!)

```sql
SELECT to_tsvector('The quick brown fox jumped over the lazy dog') @@ to_tsquery('!clown');
```

# Creating and Storing the tsvector Data Type

Let's say we have two simple tables for an article/author schema

```
author (
  id INT PRIMARY KEY,
  name TEXT
)

article (
  id INT PRIMARY KEY,
  title TEXT,
  content TEXT,
  rating INT,
  author_id INT
)
```

We will store the vectors in the same table instead of vectorizing the
documents on the fly because the execution time is faster.

```sql
ALTER TABLE article
ADD COLUMN document tsvector;
update article
set document = to_tsvector(title || ' ' || content);
```

We can take this up another notch up by adding index to the pre computed
tsvector column.

```sql
ALTER TABLE article
ADD COLUMN document_with_idx tsvector;
update artile
set document_with_idx = to_tsvector(title || ' ' || content);
CREATE INDEX document_idx
ON card
USING GIN (document_with_idx);
```

And it can be queried like this

```sql
SELECT name, artist, text from card
WHERE document_with_idx @@ to_tsquery('pandoc');
```
