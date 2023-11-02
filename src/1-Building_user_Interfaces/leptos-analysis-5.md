<h4>
BUILDING USER INTERFACES::
CONTROL FLOW
</h4>

1. The best way for complex renders, using `<Show />`:

```rs
let (value, set_value) = create_signal(0);

view! {
  <Show
    when=move || { value() > 5 }
    fallback=|| view! { <Small/> }
  >
    <Big/>
  </Show>
}
```

"`<Show/>` memoizes the when condition, so it only renders its `<Small/>` once, continuing to show the same component until value is greater than five; then it renders `<Big/>` once, continuing to show it indefinitely or until value goes below five and then renders `<Small/>` again.
This is a helpful tool to avoid rerendering when using dynamic `if` expressions. As always, there's some overhead: for a very simple node (like updating a single text node, or updating a class or attribute), a `move || if ...` will be more efficient. But it's at all expensive to render either branch, reach for `<Show/>`.

---

Other Ways To Conditionally Render A Component (Recommended for Simple Renders):

2. The primitive `if` statement way

You can reactively and conditionally render elements/components. Here is an example code;

```rs
// create a signal to make the render target reactive
let (value, set_value) = create_signal(0);
// create a logic that returns a boolean based on the state of value of the signal
let is_odd_example_logic = move || value() % 1 == 1;

// conditionally render a component or HTML element using an if statement
view! {
    <p>
      {move || if is_odd_example_logic() {
        "Render - is odd"
      } else {
        "Render - is even"
      }
      }
    </p>
}
```

3. Using an `if` statement that returns an `Option<T>` value;

You can reactively and conditionally render elements/components. Here is an example code;

```rs
// create a signal to make the render target reactive
let (value, set_value) = create_signal(0);
// create a logic that returns a boolean based on the state of value of the signal
let is_odd_example_logic = move || value() % 1 == 1;

// store a conditionally obtained Some( value, component or HTML element ) or Option::None using an if statment that returns an option value
let message = move || {
    if is_odd_example_logic() {
        Some("Ding ding ding!")
    } else {
        None
    }
}
view! {
    <p>
      {message}
    </p>
}
```

4. The primitive `match` statement way;

Here is an example code;

```rs
// create a signal to make the render target reactive
let (value, set_value) = create_signal(0);
// create a logic that returns a boolean based on the state of value of the signal
let is_odd_example_logic = move || value() % 1 == 1;

// store a match obtained value, component or HTML element
let message = move || {
    match value() {
        0 => "Zero",
        1 => "One",
        n if is_odd() => "Odd",
        _ => "Even"
    }
};

view! {
    <p>{message}</p>
}
```

---

Returning Different HTML Elements From Different Branches Of A Conditional

Here is an example code where there is multiple `HtmlElement` types that are converted into `HtmlElement<AnyElement>` with method `.into_any()`;

```rs
use leptos::*;

#[component]
fn App() -> impl IntoView {

  let (value, set_value) = create_signal(0);
  // define boolean logic that decides what gets rendered
  let is_odd_logic_example = move || value() & 1 == 1;

  view! {
    <main>
      // Simple UI to update and show a value
      <button on:click=move |_| set_value.update(
        |n| *n += 1
      )>
        "+1"
      </button>
      <p>"Value is: " {value}</p>
      // Simple dynamic style based on a condition
      <p class:hidden=is_odd_logic_example>
      "Appears if even."
      </p>

      // ** Take Note Of This Conditional Render **//
      {move || match is_odd() {
          true if value() == 1 => {
              // returns HtmlElement<Pre>
              view! { <pre>"One"</pre> }.into_any()
          },
          false if value() == 2 => {
              // returns HtmlElement<P>
              view! { <p>"Two"</p> }.into_any()
          }
          // returns HtmlElement<Textarea>
          _ => view! { <textarea>{value()}</textarea> }.into_any()
      }}
    </main>
  }
}
```

<h6>Note: If you have a variety of view types that are not all 'HtmlElement', convert them into 'View's with '.into_view()'</h6>

Perhaps you can even go futher and concatenate function calls 'into_view()' and 'into_any()' like so, `.into_view().into_any()`.
