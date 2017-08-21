---
layout: default
title: "Lab 24: JDBC"
---

Getting Started
===============
We'll continue our work on SQL databases this week by again "borrowing" from the CS320 SW Engineering labs.

Download [CS201\_Lab24.zip](CS201_Lab24.zip). Import it into your Eclipse workspace (**File&rarr;Import...&rarr;General&rarr;Existing projects into workspace&rarr;Archive File**). You will see a project called **CS201\_Lab24** in the Package Explorer.  (You will also need to have the **CS201_Derby** project in your workspace, which should already be there from [Lab 23](lab23.html).)

The lab contains a database called **test.db** which is the books database described in [Lecture 24](../lectures/lecture24.html).

You may work individually or with a small group.

Task
====

In the lab skeleton you will find a program called **TitleQuery** which demonstrates basic JDBC tasks such as loading a driver, connecting to a database, creating and executing a prepared statement, and iterating through results returned from a query.

Using **TitleQuery** as a model, write your own programs to do the following:

**(1)** Find all books written by the author whose last name is specified. Return the books in the same form as the **TitleQuery** program.

**(2)** For an existing author, given the full (first and last) name of an author, a title, and an ISBN, insert the book into the database.  The program must first retrieve the existing author's **author\_id** before inserting the new book entry into the books table.

**(3)** **CHALLENGE:** Add a book for an author that is not already in the database.  In this case, the program must first add the new author to the authors table, and then retrieve the auto-generated **author\_id** for the new author, before inserting the new book into the database.  Hint: This requires multiple steps: attempt to retrieve the **author\_id**.  If the result set is empty, add the new author to the authors table.  Then, retrieve the **author\_id** for the new author.  Now you can insert the new book into the database.

Use the SQL **insert** statement to insert the new tuple(s).
