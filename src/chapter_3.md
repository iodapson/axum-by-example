### Using a 'routes' sub-directory to keep all route handlers

HTTP method(s) used here for routing - `get`

<b>Project Setup::</b>

First things first, run command;

```
$ cargo new axum_project_name
```

..which creates a new Rust binary project file.

Next, run;
`$ cargo add axum`
..from anywhere inside your axum project dir to add 'axum' as a crate dependency for your axum project.

Next up, you need to add tokio with features 'macros' and 'rt-multi-thread';

```
$ cargo add tokio -F macros -F rt-multi-thread
```

..the upper-case "F" argument above means feature.

Axum project folder structure::

<pre>
  |-axum_project_name
   |-dist
   |-src
    |-routes
     |-mod.rs
     |-root_path_get_handler.rs
    |-lib.rs
    |-main.rs
   |-target
   |-Cargo.lock
   |-Cargo.toml
   |-index.html
</pre>

  <pre style="color:orange;">INSIDE 'Cargo.toml'</pre>

<code>
<pre>
[package]
name = "yew_project_name"
version = "0.1.0"
edition = "2021"
  
[dependencies]
axum = "0.5.17"
tokio = { version = "1.21.2", feature = ["macros", "rt-multi-thread"] }
</pre>
</code>

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/root_path_get_handler.rs'</pre>

```rust
pub async fn root_path_get_handler() -> String {
    "A proper route handler"
}

```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/mod.rs'</pre>

```rust
mod root_path_get_handler;

use root_path_get_handler::root_path_get_handler;
use axum::{body::Body, Router, routing::get};

// * This function would then be assigned to 'app' inside 'lib.rs'
pub fn create_routes() -> Router<Body> {
    Router::new().route("/", get(root_path_get_handler))
}
```

  <pre style="color:orange;">INSIDE 'lib.rs'</pre>

```rust
use routes::create_routes;

pub async fn run() {

    let app = create_routes();

    axum::Server::bind(&"0.0.0.0:8070".parse().unwrap())
    .serve(app.into_make_service())
    .await
    .unwrap();

}
```

  <pre style="color:orange;">INSIDE 'main.rs'</pre>

```rust
// * Function 'run is located
// * ...inside adjacent 'lib.rs'
use axum_project_name::run;

#[tokio::main]
async fn main() {
    run().await
}
```

To run your app, run command;

```
$ cargo watch -x run
```
