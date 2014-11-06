============
Partitioning
============
In this writeup compared two methods of partitioning, one based on routing
queries in Ruby (gem) and the other based on PL/pgSQL scripts on PostgreSQL
server.  Both methods use existing partitioning supported by Postgres starting
version 8.2.

The word "gem" below refers to "partitioned" gem unless stated otherwise.

`High level introduction into the topic
<https://github.com/fiksu/partitioned/blob/master/PARTITIONING_EXPLAINED.txt>`_
and the `official specification
<http://www.postgresql.org/docs/9.3/static/ddl-partitioning.html>`_

Postgres starting from 8.2 is able to handle SELECT, UPDATE and DELETE with
constraint_exclusion.  But, it's not able to handle INSERTs on the parent
tables.  See a `wiki <https://wiki.postgresql.org/wiki/Table_partitioning>`_ on
this topic.

Overview
========
Gem is located `here <https://github.com/fiksu/partitioned>`_, pg_partman
`there <https://github.com/keithf4/pg_partman>`_.

Gem is currently compatible with Rails ~> 3.2.19, latest commit dated
2013-12-13.  Rails 4 is currently not supported: there is a `pull request
<https://github.com/eslavich/partitioned/tree/upgrade-partitioned-gem-for-rails-4-68296828>`_
on github and authored March 27 by a different developer not the original
author of the gem.  So it looks like there is no definite answer regarding the
Rails 4.

Description
-----------
pg_partman is a set of PL/pgSQL functions and python scripts for maintenance
(dump, partition on existing table, undo partition and so on).  On INSERTs,
this approach uses database triggers to redirect inserts to correct child
tables.

Gem is a pure Ruby library.

Installation
------------
pg_partman require installation on postgresql server.  Plpgsql functions cannot
break database server but can affect the site functioning.  A collection of
python scripts serve for maintenance purpose and can be excluded from regular
work-flows.

Gem is installed together with the tddium_site.  Gem can break functioning of
the rails application if it is broken.  There is a higher risk of breaking
runtime things with this approach.  The risk exists since approach based on
monkey patching ActiveRecord code and is based on Rails 3.2.

Implementation
--------------
Gem: API (separate from Rails API) knowledge is required for usage.  A
significant code change is needed for implementation.  Technically maintenance
is harder.

pg_partman: application rails side should not require changes at all!  Require
adding a maintenance cron job taking care of partitions only.

Problems
--------
#. Add child tables

    pg_partman
      run_maintenance database function does the job.  It can be run
      periodically in a cron job

    gem
      API function exists which adds child partitions.  It can be run
      periodically in a cron job

#. INSERT

    pg_partman
      insert trigger in plpgsql
    
    gem
      stubs with a plpgsql rule to disable direct inserts to parent table.
      Monkey patch ActiveRecord to route insert to correct child

#. SELECT, UPDATE and DELETE

    pg_partman
      can be handled through constraint_exclusion option with postgres query
      planner

    gem
      Monkey patch or override ActiveRecord

#. Migrate existing tables

    pg_partman
      contains Python scripts for easy migration

    gem
      N/A

END
