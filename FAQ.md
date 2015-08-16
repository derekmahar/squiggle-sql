### How does Squiggle relate to JDBC? ###

Squiggle's only purpose is for performing text-manipulation of SQL. It does not use any of the JDBC classes. Typically you would use Squiggle to generate your statement, then pass it to JDBC for execution.

### How does Squiggle compare to JDO, Hibernate, CMP, etc? ###

It doesn't. Squiggle is not a persistence layer. In most cases I would recommend using a persistence layer such as Hibernate or JDO, but there are some occassions where you just need to revert to a plain old SQL query.

### Can I modify the Squiggle source code and use it in my own application. ###

Sure! Squiggle is open sourced under a BSD license. Please read the license thoroughly.

### Does Squiggle support joins and subselects? ###

Yep.

### Does Squiggle support INSERT, UPDATE or DELETE statements? ###

Nope. Squiggle's purpose is for building SELECT statements only. However you can extend Squiggle to support custom needs if you want.