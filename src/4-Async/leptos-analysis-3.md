<h4>ASYNC::
MUTATING DATA WITH ACTIONS
</h4>

What if you just want to call an arbitrary `async` function, and keep track of what it's doing?

Depend on an input, a non-signal, or even nothing, and perform an asynchronous action by using `create_action`.

1. `create_action` purpose - If you are trying to occasionally run an `async` function in response to something like a user clicking a button (not neccessarily a signal??), you probably want to use `create_action`.

2. `create_action` takes an `async` function that takes a reference to a single argument, which you can think of as the "input type"

Example `create_action` code;

```rs
// ✅
// if there's a single argument, just use that
let action1 = create_action(|input: &String| {
   let input = input.clone();
   async move { todo!() }
});

// if there are no arguments (inputs/deps), use the unit type `()`
let action2 = create_action(|input: &()| async {
    // no need to clone 'input' here since there are no args provided as 'input'
    todo!()
});

// if there are multiple arguments (inputs/deps), use a tuple
let action3 = create_action(
  |input: &(usize, String)| async {
    let input = input.clone();
    todo!()
  }
);
```

_N.B: Because the action function takes a reference but the `Future` needs to have `'static` lifetime, you'll usually need to clone the input valuen(as done above) to pass it into the `Future`. This is admittedly awkward but it unlocks some powerful features like optimistic UI._

3. Here is another `create_action` in action, no pun intended.

```rs
let add_todo_action = create_action(|input: &String| {
  let input = input.to_owned(); // this essentially clones input
  async move {
    add_todo_request(&input).await
  }
})
```

4. Rather than call the above action `add_todo_action` in no.3 directly, you have to call it with a `.dispatch()`, as in;

```rs
add_todo_action.dispatch("Some value".to_string()); // "some value".to_string() becomes the argument value of 'add_todo_action'.
```

5. Because `.dispatch()` isn't an `async` function, it can be called from a synchronous context. You can do this from an event listener, a timeout, or anywhere.

6. Actions provide access to a few signals that synchronize between the asynchronous action you're calling and the synchronous reactive system:

7. Actions provide access to a few signals that synchronize between an asynchronous action and a synchronous reactive system;

```rs
let submitted = add_todo_action.input(); // RwSignal<Option<String>>
let pending = add_todo_action.pending(); // ReadSignal<bool>
let todo_id = add_todo_action.value(); // RwSignal<Option<Uuid>>
```

8. With actions, you can track the current state of your request, or do "optimistic UI" based on the assumption that the submission will succeed.

Example code;

```rs
use gloo_timers::future::TimeoutFuture;
use leptos::{html::Input, *};
use uuid::Uuid;

// Here we define an async function
// This could be anything: a network request, database read, etc.
// Think of it as a mutation: some imperative async action you run,
// whereas a resource would be some async data you load
async fn add_todo(text: &str) -> Uuid {
    _ = text;
    // fake a one-second delay
    TimeoutFuture::new(1_000).await;
    // pretend this is a post ID or something
    Uuid::new_v4()
}

#[component]
fn App() -> impl IntoView {
    // an action takes an async function with single argument
    // it can be a simple type, a struct, or ()
    let add_todo = create_action(|input: &String| {
        // the input is a reference, but we need the Future to own it
        // this is important: we need to clone and move into the Future
        // so it has a 'static lifetime
        let input = input.to_owned();
        async move { add_todo(&input).await }
    });

    // actions provide a bunch of synchronous, reactive variables
    // that tell us different things about the state of the action
    let submitted = add_todo.input();
    let pending = add_todo.pending();
    let todo_id = add_todo.value();

    let input_ref = create_node_ref::<Input>();

    view! {
        <form
            on:submit=move |ev| {
                ev.prevent_default(); // don't reload the page...
                let input = input_ref.get().expect("input to exist");
                add_todo.dispatch(input.value());
            }
        >
            <label>
                "What do you need to do?"
                <input type="text"
                    node_ref=input_ref
                />
            </label>
            <button type="submit">"Add Todo"</button>
        </form>
        <p>{move || pending().then(|| "Loading...")}</p>
        <p>
            "Submitted: "
            <code>{move || format!("{:#?}", submitted())}</code>
        </p>
        <p>
            "Pending: "
            <code>{move || format!("{:#?}", pending())}</code>
        </p>
        <p>
            "Todo ID: "
            <code>{move || format!("{:#?}", todo_id())}</code>
        </p>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```

9. You would often use `create_action` along-side server functions `create_server_action`, and the `<ActionForm>` component to create really powerful progressively-enhance forms.
