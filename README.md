AspNet.Identity.MongoDB
=======================

Forked from: steentottrup/AspNet.Identity.MongoDB
Bellow are the authors note:

An updated version of an ASP.NET Identity provider using MongoDB for storage. This started out as a fork on the original project [MongoDB.AspNet.Identity by InspectorIT](https://github.com/InspectorIT/MongoDB.AspNet.Identity), but it seems the author has abandoned the project, so I've decided to create my own repository.

[![Build status](https://ci.appveyor.com/api/projects/status/1knmbosmm45mdr48/branch/master?svg=true)](https://ci.appveyor.com/project/SteenTttrup/aspnet-identity-mongodb/branch/master)
[![Test status](http://teststatusbadge.azurewebsites.net/api/status/SteenTttrup/aspnet-identity-mongodb)](https://ci.appveyor.com/project/SteenTttrup/aspnet-identity-mongodb)
[![NuGet Version](http://img.shields.io/nuget/v/aspnetidentity.mongodb.svg?style=flat)](https://www.nuget.org/packages/aspnetidentity.mongodb/)

## Purpose ##

ASP.NET MVC 5 shipped with a new Identity system (in the Microsoft.AspNet.Identity.Core package) in order to support both local login and remote logins via OpenID/OAuth, but only ships with an Entity Framework provider (Microsoft.AspNet.Identity.EntityFramework).

## News ##
__07-07-2016__ - Updating to latest MongoDb driver (2.2.4) and fixing the async/await methods to work as intended.

__22-11-2015__ - The repository was created to take the code to the latest version of the Identity assemblies and the MongoDB driver.

## Features ##
* Drop-in replacement ASP.NET Identity with MongoDB as the backing store.
* Requires only 2 mongo document type, one for users and one for roles
* Supports additional profile properties on your application's user model.
* Provides UserStore<TUser> implementation that implements these interfaces:
	* IUserLoginStore<TUser>
	* IUserClaimStore<TUser>
	* IUserRoleStore<TUser>
	* IUserPasswordStore<TUser>
	* IUserSecurityStampStore<TUser>
	* IQueryableUserStore<TUser>
	* IUserEmailStore<TUser>
	* IUserPhoneNumberStore<TUser>
	* IUserTwoFactorStore<TUser, string>
	* IUserLockoutStore<TUser, string>
	* IUserStore<TUser>
* Provides RoleStore<TRole> implementation that implements this interface:
	* IQueryableRoleStore<TRole>

## Instructions ##
These instructions assume you know how to set up MongoDB within an MVC application.

1. Create a new ASP.NET MVC 5 project, choosing the Individual User Accounts authentication type.
2. Remove the Entity Framework packages and replace with MongoDB Identity:

```PowerShell
Uninstall-Package Microsoft.AspNet.Identity.EntityFramework
Uninstall-Package EntityFramework
Install-Package AspNetIdentity.MongoDB
```
    
3. In ~/Models/IdentityModels.cs:
    * Remove the namespace: Microsoft.AspNet.Identity.EntityFramework
    * Add the namespace: AspNet.Identity.MongoDB
	* Remove the ApplicationDbContext class completely.
4. In ~/Controllers/AccountController.cs
    * Remove the namespace: Microsoft.AspNet.Identity.EntityFramework
    * Add the connection string name to the constructor of the UserStore. Or empty constructor will use DefaultConnection

```C#
public AccountController()
{
	this.UserManager = new UserManager<ApplicationUser>(
					new UserStore<ApplicationUser>("Mongo")
				) as ApplicationUserManager;
}
```

If you expect your user/role stores to contain lots of users, you should make sure the proper indexes are in place on the MongoD collections. On application start you can call these 2 initialize methods, and the indexes will be added if they're missing.

```C#
public Application_Start()
{
        UserStore<ApplicationUser>.Initialize("Mongo");
        RoleStore<ApplicationUser>.Initialize("Mongo");
}
```

Or you can create the indexes yourself, directly in the MongoDb console. Here an example where the user collection is named 'user' and the role collection is named 'role'.

```Javascript
db.user.createIndex({ "lun": 1 }, { "unique": true })
db.user.createIndex({ "lem": 1 })
db.role.createIndex({ "ln": 1 }, { "unique": true })
```

## Connection Strings ##
The UserStore has multiple constructors for handling connection strings. Here are some examples of the expected inputs and where the connection string should be located.

### 1. SQL Style ###
```C#
UserStore(string connectionNameOrUrl)
```
<code>UserStore("Mongo")</code>

**web.config**
```xml
<add name="Mongo" connectionString="Server=localhost:27017;Database={YourDataBase}" />
```

### 2. Mongo Style ###
```C#
UserStore(string connectionNameOrUrl)
```
<code>UserStore("Mongo")</code>

**web.config**
```xml
<add name="Mongo" connectionString="mongodb://localhost/{YourDataBase}" />
```

**OR**

```C#
UserStore(string connectionNameOrUrl)
```
<code>UserStore("mongodb://localhost/{YourDataBase}")</code>
