# fy dropship

## Description

An online store that gives suppliers daily exposure to thousands of the most interested customers, by creating an addictive shopping experience that helps customers find what they want.

Any action that a customer or supplier makes is recorded by the store as an event, and the current state of facts are a projection of all past events (CQRS pattern).

The folders are represent the following components:

### Auth
Functions to login/create user. Login means to get authenticated token. Create saves a new user with a unique token.

### Data
Storage and transit of data - Message queues to send events, Collections that hold data about entities and projections to cast current state of all events, images, etc. These would contain all the functions that upsert/query the data. The same folder structure would exist in Ops for deployment of all these components.

### Diagrams
Rough sketches of interactions and relationships between system components

### Entities
All the different values or data objects that are sent around and/or persisted. They represent immutable events, and each have an associated timestamp.

### Notifications
Functions to send user notifications. Eg. stock levels to suppliers, email notifications, sms, app-messages, marketing etc.

### Ops
Application Deployment & restore of all data stores and lambda functions, as well as development related scripts & documents (eg if applicable - ci, dev setup docker images, etc).

### Recommendation
The recommendation engine that takes as input latest user event, uses it with current stock & history, and outputs a sorted list of items and collections that the customer might be most likely to buy to display to the user.

### Stock
Functions to create/update stock and orders.

### UI
Progressive web app and mobile shell apps.

## Assumptions

* SKU means product category, not GUID.
* There isn't a need (yet) to distinguish between stock and product as different entities. It's sufficient to work with the amount of products as a number, and not consider individual products that have the same SKU.
* AWS SQS queues can trigger lambda events
* If a supplier wants to use a barcode scanner, they manage all the drivers etc, and we just think of it as a keyboard.
* Mobile apps are mostly wrappers around the same progressive web app, with native styling.
* Closed mobile apps do not need an open connection, they poll on interval if an order is in a pending state.
* Payments are done via 3rd party via API.
* All the serverless services scale automagically as advertised.

## Diagrams (e.g entity relationship, system context, architecture)

## Questions

* Is setting up DynamoDB as a storage service for Datomic sufficient for auto-scaling needs ?

## Deploy

AWS cloudfront (wrapped in scripting langauge where needed) to create an idempotent deploy, that (re-)creates the entire environment from scratch/backup.

The site is on S3, messages on SQS, storage on DynamoDB and all processing done with Lambda/Step functions.

## Test

Test environment is as close as possible to production.

Every 1% of entities are also used as test-clone users, meaning all their events (and relational entities' events) are replicated and sent to the test environment, to simulate a load on test environment. This test-event stream can be stopped/started as needed for testing.

The test event log and projection can be destroyed can duplicated from the production data all at once, or specifically for one (and related) entity by re-playing all events into the test environment.

## Maintain

As part of development process, portions of the test environment are modified and re-created by the same scripts (except for specific config like url or secrets), so that updating production should have the same 'muscle-memory' of changing things on test environment. 

## Scaling

With a complete serverless architecture, there isn't a need to provision servers.

Using AWS SQS, Lambda/Step functions and DynamoDB the memory and compute auto scales.

Cloudfront CDN can be used to serve the site to website/app customers.

Monitoring is still important, to identify areas where billing could be improved. For example, maybe DynamoDB could be replaced by auto-scaling EC2 vm's with CockroachDB.