### Extracting custom header(s) from a route

HTTPS method used herein for routing - `get`

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

N.B: You also need 'serde' with 'derive' feature for this axum demo to parse json data
To add this to your axum project, run;

```
$ cargo add serde -F derive
```

Axum project folder structure::

<pre>
  |-axum_project_name
   |-dist
   |-src
    |-routes
     |-mod.rs
     |-send_path_post_handler.rs
     |-id_path_extractor_get_handler.rs
     |-query_params_get_handler.rs
     |-standard_header_user_agent_get_handler.rs
     |-custom_header_get_handler.rs
    |-lib.rs
    |-main.rs
   |-target
   |-Cargo.lock
   |-Cargo.toml
   |-index.html
</pre>

  <pre>INSIDE 'Cargo.toml'</pre>

<code>
<pre>
[package]
name = "yew_project_name"
version = "0.1.0"
edition = "2021"
  
[dependencies]
axum = { version = "0.5.17", features = ["headers"] }
serde = { version = "1.0.148", features = ["derive"] }
tokio = { version = "1.21.2", feature = ["macros", "rt-multi-thread"] }
</pre>
</code>

  <pre  style="color:orange;">INSIDE 'axum_project_name/src/routes/send_path_post_handler.rs'</pre>

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

pub async fn send_path_post_handler(Json(sent_file): Json<ExpectedJsonData>) -> Json<ReturnedJsonData> {
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
use axum::extract::Path; // Path is an axum extractor

/*pub async fn id_path_extractor_get_handler(Path(id): Path<u8>) -> String {
  format!("Here, the path ':id' received: {}", id)
}*/ // This is for single paths.

// For extracting multiple paths, use this function signature instead:
pub async fn id_path_extractor_get_handler( Path(paths): Path<(u8, String)> ) -> String {
  format!("Here are the paths: first one is u8; {}, second one is String; {}")
}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/query_params_get_handler.rs'</pre>

```rust
use serde::{Serialize, Deserialize};
// You need to import Query to be able to mange a set of query parameters
use axum::extract::Query;
use axum::Json;

// Create a struct that would manage a given set of query parameters
// You need to also derive serde 'Serialize' and 'Deserialize' on the struct
#[derive(Serialize, Deserialize)]
pub structFirstQuerySet {
  one: u8,
  two: String,
}

pub async fn query_params_get_handler( Query(query_set): Query<FirstQuerySet> ) -> Json<FirstQuerySet> {
  Json (
    FirstQuerySet {
      one: query_set.one,
      two: query_set.two,
    }
  )
  // Alternatively, you could simply return
  // Json(query_set)
  // ..which is less verbose, more succinct
}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/standard_header_user_agent_get_handler.rs'</pre>

```rust
use axum::{TypedHeader, headers::UserAgent};

pub async fn standard_header_user_agent(TypedHeader(user_agent): TypedHeader<UserAgent>) -> String {
  user_agent.to_string()
}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/custom_header_get_handler.rs'</pre>

```rust
//* Priority rust file
use axum::http::HeaderMap;

pub async fn custom_header(headers: HeaderMap) -> String {
  // * You could get literally any custom-named header, e.g. 'x-code'
  let x_code_key_value: headers.get("x-code").unwrap(); // * Try and use better error handling
  let x_code_value = x_code_key_value.to_str().unwrap().to_owned();
  x_code_value
}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/mod.rs'</pre>

```rust
mod send_path_post_handler;
mod id_path_extractor_get_handler;
mod query_params_get_handler;
mod standard_header_user_agent_get_handler;
mod custom_header_get_handler; // * Priority mod

use axum::{body::Body, Router};
use axum::{routing::post};
use send_path_post_handler::send_path_post_handler;
use id_path_extractor_get_handler::id_path_extractor_get_handler;
use query_params_get_handler::query_params_get_handler;
use standard_header_user_agent_get_handler::standard_header_user_agent;
use custom_header_get_handler::custom_header; // * Priority use


// This function would be assigned to 'app' inside 'run' function inside 'lib.rs'
pub fn create_routes() -> Router<Body> {
  Router::new()
    .route("/send", post( send_path_post_handler ))
    // .route("/id_path_extractor/:id", get( id_path_extractor_get_handler )) // for single path
    .route("/id_path_extractor/:id/id2", get( id_path_extractor_get_handler )) // for multiple paths
    .route("/enter_queries", get( query_params_get_handler ))
    .route( "/show_a_standard_header", get(standard_header_user_agent) )
    .route("/show_custom_header", get( custom_header )) // * Priority route
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
