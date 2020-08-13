---
layout: inner
title: How Does it Work?
lead_text: ''
permalink: /how-does-it-work/
---

# How Does it Work?

## MarkLogic
Starting from the bottom of the stack, Grove uses [grove-ml-gradle](https://github.com/marklogic-community/grove-ml-gradle), to bootstrap and deploy vanilla MarkLogic-related configuration and artifacts.  It is nearly an out-of-the-box [ml-gradle](https://github.com/marklogic-community/ml-gradle) project but does add configurations for a different paths to specify the location of the [modules, schemas, and data](https://github.com/marklogic-community/grove-ml-gradle/blob/master/gradle.properties#L22). This has been designed to complement the Data Hub, but does not deploy one.  The search options used by the project are located in [src/main/ui-modules/options](https://github.com/marklogic-community/grove-ml-gradle/tree/master/src/main/ui-modules/options) where others may be deployed as well.

## Middle Tier
The middle tier is implemented with [grove-node](https://github.com/marklogic-community/grove-node), but can be swapped out for [grove-spring-boot](https://github.com/marklogic-community/grove-spring-boot) if Java is more desirable.  Simply replace the contents of the middle-tier directory with a clone of grove-spring-boot to do this switcheroo.  The grove-node project has rich documentation inside the project written in Markdown.  Instructions on code customizations and details on how things work reside there.


## UI
Lastly, the UI's presentation is powered by either [React](https://github.com/marklogic-community/grove-react-ui) or [Vue](https://github.com/marklogic-community/grove-vue-ui).  State management is handled by either Redux (React) or Veux (Vue).  It is important for each to use proper state management to interact with the middle tier.  

For React, UI customizations are done in [React components](https://github.com/marklogic-community/grove-react-ui/tree/master/src/components).  A component is "hooked up" to the Redux via a [container](https://github.com/marklogic-community/grove-react-ui/tree/master/src/containers).  The container that makes performs actions with the [current state](https://github.com/marklogic-community/grove-react-ui/tree/master/src/redux) of the application.  A similar pattern exists with Vue, though it is much lighter weight.