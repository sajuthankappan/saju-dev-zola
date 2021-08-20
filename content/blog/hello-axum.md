+++
title = "Hello Axum"
description = "Playing with axum, a new web framework for rust"
date = 2021-08-20
[taxonomies]
tags =["rust"]
+++


Saw this announcement https://tokio.rs/blog/2021-07-announcing-axum, via twitter feed. Got interested to try it out, and see how it will fit my needs.

Initial impressions, after reading [the documentation](https://docs.rs/axum/0.1.3/axum/), is quite impressive. So, here I am, taking a test ride..


### Steps

Create new rust app/binary
```bash
cargo new hello-axum
```

Add axum dependency in Cargo.toml
```
axum = "0.1.3"
tokio = { version = "1.9.0", features = ["full"] }
```

Update main.rs (taken from axum documentation / hello world )
```rust
use axum::prelude::*;

#[tokio::main]
async fn main() {
    // build our application with a single route
    let app = route("/", get(|| async { "Hello, World!" }));

    // run it with hyper on localhost:3000
    axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

Run
```
cargo run
```

Check root url with a GET request (I use VS Code [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension)
```
GET http://localhost:3000/
```

It gives 
```
HTTP/1.1 200 OK
content-type: text/plain
content-length: 13
date: Fri, 20 Aug 2021 16:53:45 GMT

Hello, World!
```

Great! Hello World with Axum is success!!

Let's add more endpoints
```rust
let app = route("/", get(|| async { "Hello, World!" }))
        .route("/ping", get(|| async { "Pong!" }))
        .route("/api/profile", get(|| async { "Profiles!" }))
        .route("/api/users", get(|| async { "Users!" }));
```

Run again, and test the endpoints
```
GET http://localhost:3000/

###
GET http://localhost:3000/ping

###
GET http://localhost:3000/api/profile

###
GET http://localhost:3000/api/users
```

All endpoints are returning the expected results.

Let me try to use the axum's nest function to nest the api routes :)
```rust
let app = route("/", get(|| async { "Hello, World!" }))
        .route("/ping", get(|| async { "Pong!" }))
        .nest("/api", api_routes());
```

Add new function to return the api routes
```
fn api_routes() -> BoxRoute<Body> {
    route("/profile", get(|| async { "Profiles!" }))
        .route("/users", get(|| async { "Users!" }))
        .boxed()
}
```

Well, that continues to work the same, albet with little bit more modular/organized code. Sweat!!

I could read and make further changes, as per Axum documentation. The documentation is neat and clean! Really impressed.

So, taking it to the next level, I wanted to solve some of the problems I typically had to solve in my other api projects. One of them being, able to 'guard' the api enddpoints using an secure api-key in the request header. So, how to do this?

I noticed the documentation had a reference to using middlewares/tower middlewares. So, initial attempt was to look, if there is already a middleware available to do my task. I could find a few similar middlewares (here)[https://docs.rs/tower-http/0.1.1/tower_http/auth/index.html]. But, my needs were a little different. Tried to build a tower middleware myself, but, I was getting a few errors, when integrating with Axum (Eventually, figured out I did a few silly mistakes :P)

Reached out to the axum folks in discord. The folks are very friendly, and provided  the inputs almost real time, especially, davidpdrsn and ChillFish8 :)

davidpdrsn provided the valuabel input to use axum (extractor middleware). So, created one as below..

```rust
struct RequireApiKey;

#[async_trait]
impl<B> extract::FromRequest<B> for RequireApiKey
where
    B: Send,
{
    type Rejection = StatusCode;

    async fn from_request(req: &mut RequestParts<B>) -> Result<Self, Self::Rejection> {
        let my_secret_password = "secretlol";

        let auth_header = req
            .headers()
            .and_then(|headers| headers.get("api-key"))
            .and_then(|value| value.to_str().ok());

        if let Some(value) = auth_header {
            if value == my_secret_password {
                return Ok(Self);
            }
        }

        Err(StatusCode::UNAUTHORIZED)
    }
}
```

Note: I had to add async_trait dependency to Cargo.toml

and apply this middleware to api_routes
```rust
fn api_routes() -> BoxRoute<Body> {
    route("/profile", get(|| async { "Profiles!" }))
        .route("/users", get(|| async { "Users!" }))
        .layer(extractor_middleware::<RequireApiKey>())
        .boxed()
}
```
Now, trying below endpoint 
```
GET http://localhost:3000/api/profile
```
gives following error properly!
```
HTTP/1.1 401 Unauthorized
content-length: 0
date: Fri, 20 Aug 2021 17:51:15 GMT
```

Let's add the required api-key headers
```
GET http://localhost:3000/api/profile
api-key: secretlol
```

Now, it gives the right results. Great!

That still misses something. i.e., the secret is hardcoded in the middleware code. That's not what I don't want. Ideally, I would like to keep the real secret out of the middleware, and pass it from main. Again, davidpdrsn comes to the rescue and suggested how to use an extractor from another extractor, by giving an example code in axum (examples)[https://github.com/tokio-rs/axum/blob/main/examples/tokio-postgres/src/main.rs]

So, I had to create another 'extractor' to the code :)

```rust
#[derive(Clone)]
struct MySecretPassword(pub String);
```

And, add this to main, as an additional layer using AddExtensionLayer
```rust
let my_secret_password = MySecretPassword("secretlol".into());

let app = route("/", get(|| async { "Hello, World!" }))
    .route("/ping", get(|| async { "Pong!" }))
    .nest("/api", api_routes())
    .layer(AddExtensionLayer::new(my_secret_password));

```

And, retrieving this value from the RequireApiKey middleware is easy as below one liner
```rust
let Extension(my_secret_password) = Extension::<MySecretPassword>::from_request(req)
    .await
    .unwrap();
let my_secret_password = my_secret_password.0;
```

That's pretty much it. We managed to read the secret from another extractor. Yay!!

Testing the endpoints still works great!

### Conclusion

So, did i Like Axum? Yes. It does look promising. I currently use warp for some of my side projects. I think, both warp and axum has their respective share of plus & minuses. But, IMHO, both are really good choices! 

Expecting axum to mature a little bit more. I currently see the api's are still being tweaked.  I will probably use it for my next side project, once it gets a little bit more stable!

Thanks for eading!!
