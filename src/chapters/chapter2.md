## Connecting an Azure SQL Database

In this chapter you'll learn about Azure SQL Database, how to provision one in the Azure Portal, and how to use Entity Framework Core (EF Core) to seed a local (development database) and an Azure SQL Database.


### Overview

Azure SQL Database is a general-purpose relational database-as-a-service (DBaaS) based on the latest stable version of Microsoft SQL Server Database Engine. SQL Database is a high-performance, reliable, and secure cloud database that you can use to build data-driven applications and websites in the programming language of your choice, without needing to manage infrastructure.


### Provisioning in Azure

Now that you know what Azure SQL Database can do, let's start using it! You'll start by creating a SQL Database instance in the Azure portal. 

> **SQL Servers and Databases**
>
> When creating a SQL Database, you also need to create a SQL Server. The steps below walk you through this process.

<h4 class="exercise-start">
    <b>Exercise</b>: Creating a SQL Server and Database instance
</h4>

Start by jumping back to the Azure portal, and create a new resource by clicking the *Create a resource* button.

Search for *SQL Database*:

<img src="images/chapter2/sql-search.png" class="img-medium" />

Fill out the required parameters as you create an instance:
- Resource group: the resource group you created earlier
- Database name: ContosoUniversity3
- Server: *Press Create new button*, complete right side panel
- Server name: unique SQL server name to create
- Server admin login: workshopAdmin
- Password: *Enter complex password*, write it down
- Location: East US

<img src="images/chapter2/create-sql.png" class="img-override" />

Press the *Select* button after completing the server information in the right panel, then press *Review and create*.

<img src="images/chapter2/sql-review-create.png" />

When the SQL Server and database instances are provisioned, they appear in your resource group:

<img src="images/chapter2/sql-deployed.png" class="img-override" />

The final step is to navigate to the SQL Server instance by clicking on it. 

> **Secured by default**
>
> As with most Azure resources, SQL Server is secured by default, meaning that you cannot access the SQL Server or databases without opening up the attached firewall. In this next step, we'll do just that.

Locate the *Firewalls and virtual networks* area, click *Show firewall settings*.

<img src="images/chapter2/sql-firewall.png" class="img-override" />

Click the *Add client IP* button and notice an entry being made in the table below. Then, click *Save*. This opens the SQL Server firewall to allow you to communicate with it directly for the duration of the workshop.

<img src="images/chapter2/add-client-ip.png" class="img-override" />

Next, navigate to the *SQL Databases* link on the left, and select the ContosoUniversity3 database.

<img src="images/chapter2/nav-sql.png" class="img-medium" />

Click the *Show database connection strings* link:

<img src="images/chapter2/conn-strings.png" class="img-override" />

Copy the connection string and paste it into Notepad, or another text editor:

<img src="images/chapter2/copy-conn-string.png" class="img-override" />

After pasting the connections tring into a text editor, locate the `{your_username}` and `{your_password}` sections and update them with the SQL Server username and password you created previously.

<img src="images/chapter2/update-conn-string.png" class="img-medium" />

Save this connection string -- you'll need it later!

This concludes the exercise. 

<div class="exercise-end"></div>

### Seeding your databases

You may have noticed that the Contoso University app crashes when you navigate to various pages. That's because it's expecting a database exists, but we haven't created it.

In this next step, we'll use Entity Framework's Migrations to create and seed a local SQL database, then do the same for the Azure SQL Database we just created.

<h4 class="exercise-start">
    <b>Exercise</b>: Seeding the local SQL Database
</h4>

Every installation of Visual Studio 2019 comes with a development version of SQL Server called MS SQL Local DB. In fact, the Contoso University app is pre-configured to use this SQL Server instance already.

Open the `appsettings.json` file to see the connection string:

```json
"ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity3;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
```

Notice the server name of **(localdb)\\mssqllocaldb**. This is the local development version of SQL I mentioned above. We'll use this in the next section to "seed" the database with Entity Framework Migrations.

> **Entity Framework and Migrations**
>
> If you're not familiar with Entity Framework and Migrations, don't worry. Entity Framework (EF) is a technology called an object-relational mapper (O/RM). O/RMs (like EF) enable .NET developers to work with a database using .NET objects, and eliminating the need for most of the data-access code they usually need to write (like stored procedures, views, and queries). 
>
> EF Migrations are a way to use .NET objects and code to create a SQL database and tables, then seed the database with preliminary data.

The Contoso University app already has Entity Framework installed and configured. Let's use it to create the ContosoUniversity3 database locally by running the migrations.

#### Using EF Migrations

In Visual Studio, go to the *Tools* menu, select *NuGet Package Manager*, then *Package Manager Console*.

<img src="images/chapter2/open-pm.png" class="img-medium" />

This opens a new area at the bottom of Visual Studio:

<img src="images/chapter2/pm-console.png" class="img-medium" />

In the Package Manager Console, change the Default project to *ContosoUniversityData*, the type *Update-Database* and press *Enter*. The EF Migrations will read the connection string from your configuration file, create the ContosoUniversity3 database, create the tables, and load seed data into the tables. You can see the process happening below:

<img src="images/chapter2/ef-migrations.gif" class="img-medium" />

When the EF Migrations have finished, run the web site, and navigate to the *Courses* page. You will see the page displays a variety of course data:

<img src="images/chapter2/courses.png" class="img-medium" />

This concludes the exercise. 

<div class="exercise-end"></div>

Now that you've used EF Migrations, let's seed the ContosoUniversity3 database in Azure. 

<h4 class="exercise-start">
    <b>Exercise</b>: Seeding the Azure SQL Database
</h4>

Open the `appsettings.json` file to adjust the connection string. You'llrecall that we saved the Azure SQL Database connection string in our text editor earlier. Copy the value and paste it into the configuration file.

Mine looks like:

<img src="images/chapter2/appsettings.png" class="img-medium" />

Now that the connection string has been updated, return to the Package Manager Console and run the EF Migrations again.

> **ContosoUniversityData project**
>
> Don't forget to change the default project to ContosoUniversityData before running the migrations.

```
> Update-Database
```

This will use the updated connection string to provision and seed the ContosoUniversity3 database in Azure.

When the migrations have finished, run the website to verify it still works. 

Now, as a final step, re-deploy your web app to Azure, so it has the updated connection string (pointing to your Azure SQL Database).

This concludes the exercise. 

<div class="exercise-end"></div>

That's it. In the next chapter, you'll learn how to secure your connection string so it isn't visible int he configuration file.
