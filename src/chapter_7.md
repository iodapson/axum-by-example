### Extracting multiple paths from a route

HTTPS method used herein for routing - `get`

**N.B:** Here, you'll be able to respond to a get request to a URL with a path (:id)

<b>Project Setup::</b>

First things first, run command;

```
$ cargo new axum_project_name
```

..which creates a new Rust binary project file.

Next, run;

```
$ cargo add axum
```

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
     |-send_path_post_handler.rs
     |-id_path_extractor_get_handler.rs
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
serde = { version = "1.0.148", features = ["derive"] }
tokio = { version = "1.21.2", feature = ["macros", "rt-multi-thread"] }
</pre>
</code>

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/send_path_post_handler.rs'</pre>

```rust
use serde::{Deserialize, Serialize};
use axum::Json;

#[derive(Serialize, Deserialize, Debug)]
pub struct ExpectedJsonData {
  message: String,
}

#[derive(Serialize)]
pub struct ReturnedJsonData {
  received_message: String,
  success_value: u8,
}

pub async fn send_path_post_handler(
  Json(sent_file): Json<ExpectedJsonData>
) -> Json<ReturnedJsonData> {

  Json(
    ReturnedJsonData {
      recieved_message: sent_file.message,
      success_value: 1,
    }
  )

}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/id_path_extractor_get_handler.rs'</pre>

```rust
// * Priority code file
use axum::extract::Path; // * Path is an axum extractor


// * PLEASE TAKE NOTE *//
// * Function signature for extracting a single paths
/*pub async fn id_path_extractor_get_handler(
  Path(id): Path<u8>
) -> String {

  format!("Here, the path ':id' received: {}", id)

}*/


// * For extracting multiple paths, use this function signature instead:
pub async fn id_path_extractor_get_handler(
  Path(paths): Path<(u8, String)>
) -> String {

  format!(
    "Here are the paths: first one is u8; {}, second one is String; {}",
    paths.0, paths.1
  )

}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/mod.rs'</pre>

```rust
mod send_path_post_handler;
// * The get handler with path extraction
mod id_path_extractor_get_handler;

use axum::{body::Body, Router};
use axum::{routing::post};
use send_path_post_handler::send_path_post_handler;
use id_path_extractor_get_handler::id_path_extractor_get_handler;


// This function would be assigned to 'app' inside 'run' function
// ...inside 'lib.rs'
pub fn create_routes() -> Router<Body> {

  Router::new()
    .route("/send", post( send_path_post_handler ))
    // * Now it is time to create a route handler for
    // * ...HTTP get request to 'id_path_extractor'
    // * ...with paths - ':id' and 'id2'
    .route(
      "/id_path_extractor/:id/id2", get( id_path_extractor_get_handler )
    )
    // * For a single path, use;
    /*
    .route(
      "/id_path_extractor/:id", get( id_path_extractor_get_handler )
    )
    */

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
use axum_project_name::run; //* Function 'run' is located inside 'lib.rs'

#[tokio::main]
async fn main() {
  run().await
}
```

To run your app, run command;

```
$ cargo watch -x run
```
