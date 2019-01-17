---
layout: inner
title: Getting Started with Grove
lead_text: ''
permalink: /guides/getting-started/
---

This guide demonstrates how to create a new Grove project.

# Quick Creation of a new Grove project:

The commands listed below should be typed into a terminal/console window.

## 1. Check Prerequisites

  - **MarkLogic**: If you have not yet done so, [install MarkLogic 9](https://developer.marklogic.com/products), start it, and initialize it at `localhost:8001`.
  - **Node.js**: Check if you have Node.js version 8.10.0 or above installed by running `node -v`. If not, [install Node.js](https://nodejs.org/).
  - **npm**: Check if you have npm version 5.7.0 or above installed by running `npm -v`. If not, run `npm install -g npm` to get the latest.
  - **Git**: The grove-cli requires git. Check if you have git installed by running `git --version`. If not, [install Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
  - **Java**: (If you are an advanced user and know that you do not plan to use the included ml-gradle deployer, you can skip the Java requirement.) Check if you have Java 1.8 or above installed by running `java -version`. If not, [install Java 1.8](https://www.java.com/en/download/help/download_options.xml). Java is required because the generated Grove project will use [ml-gradle](https://github.com/marklogic-community/ml-gradle) to administer MarkLogic and [mlcp](https://developer.marklogic.com/products/mlcp) to load data.

## 2. Install the grove-cli

The [grove-cli](https://project.marklogic.com/repo/users/pmcelwee/repos/grove-cli/browse) is the best way to install and configure new Grove Projects. Run the following to install the grove-cli:

```bash
npm install -g @marklogic-community/grove-cli
```

This makes the `grove` command available on your command-line.

## 3. Generate a new Project:

```bash
grove new your-project-name
```

Answer the questions in the prompts in order to select the React or Vue UI, as desired.

```bash
cd your-project-name
```

## 4. Configure your project:

```bash
grove config
```

This command will update settings in your React or Vue UI, Node middle-tier, and the provided ml-gradle project. It is helpful to use the grove-cli for this task, so the configuration is the same for all parts of your project.

## 5. OPTIONAL: Provision MarkLogic

We provide an ml-gradle configuration that matches your Grove settings as a convenience in the `marklogic` directory of your generated Project. You can also just point Grove at an existing MarkLogic app server using the `grove config` command. If you are running alongside the DataHub Framework, take a look at the notes in [Grove and Data Hub Framework](/grove/guides/data-hub-framework/).

```bash
cd marklogic
./gradlew mlDeploy
cd ..
```

Or, on Windows:

```bash
cd marklogic
gradlew.bat mlDeploy
cd ..
```

## 6. Install npm dependencies

Run from the top-level of your generated Project:

```bash
npm install
```

## 7. Start development servers to run your Grove Project

Run from the top-level of your generated Project:

```bash
npm start
```

## 8. Optional: Load sample data

This command uses [ml-gradle](https://github.com/marklogic-community/ml-gradle) and [mlcp](https://developer.marklogic.com/products/mlcp) to load 3000 sample json documents, about people. These documents are stored in the `marklogic/data` directory of this source code.

This Template includes ml-gradle in the `marklogic` directory. You must be in that directory to invoke ml-gradle commands.

```bash
cd marklogic
./gradlew loadSampleData
cd ..
```

Or, on Windows:

```bash
cd marklogic
gradlew.bat loadSampleData
cd ..
```

## Next Steps

If you have created a React UI, see the [React Developers Guide](/grove/guides/react/) for information.

# VERY Quick Start

This sections leverages the grove demo command, which is designed for limited use cases. In most cases, it is not appropriate for normal Project development. But, if all prerequisites are in place, it will quickly start a running Grove Project loaded with sample data.

First, start MarkLogic. Then, the following grove-cli command will clone the application source code, create a content database, create a modules database, create a MarkLogic REST server, load sample people data, install the Web app dependencies locally, and start your application.

```bash
grove demo your-demo-name
```

The grove-cli will walk you through the creation of a running Grove project, based on the code in this repository.
