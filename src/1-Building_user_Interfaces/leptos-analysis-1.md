<h3 style="text-align:center;">Building User Interfaces</h3>

---

#### [Components](#a--basic-component),

#### [Event Listeners](#b---event-listeners),

#### [Signals](#c--signals-basics),

#### [Derived Signals for Dynamically Derived Values](#d--derived-signals-for-dynamically-derived-values),

#### [Dynamic Classes, HTML Attributes & Inline-Styles](#e--dynamic-classes-html-attributes--inline-styles),

#### [HTML Injection](#f--html-injection)

---

##### A- BASIC COMPONENT

- Has either `#[component]` (for client-side-render only components) or `#[server]` (for both server-side and client-side rendering with hydration), at the start of its definition.
- Is a function that is CamelCased
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

#[server]
fn {{TheNameOfTheServerComponent}}() -> impl IntoView {
  view! {
    <p>"This is a server component"</p>
  }
}

// Now render the component you wish to
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

##### B - EVENT LISTENERS

- Event listeners have the following syntax;

```rs
  on:{{event-name}} = move |_| {
      ...
  }
```

OR

```rs
  on:{{event-name}} = move | {{received_event_value}} | {
      ...
  }
```

<details>
<summary>Usage Example</summary>

Create a module named 'counter_component.rs';

```rs
use leptos::*;
/* Imagine component 'CounterComponent' as a stand-alone
 component inside its own rust module
*/
#[component]
pub fn CounterComponent() -> impl IntoView {
  // Let's create a signal
  let (counter, set_counter) = create_signal(0);
  // Let's return a single view
  view! {
    <button on:click = move |_| {
      set_counter = counter()+1;
    }>"Increase count"
    </button>
  }
}
```

Inside module 'app.rs';

```rs
use crate::counter_component::CounterComponent;
use leptos::*;
// Now render the component you wish to
// Let's import CounterComponent
#[component]
fn App() -> impl IntoView {
  let (count, set_count) = create_signal(0);
  view! {
    <CounterComponent/>
  }
}

fn main() {
  leptos::mount_to_body(|| view! { <App/> })
}
```

</details>

##### C- SIGNALS BASICS

- Creating a signal in Leptos;

```rs
  let ( {{signal_name}}, {{set_signal_name}} ) = create_signal(
    {{intial-value}}
  )
```

- Reading a signal syntax

```rs
  <button /*...*/>
    "Click me: "
    {count}
  </button>
  // The value will evaluate and dynamically change per signal change
```

OR

```rs
  <button /*...*/>
    "Click me: "
    {count()}
  </button>
```

- Update a signal dynamically

```rs
  <button on:click=move |_| {
      set_count.update(|n| *n += 1);
  }>
    "Click me: "
    {move || count()}
  </button>
```

##### D- DERIVED SIGNALS FOR DYNAMICALLY DERIVED VALUES

- Derived signals are controlled by signals.

- Signals are functions, e.g, the single tuple value returned from 'create_signal()' contains two elements and each elements are the signals. Derived signals on the other hand, are closures which access a signal.

- Derived signal syntax

```rs
// double_count is a derived signal
let double_count = move || count() * 2;
```

OR

```rs
<progress
  max="50"
  value=move || count() * 2 // value is subtly a derived signal
/>
```

- Derived signal usage example

```rs
// First create a Read & Write signal
let (count, set_count) = create_signal(0);
/* Then create a closure that calls a Read signal to clone its value,
 and returns a double of the value as a variable
*/
let double_count = move || count() * 2;

view! {
  // An HTML element with an attribute whose value dynamically changes
  <progress
    max="50"
    value=double_count
  />
  // Another example
  <p>
    "Double Count: "
    {double_count}
  </p>
}
```

- Speaking of derived signals, you can use them to create;

  - dynamic HTML values
  - dynamic event-listeners updates
  - dynamic class values
  - dynamic inline-style values
  - & dynamic HTML attribute values

- Derived signals compute twice or more times. A derived signal will run once for the variable that stores, and once for/inside each and every place where the variable that stores it is used.

- Use memos for resource intensive values that need to change like a signal.

##### E- DYNAMIC CLASSES, HTML ATTRIBUTES & INLINE STYLES

- Dynamic class' syntax

```rs
  <p class:red=move || count() % 2 == 1>
     "Dynamically change the color of this text"
  </p>
```

OR

```rs
  /* Use this approach if your HTML element
   has a class name with one or more dashes in its definition
  */
  class=("button-20", move || count() % 2 == 1)
```

- Dynamic HTML attribute syntax

```rs
<progress
  max="50"
  value=move || count() * 2 // value is subtly a derived signal
/>
```

- Dynamic inline-style syntax

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

##### F- HTML INJECTION

- You can inject HTML directly into a DOM element or Component's render using the `inner_html` Leptos property like this;

```rs
let html = "<p>This HTML will be injected.</p>";
view! {
  <div inner_html=html/>
}
```
