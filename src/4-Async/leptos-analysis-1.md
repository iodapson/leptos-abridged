<h4>ASYNC::
LOADING DATA WITH RESOURCES - `create_resource`
</h4>

1. `create_resource` is great for waiting for a single asynchronous task `Future` inside Leptos synchronous reactive system. It takes two arguments

- i. a source signal, which will generate a new `Future` whenever it changes
- ii. a fetcher function, which takes the data from that signal and returns a `Future`.

2. `create_resource` also provides a `refetch()` method that allows you to manually reload data (for example, in response to a button click) and a `loading()` method that returns a `ReadSignal<bool>` indicating whether the resource is currently loading or not.

Example `create_resource()` code;

```rs
use gloo_timers::future::TimeoutFuture;
use leptos::*;

// Here we define an async function
// This could be anything: a network request, database read, e.t.c
// Here, we just multiply a number by 10
async fn load_data(value: i32) -> i32 {
    // fake a one-second delay
    TimeoutFuture::new(1_000).await;
    value * 10
}

#[component]
fn App() -> IntoView {
  // our source signal: some synchronous, local state
  let (count, set_count) = create_signal(0);

  // our resource
  let async_data = create_resource(
    // synchronous 'count' triggers an asynchronous action (the second argument to 'create_resource') everytime 'count' changes
    count,
    // every time `count` changes, this will run
    // ..'value' is the value of 'count', this fetcher function receives it
    |value| async move {
        logging::log!("loading data from API");
        load_data(value).await
    },
  );

// Usage:
    // we can access the resource values with .read()
    // this will reactively return None before the Future has resolved
    // and update to Some(T) when it has resolved
    let async_result = move || {
        async_data
            .read()
            .map(|value| format!("Server returned {value:?}"))
            // This loading state will only show before the first load
            .unwrap_or_else(|| "Loading...".into())
    };

    // the resource's loading() method gives us a
    // signal to indicate whether it's currently loading
    let loading = async_data.loading();
    let is_loading = move || if loading() { "Loading..." } else { "Idle." };

  view! {
    <button
        on:click=move |_| {
            set_count.update(|n| *n += 1);
        }
    >
        "Click me"
    </button>
    // display 'async_result' of 'async_data'
    <p>
        <code>"async_value"</code>": "
        {async_result}
        <br/>
        {is_loading}
    </p>
  }
}
```

3. To create a resource that simply runs once, you can pass a non-reactive, empty source signal:

```rs
// an async function
async fn load_data(value: i32) -> i32 {
    // fake a one-second delay
    TimeoutFuture::new(1_000).await;
    value * 10
}

let once = create_resource(|| (), |_| async move { load_data(1).await }); //  this resource does not receive, depend on, or utilize any source signal

// Usage:
view! {
    // ✅ Option 1:
    <h1>"My Data"</h1>
    {move || match once.get() {
        None => view! { <p>"Loading..."</p> }.into_view(),
        Some(data) => view! { <ShowData data/> }.into_view()
    }}
    // Option 2:
    <p>
        <code>"stable"</code>": " {move || stable.read()}
    </p>
    <p>
        "Render something else"
    </p>
}
```
