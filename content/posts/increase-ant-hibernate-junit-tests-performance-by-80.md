---
title: 'Increase Ant Hibernate Junit tests Performance by 80%'
date: Tue, 27 Dec 2016 20:52:25 +0000
draft: false
tags: ['ant', 'hibernate', 'junit', 'mysql', 'performance', 'robolucha', 'selfhackaton']
---

![](/images/2016/12/performance_comparision_junit_ant_hibernate.png) If don´t want to read everything, just set **forkingmode="once"** to have huge improvement in your tests processing time. :)

### Now the details...

I have several, around 100 junit tests in [Robolucha](http://www.robolucha.com/) project, most of then use [Hibernate](http://hibernate.org/orm/) and are run by an ant script. When you start Java Virtual machine Hibernate prepares itself to connect to the database by checking the necessary tables and preparing all the internals needed to work, but this process takes some time, some seconds to say the true. When you are running the unit tests and forking each process in a separated VM by using fork="yes" the default behavior is to run one VM per test. With one VM per test, the result would be Hibernate running preparing its internals for each test, and the results are around 5 minutes as you can see at the table above. So reuse the Hibernate preparation was the main goal of the optimization but some other options were explored and some nice findings came from that. Here are the steps I did trying to optimize the test performance:

*   Reuse Hibernate preparation
*   Use in Memory tables
*   Use MyISAM tables
*   Set hbm2ddl to none - no significant results from this one

### Reuse Hibernate preparation

As you can see at [junit ant task documentation](https://ant.apache.org/manual/Tasks/junit.html), the default behavior for fork=yes is to set forkmode="perTest", and that was the main reason for the poor performance on my tests. set forkmode="once" and only one VM will be created to the tests and Hibernate preparation will run only once, changing the times from around 5 minutes to one.

### Use different types of tables

InnoDB offers several nice features like referential integrity and so on, by there are other options with less features and much better performance, for the tests I tried to use then to check the performance, but it wasn´t that easy :) First I tried myISAM table tyble just changed the hibernate.dialect to [org.hibernate.dialect.MySQLMyISAMDialect](http://grepcode.com/file/repo1.maven.org/maven2/org.hibernate/hibernate-core/4.3.6.Final/org/hibernate/dialect/MySQLMyISAMDialect.java#MySQLMyISAMDialect), but sadly this class is using a wrong parameter at the method [getTableTypeString](http://grepcode.com/file/repo1.maven.org/maven2/org.hibernate/hibernate-core/4.3.6.Final/org/hibernate/dialect/Dialect.java#Dialect.getTableTypeString%28%29 "Overrides org.hibernate.dialect.Dialect.getTableTypeString()")(), its returning **type=MyISAM** where it should be: **ENGINE=MyISAM**, So in order to fix it I extended MySQL5InnoDBDialect by overriding getTableTypeString()

```
public class **MySQL5MyISAMDBDialectEngine** extends MySQL5InnoDBDialect {

  public MySQL5MyISAMDBDialectEngine() {
    super();
  }

  @Override
  public String getTableTypeString() {
    return " ENGINE=MyISAM";
  }
}
```

Used a similar strategy for the in memory tables, with an extra detail that in [Memory table DON´T support BLOB types](http://dev.mysql.com/doc/refman/5.7/en/memory-storage-engine.html), so I need to create a custom mapping for large strings to use VARCHAR(1024) instead of BLOB.

```
public class **MySQL5InnoDBDialectInMemory** extends MySQL5InnoDBDialect {

  public MySQL5InnoDBDialectInMemory() {
    super();
  }

  @Override
  public String getTableTypeString() {
    return " ENGINE=MEMORY";
  }

 /\*\*
  \* avoid LONGTEXT that are not supported by In Memory tables
  \*/
  @Override
  protected void registerVarcharTypes() {
    registerColumnType(Types.VARCHAR, "varchar(1024)");
    registerColumnType(Types.VARCHAR, 255, "varchar($l)");
    registerColumnType(Types.LONGVARCHAR, "varchar(1024)");
  }
}
```

### Conclusion

After all the tests my final configuration is:

*   junit fork="yes" forkmode="once"
*   hbm2ddl=update
*   hibernate.dialect=com.robolucha.test.MySQL5MyISAMDBDialectEngine

Forcing the use of only one VM for all the tests, checking if all the necessary tables are in place, as this dont take that much time and using myISAM tables that has almost the same performance as in Memory but uses less memory.