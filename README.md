# A Subscription Application <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [The Question](#the-question)
- [Stepwise Implementation](#stepwise-implementation)
- [Prerequisites](#prerequisites)
- [Installing laravel](#installing-laravel)
- [Configuring a database](#configuring-a-database)
- [Project Documentation](#project-documentation)
  - [Database Migrations](#database-migrations)
  - [API Routes](#api-routes)
    - [Websites Routes](#websites-routes)
    - [Posts Routes](#posts-routes)
    - [Mailing List (Subscription Route)](#mailing-list-subscription-route)
  - [Implementing the models](#implementing-the-models)
  - [Implementing the controllers...](#implementing-the-controllers)
  - [Assigning an Observer & listing to the events.](#assigning-an-observer--listing-to-the-events)
  - [Notifying the User (Email)](#notifying-the-user-email)
- [Links](#links)
  - [Documentation:](#documentation)
  - [API Implementation](#api-implementation)
- [Ending thoughts.](#ending-thoughts)

## The Question

Create a simple subscription platform (only RESTful APIs with MySQL) in which users can subscribe to a website *(there can be multiple websites in the system)*. Whenever a new post is published on a particular website, all it's subscribers shall receive an email with the post title and description in it. **(No authentication of any type is required)**.

## Stepwise Implementation
- [x] Install laravel
- [x] Prerequisites
- [x] Configuring a database
- [x] Project Planning.
- [x] Links 
- [x] Ending Thoughts. 

## Prerequisites
- Assumed you have composer installed from https://getcomposer.org
- Assumed you have node installed from https://nodejs.org **(optional)**
- Assumed you are using a Visual Studios Code as your IDE. **(optional)**
- Assumed you have `PHP>=7.*` installed in your system with the path of the executable in your `$PATH` variable.

## Installing laravel

```sh
$ composer global require laravel/installer
$ laravel new your-app-name
$ cd your-app-name
$ composer update
```

**If you use VS Code as your IDE, please continue with this command**

```sh
$ code .
```

> :bulb: Please open an integrated terminal for your `laravel artisan` commands.

## Configuring a database

By default, laravel & this project assumes you are using **MySQL**.

- Driver: MySQL
- Username: *any_username*
- Password: *any_password*
- Database: *some_name*

> :alert: Please update the necessary credentials in the `.env` found in the project's root. If you do not find one, use the `.env.local` file to create one.

## Project Documentation

### Database Migrations

We have considered the following tables in the database:

1. A table for multiple websites is _websites_.
2. A table for posts on a website is _posts_.
3. A table for list of subscriptions is _mailing\_list_.
4. A table for maintaining a email queue named `jobs` generated by `php artisan queue:table`.


### API Routes

The following tables define routes to the different API's.

> Please append `/api` to the routes

#### Websites Routes

| Method |   Route         |       Description               |
|:------:|:----------------|:--------------------------------|
|  GET   | `/websites`     | Get all websites                |
|  POST  | `/websites`     | Add a new website               |
|  GET   | `/websites/{id}`| Get details of a website        |
|  POST  | `/websites/{id}`| Update all details of a website |
|  PUT   | `/websites/{id}`| Specific  details of a website  |

---

#### Posts Routes

| Method |      Route                  |                        Description                      |
|:------:|:----------------------------|:--------------------------------------------------------|
|  GET   | `/websites/{id}/posts`      | Gets details of all posts of a website.                 |
|  POST  | `/websites/{id}/posts`      | Add a new post to the website.                          |
|  GET   | `/websites/{id}/posts/{pid}`| Get details of a particular post of a website.          |
|  POST  | `/websites/{id}/posts/{pid}`| Update all details of particular post of a website.     |
|  PUT   | `/websites/{id}/posts/{pid}`| Update specific details of particular post of a website.|

---

#### Mailing List (Subscription Route)

| Method |      Route                 |                        Description                        |
|:------:|:---------------------------|:----------------------------------------------------------|
| POST   | `/websites/{id}/subscribe` | Add a new subscriber to the website.                      | 
| PUT    | `/websites/{id}/subscribe` | Update name by email in query like `?email=me@example.com`|
| DELETE | `/websites/{id}/subscriber`| Remove subscription from the specific website by email in query like `?email=me@example.com`|
| GET    | `/websites/{id}/subscriber`| Show the details of subscriber of a website by email in query like `?email=me@example.com`|


### Implementing the models

Three models have been considered.

1. A model for [Website](./app/Models/Website.php)
2. A model for [Posts](./app/Models/Posts.php)
3. A model for [Subscribers](./app/Models/Subscriber.php)


### Implementing the controllers...

The routes [above](#api-routes), have their respective controllers bound by the [Models](#implementing-the-models).

1. The [WebsiteController](./app/Http/Controllers/WebsiteController.php), uses the [Website](./app/Models/Website.php) Model to implement the [routes defined for websites](#websites-routes). The controller handles all functions of *C*reate, *U*pdate, *V*iew/*R*ead actions of a website. 
2. The [PostController](./app/Http/Controllers/PostController.php), uses the [Posts](./app/Models/Posts.php) Model to implement the [routes defined for posts](#posts-routes) *C*reate, *U*pdate, *V*iew/*R*ead actions of a post.
3. The [MailingListController](./app/Http/Controllers/MailingListController.php), uses the [Subscriber](./app/Models/Subscriber.php) Model to implement the [routes defined for mailing-list/subscription](#mailing-list-subscription-route). This controller is responsible for handling the mailing-list or subscription-table of a particular website.


### Assigning an Observer & listing to the events.

A dedicated [PostObserver](./app/Observers/PostObserver.php) is declared to observe for any new post created & dispatch a [mailing job](#notifying-the-user-email) which is enqueued to the bottom of Queue.

The [PostObserver](./app/Observers/PostObserver.php) is "booted" in the [EventServiceProvider](./app/Providers/EventServiceProvider), defining that the [Post Model](./app/Models/Post.php) is bound by this observer.


### Notifying the User (Email)

> Notifications are not perfectly handled. There was no attention given to the templating. Only a working implementation can be guaranteed.

- To notify the subscriber about any new post (template by `php artisan make:mail`), [NotifySubscriber](./app/Mail/NotifySubscriber.php) is defined.
- An asyncronous queue job (template generated by `php artisan make:job`) [SendEmailJob](./app/Jobs/SendEmailJob.php) handles the queue as well as finally invokes the "notifying" with the help of [NotifySubscriber](./app/Mail/NotifySubscriber.php).


## Links

### Documentation:

The following links may be helpful:

1. Laravel Docs - https://laravel.com/docs \[At the time of writing the version of the docs was 9.x\]
2. PHP Docs - https://www.php.net/manual/
3. Composer - https://getcomposer.org

### API Implementation

If you want to run the api, you can use the button below:

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/13488408-eed1b06e-68a2-4c82-b8e9-6cb8edef2e86?action=collection%2Ffork&collection-url=entityId%3D13488408-eed1b06e-68a2-4c82-b8e9-6cb8edef2e86%26entityType%3Dcollection%26workspaceId%3Db92b2c7f-201e-44d0-bf3a-bffc2a4e510e)

[You can also use the `JSON` link](https://www.getpostman.com/collections/f2926699d6f9ba10c57b) to deploy on Postman.

## Ending thoughts.

The front-end of this application will be a work-in progress. With this simple application, I have actually learnt a lot more than any other projects undertaken.




Thanks to Team Inisev....

Thank You & Regards,

`@anrcry (Anweshan Roy Chowdury)`
