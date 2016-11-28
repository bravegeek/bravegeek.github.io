---
title: Entity Framework 6 and Sybase SQL Anywhere
date: 2016-07-13 23:45:02
tags:
    - Entity Framework
    - Performance Tuning
---
![](/content/images/2016/ef6.sybase.png)

## The Problem
Getting Sybase SQL Anywhere 16 (SA16) to work with Entity Framework 6 (EF6) can be a bit of a challenge. What made our situation particularly challenging was that we had older versions of Entity Framework (EF4) that needed to work with SA16.

Sybase documentation instructs you to install the latest version of Sybase and then run the special .exe for EF6. Unfortunately, this leaves you with SA16 drivers that ONLY work with EF6. Any other code that needs to use an older version of EF won't work correctly.

## The Fix
It turns out there is a way to have both! Some simple config file changes will accomplish this.

You'll need to modify three sections: 
1. connectionStrings
2. system.data
3. entityFramework

```xml
<connectionStrings>
     <add name="YourSybaseConnection" connectionString="Host=your_hosting_server;ServerName=your_Sybase_DB_server_instance;DatabaseName=your_db_name;uid=db_user_id;pwd=db_pwd" providerName="iAnywhere.Data.SQLAnywhere" />
</connectionStrings>

<system.data>
    <DbProviderFactories>
      <clear />
      <add name="SQL Anywhere 16 Data Provider" invariant="iAnywhere.Data.SQLAnywhere" description=".Net Framework Data Provider for SQL Anywhere 16" type="iAnywhere.Data.SQLAnywhere.SAFactory, iAnywhere.Data.SQLAnywhere.EF6, Version=16.0.0.20434, Culture=neutral, PublicKeyToken=f222fc4333e0d400" />
    </DbProviderFactories>
</system.data>

<entityFramework>
    <defaultConnectionFactory type="System.Data.Entity.Infrastructure.LocalDbConnectionFactory, EntityFramework">
      <parameters>
        <parameter value="mssqllocaldb" />
      </parameters>
    </defaultConnectionFactory>

    <providers>
      <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
      <provider invariantName="iAnywhere.Data.SQLAnywhere" type="iAnywhere.Data.SQLAnywhere.SAProviderServices, iAnywhere.Data.SQLAnywhere.EF6, Version=16.0.0.20434, Culture=neutral, PublicKeyToken=f222fc4333e0d400" />
    </providers>
</entityFramework>
```
Now your code that uses the older EF should run side-by-side with the new EF6 code.

### Noteworthy
The Sybase connection string is definitely different than a MS SQL connection string.
If you follow my example, you won't need to setup an ODBC DSN; it will simply connect.

Make sure you're running a newer version of Sybase SA16, some older versions don't have the .dll's for EF6.


Helpful links:  
[How to add Sybase EF6 provider](http://sqlanywhere-forum.sap.com/questions/22161/entity-framework-6-provider)
[Sybase .Net client deployment](http://dcx.sybase.com/index.html#sa160/en/dbprogramming/deploying-adonet-deploy.html)