---
title: "Building a Web Service with Actix and Rust"
description: "Explore the steps to create a high-performance web service using Actix and Rust. This guide includes building a simple To-Do application with CRUD operations."
pubDate: "May 23, 2024"
heroImage: "/blog1.png"
tags: ["Rust", "Actix", "Web Development"]
---

Rust provides safety and performance, which are crucial for building reliable web services. Actix is a powerful and flexible web framework for Rust that enables developers to build scalable and maintainable applications. In this blog, we'll create a basic To-Do application with CRUD operations.

### Setting up the Project

Before diving into the code, you need to set up your development environment. Here are the steps to get started:

1. **Install Rust:**  
   Rust can be installed by following the instructions on the official Rust website. Visit [Install Rust](https://www.rust-lang.org/tools/install) and follow the steps for your operating system.

2. **Set up Actix:**  
   For building web applications with Rust, we'll use the Actix web framework. Detailed instructions can be found at [Actix Getting Started](https://actix.rs/docs/getting-started/).

3. **Create a New Project:**  
   Use Cargo, the Rust package manager, to create a new project by running the following command:
   ```bash
   cargo new my_todo_app
   cd my_todo_app
   ```

4. **Add Dependencies:**  
   Open your `Cargo.toml` file and add the following lines under `[dependencies]` to include Actix and other required crates:
   ```toml
   actix-web = "4"
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   ```

### Building the To-Do Application

Here is the complete Rust code for a simple To-Do application with CRUD operations:

```rust
use actix_web::{web, App, HttpServer, Responder, HttpResponse};
use serde::{Deserialize, Serialize};
use std::sync::Mutex;
use std::collections::HashMap;

#[derive(Serialize, Deserialize)]
struct TodoItem {
    id: u32,
    description: String,
}

struct AppState {
    todos: Mutex<HashMap<u32, String>>,
}

async fn add_todo(data: web::Data<AppState>, item: web::Json<TodoItem>) -> impl Responder {
    let mut todos = data.todos.lock().unwrap();
    todos.insert(item.id, item.description.clone());
    HttpResponse::Created().finish()
}

async fn get_todos(data: web::Data<AppState>) -> impl Responder {
    let todos = data.todos.lock().unwrap();
    let todo_list: Vec<_> = todos.iter().map(|(id, desc)| format!("{}: {}", id, desc)).collect();
    HttpResponse::Ok().json(todo_list)
}

async fn update_todo(data: web::Data<AppState>, item: web::Json<TodoItem>) -> impl Responder {
    let mut todos = data.todos.lock().unwrap();
    if let Some(desc) = todos.get_mut(&item.id) {
        *desc = item.description.clone();
        HttpResponse::Ok().finish()
    } else {
        HttpResponse::NotFound().finish()
    }
}

async fn delete_todo(data: web::Data<AppState>, id: web::Json<u32>) -> impl Responder {
    let mut todos = data.todos.lock().unwrap();
    if todos.remove(&id.0).is_some() {
        HttpResponse::Ok().finish()
    } else {
        HttpResponse::NotFound().finish()
    }
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let app_data = web::Data::new(AppState {
        todos: Mutex::new(HashMap::new()),
    });

    HttpServer::new(move || {
        App::new()
            .app_data(app_data.clone())
            .route("/todos", web::post().to(add_todo))
            .route("/todos", web::get().to(get_todos))
            .route("/todos", web::put().to(update_todo))
            .route("/todos", web::delete().to(delete_todo))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```
**To run the application**
```bash
   cargo run
```
To test if the application is running correctly, you can use curl or any API testing tool like Postman. Here are some curl commands to test the CRUD operations:

**Create a To-Do Item:**
```bash
curl -X POST -H "Content-Type: application/json" -d '{"id":1, "description":"Buy milk"}' http://127.0.0.1:8080/todos
```
**Retrieve All To-Do Items:**
```bash
curl http://127.0.0.1:8080/todos
```
**Update a To-Do Item:**
```bash
curl -X PUT -H "Content-Type: application/json" -d '{"id":1, "description":"Buy bread"}' http://127.0.0.1:8080/todos
```
**Delete a To-Do Item:**
```bash
curl -X DELETE -H "Content-Type: application/json" -d '{"id":1}' http://127.0.0.1:8080/todos
```

