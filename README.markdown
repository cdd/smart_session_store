SmartSession
============
[![Build Status](https://travis-ci.org/fcheung/smart_session_store.png)](https://travis-ci.org/fcheung/smart_session_store)

A session store that avoids the pitfalls usually associated with concurrent access to the session (see http://www.paulbutcher.com/2007/05/race-conditions-in-rails-sessions-and-how-to-fix-them/)

Derived from SqlSessionStore, see http://railsexpress.de/blog/articles/2005/12/19/roll-your-own-sql-session-store

Requires rails 3 or higher.

Installation
==========

Step 1
-------
Add the gem to your gemfile
gem 'smart_session'

Step 2
-------
Generate your sessions table using rake db:sessions:create

Step 3
-------
Add the code below in an initializer

    
    YourApp::Application.config.session_store SmartSession::Store, key: '_test_app_session'

Depending on your database type, add

    SmartSession::Store.session_class = :mysql2
    SmartSession::Store.session_class = :postgres
    SmartSession::Store.session_class = :sqlite

after the initializer section in environment.rb


Step 4 (optional)
-------
If you want to use a database separate from your default one to store
your sessions, specify a configuration in your database.yml file (say
sessions), and establish the connection on SqlSession in
environment.rb:

   SmartSession::SqlSession.establish_connection :sessions

Testing
=======
To run tests with a certain database, set the DATABASE environment variable.
You may need to configure the database.yml or your database server so that the correct connection options are used.

For example:

   rake test    # defaults to mysql
   rake test DATABASE=postgres


Performance Implications
========

There's no such thing as a free lunch. On top of the overhead of a database backed session store, whenever you make changes to the session you'll incur a small extra hit: we lock the current session row, we reload it to determine if there is updated data and we merge the changes before doing the actual save. On the flipside, we only save the session at all if changes have been made.

Optimistic locking
=======
You can enable optimistic locking by adding an integer column called lock_version to your sessions table. It should have a default value of 0. The optimistic locking feature is identical in concept to optimistic locking within Rails and saves a database query in those cases where there is no need to merge changes, which is usually the most common case.


IMPORTANT NOTES
=======
1. You will need the binary drivers for Mysql or Postgresql.
   These have been verified to work:

   * ruby-postgres (0.7.1.2005.12.21) with PostgreSQL 8.1
   * postgres (0.7.9.2008.01.28) with PostgreSQL 8.3
   * pg (0.7.9.2008.10.13) with PostgreSQL 8.3
   * ruby-mysql 2.7.1 with Mysql 4.1
   * ruby-mysql 2.7.2 with Mysql 5.0
   * mysql2 ~>0.2.7 with Mysql 5.0
   * mysql2 ~>0.3.1 with Mysql 5.1

2. Tests have been done with SqlLiteSession, SqlSession, PostgresqlSession
   and Mysql2Session. Feedback would be very much appreciated.
