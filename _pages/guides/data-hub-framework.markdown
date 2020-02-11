---
layout: inner
title: Grove and the Data Hub
lead_text: ''
permalink: /guides/data-hub-framework/
---

# Grove + MarkLogic Data Hub

We often want to run Grove as a UI working alongside a [MarkLogic Data Hub](https://marklogic.github.io/marklogic-data-hub/). This guide provides the mechanics on how to do this, and provides a choice of method.

This guide has been vetted on both Data Hub 5.x and the Data Hub Framework 4.3.1. 

## Migrating Grove-related artifacts and code to the Data Hub

In order to run a Grove UI project alongside a MarkLogic Data Hub, we recommend migrating Grove related artifacts and code from its code base into your Data Hub project.  This approach was tested against both the Data Hub and the Data Hub Service.

Note that this implies some coordination between the Data Hub project and Grove even after migration of Grove's core capabilities. If possible, it is easiest to manage both projects within the same code repository. The Grove files should all be within a dedicated parent directory. (For example, `grove/ui` and `grove/middle-tier`. Note that there is nothing magic about the `grove` directory name, so call it whatever you want.)

You can accomplish this recommended approach by moving a subset of files from inside the `marklogic` directory of your Grove project into your Data Hub project. Grove projects generated using the grove-cli's `grove new` command ship with this `marklogic` directory, which contains configuration files for a standalone MarkLogic database, optimized for Grove's sample data set.  Once the relevant files are migrated, the `marklogic` directory can be deleted entirely.  Additionally, an environment variable will need to be updated to reflect the data-hub-FINAL HTTP app server as configured by the Data Hub project.

The advantages of this approach include: 
- Combining the Grove-related artifacts with the Data Hub simplifies development and deployments.  The Grove code is logically separated inside its own directory under `data-hub/src/main/ui-modules` for clarity and distinction from Data Hub specific modules.
- If code is maintained together in the same repository, consistent versioning can be ensured.
- The Grove UI becomes a downstream consumer of the Data Hub.

There are some disadvantages to this approach:
- UI specific code will reside in close proximity to backend specific configurations.  This might not be as preferable to some development teams.
- Any CRUD operations performed by the Grove project will write to the Data Hub's FINAL database.
- The security approach for Data Hub differs from that of Grove.  It is best to drop that direction and use MarkLogic's guidance for the Data Hub.

### How to implement the recommended approach, step-by-step:

Prerequisite: MarkLogic Data Hub has been deployed to your MarkLogic host.  

[Optional] If you previously installed Grove to its default database via the `mlDeploy` command within the `marklogic` directory, remove the Grove-specific configs before proceeding:
        
    > cd grove/marklogic
    > ./gradlew mlUndeploy -Pconfirm=true
    

### Steps to move Grove modules to the Data Hub

1. Determine the MarkLogic host and port the data-hub-FINAL app server is running on.
2. At the command line, from within the top level of your Grove project, (the directory with `middle-tier` and `ui` directories inside it), run `grove config` and enter the host and port of the data-hub-FINAL app server.  This will update environment variables for your project.
3. Create the `ui-modules` directory inside the Data Hub project: `data-hub/src/main/ui-modules`.
4. Update the Data Hub's `gradle.properties` to register this new location for MarkLogic module deployments.  Add the following line to `gradle.properes`:

    ```
    mlModulePaths=src/main/ml-modules,src/main/ui-modules
    ```

<<<<<<< Updated upstream
5. __Only necessary for Data Hub 5.0.1__ Update the Data Hub's `build.gradle` to use a newer version of ml-gradle.  Add the `dependencies` to the `buildscript` object around line 5: 
=======
5. If you are using a version of the Data Hub older than 5.0.1, update the Data Hub's `build.gradle` to use a newer version of ml-gradle (3.14.0 or higher).  Add the `dependencies` to the `buildscript` object around line 5: 
>>>>>>> Stashed changes

    ```JSON
    dependencies {
		  classpath "gradle.plugin.com.marklogic:ml-gradle:3.14.0"
    }
    ```

6. Copy the contents of the Grove modules to the Data Hub project. Copy 
    ```
    grove/marklogic/ml-modules/options
    grove/marklogic/ml-modules/root 
    grove/marklogic/ml-modules/services
    grove/marklogic/ml-modules/transforms
    
    to 
    
    data-hub/src/main/ui-modules
    ```

7. Copy the contents of the Grove default user profile and a dictionary document to the Data Hub project. Copy the files in 
    ```
    grove/marklogic/ml-content
    
    to 
    
    data-hub/src/main/ui-data
    ```

To ensure the deployment process deploys these files, also add the folllowing line to your `gradle.properies` file:
  
    mlDataPaths=src/main/ml-data,src/main/ui-data


8. Edit the query options used by your Grove middle-tier's search route (by default, these query options are called `all` and are found in the `data-hub/ui-modules/options/all.xml` file) to remove the following blocks:

    ```xml
    <constraint name="eyecolor">
      ...
    </constraint>
    <constraint name="docFormat">
      ...
    </constraint>
    <constraint name="gender">
      ...
    </constraint>
    ```

9. Make any other necessary changes to the query options file. For example, you may need to update the `<additional-query>` specifying that only docs in the `data` collection are returned.  There may be a Data Hub specific collections that are a natural fit to limit the search results for your Grove application.  For example, 
    ```xml
    <cts:collection-query>
      <cts:uri>Entity1</cts:uri>
      <cts:uri>Entity2</cts:uri>
       .....
    </cts:collection-query>
    ```

10. Run `./gradlew mlLoadModules` from inside the `data-hub` directory. This command will deploy the new modules and supporting documents.  

11. Delete the contents of `grove/marklogic`. 


## Alternative Approach to Run Grove with the Data Hub

In order to run a Grove UI project alongside the MarkLogic Data Hub project, we alternatively suggest creating a Grove project-specific app-server, which has its own modules database but points at the content database with the data you wish to visualize: most often the Data Hub project's FINAL database. The Data Hub project will be responsible for managing the content database (including setting up indexes - needed for facets - and security permissions), as well as the content database's related triggers and schemas databases, while the Grove project will manage the Grove project's specific app-server and modules database.  Security will be a shared responsibility, where the Data Hub has to give low-level access to data, and Grove can arrange higher-level access.

Note that this implies some coordination between the Data Hub project and Grove. If possible, it is easiest to manage both projects within the same code repository. The Grove files should all be within a dedicated parent directory. (For example, `grove/ui`, `grove/middle-tier`, and `grove/marklogic`. Note that there is nothing magic about the `grove` directory name, so call it whatever you want.)

If you need more independence than that provided by this approach, you may consider replicating the Data Hub data to a Grove-specific content database - but that is currently beyond the scope of this guide.

You can accomplish this approach by making some changes inside the `marklogic` directory of your Grove project. Grove projects generated using the grove-cli's `grove new` command ship with this `marklogic` directory, which contains configuration files for a standalone MarkLogic database, optimized for Grove's sample data set.

The advantages to this approach include:

- It does not pollute the Data Hub project's modules database.
- It creates a line between configuration to support the Data Hub project and configuration to support the Grove UI project (with some exceptions, described in the next section on downsides). For example, the Grove project can set up its own users, roles, and security permissions (though those roles need to be consistent with doc permissions set in Data Hub).

There are some downsides to this approach:

- You will have two different ml-gradle installations, one for the Grove project and one for the Data Hub project, which can be confusing and time-consuming, because you have to run gradle tasks in two places, for example when bootstrapping the project. You could mitigate this by creating scripts that automatically run gradle scripts in both places.
- You will have to add some configuration to the Data Hub project in order to support the Grove UI. For example, the Data Hub project may need to add new indexes in order to support facets for the Grove UI. Content permissions will need to correspond to Grove users and roles. And if triggers and schemas are desired for the Grove project, those will have to be set up in the Data Hub ml-gradle configuration. (Note that this kind of demand will come from any downstream system connecting to a Data Hub content database.)
- Any CRUD operations performed by the Grove project will write to the Data Hub project's content database.

### How to implement the alternative approach, step-by-step:

1. Run `grove config` at the top-level of your Grove project to ensure that, for example, an available port is specified.

2. Delete `content-database.json`, `schemas-database.json`, and `triggers-database.json` from `marklogic/ml-config/databases/`.

3. Edit `marklogic/ml-config/servers/app-server` and `marklogic/ml-config/rest-api.json` to point to the correct content-database name.

4. Edit the search options used by your Grove middle-tier's search route (by default, these search options are called `all` and are found in the `marklogic/ml-modules/options/all.xml` file) to remove the following blocks:

    ```xml
    <constraint name="eyecolor">
      ...
    </constraint>
    <constraint name="docFormat">
      ...
    </constraint>
    <constraint name="gender">
      ...
    </constraint>
    ```

5. Make any other necessary changes to the search options file. For example, you may need to remove the `<additional-query>` specifying that only docs in the `data` collection are returned.

6. Remove vestiges of the permissions being set on the now OBE schemas database.
  * Remove the entire `setSchemaPermissions` task beginning on line 42
  * Remove line 56: `mlLoadSchemas.finalizedBy setSchemasPermissions`
  * Remove line 58: `mlDeploy.finalizedBy setSchemasPermissions`


7. Run `./gradlew mlDeploy` from inside the `marklogic` directory. This command will deploy the new configuration, but it will not change the Data Hub project's content database, because you have removed all related configuration files.

