### Receiving and returning JSON data in a HTTP POST request with JSON payload

HTTP method(s) used herein for routing - `post`

**N.B:** Here, you'll have to parse data recieved as json.

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

// * Valid JSON data has to be provided with the HTTP-POST request
// * ..to this sample route handler
pub async fn send_path_post_handler(
  Json(received_json_data): Json<ExpectedJsonData>
) -> Json<ReturnedJsonData> {

    Json(
        ReturnedJsonData {
            recieved_message: recieved_json_data.message,
            success_value: 1,
        }
    ) // * return a json response

}
```

  <pre style="color:orange;">INSIDE 'axum_project_name/src/routes/mod.rs'</pre>

```rust
// * This file is contained in 'src/routes' dir
mod send_path_post_handler;

use axum::{body::Body, Router};
use axum::{routing::post};
use send_path_post_handler::send_path_post_handler;

// This function would be assigned to 'app' inside 'run' function
// ...inside 'lib.rs'
pub fn create_routes() -> Router<Body> {
    Router::new()
    .route("/", get( root_path_get_handler ))
    .route("/send", post( send_path_post_handler ))
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
// * Function 'run' is located inside adjacent 'lib.rs'
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
