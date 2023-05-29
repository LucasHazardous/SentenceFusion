# Exploring MongoDB App Services

> Warning! This tutorial will involve serverless functions, Atlas Realm and making an exciting project.

> For now basic MongoDB knowledge is assumed. The tutorial will be updated to be more beginner friendly later.

---

## Introduction

* ## What are serverless functions?

    Serverless functions, also known as Function as a Service (FaaS), are a type of cloud computing service that allows developers to run and execute code without the need to manage or provision servers. In a traditional server-based model, developers have to set up and manage servers to host their applications and handle incoming requests. With serverless functions, developers can focus solely on writing and deploying code without worrying about the underlying infrastructure.

    Benefits:

    - **No infrastructure necessary** - in a serverless architecture, the cloud provider takes care of managing the servers
    - **Pay for exact usage** - billing is typically based on the actual execution time and resource usage
    - **Autoscaling** - the serverless platform automatically scales up the necessary resources to handle the incoming request
    - **Multiple triggers** - functions are triggered by events, such as HTTP requests, database updates, file uploads, timers, or messages from queues

* ## What is MongoDB Atlas Realm?

    MongoDB Atlas Realm is a cloud-based serverless platform provided by MongoDB. It is designed to simplify the development and deployment of real-time applications that require data synchronization and event-driven workflows.

    Key features:

    - Real-Time Data Sync
    - Authentication and User Management
    - Serverless Functions
    - Integration with MongoDB Atlas
    - GraphQL API
    - Multiple SDKs

---

## Project

### Goal

Create an application that will allow user creation. Created users will be able to participate in writing a story. There will be one rule: a single user can't write two sentences in a row - that ensures that at least two people will be writing a story. Later we will also create a representative website of this project that will showcase the first sentence, which will be shared through a public endpoint.

### Setup

Login to MongoDB Atlas and create a new project. When creating a new free-tier cluster: choose a nearby location, create a new user and **set access to anywhere (0.0.0.0/0)**. Inside the cluster create a database named ***fusion*** with a collection named ***sentence***.

### Configuring a Realm App

We will begin by adding user authentication configuration. 

![authentication](./img/authentication.png)

![authentication configuration](./img/auth_conf.png)

![reset password function](./img/reset_func.png)

![enabled authentication](./img/enabled_auth.png)

For now users won't be able to reset their passwords but a reset function must exist. We will go with the default function, without any changes.

When you are done click *Save draft* button at the bottom. When you do changes won't be instantly deployed until you click *Review draft & deploy* button, then you get a chance to review all changes and deploy them.

Let's create a function that we will use in a rule later. This function will be responsible for determining if a user can insert a new document.

Enter function list view from the left side panel. You should see resetFunc created earlier. Click *Create New Function*.

![function list](./img/funcs.png)

In the *Settings* tab begin configuration:

![function configuration](./img/func_conf.png)

Authentication **System** will allow the function to execute with full priviliges and setting it to private will make it unreachable for regular users. We configure it that way because the only user of this function will be our rule.

Now set head to *Function Editor* and write the following code:

```js
exports = async function(arg){
  const mongodb = context.services.get("mongodb-atlas");
  const db = mongodb.db("fusion");
  const coll = db.collection("sentence");
  
  const latestDocument = await coll.find().sort({_id: -1}).limit(1).toArray();
  if(latestDocument.length == 0) return true;

  return latestDocument[0].userId != arg;
};
```

