<h4>
BUILDING USER INTERFACES::
COMPONENTS, 
EVENT LISTENERS, 
SIGNALS, 
DYNAMICALLY-DERIVED VALUES, 
ATTRIBUTES, 
STYLES, AND HTML INJECTION
</h4>

A - A BASIC COMPONENT

- Has either `#[component]` or `#[server]` at the start of its definition
- Is a function that is CamelCase
- Returns `impl IntoView`
- Returns value `view! {...}`

Example;

```rs
#[component]
fn {{TheNameOfTheComponent}}() -> impl IntoView {
  view! {
    <p>"This is a sub-component"</p>
  }
}

// Now render the component you define
#[component]
fn App() -> impl IntoView {
  let (count, set_count) = create_signal(0);
  view! {
    <{{TheNameOfTheComponet}}/>
  }
}

fn main() {
  leptos::mount_to_body(|| view! { <App/> })
}
```

B - VIEW: DYNAMIC CLASSES, STYLES, AND ATTRIBUTES

1. Event listeners have the following syntax;

```rs
  on:{{event-name}} = move |_| {
      ...
  }
```

OR

```rs
  on:{{event-name}} = move |{{received_event_value}} {
      ...
  }
```

2. Event listener functions (always closures) usually change the value of a signal. Check No.5 to see how to change the value of a signal dynamically.

3. Signal Syntax Looks Like This:

```rs
  let ({{signal_name}}, {{set_signal_name}}) = create_signal({{intial-value}})
```

4. Reading a signal syntax: Inside a button value, as in;

```rs
  <button /*...*/>
    "Click me: "
    {count}
  </button> ----> // The value will dynamically change as the signal changes
```

OR

```rs
  <button /*...*/>
    "Click me: "
    {count()}
  </button> ----> // The value will not dynamically change as the signal changes. The calue is evaluated once only throughout the entire lifetime of the component
```

5. Update a Signal Dynamically;

```rs
  <button on:click=move |_| {
      set_count.update(|n| *n += 1);
  }>
    "Click me: "
    {move || count()}
  </button>
```

6. Only Event Handlers and signal-derived values (like in no.5) require Closures.

7. Dynamic Values Are Controlled By Signals. See An Example Below.

8. Dynamic class Syntax looks like this:
   ...inside an HTML/Leptos element or component as an attribute;

```rs
  class:red=move || count() % 2 == 1
```

OR

```rs
  class=("button-20", move || count() % 2 == 1) ----> // For class names with dashes
```

9. Dynamic Styles syntax

```rs
// First create a signal that would control the value fo the style
let (o, set_o) = create_signal(0);
let (p, set_p) = create_signal(0);
view! {
    <div
      style="position: absolute"
      // Now create dynamic style
      style:left=move || format!("{}px", o() + 100)
      style:top=move || format!("{}px", p() + 100)
      style:background-color=move || format!("rgb{}, {}, 100)", o() p())
      style=("--columns", x)
    >
      "Moves when coordinates change"
    </div>
}
```

STRONG ADVICE; SKIP THIS. YOU ALREADY POSSESS FUNCTIONING KNOWLEDGE. DON'T GET CAUGHT BY THIS TRIPPER.

10. Derived Signals. Signals are functions, e.g, the tuple values from 'create_signal()', while derived signals are signal' tuple values function calls made inside a closure, hence, such closure access the signal.
    Derived signals are closures which access a signal.

Derived signal example:

```rs
let double_count = move || count() * 2; // double_count is a derived signal
```

OR

```rs
<progress
  max="50"
  value=move || count() * 2 // value is a derived signal
/>
```

11. Derived Signal syntax;

```rs
// First create a Read & Write signal
let (count, set_count) = create_signal(0);
// The create a closure that calls a Read signal to clone its value, and returns a double of the value as a variable
let double_count = move || count() * 2;
// Now setup an HTML element with an attribute whose value is to change dynamically
<progress
  max="50"
  value=double_count
/>
// Another example
<p>
  "Double Count: "
  {double_count}
</p>
```

12. You can have;

- dynamic HTML values. Check No.4
- dynamic event-listeners. Check No.5
- dynamic classes. Check No.8
- dynamic styles. Check No.9
- & dynamic attributes. Check No.11

13. Derived signals compute twice or more times. A derived signal will run once for the variable that stores, and once for/inside each and every place where the variable that stores it is used.

14. Use memos for resource intensive values that need to change like a signal.

15. You can inject HTML directly into a DOM element or Component's render using the `inner_html` Leptos property like this;

```rs
let html = "<p>This HTML will be injected.</p>";
view! {
  <div inner_html=html/>
}
```
