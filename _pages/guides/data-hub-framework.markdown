---
layout: inner
title: Grove and DHF
lead_text: ''
permalink: /guides/data-hub-framework/
---

We often want to run Grove as a UI working alongside a [MarkLogic Data Hub Framework (DHF)](https://marklogic.github.io/marklogic-data-hub/). This page demonstrates how to do that.

# Recommended Method to Run Grove with DHF

In order to run a Grove UI project alongside the MarkLogic Data Hub Framework (DHF) project, we recommend creating a Grove project-specific app-server, which has its own modules database but points at the content database with the data you wish to visualize: most often the DHF project's FINAL database. The DHF project will be responsible for managing the content database (including setting up indexes - needed for facets - and security permissions), as well as the content database's related triggers and schemas databases, while the Grove project will manage the Grove project's specific app-server and modules database.

Note that this implies some coordination between the DHF project and Grove. If possible, it is easiest to manage both projects within the same code repository. The Grove files should all be within a dedicated parent directory. (For example, `grove/ui`, `grove/middle-tier`, and `grove/marklogic`. Note that there is nothing magic about the `grove` directory name, so call it whatever you want.)

If you need more independence than that provided by this recommended approach, you may consider replicating the DHF data to a Grove-specific content database - but that is currently beyond the scope of this guide.

You can accomplish this recommended approach by making some changes inside the `marklogic` directory of your Grove project. Grove projects generated using the grove-cli's `grove new` command ship with this `marklogic` directory, which contains configuration files for a standalone MarkLogic database, optimized for Grove's sample data set.

The advantages to this approach include:

- It does not pollute the DHF project's modules database.
- It creates a line between configuration to support the Data Hub project and configuration to support the Grove UI project (with some exceptions, described in the next section on downsides). For example, the Grove project can set up its own users, roles, and security permissions (though those roles need to be consistent with doc permissions set in DHF).

There are some downsides to this approach:

- You will have two different ml-gradle installations, one for the Grove project and one for the DHF project, which can be confusing and time-consuming, because you have to run gradle tasks in two places, for example when bootstrapping the project. You could mitigate this by creating scripts that automatically run gradle scripts in both places.
- You will have to add some configuration to the DHF project in order to support the Grove UI. For example, the DHF project may need to add new indexes in order to support facets for the Grove UI. Content permissions will need to correspond to Grove users and roles. And if triggers and schemas are desired for the Grove project, those will have to be set up in the DHF ml-gradle configuration. (Note that this kind of demand will come from any downstream system connecting to a DHF content database.)
- Any CRUD operations performed by the Grove project will write to the DHF project's content database.

## How to implement the recommended approach, step-by-step:

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

6. Run `./gradlew mlDeploy` from inside the `marklogic` directory. This command will deploy the new configuration, but it will not change the DHF project's content database, because you have removed all related configuration files.

## Less-Recommended Option: Remove Grove-specific Config Altogether

The Grove middle-tier and front-end just need a MarkLogic REST server to work against. So, you can remove the Grove project's ml-gradle configuration altogether and just use an app-server provided by the DHF project.

As with the recommended option, there will be dependencies on configuration provided by the DHF project, so keeping both in the same code repository has some advantages for mutual versioning.

This approach in some ways simplifies things because all MarkLogic management is handled in one place: in the DHF project. The Grove project is clearly just a downstream consumer.

A downside is that you will have to add Grove-specific modules, rest extensions, and named search options to the DHF project's modules database. 

Steps:

1. Determine which host and port the DHF FINAL app server is running on.
2. Go to top level of your Grove project (the directory with `middle-tier` and `ui` directories inside it), run `grove config` and give it the host and port of the DHF FINAL app server.
3. Use a set of named search options provided by the DHF project. Do this by editing `middle-tier/routes/api/index.js`: Find the mounted search route, around lines 48-58, and change the namedOptions to `'final-entity-options'` (or any other named options available in your FINAL database module). Note that you need to include quotes around `'final-entity-options'`. (Note that you can go to {ML-HOST}:{FINAL-REST-PORT}/v1/config/query to see what named options are available.)
4. Migrate modules you are using to the DHF project. (These are stored in `marklogic/ml-modules`.)
5. Run `./gradlew mlUndeploy` from `marklogic` if you ran `mlDeploy` previously, to get rid of Grove-specific artifacts.
6. remove the entire "marklogic/" directory.
