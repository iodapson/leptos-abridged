<h3 style="text-align:center;">Components, Event Listeners, Signals, Derived Signals (for Dynamically Derived Values, Dynamic Classes, HTML Attributes & Inline-Styles), and HTML Injection</h3>

---

#### [Components](#a--basic-component),

#### [Event Listeners](#b---event-listeners),

#### [Signals](#c---signals-basics),

#### [Derived Signals for Dynamically Derived Values](#d---derived-signals-for-dynamically-derived-values),

#### [Dynamic Classes, HTML Attributes & Inline-Styles](#e---dynamic-classes-html-attributes--inline-styles),

#### [HTML Injection](#f---html-injection)

---

##### A - LEPTOS COMPONENT

- A Leptos component would at the start of its definition either have an attribute `#[component]` (for client-side-render only components) or attribute `#[server]` (for both server-side and client-side rendering with hydration).
- Has a function-name that is CamelCased
- Returns `impl IntoView`
- Returns value `view! {...}` to satisfy `impl IntoView`'s requirement

Example;

```rs
use leptos::*;

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

// Now render any component you wish to
#[component]
fn App() -> impl IntoView {
  let (count, set_count) = create_signal(0);
  view! {
    <{{TheNameOfTheComponent}}/>
  }
}

fn main() {
  leptos::mount_to_body(|| view! { <App/> })
}
```

Check out the official documentation on ["A Basic Component"](https://leptos-rs.github.io/leptos/view/01_basic_component.html) for a thorough explanation.

##### B - EVENT LISTENERS

- Event listeners have the following syntax:

```rs
  on:{{event_name}} = move |_| {
      ...
  }
```

OR

```rs
  on:{{event_name}} = move | {{received_event_value}} | {
      ...
  }
```

<details>
<summary>Usage Example</summary>

Create a module inside `src` e.g, `counter_component.rs`;

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

Inside module `app.rs`;

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

##### C - SIGNALS BASICS

- Creating a signal in Leptos:

```rs
  let ( {{signal_name}}, {{set_signal_name}} ) = create_signal(
    {{intial-value}}
  )
```

> Here, in the above code snippet, calling `create_signal()` returns a tuple, where `signal_name` is a [ReadSignal](https://docs.rs/leptos/latest/leptos/struct.ReadSignal.html), and `set_signal_name` is a [WriteSignal](https://docs.rs/leptos/latest/leptos/struct.WriteSignal.html)

- Reading a signal syntax:

```rs
  use leptos::*;

  #[component]
  fn App() -> impl IntoView {

      // First create a signal
      let (count, set_count) = create_signal(0);

      view! {
          // Then read the signal
          <button /*...*/>
            "Click me: "
            {count} // reactive
          </button>
          // count is reactive, because it is a ReadSignal
      }
  }
```

> Here, in the code snippet above, `{count}` inside the view's `<button></button>` is actually a clone of signal `count` of type `ReadSignal<i32>` from '`let (count, set_count) = create_signal(0);`'.

OR

```rs
  use leptos::*;

  #[component]
  fn App() -> impl IntoView {

      // First create a signal
      let (count, set_count) = create_signal(0);

      view! {// Then read the signal
          <button /*...*/>
            "Click me: "
            {count()} // unreactive
          </button>
      }
  }
```

> Here, in the code demo above, `count()` will evaluate `ReadSignal` - `count`'s value immediately, and access it only once. Calling `count()` is actually syntax-sugar for `count.clone().get()`

- Update a signal dynamically:

```rs
  use leptos::*;

  #[component]
  fn App() -> impl IntoView {

      /* First create a signal so you can cause a
         re-render whenever you update the signal
      */
      let (count, set_count) = create_signal(0);

      view! {
          // Pay attention to 'set_count', which is a 'WriteSignal'
          <button on:click=move |_| {
            set_count.update(|n| *n += 1);
          }>
            "Click me: "
            {move || count()}
            /* Interestingly, '{move || count()}' is reactive because
                you're only calling 'count()' within a closure!
            */
          </button>
      }
  }
```

##### D - DERIVED SIGNALS FOR DYNAMICALLY DERIVED VALUES

- Derived signals are controlled by signals.

- Derived signals are closures that access a signal source.

- Derived signals gived you the opportunity to access a `ReadSignal` to make futher computations, and then use the computed value instead.

- Derived signals allow you derive values from a signal source reactively. You'll soon see an example of accessing a derived signal reactively.

- Derived signal syntax:

```rs
// double_count is a derived signal, count() is a ReadSignal value
let double_count = move || count() * 2;
```

OR

```rs
view! {
    <progress
      max="50"
      value=move || count() * 2
      // 'move || count() * 2' is a derived signal
    />
}
```

- Derived signal usage example:

```rs
use leptos::*;

pub fn MyComponent() -> impl IntoView {
    // First create a Read & Write signal
    let (count, set_count) = create_signal(0);

    /* Then create a closure that gets a ReadSignal's data,
       and returns a double of the ReadSignal value as a variable
    */
    let double_count = move || count() * 2;

    // Usage of derived signal 'double_count':
    view! {
        /* An HTML element with an attribute whose value
           dynamically changes
        */
        <progress
          max="50"
          value=double_count
        />

        // Accessing a derived signal value reactively
        <p>
          "Double Count: "
          {double_count}
        </p>
    }
}
```

- Speaking of derived signals, you can use them to create;

  - dynamic HTML values
  - dynamic event-listeners updates
  - dynamic class values
  - dynamic inline-style values
  - & dynamic HTML attribute values

- Derived signals compute twice or more times - once per source signal signal change (acceptable), and once again for each and every place where the derived-signal is called to make its source-signal' derived-computation (not great for resource intensive computations!).

- Use memos for resource intensive values that need to change like a signal.

##### E - DYNAMIC CLASSES, HTML ATTRIBUTES & INLINE STYLES

- Dynamic class' syntax

```rs
  ...

  /* First create a source-signal for the dynamic signal
     that would make the HTML element's class value to be dynamic
  */
  let (count, set_count) = create_signal(0);

  // Then make the class dynamic
  <p class:red=move || count() % 2 == 1>
     "Dynamically change the color of this text"
  </p>

  ...

```

OR

```rs
  ...

  /* First create a source-signal for the dynamic signal
     that would make the HTML element's class value to be dynamic
  */
  let (count, set_count) = create_signal(0);

  /* Use this approach if your HTML element
   has a class name with one or more dashes in its definition
  */
  <p class=("p-20", move || count() % 2 == 1)>
    "Dynamically change the font-size of this text"
  </p>

  ...

```

- Dynamic HTML attribute syntax

```rs
<progress
  max="50"
  value=move || count() * 2 // attribute 'value' is now dynamic
/>
```

- Dynamic inline-style syntax

```rs
...

// First create a signal that would control the value of the style
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

...

```

##### F - HTML INJECTION

- You can inject HTML directly into a DOM element or Component's render using the `inner_html` Leptos property like this;

```rs
let html = "<p>This HTML will be injected.</p>";
view! {
  <div inner_html=html/>
}
```
