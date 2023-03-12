### Hello World axum app

<b>Project Setup::</b>

First things first, run command;
`$cargo new axum_project_name`
..which creates a new Rust binary project file.

Next, run;
`
$ cargo add axum`
..from anywhere inside your axum project dir to add 'axum' as a crate dependency for your axum project.

Next up, you need to add tokio with features 'macros' and 'rt-multi-thread';
`$ cargo add tokio -F macros -F rt-multi-thread`
..the upper-case "`F`" argument here means feature.

Axum project folder structure::

<pre>
  |-axum_project_name
   |-dist
   |-src
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

  <pre style="color:orange;">INSIDE 'main.rs'</pre>

```rust
use axum::{routing::get, Router};

#[tokio::main]
async fn main() {

    let app = Router::new().route("/", get(root_path_get_handler));

    axum::Server::bind(&"0.0.0.0:8070".parse().unwrap())
    .serve(app.into_make_service())
    .await
    .unwrap();

}

// * handler_functions should ideally return
// * ...something or cause a side-effect
async fn root_path_get_handler() -> String {
    "Hello world!!!".to_owned()
}
```

By the way, to watch and auto-restart your server upon each saved change to your code, use the cargo tool 'cargo-watch'.
You need to first install 'cargo-watch';

```
$ cargo install cargo-watch
```

Having installed 'cargo-watch', you can now run your axum project using the following command;

```
$ cargo watch -x run
```

Also note that you can view your documentation (for your project and all of its dependencies) locally by using command;

```
$ cargo doc --open
```
