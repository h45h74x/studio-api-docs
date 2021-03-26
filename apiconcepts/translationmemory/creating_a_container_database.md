Creating a Container Database
====
In this chapter you will learn how to programmatically create a container on a database server for storing translation memories.

Add a New Class
---
Start by adding a new class called `ServerContainer`. Then implement a public Create function, which takes a `TranslationProviderServer` object as parameter. The function should look as shown below:

```
public void Create(TranslationProviderServer tmServer)
{
    TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);

    DatabaseServer dbServ = tmServer.GetDatabaseServer("DB01", this.GetDbServProperties());
    container.DatabaseServer = dbServ;
    container.DatabaseName = "DbName";
    container.Name = "NiceName";
    container.Save();
}
```
First, we create the database server object from the TM Server. For this, the `GetDatabaseServer` method is applied, which takes the (friendly) database server name and the database server properties as parameters. The database server properties are returned by function that is shown below. Note that this function has been simplified for our sample implementation. It actually returns only an empty property, however, we need it to provide the required parameter for the `GetDatabaseServer` method.

```
private DatabaseServerProperties GetDbServProperties()
{
    DatabaseServerProperties props = new DatabaseServerProperties();

    return props;
}
```
When creating the container we specify the
* database server object
* physical database name (note that this name is subject to DB naming conventions)
* friendly name for the container database
Afterwards, we apply the Save method, which creates the actual container database. The complete function should look as shown below:

```
public void Create(TranslationProviderServer tmServer)
{
    TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);

    DatabaseServer dbServ = tmServer.GetDatabaseServer("DB01", this.GetDbServProperties());
    container.DatabaseServer = dbServ;
    container.DatabaseName = "DbName";
    container.Name = "NiceName";
    container.Save();
}
```

Enhance the Code Example
---
In the next step we would like to show to you how to enhance the function for creating containers. Add a new function called CreateAdvanced to your class, which takes a TM Server object and the container database name as parameters. The aim is to make the function for creating the container more robust by first checking whether the TM Server has any database servers registered in the first place. To do this determine the number of available database servers. If the count equals zero, an error message should be thrown:

```
ReadOnlyCollection<DatabaseServer> dbs = tmServer.GetDatabaseServers(DatabaseServerProperties.Containers);
if (dbs.Count == 0)
{
    throw new Exception("No DB server registered.");
}
```

Afterwards, check whether a container with the name that we want to give already exists on the first available database server. For simplicity reasons we are just checking the first database server, as we can assume that most TM Server setups bank on only a single DB server.

```
foreach (TranslationMemoryContainer item in dbs[0].Containers)
{
    if (item.Name == newContainerName)
    {
        throw new Exception("Container with that name already exists.");
    }
}
```

If at least one database server is available, and a container by that name does not exist yet, we create the container on the database server:

```
TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);
container.DatabaseServer = dbs[0];
container.DatabaseName = newContainerName + "DB";
container.Name = newContainerName;
container.ParentResourceGroupPath = organization;
container.Save();
```

Last, check whether the container was actually created by applying the Contains method to the Containers collection. If the container object was not found on the first available database server, an error message should be thrown:

```
if (!dbs[0].Containers.Contains(container))
{
    throw new Exception("Container was not created.");
}
```
The complete function should look as shown below:

```
public void CreateAdvanced(TranslationProviderServer tmServer, string organization, string newContainerName)
{
    #region "count"
    ReadOnlyCollection<DatabaseServer> dbs = tmServer.GetDatabaseServers(DatabaseServerProperties.Containers);
    if (dbs.Count == 0)
    {
        throw new Exception("No DB server registered.");
    }
    #endregion

    #region "CheckExists"
    foreach (TranslationMemoryContainer item in dbs[0].Containers)
    {
        if (item.Name == newContainerName)
        {
            throw new Exception("Container with that name already exists.");
        }
    }
    #endregion

    #region "CreateContainer"
    TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);
    container.DatabaseServer = dbs[0];
    container.DatabaseName = newContainerName + "DB";
    container.Name = newContainerName;
    container.ParentResourceGroupPath = organization;
    container.Save();
    #endregion

    #region "CheckCreated"
    if (!dbs[0].Containers.Contains(container))
    {
        throw new Exception("Container was not created.");
    }
    #endregion
}
```
Delete a Container
----
The short sample function below demonstrates how to remove a container database from the TM Server system by applying the Delete method. This method takes a boolean parameter to indicate whether the actual physical container database should be deleted (True), or whether the database should only be unregistered from the system (False). Note that deleting the physical database represents a risk, as it may contain a lot of valuable data. Therefore, unregistering the database is the safer option, as it can be re-registered later. This method should only be applied when you are absolutely certain that the database content is no longer needed. Note that the delete operation can only be performed by a user who has the required credentials.

```
public void DeleteContainer(TranslationProviderServer tmServer, string organization, string containerName)
{
    if (!organization.EndsWith("/")) 
        organization += "/";
    TranslationMemoryContainer container = tmServer.GetContainer(organization + containerName, ContainerProperties.None);
    container.Delete(false);
}
```

Putting it All Together
----

The complete class should now look as shown below:

```
namespace Sdl.SDK.LanguagePlatform.Samples.TmAutomation
{
    using System;
    using System.Collections.ObjectModel;
    using Sdl.LanguagePlatform.TranslationMemoryApi;

    public class ServerContainer
    {
        #region "CreateSimple"
        public void Create(TranslationProviderServer tmServer)
        {
            TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);

            DatabaseServer dbServ = tmServer.GetDatabaseServer("DB01", this.GetDbServProperties());
            container.DatabaseServer = dbServ;
            container.DatabaseName = "DbName";
            container.Name = "NiceName";
            container.Save();
        }
        #endregion

        #region "props"
        private DatabaseServerProperties GetDbServProperties()
        {
            DatabaseServerProperties props = new DatabaseServerProperties();

            return props;
        }
        #endregion

        #region "CreateAdvanced"
        public void CreateAdvanced(TranslationProviderServer tmServer, string organization, string newContainerName)
        {
            #region "count"
            ReadOnlyCollection<DatabaseServer> dbs = tmServer.GetDatabaseServers(DatabaseServerProperties.Containers);
            if (dbs.Count == 0)
            {
                throw new Exception("No DB server registered.");
            }
            #endregion

            #region "CheckExists"
            foreach (TranslationMemoryContainer item in dbs[0].Containers)
            {
                if (item.Name == newContainerName)
                {
                    throw new Exception("Container with that name already exists.");
                }
            }
            #endregion

            #region "CreateContainer"
            TranslationMemoryContainer container = new TranslationMemoryContainer(tmServer);
            container.DatabaseServer = dbs[0];
            container.DatabaseName = newContainerName + "DB";
            container.Name = newContainerName;
            container.ParentResourceGroupPath = organization;
            container.Save();
            #endregion

            #region "CheckCreated"
            if (!dbs[0].Containers.Contains(container))
            {
                throw new Exception("Container was not created.");
            }
            #endregion
        }
        #endregion

        #region "DeleteContainer"
        public void DeleteContainer(TranslationProviderServer tmServer, string organization, string containerName)
        {
            if (!organization.EndsWith("/")) 
                organization += "/";
            TranslationMemoryContainer container = tmServer.GetContainer(organization + containerName, ContainerProperties.None);
            container.Delete(false);
        }
        #endregion
    }
}
```