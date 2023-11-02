<h4>ASYNC::
SUSPENSE,
AWAIT,
TRANSITION
</h4>

1. `<Suspense> </Suspense/>` lets you work with multiple `create_resource`. `<Suspense><Suspense/>` provides a `fallback` component when two or more resources not yet available, and then lets you render the resources once available at its `children`.

Code example;

```rs
let (count_a, set_count) = create_signal(0);
let (count_b_ set_count2) = create_signal(0);

// async operations
async fn load_data_a(value: i32) -> i32 {
    // ...imagine some api call or something ...
}
async fn load_data_b(value: i32) -> i32 {
    // ...imagine some api call or something ...
}

// create_resource a
let loaded_data_a = create_resource(
    count_a,
    |count| async move { load_data_a(count_a).await }
);
// create_resource b
let loaded_data_b = create_resource(
    count_b,
    |count| async move { load_data_b(count_b).await }
);

// render the loaded resources from 'create_resource'
view! {
    <h2>"The Data"</h2>
    // Use Suspense to provide 'fallback' in-case, resource-data not loaded
    <Suspense
      fallback=move || view! {
        <p>"Loading..."</p>
      }
    >
    // The children now
      <h2>"Now Loaded!"</h2>
      <br/>
      <h3>"Loaded data a:"</h3>
      {
        move || {
            // Note that you do not need to match or call unwrap on the created resource
            loaded_data_a.get().map(
                |a| view! { <ShowA a/> }
            )
        }
      }
      <h3>"Loaded data b:"</h3>
      {
        move || {
            loaded_data_b.get().map(
                |b| view! { <ShowB b/> }
            )
        }
      }
    </Suspense>
}
```

2. `<Await></Await>` lets you poll a future once, wait for it to finish, and then render its content without ever reactively re-polling.
   It is the equivalent of a `create_resource` that does not depend on anything and runs just once, except that it's great for having two or more of such `create_resource` received data' render.

Code example;

```rs
// the async function
async fn fetch_monkeys(monkey: i32) -> i32 {
    // maybe this didn't need to be async
    monkey * 2
}
view! {
    <Await
        // `future` provides the `Future` to be resolved
        future=|| fetch_monkeys(3)
        // the data is bound to whatever variable name you provide - in this case, it's also called 'data'
        let:data
    >
        // you receive the data by reference and can use it in your view here
        <p>{*data} " little monkeys, jumping on the bed."</p>
    </Await>
}
```

3. Take note that `<Suspense></Suspense>` would keep flickering its `fallback` if you keep reloading the data, as in, each time any one of its embedded `create_resource` dep signal changes.
   To avoid this flickering, consider using a `<Transition></Transition>` instead.

4. `<Transition></Transition>` behaves exactly as `<Suspense></Suspense>`, but instead of falling back every time, it only shows the fallback the first time. On all subsequent loads, it continues showing the old data until the new data are ready. This can be really handy to prevent the flickering effect, and to allow users to continue interacting with your application.

Code example;

```rs
let (count_a, set_count) = create_signal(0);
let (count_b_ set_count2) = create_signal(0);

// async operations
async fn load_data_a(value: i32) -> i32 {
    // ...imagine some api call or something ...
}
async fn load_data_b(value: i32) -> i32 {
    // ...imagine some api call or something ...
}

// create_resource a
let loaded_data_a = create_resource(
    count_a,
    |count| async move { load_data_a(count_a).await }
);
// create_resource b
let loaded_data_b = create_resource(
    count_b,
    |count| async move { load_data_b(count_b).await }
);

// render the loaded resources from 'create_resource'
view! {
    <h2>"The Data"</h2>
    // Use Transition to provide 'fallback' in-case, resource-data not loaded, this fallback loads once, and remains upon subsequent `create_resource`' data reloads without ever flickering
    <Transition
      fallback=move || view! {
        <p>"Loading..."</p>
      }
    >
    // The children now
      <h2>"Now Loaded!"</h2>
      <br/>
      <h3>"Loaded data a:"</h3>
      {
        move || {
            // Note that you do not need to match or call unwrap on the created resource
            loaded_data_a.get().map(
                |a| view! { <ShowA a/> }
            )
        }
      }
      <h3>"Loaded data b:"</h3>
      {
        move || {
            loaded_data_b.get().map(
                |b| view! { <ShowB b/> }
            )
        }
      }
    </Transition>
}
```
