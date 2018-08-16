---
layout: post
title: liquibase
date: 2018-08-16
categories: liquibase
---
* summary
* create a change log file
* load data from liquibase by csv
* include change log file into comprehensive file
* using db.changelog_master.xml file manager change log file
* liquibase.properties
* liquibase maven plugin


### create a change log file

In liquibase , using many db change log file for maintaining db meta-data or  structure   , These db change log includes  database operation of DDL , These db change log file would resides in resources directory , like below. 

![change log file]({{ site.url }}/assets/images/liquibase-changelogfile.PNG "change log file")

A change log file is a XML format file . The root element and xml schema is below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

</databaseChangeLog> 

```

While i want to create a new table named as `TblLiquibaseTest` , then i create a change log xml file named as `db.changelog.create.TblLiquibaseTest.xml` placed into resources directory , The xml content is showing below:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">

    <changeSet id="createtion-com.aiatss.coast.AppTest.LiquibaseTest" author="AndyLi">
        <createTable tableName="TblLiquibaseTest">
            <column name="testId" type="java.sql.Types.BIGINT">
                <constraints nullable="false" primaryKey="true"></constraints>
            </column>
            <column name="testName" type="java.sql.Types.VARCHAR(50)">
                <constraints nullable="true"></constraints>
            </column>
            <column name="createDate" type="java.sql.Types.TIMESTAMP"></column>
            <column name="status" type="java.sql.Types.CHAR(1)"></column>
        </createTable>
    </changeSet>


</databaseChangeLog>
```
1. `databaseChangeLog` is the root element of change log xml
2. `chanageSet` is an operation of DDL
3. `changeSet.id` `changeSet.author` is a unique identification using by liuqibase to identify which operation has been finished or not .  

### comprehensive file

In the same directory with `db.changelog.create.TblLiquibaseTest.xml` , i created comprehensive file `db.changelog.TblLiquibaseTest.xml` to include `db.changelog.create.TblLiquibaseTest.xml` in order to maintain familial change log file.

```xml

<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
    <include file="com/xx/xx/infrastructure/persistence/db.changelog.create.TblLiquibaseTest.xml"/>
</databaseChangeLog>

```

### db.changelog_master.xml