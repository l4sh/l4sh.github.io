---
title: "DevLog: Building a rust app - Part 1"
date: 2025-03-16
layout: post
current: post
navigation: True
author: l4sh
category: development
tags:
  - rust
  - backend
  - devlog
comments: true
---

Been putting off seriously learning Rust for a while now. I've toyed with it but never really completed a project. So I've decided to build a small app.

The idea is to build a REST API that will serve as a backend for the app and document the process in a series of posts.

## Development process

I'll be following a MVP approach. Starting with a really basic version of the app and adding features as I go. So don't really expect a full blown design document or anything like that.


## The app

- **Name:** Homly
- **Description:** A fast and lightweight home management app.
- **Features:**
  - User management
    - Users
    - Roles
  - Inventory management
    - Food and supplies
    - Appliances
    - Tools
  - Task management
    - To-do lists
- **Nice to have:**
  - Barcode & QR code scanning for food and supplies
  - Code generator for tagging items
  - Shopping list generator
  - Automated reminders for upcoming tasks
  - Automated messages with tasks, shopping lists, etc
  - Permissions. By default every user would be able to edit and view everything.
  - Vehicle management
  - Maintenance tracking
    - Track scheduled maintenance
    - Log maintenance history
    - Generate reminders for upcoming maintenance

I'll focus on getting the main features first and adding the rest later on.

### Tech stack

From my research it seems that the most well documented backend stack for a beginner would be Actix Web with Diesel.

So far this is the stack I'm planning to use:

- **Backend:** Rust / Actix Web / Diesel
- **Frontend:** Svelte / Tailwind CSS
- **Database:** SQLite / Postgres

I'm targeting both sqlite and postgres in order to be able to both run on a regular PC or do a proper deployment.

## First steps

### Hello world

First thing I created my project with `cargo new homly`. Then I added the dependencies to the `Cargo.toml` file.

```toml
[package]
name = "homly"
version = "0.1.0"
edition = "2024"

[dependencies]
actix-web = "4"
diesel = { version = "2.2.0", features = ["sqlite", "returning_clauses_for_sqlite_3_35", "postgres"] }
dotenvy = "0.15"
```

Updated the `main.rs`

```rust
use actix_web::{web, App, HttpServer, HttpResponse, Responder, get, post};

#[get("/")]
async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(index)
            .service(echo)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

Finally did a quick test with `cargo run`, sent a couple of requests and it seems to be working.

### User management

First step (after the previous first step) is adding user support I guess, since it's the most basic feature and one of the easiest to mess up when learning.

#### User model

Before starting I had to install diesel cli and set it up.

```bash
cargo binstall diesel_cli
diesel setup
```

Generate the respective migration for the users table.

```bash
diesel migration generate add_users users
```

Edit `migrations/2025-03-16-034512_add_users/up.sql`

```sql
CREATE TABLE users (
  id VARCHAR(36) NOT NULL PRIMARY KEY,
  username VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  password VARCHAR(255) NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  deleted_at TIMESTAMP DEFAULT NULL
);
```

Then edit `migrations/2025-03-16-034512_add_users/down.sql`

```sql
DROP TABLE IF EXISTS users;
```

Add the `chrono` and `serde` crates to the dependencies.

```toml
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
```

Create a module `models/user.rs`

```rust
use chrono::{DateTime, Utc};
use crate::schema::users;
use serde::{Serialize, Deserialize};

#[derive(Debug, Serialize, Deserialize, Queryable)]
pub struct User {
    pub id: i32,
    pub username: String,
    pub name: String,
    pub email: String,
    pub password: String,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
    pub deleted_at: Option<DateTime<Utc>>,
}
```

I'll be using UTC for all the timestamps since it's easier to work with and avoids annoying timezone issues like having your local on UTC+-whatever then deploying to a server on UTC and breaking displayed datetimes, scheduled jobs, etc.


Finally to populate apply the mgiration and populate the `schema.rs` file.

```bash
diesel migration run
diesel print-schema > src/schema.rs
```






#### Password hashing / verification

For hashing / verifying passwords I'll be using argon2.

Added the crate to the `Cargo.toml` file.

```toml
argon2 = "0.5"
```

Then created a module `utils/password.rs`

```rust
use argon2::{Argon2, PasswordHash, PasswordHasher, PasswordVerifier};
use argon2::password_hash::{rand_core::OsRng, SaltString};


pub fn hash_password(password: &str) -> Result<String, argon2::password_hash::Error> {
    let salt = SaltString::generate(&mut OsRng);
    let argon2 = Argon2::default();
    let password_hash = argon2.hash_password(password.as_bytes(), &salt)?.to_string();
    Ok(password_hash)
}

pub fn verify_password(hash: &str, password: &str) -> Result<bool, argon2::password_hash::Error> {
    let parsed_hash = PasswordHash::new(hash)?;
    let argon2 = Argon2::default();
    Ok(argon2.verify_password(password.as_bytes(), &parsed_hash).is_ok())
}
```



## Resources / References

Along the way I found these helpful.

- Actix Web documentation - https://actix.rs/docs
- Diesel Getting Started - https://diesel.rs/guides/getting-started/
- Serde - https://serde.rs/
- Rust Book - https://doc.rust-lang.org/book/
- Auth0's Build an API in Rust with JWT Authentication - https://auth0.com/blog/build-an-api-in-rust-with-jwt-authentication-using-actix-web/

