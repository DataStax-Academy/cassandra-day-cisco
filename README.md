## 🎓🔥 Cassandra Datastax Cloud Day @Cisco 🔥🎓

[![License Apache2](https://img.shields.io/hexpm/l/plug.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Discord](https://img.shields.io/discord/685554030159593522)](https://discord.com/widget?id=685554030159593522&theme=dark)

![image](pics/splash.png?raw=true)

This intructions will lead you to step by step operations for the Cassandra Day at Cisco.

## Table of content

1. [Create Astra Instance](#1-create-astra-instance)
2. [Working with Cassandra](#2-working-with-cassandra)
3. [Working with REST API](#3-working-with-rest-api)
4. [Working with DOCUMENT API](#4-working-with-document-api)
5. [Working with GRAPHQL API](#5-working-with-graphql-api)
6. [Getting Started with SAI](#6-getting-started-with-sai-storage-attached-index)

## 1. Create Astra Instance

**`ASTRA`** is the simplest way to run Cassandra with zero operations at all - just push the button and get your cluster. No credit card required, $25.00 USD credit every month, roughly 5M writes, 30M reads, 40GB storage monthly - sufficient to run small production workloads.

✅ **Step 1a** Register (if needed) and Sign In to Astra [https://astra.datastax.com](https://dtsx.io/workshop): You can use your `Github`, `Google` accounts or register with an `email`.

_Make sure to chose a password with minimum 8 characters, containing upper and lowercase letters, at least one number and special character_

✅ **Step 1b** Create a "free tier" plan

Follow this [guide](https://docs.datastax.com/en/astra/docs/creating-your-astra-database.html), to set up a pay as you go database with a free $25 monthly credit.

- **Select the pay as you go option**: Includes $25 monthly credit - no credit card needed to set up.

You will find below which values to enter for each field.

- **For the database name** - `free_db.` While Astra allows you to fill in these fields with values of your own choosing, please follow our recommendations to ensure the application runs properly.

- **For the keyspace name** - `keyspace1`. It's really important that you use the name "free" for the code to work.

_You can technically use whatever you want and update the code to reflect the keyspace. This is really to get you on a happy path for the first run._

- **For provider and region**: Choose and provider (either GCP or AWS). Region is where your database will reside physically (choose one close to you or your users).

- **Create the database**. Review all the fields to make sure they are as shown, and click the `Create Database` button.

You will see your new database `pending` in the Dashboard.

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/dashboard-pending-1000-update.png?raw=true)

The status will change to `Active` when the database is ready, this will only take 2-3 minutes. You will also receive an email when it is ready.

## 2. Working with Cassandra

**✅ 2a Check that our keyspace exist**

```sql
describe keyspaces;
```

**✅ 2b Create Entities**

```sql
use keyspace1;

CREATE TYPE IF NOT EXISTS video_format (
  width   int,
  height  int
);

CREATE TABLE IF NOT EXISTS videos (
 videoid   uuid,
 title     text,
 upload    timestamp,
 email     text,
 url       text,
 tags      set <text>,
 frames    list<int>,
 formats   map <text,frozen<video_format>>,
 PRIMARY KEY (videoid)
);

describe keyspace1;
```

**✅ 2c Use the data model** :

- Insert value using plain CQL

```sql
INSERT INTO videos(videoid, email, title, upload, url, tags, frames, formats)
VALUES(uuid(), 'clu@sample.com', 'sample video',
     toTimeStamp(now()), 'http://google.fr',
     { 'cassandra','accelerate','2020'},
     [ 1, 2, 3, 4],
     { 'mp4':{width:1,height:1},'ogg':{width:1,height:1}});

INSERT INTO videos(videoid, email, title, upload, url)
VALUES(uuid(), 'clu@sample.com', 'video2', toTimeStamp(now()), 'http://google.fr');
```

- Insert Value using JSON

```sql
INSERT INTO videos JSON '{
   "videoid":"e466f561-4ea4-4eb7-8dcc-126e0fbfd573",
     "email":"clunven@sample.com",
     "title":"A Second videos",
     "upload":"2020-02-26 15:09:22 +00:00",
     "url": "http://google.fr",
     "frames": [1,2,3,4],
     "tags":   [ "cassandra","accelerate", "2020"],
     "formats": {
        "mp4": {"width":1,"height":1},
        "ogg": {"width":1,"height":1}
     }
}';
```

**✅ 2d Select Values** :

- Read values

```sql
select * from videos;
```

- Read by id

```sql
select * from videos where videoid=e466f561-4ea4-4eb7-8dcc-126e0fbfd573;
```

[🏠 Back to Table of Contents](#table-of-content)

## 3. Working with REST API

To use the API we will need a token please create a token following the instructions here

✅ **Step3a** [Create a token for your app](https://docs.datastax.com/en/astra/docs/manage-application-tokens.html) to use in the settings screen

Copy the token value (eg `AstraCS:KDfdKeNREyWQvDpDrBqwBsUB:ec80667c....`) in your clipboard and save the CSV this value would not be provided afterward.

**👁️ Expected output**
![image](pics/astra-token.png?raw=true)

Now launch the swagger UI
![image](pics/launch-swagger.png?raw=true)

```diff
+ Info
+ --------------------
+ During this hands-on we will use the API in version 2
```

**✅ Step 3b. Create a new keyspaces** :

On the dashboard locate the `Add Keyspace` button in the bottom, on the tab enter the value `keyspace2`.

![image](pics/add-keyspace.png?raw=true)

**✅ Step 3c. List keyspaces** :

Locate the `SCHEMAS` part of the API and then `[GET] /v2/schemas/keyspaces`

![image](pics/swagger-schemas.png?raw=true)

- Click `Try it out`
- Provide your token in the field `X-Cassandra-Token` _(looks like AstraCS:...)_
- Click on `Execute`

**✅ Step 3d. Creating a Table** :

- [POST] ​/v2/schemas/keyspaces/{keyspaceName}/tables
- X-Cassandra-Token: `<your_token>`
- keyspace: `keyspace2`
- Data

```json
{
  "name": "users",
  "columnDefinitions": [
    {
      "name": "firstname",
      "typeDefinition": "text"
    },
    {
      "name": "lastname",
      "typeDefinition": "text"
    },
    {
      "name": "email",
      "typeDefinition": "text"
    },
    {
      "name": "color",
      "typeDefinition": "text"
    }
  ],
  "primaryKey": {
    "partitionKey": ["firstname"],
    "clusteringKey": ["lastname"]
  },
  "tableOptions": {
    "defaultTimeToLive": 0,
    "clusteringExpression": [{ "column": "lastname", "order": "ASC" }]
  }
}
```

Now Locate the `DATA` part of the API

**👁️ Expected output**
![image](pics/astra-data.png?raw=true)

**✅ Step 3e. Insert a row** :

- [createRow]
- X-Cassandra-Token: `<your_token>`
- keyspace: `keyspace2`
- table: `users`
- Data

```json
{
  "columns": [
    { "name": "firstname", "value": "Mookie" },
    { "name": "lastname", "value": "Betts" },
    { "name": "email", "value": "mookie.betts@gmail.com" },
    { "name": "color", "value": "blue" }
  ]
}
```

**✅ Step 3f. Read data** :

- [getAllRows]
- X-Cassandra-Token: `<your_token>`
- keyspace: `keyspace2`
- table: `users`

[🏠 Back to Table of Contents](#table-of-content)

## 4. Working with DOCUMENT API

This walkthrough has been realized using the [Quick Start](https://stargate.io/docs/stargate/1.0/quickstart/quick_start-document.html)

locate the Document part in the Swagger UI

![image](pics/swagger-docs.png?raw=true)

**✅ Creating a namespace** :

- Access [createNamespace]
- Fill with Header `X-Cassandra-Token` with `<your_token>`
- Use this payload as JSON

```json
{ "name": "namespace1", "replicas": 3 }
```

**✅ Checking namespace existence** :

- Access [getAllNamespaces]
- Fill with Header `X-Cassandra-Token` with `<your_token>`
- For `raw` you can use either `true` or `false`

**👁️ Expected output**

```json
{
  "data": [
    { "name": "system_distributed" },
    { "name": "system" },
    { "name": "data_endpoint_auth" },
    { "name": "keyspace1" },
    { "name": "namespace1" },
    { "name": "system_schema" },
    { "name": "keyspace2" },
    { "name": "stargate_system" },
    { "name": "system_auth" },
    { "name": "system_traces" }
  ]
}
```

**✅ Create a document** :

```json
{
  "videoid": "e466f561-4ea4-4eb7-8dcc-126e0fbfd573",
  "email": "clunven@sample.com",
  "title": "A Second videos",
  "upload": "2020-02-26 15:09:22 +00:00",
  "url": "http://google.fr",
  "frames": [1, 2, 3, 4],
  "tags": ["cassandra", "accelerate", "2020"],
  "formats": {
    "mp4": { "width": 1, "height": 1 },
    "ogg": { "width": 1, "height": 1 }
  }
}
```

**👁️ Expected output**:

```json
{
  "documentId": "5d746e40-97cf-490b-ab0d-68cfbc5d2ef3"
}
```

**✅ Retrieve documents** :

```bash
curl --location \
--request GET 'localhost:8082/v2/namespaces/namespace1/collections/videos?page-size=3' \
--header "X-Cassandra-Token: $AUTH_TOKEN" \
--header 'Content-Type: application/json'
```

**👁️ Expected output**:

```json
{
  "data": {
    "5d746e40-97cf-490b-ab0d-68cfbc5d2ef3": {
      "email": "clunven@sample.com",
      "formats": {
        "mp4": { "height": 1, "width": 1 },
        "ogg": { "height": 1, "width": 1 }
      },
      "frames": [1, 2, 3, 4],
      "tags": ["cassandra", "accelerate", "2020"],
      "title": "A Second videos",
      "upload": "2020-02-26 15:09:22 +00:00",
      "url": "http://google.fr",
      "videoid": "e466f561-4ea4-4eb7-8dcc-126e0fbfd573"
    }
  }
}
```

**✅ Retrieve 1 document** :

```bash
curl -L \
-X GET 'localhost:8082/v2/namespaces/namespace1/collections/videos/5d746e40-97cf-490b-ab0d-68cfbc5d2ef3' \
--header "X-Cassandra-Token: $AUTH_TOKEN" \
--header 'Content-Type: application/json'
```

**👁️ Expected output**:

```json
{
  "documentId": "5d746e40-97cf-490b-ab0d-68cfbc5d2ef3",
  "data": {
    "email": "clunven@sample.com",
    "formats": {
      "mp4": { "height": 1, "width": 1 },
      "ogg": { "height": 1, "width": 1 }
    },
    "frames": [1, 2, 3, 4],
    "tags": ["cassandra", "accelerate", "2020"],
    "title": "A Second videos",
    "upload": "2020-02-26 15:09:22 +00:00",
    "url": "http://google.fr",
    "videoid": "e466f561-4ea4-4eb7-8dcc-126e0fbfd573"
  }
}
```

**✅ Search for document by properties** :

```JSON
{"email":
   { "$eq":"clunven@sample.com" }
}
```

**👁️ Expected output**:

```json
{
  "data": {
    "5d746e40-97cf-490b-ab0d-68cfbc5d2ef3": {
      "email": "clunven@sample.com",
      "formats": {
        "mp4": { "height": 1, "width": 1 },
        "ogg": { "height": 1, "width": 1 }
      },
      "frames": [1, 2, 3, 4],
      "tags": ["cassandra", "accelerate", "2020"],
      "title": "A Second videos",
      "upload": "2020-02-26 15:09:22 +00:00",
      "url": "http://google.fr",
      "videoid": "e466f561-4ea4-4eb7-8dcc-126e0fbfd573"
    }
  }
}
```

[🏠 Back to Table of Contents](#table-of-content)

## 5. Working with GRAPHQL API

This walkthrough has been realized using the [GraphQL Quick Start](https://stargate.io/docs/stargate/1.0/quickstart/quick_start-graphql.html)

**✅ Open GraphQL Playground** :

Open the playground
![image](pics/launch-graphql.png?raw=true)

**👁️ Expected output**
![image](pics/playground-home.png?raw=true)

**✅ Creating a keyspace** :

Before you can start using the GraphQL API, you must first create a Cassandra keyspace and at least one table in your database. If you are connecting to a Cassandra database with existing schema, you can skip this step.

Inside the GraphQL playground, navigate to http://localhost:8080/graphql-schema and create a keyspace by executing the following mutation:

```
mutation createKeyspaceLibrary {
  createKeyspace(name:"library", replicas: 1)
}
```

Add the auth token to the HTTP Headers box in the lower lefthand corner:

```
{
  "x-cassandra-token":"7c37bda5-7360-4d39-96bc-9765db5773bc"
}
```

**👁️ Expected output**

![image](pics/graphql-createkeyspace.png?raw=true)

**✅ Creating a Table** :

- Use this query

```
mutation {
  books: createTable(
    keyspaceName:"library",
    tableName:"books",
    partitionKeys: [ # The keys required to access your data
      { name: "title", type: {basic: TEXT} }
    ]
    values: [ # The values associated with the keys
      { name: "author", type: {basic: TEXT} }
    ]
  )
  authors: createTable(
    keyspaceName:"library",
    tableName:"authors",
    partitionKeys: [
      { name: "name", type: {basic: TEXT} }
    ]
    clusteringKeys: [ # Secondary key used to access values within the partition
      { name: "title", type: {basic: TEXT}, order: "ASC" }
    ]
  )
}
```

**👁️ Expected output**

![image](pics/graphql-createtables.png?raw=true)

**✅ Populating Table** :

Any of the created APIs can be used to interact with the GraphQL data, to write or read data.

First, let’s navigate to your new keyspace `library` inside the playground. Change tab to `graphql` and pick url `/graphql/library`.

- Use this query

```
mutation insert2Books {
  moby: insertbooks(value: {title:"Moby Dick", author:"Herman Melville"}) {
    value {
      title
    }
  }
  catch22: insertbooks(value: {title:"Catch-22", author:"Joseph Heller"}) {
    value {
      title
    }
  }
}
```

- Don't forget to update the header again

```
{
  "x-cassandra-token":"7c37bda5-7360-4d39-96bc-9765db5773bc"
}
```

**👁️ Expected output**

![image](pics/graphql-insertdata.png?raw=true)

**✅ Read data** :

Stay on the same screen and sinmply update the query with

```
query oneBook {
    books (value: {title:"Moby Dick"}) {
      values {
        title
        author
      }
    }
}
```

**👁️ Expected output**

![image](pics/graphql-readdata.png?raw=true)

[🏠 Back to Table of Contents](#table-of-content)

## 6. Getting started with SAI (Storage Attached Index)

**SAI** is short for **Storage Attached Indexes**, it allows us to build indexes on Cassandra tables that dramatically improve the flexibility of Cassandra queries.

For a **non-technical introduction** to **SAI**, have a look at this [recent blog post](https://www.datastax.com/blog/get-your-head-clouds-part-1-3-build-cloud-native-apps-datastax-astra-dbaas-now-aws-gcp).

To learn more about **SAI** from a **technical perspective**, have a look at our [docs on SAI](https://docs.datastax.com/en/storage-attached-index/6.8/sai/saiQuickStart.html). Honestly, these docs are pretty great IMO especially the [SAI FAQ](https://docs.datastax.com/en/storage-attached-index/6.8/sai/saiFaqs.html). Definitely take a moment to read through these to get a better understanding of how all of this works and even more examples on top of what we are presenting in this repo.

Now, let's get into some examples. The first thing we'll need is a table and some data to work with. For that we need to talk about my dentist, or really, a contrived example of a client data model a dentist might need to use.

**✅ Step 6a. Navigate to the CQL Console and login to the database**

In the Summary screen for your database, select **_CQL Console_** from the top menu in the main window. This will take you to the CQL Console with a login prompt.

![astra cqlsh console](https://user-images.githubusercontent.com/23346205/97186856-35bc9d80-1778-11eb-80fd-df1a2f264a25.png)

Once you click the _`CQL Console`_ tab it will automatically log you in and present you with a `token@cqlsh>` prompt.

**✅ Step 6b. Describe keyspaces and USE `sa_index`**

Ok, you're logged in, and now we're ready to rock. Creating tables is quite easy, but before we create one we need to tell the database which keyspace we are working with.

First, let's **_DESCRIBE_** all of the keyspaces that are in the database. This will give us a list of the available keyspaces.

📘 **Command to execute**

```
desc KEYSPACES;
```

_"desc" is short for "describe", either is valid_

📗 **Expected output**

![desc keyspace output](https://user-images.githubusercontent.com/23346205/97188152-939db500-1779-11eb-8e47-0d6b4ebea74b.png)

Depending on your setup you might see a different set of keyspaces then in the image. The one we care about for now is **_sa_index_**. From here, execute the **_USE_** command with the **_sa_index_** keyspace to tell the database our context is within **_sa_index_**.

📘 **Command to execute**

```
use sa_index;
```

📗 **Expected output**

![use sa_index output](https://user-images.githubusercontent.com/23346205/97188451-e4151280-1779-11eb-98a2-f7292621f6ac.png)

Notice how the prompt displays `token@cqlsh:sa_index>` informing us we are **using** the **_sa_index_** keyspace. Now we are ready to create our tables.

**✅ Step 6c. Create a _`clients`_ table and insert some data**

Create the table.

📘 **Command to execute**

```SQL
CREATE TABLE IF NOT EXISTS clients (
    uniqueid uuid primary key,
    firstname text,
    lastname text,
    birthday date,
    nextappt timestamp,
    newpatient boolean,
    photo text
);
```

Insert some data into the table.

_We don't have real image URLs, so we're just using a placeholder string._

📘 **Commands to execute**

```SQL
INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (D85745B1-4BEC-43D7-8B77-DD164CB9D1B8, 'Alice', 'Apple', '1984-01-24', '2020-10-20 12:00:00', true, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (2A4F139F-0BBF-4A6F-B982-5400F11D2F2B, 'Zeke', 'Apple', '1961-12-30', '2020-10-20 12:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (DF649261-89CB-446B-9998-FFA2D17506F9, 'Lorenzo', 'Banana', '1963-09-03', '2020-10-20 13:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (808E6BBF-A0F4-4E4C-9C97-E36751D51A8B, 'Miley', 'Banana', '1969-02-06', '2020-10-20 13:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (3D458A4D-2F54-4271-BEDC-1FC316B3CC96, 'Cheryl', 'Banana', '1970-07-11', '2020-10-20 14:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (287AB6B4-1AA6-45DF-B6F8-2BE253B9AACE, 'Red', 'Currant', '1974-02-18', '2020-10-20 15:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (AB49D151-CC04-40DC-AEEA-0A4E5F59D69A, 'Matthew', 'Durian', '1976-11-11', '2020-10-19 12:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (783CE790-16B4-4645-B27C-4FDF3994A755, 'Vanessa', 'Elderberry', '1977-12-03', '2020-10-20 15:30:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (D23997E4-CCCB-46BB-B92F-0D4582A68809, 'Elaine', 'Elderberry', '1979-11-16', '2020-10-20 10:00:00', true, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (36C386C1-3C3B-49FC-81B1-391D5537453D, 'Phoebe', 'Fig', '1986-01-27', '2020-10-21 11:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (00FEE7EE-8F93-4C2E-A8BE-3ADD81235822, 'Patricia', 'Grape', '1986-06-24', '2020-10-21 12:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (B9DB7E99-AD1C-49B1-97C6-87154663AEF4, 'Herb', 'Huckleberry', '1990-07-09', '2020-10-21 13:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (F4DB7673-CA4E-4382-BDCD-2C1704363590, 'John-Henry', 'Huckleberry', '1979-11-16', '2020-10-21 14:00:00', false, 'imageurl');

INSERT INTO clients (uniqueid, firstname, lastname, birthday, nextappt, newpatient, photo)
VALUES (F4DB7673-CA4E-4382-BDCD-2C1704363595, 'Sven', 'Åskådare', '1967-11-07', '2020-10-21 14:00:00', false, 'imageurl');
```

**✅ Step 26. Verify data exists**

Now let's take a look at the data we just inserted.

📘 **Command to execute**

```
SELECT * FROM clients;
```

📗 **Expected output**

![select from client table](https://user-images.githubusercontent.com/23346205/97189410-f2176300-177a-11eb-90ca-3604f113520f.png)

**✅ Step 6e. Create some indexes**

Ok great, we have data in our table, but remember we used **_`uniqueid`_** as our **primary key** when we created the table. If we want to query a single patient, we'd have to do that by the **_`uniqueid`_** column because that's our **partition key** _(don't forget, a single value in the primary key is always the partition key)_.

As a matter of fact, let's try an example. Let's say I want to find a user by their lastname.

📘 **Command to execute**

```SQL
SELECT * FROM clients WHERE lastname = 'Apple';
```

📗 **Expected output**

![clients where lastname allow filtering](https://user-images.githubusercontent.com/23346205/97208146-33b30880-1791-11eb-8470-13b6381de70e.png)

Right, the database is telling me here I **CANNOT** query against the **lastname** column because it is NOT in my primary key **_`uniqueid`_**.

But how would we search for users outside of using their unique ID's? We need to look for clients based on information they give us when they walk in the office. Namely, information like first and last name, or birthdate. Maybe a combination of those. Let's set up some indexes to do that.

_Don't worry about options in the below statements just yet. We'll get to that. For now, just execute the commands to create your indexes._

📘 **Commands to execute**

```SQL
CREATE CUSTOM INDEX IF NOT EXISTS ON clients(firstname) USING 'StorageAttachedIndex'
WITH OPTIONS = {'case_sensitive': false, 'normalize': true };

CREATE CUSTOM INDEX IF NOT EXISTS ON clients(lastname) USING 'StorageAttachedIndex'
WITH OPTIONS = {'case_sensitive': false, 'normalize': true };

CREATE CUSTOM INDEX IF NOT EXISTS ON clients(birthday) USING 'StorageAttachedIndex';
```

**✅ Step 6f. Execute queries that use firstname, lastname, and birthday using our indexes**

Remember, the **`clients`** table data model only includes **`uniqueid`** in the primary key. In the traditional Cassandra sense I can only query against the **`uniqueid`** column in the **WHERE** clause. However, with our **SAIndexes** now added we can do a lot more.

📘 **Command to execute**

```SQL
// Look for a client by ONLY their lastname. Notice the case used.
SELECT * FROM clients WHERE lastname = 'Apple';
```

📗 **Expected output**

![clients where lastname](https://user-images.githubusercontent.com/23346205/97198417-25f78600-1785-11eb-8632-9a7d30456ec4.png)

📘 **Command to execute**

```SQL
// Look for a client by their lastname and firstname. Notice the case used.
SELECT * FROM clients WHERE lastname = 'apple' AND firstname = 'alice';
```

📗 **Expected output**

![clients where firstname and lastname](https://user-images.githubusercontent.com/23346205/97198713-871f5980-1785-11eb-8416-bd595a70bf6d.png)

📘 **Command to execute**

```SQL
// Look for a client by an exact match to their birthday.
SELECT * FROM clients WHERE birthday = '1984-01-24';
```

📗 **Expected output**

![clients where birthday](https://user-images.githubusercontent.com/23346205/97198969-d9607a80-1785-11eb-8a5a-f754995634f1.png)

📘 **Command to execute**

```SQL
// Look for a client by a range match for the year of their birthday.
SELECT * FROM clients WHERE birthday > '1984-01-01' AND birthday < '1985-01-01';
```

📗 **Expected output**

![clients where birthday range](https://user-images.githubusercontent.com/23346205/97199744-baaeb380-1786-11eb-829d-5e05d99d4ba5.png)

📘 **Command to execute**

```SQL
// Look for a client by their firstname
// and a range match for the year of their birthday. Again, notice the case used.
SELECT * FROM clients
WHERE firstname = 'aLicE'
AND birthday > '1984-01-01' AND birthday < '1985-01-01';
```

📗 **Expected output**

![clients where name and birthday range](https://user-images.githubusercontent.com/23346205/97200012-0f522e80-1787-11eb-8aa9-d3c22049dd4b.png)

**✅ Step 6g. Digest everything we just did there**

Ok, so let's break that all down. I said earlier when we created the indexes I would explain the options included with some of the indexes.

```SQL
WITH OPTIONS = {'case_sensitive': false, 'normalize': true };
```

So what does the **“WITH OPTIONS”** part mean?

Well, [case_sensitive](https://docs.datastax.com/en/dse/6.8/cql/cql/cql_reference/cql_commands/cqlCreateCustomIndex.html#cqlCreateCustomIndex__cqlCreateCustomIndexOptions) is fairly straightforward. Setting this **false** allows us to match any combination of case for the terms we are querying against, **firstname** or **lastname** fields according to the indexes we created.

This is why I kept varying the case used in our queries above. You could **NOT** have done does this with a traditional Cassandra query.

How about [normalize](https://docs.datastax.com/en/dse/6.8/cql/cql/cql_reference/cql_commands/cqlCreateCustomIndex.html#cqlCreateCustomIndex__cqlCreateCustomIndexOptions)? Basically, this means that special characters, like vowels with diacritics can be represented by multiple binary representations for the same character, which also makes things easier to match.

An example would be a row with a column value that contained the character `Å (U+212B)`. With **normalize** enabled a query that used the character `Å (U+00C5)` would find that row. This saves from the need to find all unicode variations for a single character.

📘 **Command to execute**

```SQL
SELECT * FROM clients WHERE lastname = 'Åskådare';
```

_To be clear, this is not Ascii folding where I might insert code that uses `é` and a select using `e`. This is coming as a future feature._

To sum up, we queried against a combination of string and date fields using exact matches, multiple string cases, and date ranges. Just by adding an index on 3 fields we significantly expanded the flexibility of our data model.

Let's do more.

**✅ Step 6h. Add another index to support a new data model requirement**

Imagine a case where we now have a requirement to find clients based off of their next appointment.

Prior to **SAI**, if I wanted to accomplish this same thing in Cassandra, I would set up a new table using the **date** as the **partition key**, and I'd probably have the **appointment** slots as a **clustering column**, along with the **`uniqueid`** rounding out the primary key.

Then, I would retrieve the days partition to get a list of the appointments for the day. Now, I have **two tables** that I need to worry about to support that query.

Let's see what this looks like with **SAI**.

📘 **Command to execute**

```SQL
CREATE CUSTOM INDEX IF NOT EXISTS ON clients(nextappt) USING 'StorageAttachedIndex';
```

📘 **Command to execute**

```SQL
SELECT * FROM clients WHERE nextappt > '2020-10-20 09:00:00';
```

📗 **Expected output**

![clients where nextappt](https://user-images.githubusercontent.com/23346205/97206362-01081080-178f-11eb-8b7d-6b002da6f9fb.png)

[🏠 Back to Table of Contents](#table-of-content)

## THE END

Congratulation your made it to the END.
