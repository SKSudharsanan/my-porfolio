---
title: "User Management System using Actix"
description: "Handson section for SRM Vadapalani"
pubDate: "Aug 21, 2024"
heroImage: "/rust.jpg"
tags: ["RUST", "SRM"]
---
# Building a User Management API with Rust and Actix: A Step-by-Step Guide

## 1. Introduction to Actix and Hello World

Let's start by creating a simple "Hello World" application with Actix.

First, create a new Rust project:

```bash
cargo new user_management_api
cd user_management_api
```

Update your `Cargo.toml` file:

```toml
[dependencies]
actix-web = "4.0"
```

Now, create a simple "Hello World" route in `src/main.rs`:

```rust
use actix_web::{get, App, HttpServer, HttpResponse, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().service(hello)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

Run the application with `cargo run` and visit `http://localhost:8080` in your browser to see "Hello world!".

## 2. User Management Use Case

Our User Management system will perform CRUD (Create, Read, Update, Delete) operations on user data. This is a common requirement in many applications, allowing us to manage user accounts effectively.

## 3. Introducing CRUD Operations

Let's define the structure for our User data and create placeholder functions for our CRUD operations:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

async fn create_user(user: web::Json<User>) -> impl Responder {
    // TODO: Implement user creation
    HttpResponse::Ok().body("User created")
}

async fn get_user(id: web::Path<u32>) -> impl Responder {
    // TODO: Implement user retrieval
    HttpResponse::Ok().body(format!("Get user with id: {}", id))
}

async fn update_user(id: web::Path<u32>, user: web::Json<User>) -> impl Responder {
    // TODO: Implement user update
    HttpResponse::Ok().body(format!("Update user with id: {}", id))
}

async fn delete_user(id: web::Path<u32>) -> impl Responder {
    // TODO: Implement user deletion
    HttpResponse::Ok().body(format!("Delete user with id: {}", id))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/users", web::post().to(create_user))
            .route("/users/{id}", web::get().to(get_user))
            .route("/users/{id}", web::put().to(update_user))
            .route("/users/{id}", web::delete().to(delete_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 4. Serialization and Deserialization with Serde

In the previous step, we used `serde` for serialization and deserialization. Serde is a framework for serializing and deserializing Rust data structures efficiently and generically.

The `#[derive(Serialize, Deserialize)]` attribute on our `User` struct automatically implements the necessary traits for converting between Rust structures and JSON.

Example of how Serde works:

```rust
use serde::{Serialize, Deserialize};
use serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct User {
    id: u32,
    name: String,
    email: String,
}

fn main() {
    let user = User {
        id: 1,
        name: String::from("John Doe"),
        email: String::from("john@example.com"),
    };

    // Serialize
    let serialized = serde_json::to_string(&user).unwrap();
    println!("Serialized: {}", serialized);

    // Deserialize
    let deserialized: User = serde_json::from_str(&serialized).unwrap();
    println!("Deserialized: {:?}", deserialized);
}
```

## 5. Adding Chrono for Timestamp Handling

Let's add a `created_at` field to our `User` struct using the `chrono` crate for better timestamp handling:

Update `Cargo.toml`:

```toml
[dependencies]
actix-web = "4.0"
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
```

Update the `User` struct:

```rust
use chrono::{DateTime, Utc};

#[derive(Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
    created_at: DateTime<Utc>,
}
```

## 6. Introducing Diesel ORM

Now, let's integrate Diesel ORM for database operations. We'll use Neon, a serverless PostgreSQL service.

Add Diesel to your `Cargo.toml`:

```toml
[dependencies]
actix-web = "4.0"
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
diesel = { version = "2.0.0", features = ["postgres", "r2d2", "chrono"] }
dotenv = "0.15"
```

## 7. Setting up Neon Database

1. Sign up for a Neon account at https://neon.tech/
2. Create a new project
3. Get your connection string

Create a `.env` file in your project root:

```
DATABASE_URL=postgres://your-username:your-password@your-neon-host/your-database
```

## 8. Creating Migrations

Install the Diesel CLI:

```bash
cargo install diesel_cli --no-default-features --features postgres
```

Set up Diesel and create a migration:

```bash
diesel setup
diesel migration generate create_users
```

In the `up.sql` file:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR NOT NULL,
  email VARCHAR NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Run the migration:

```bash
diesel migration run
```

## 9. Connecting to the Database

Update `src/main.rs` to include database connection:

```rust
use diesel::prelude::*;
use diesel::r2d2::{self, ConnectionManager};
use dotenv::dotenv;

type DbPool = r2d2::Pool<ConnectionManager<PgConnection>>;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok();
    let database_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let manager = ConnectionManager::<PgConnection>::new(database_url);
    let pool = r2d2::Pool::builder()
        .build(manager)
        .expect("Failed to create pool.");

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            // ... routes ...
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

## 10. Implementing CRUD Operations

Now, let's implement our CRUD operations using Diesel:

```rust
use diesel::prelude::*;
use crate::schema::users;

#[derive(Queryable, Serialize)]
struct User {
    id: i32,
    name: String,
    email: String,
    created_at: chrono::NaiveDateTime,
}

#[derive(Insertable, Deserialize)]
#[diesel(table_name = users)]
struct NewUser {
    name: String,
    email: String,
}

async fn create_user(pool: web::Data<DbPool>, new_user: web::Json<NewUser>) -> impl Responder {
    let conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || {
        diesel::insert_into(users::table)
            .values(&new_user.0)
            .get_result::<User>(&conn)
    })
    .await
    .unwrap();

    HttpResponse::Ok().json(user)
}

async fn get_user(pool: web::Data<DbPool>, user_id: web::Path<i32>) -> impl Responder {
    let conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || users::table.find(user_id.into_inner()).first::<User>(&conn))
        .await
        .unwrap();

    HttpResponse::Ok().json(user)
}

async fn update_user(
    pool: web::Data<DbPool>,
    user_id: web::Path<i32>,
    updated_user: web::Json<NewUser>,
) -> impl Responder {
    let conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || {
        diesel::update(users::table.find(user_id.into_inner()))
            .set(&updated_user.0)
            .get_result::<User>(&conn)
    })
    .await
    .unwrap();

    HttpResponse::Ok().json(user)
}

async fn delete_user(pool: web::Data<DbPool>, user_id: web::Path<i32>) -> impl Responder {
    let conn = pool.get().expect("couldn't get db connection from pool");

    let deleted = web::block(move || {
        diesel::delete(users::table.find(user_id.into_inner())).execute(&conn)
    })
    .await
    .unwrap();

    if deleted > 0 {
        HttpResponse::Ok().body("User deleted successfully")
    } else {
        HttpResponse::NotFound().body("User not found")
    }
}
```

## 11. Error Handling

To handle errors properly, we can create a custom error type:

```rust
use actix_web::{error::ResponseError, HttpResponse};
use thiserror::Error;

#[derive(Error, Debug)]
enum MyError {
    #[error("Database error: {0}")]
    DbError(#[from] diesel::result::Error),
    #[error("Environment error: {0}")]
    EnvError(#[from] std::env::VarError),
    #[error("IO error: {0}")]
    IOError(#[from] std::io::Error),
    #[error("Blocking error: {0}")]
    BlockingError(#[from] actix_web::error::BlockingError),
}

impl ResponseError for MyError {
    fn error_response(&self) -> HttpResponse {
        HttpResponse::InternalServerError().json(format!("Internal Server Error: {}", self))
    }
}
```

Then update your handler functions to return `Result<HttpResponse, MyError>`.

## 12. Final Main.rs File

Here's the complete `main.rs` file with all the changes:

```rust
use actix_web::{web, App, HttpResponse, HttpServer, ResponseError};
use diesel::prelude::*;
use diesel::r2d2::{self, ConnectionManager};
use dotenv::dotenv;
use serde::{Deserialize, Serialize};
use std::env;
use thiserror::Error;

mod schema;
use schema::users;

type DbPool = r2d2::Pool<ConnectionManager<PgConnection>>;

#[derive(Queryable, Serialize)]
struct User {
    id: i32,
    name: String,
    email: String,
    created_at: chrono::NaiveDateTime,
}

#[derive(Insertable, AsChangeset, Deserialize)]
#[diesel(table_name = users)]
struct NewUser {
    name: String,
    email: String,
}

#[derive(Error, Debug)]
enum MyError {
    #[error("Database error: {0}")]
    DbError(#[from] diesel::result::Error),
    #[error("Environment error: {0}")]
    EnvError(#[from] std::env::VarError),
    #[error("IO error: {0}")]
    IOError(#[from] std::io::Error),
    #[error("Blocking error: {0}")]
    BlockingError(#[from] actix_web::error::BlockingError),
}

impl ResponseError for MyError {
    fn error_response(&self) -> HttpResponse {
        HttpResponse::InternalServerError().json(format!("Internal Server Error: {}", self))
    }
}

async fn create_user(pool: web::Data<DbPool>, new_user: web::Json<NewUser>) -> Result<HttpResponse, MyError> {
    let mut conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || {
        diesel::insert_into(users::table)
            .values(new_user.into_inner())
            .get_result::<User>(&mut conn)
    })
    .await??;

    Ok(HttpResponse::Ok().json(user))
}

async fn get_user(pool: web::Data<DbPool>, user_id: web::Path<i32>) -> Result<HttpResponse, MyError> {
    let mut conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || users::table.find(user_id.into_inner()).first::<User>(&mut conn))
        .await??;

    Ok(HttpResponse::Ok().json(user))
}

async fn update_user(
    pool: web::Data<DbPool>,
    user_id: web::Path<i32>,
    updated_user: web::Json<NewUser>,
) -> Result<HttpResponse, MyError> {
    let mut conn = pool.get().expect("couldn't get db connection from pool");

    let user = web::block(move || {
        diesel::update(users::table.find(user_id.into_inner()))
            .set(updated_user.into_inner())
            .get_result::<User>(&mut conn)
    })
    .await??;

    Ok(HttpResponse::Ok().json(user))
}

async fn delete_user(pool: web::Data<DbPool>, user_id: web::Path<i32>) -> Result<HttpResponse, MyError> {
    let mut conn = pool.get().expect("couldn't get db connection from pool");

    let deleted = web::block(move || {
        diesel::delete(users::table.find(user_id.into_inner())).execute(&mut conn)
    })
    .await??;

    if deleted > 0 {
        Ok(HttpResponse::Ok().body("User deleted successfully"))
    } else {
        Ok(HttpResponse::NotFound().body("User not found"))
    }
}

#[actix_web::main]
async fn main() -> Result<(), MyError> {
    dotenv().ok();
    env_logger::init();

    let database_url = env::var("DATABASE_URL")?;
    let manager = ConnectionManager::<PgConnection>::new(database_url);
    let pool = r2d2::Pool::builder()
        .build(manager)
        .expect("Failed to create pool.");

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            .route("/users", web::post().to(create_user))
            .route("/users/{id}", web::get().to(get_user))
            .route("/users/{id}", web::put().to(update_user))
            .route("/users/{id}", web::delete().to(delete_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await?;

    Ok(())
}
```

Now, let's explain some of the key concepts we've used in this code:

## Explanation of Key Concepts

1. `NewUser` vs `User`:
   - `User` represents a user as stored in the database, including an `id` and `created_at` timestamp.
   - `NewUser` represents the data needed to create a new user, typically just `name` and `email`.

2. Derive Attributes:
   - `#[derive(Queryable)]`: This attribute is provided by Diesel and automatically implements the `Queryable` trait, allowing the struct to be the result of a database query.
   - `#[derive(Serialize)]`: This attribute from Serde allows the struct to be serialized (e.g., to JSON).
   - `#[derive(Insertable)]`: This Diesel attribute allows the struct to be inserted into the database.
   - `#[derive(Deserialize)]`: This Serde attribute allows the struct to be deserialized (e.g., from JSON).
   - `#[derive(AsChangeset)]`: This Diesel attribute allows the struct to be used for updating existing records.

3. Database Pool:
   - `type DbPool = r2d2::Pool<ConnectionManager<PgConnection>>;`
   This creates a type alias for a connection pool. It uses r2d2 to manage a pool of database connections, which is more efficient than creating a new connection for each request.

4. `web::Data<DbPool>`:
   This is how we pass the database pool to our handlers. `web::Data` is an Actix construct that allows us to share application state across handlers.

5. Custom Error Type:
   We create a custom error type (`MyError`) to handle different types of errors that might occur in our application. The `Debug` and `Display` traits are used for formatting the error:
   - `Debug` provides a format intended for developers.
   - `Display` provides a user-friendly format.

## Testing the API

Now that we have our API set up, let's test it using curl commands:

1. Create a User:
```bash
curl -X POST http://localhost:8080/users \
     -H "Content-Type: application/json" \
     -d '{"name": "John Doe", "email": "john@example.com"}'
```

2. Get a User:
```bash
curl http://localhost:8080/users/1
```

3. Update a User:
```bash
curl -X PUT http://localhost:8080/users/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "John Updated", "email": "john_updated@example.com"}'
```

4. Delete a User:
```bash
curl -X DELETE http://localhost:8080/users/1
```

## Using Postman

While curl is great for quick command-line tests, Postman provides a more user-friendly interface for API testing. Here's how to use Postman with our API:

1. Open Postman and click on "Import" in the top left corner.
2. Choose the "Raw text" tab.
3. Paste one of the curl commands from above.
4. Click "Continue" and then "Import".
5. Postman will create a new request with the correct method, URL, headers, and body.
6. You can then send the request and see the response directly in Postman.

Remember to update the user ID in the URL for GET, PUT, and DELETE requests to match the ID of an existing user in your database.

## Conclusion

In this tutorial, we've built a robust User Management API using Rust, Actix, and Diesel ORM. We've covered:

1. Setting up a basic Actix web server
2. Defining data structures with Serde for serialization and deserialization
3. Using Chrono for timestamp handling
4. Integrating Diesel ORM for database operations with Neon (serverless PostgreSQL)
5. Implementing CRUD operations for user management
6. Adding proper error handling
7. Testing the API using curl and Postman

This API provides a solid foundation for building more complex applications. Remember to add proper authentication and authorization before using this in a production environment.

From here, you could extend this API by adding more complex queries, implementing authentication, or adding additional endpoints as needed for your application.