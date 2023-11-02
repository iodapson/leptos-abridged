<h4>
BUILDING USER INTERFACES::
ITERATING LISTS STATICALLY OR DYNAMICALLY
</h4>

1. Whether you're listing todos, displaying a table, or showing product images, iterating over a list of items is a common task in web applications.

   Leptos provides two different patterns for iterating over items:

   - For static views: `Vec<_>`
   - For dynamic lists (e.g, controlled by signals): `<For/>`

2. Example code of iterating a list `Vec<IV>` where `IV: IntoView`;

```rs
// the list
let values = vec![0, 1, 2];
// time to render the list
view! {
    //  render the list as-is, bundled together
    <p>{values.clone()}</p>
    // or iterate over the list for precise renders
    <ul>
      {values.into_iter().map(
        |item| view! {
            <li>{item}</item>
        }
      ).collect::<Vec<_>>()}
    </ul>
}
```

3. Example code of iterating a list `Vec<IV` where `IV: IntoView`, using Leptos iterator helper-function - `.collect_view()`

```rs
// the list
let values = vec![0, 1, 2];
// time to render the list
view! {
    // again, render the list as-is, bundled together
    <p>{values.clone()}</p>
    // or iterate over the list for precise renders
    <ul>
      {values.into_iter().map(
        |item| view! {
            <li>{item}</li>
        }
      ).collect_view()}
    </ul>
}
```

4. Example of making a list of signals whose individual items can change reactively, but the entire list as a whole would remain unreactive.

```rs
// define the signal list's length
let signal_list_length = 5;
// create a list of 5 counter signals
let counters_list = (1..=signal_list_length).map(
    |current_index| create_signal(current_index)
);

// each item manages a reactive view
// ...but, the list itself will never change
let counter_buttons = counters_list.map(
    |(count, set_count)| {
        // display a rendered list of buttons, each with their own independent signals
        view! {
            <li>
              <button on:click=move |_| {set_count.update(
                |its_current_count| *its_current_count += 1
              )}>
                {count}
              </button>
            </li>
        }
    }
).collect_view(); // This, 'counter_buttons' is an unordered list of signals, each list with its own internally managed reactive view

view! {
    <ul>{counter_buttons}</ul>
}
```

5. Here is an example code that creates a list of signals, whereby the entire list is a signal, hence new new list item additions can be added, or even old list items removed, and the list would get updated dynamically;

_N.B: The iteration is done using `<For/>`_

```rs
use leptos::*;

/// A list of counters that allows you to add or
/// remove counters.
#[component]
fn DynamicList(
    /// The number of counters to begin with.
    initial_length: usize,
) -> impl IntoView {
    // This dynamic list will use the <For/> component.
    // <For/> is a keyed list. This means that each row
    // has a defined key. If the key does not change, the row
    // will not be re-rendered. When the list changes, only
    // the minimum number of changes will be made to the DOM.

    // `next_counter_id` will let us generate unique IDs
    // we do this by simply incrementing the ID by one
    // each time we create a counter
    let mut next_counter_id = initial_length;

    // we generate an initial list as in <StaticList/>
    // but this time we include the ID along with the signal
    let initial_counters = (0..initial_length)
        .map(|id| (id, create_signal(id + 1)))
        .collect::<Vec<_>>();

    // now we store that initial list in a signal
    // this way, we'll be able to modify the list over time,
    // adding and removing counters, and it will change reactively
    let (counters, set_counters) = create_signal(initial_counters);

    // clousure to update list-signal - 'initial_counters'
    let add_counter = move |_| {
        // create a signal for the new counter
        let sig = create_signal(next_counter_id + 1);
        // add this counter to the list of counters
        set_counters.update(move |counters| {
            // since `.update()` gives us `&mut T`
            // we can just use normal Vec methods like `push`
            counters.push((next_counter_id, sig))
        });
        // increment the ID so it's always unique
        next_counter_id += 1;
    };

    view! {
        <div>
            <button on:click=add_counter>
                "Add Counter"
            </button>
            <ul>
                // The <For/> component is central here
                // This allows for efficient, key list rendering
                <For
                    // `each` takes any function (in this case, a ReadSignal<_>) that returns an iterator (usually a list of signals or derived signal)
                    // if the list is not reactive, just render a Vec<_> instead of <For/>
                    each=counters
                    // the key should be unique and stable for each row
                    // using an index is usually a bad idea, unless your list
                    // can only grow, because moving items around inside the list
                    // means their indices will change and they will all rerender
                    key=|counter| counter.0
                    // `children` receives each item from your `each` iterator
                    // and returns a view
                    children=move |(id, (count, set_count))| {
                        view! {
                            <li>
                                <button
                                    on:click=move |_| set_count.update(|n| *n += 1)
                                >
                                    {count}
                                </button>
                                <button
                                    on:click=move |_| {
                                        set_counters.update(|counters| {
                                            counters.retain(|(counter_id, _)| counter_id != &id)
                                        });
                                    }
                                >
                                    "Remove"
                                </button>
                            </li>
                        }
                    }
                />
            </ul>
        </div>
    }
}

#[component]
fn App() -> impl IntoView {
    view! {
        <h2>"Dynamic List"</h2>
        <p>"Use this pattern if the rows in your list will change."</p>
        <DynamicList initial_length=5/>
    }
}

fn main() {
    leptos::mount_to_body(App)
}

```

Click this link to see its result: https://pwdn2y-8000.csb.app/
