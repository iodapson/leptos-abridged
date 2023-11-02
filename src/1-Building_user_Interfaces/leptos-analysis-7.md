<h4>
BUILDING USER INTERFACES::
PARENT-CHILD COMMUNICATION (PASSING PROPS TO GRAND-SUB-COMPONENTS, MAKING A PARENT COMPONENT AWARE OF CHANGES MADE TO ITS PROP FROM A SUB-COMPONENT)
</h4>

How can a child send notifications about signal(state) changes back up to the parent?

---

Using `WriteSignal` in Sub-Component

1. You can pass a `WriteSignal` from the parent down to the child, and update it in the child. This lets you manipulate the state of the parent from inside the child.

Example code;

```rs
#[component]
pub fn App() -> impl IntoView {
    let (toggled, set_toggled) = create_signal(false);
    view! {
        <p>"Toggled? " {toggled}</p>
        <ButtonA setter=set_toggled/> // ** Take Note **
    }
}

// The {{SubComponent}} here is 'ButtonA'
#[component]
// ** Take note of the prop's data-type
pub fn ButtonA(setter: WriteSignal<bool>) -> impl IntoView {
    view! {
        <button
            // and now you can freely control the state of the parent component, which of course would impact the parent component too
            on:click=move |_| setter.update(|value| *value = !*value)
        >
            "Toggle"
        </button>
    }
}
```

This pattern is simple, but you should be careful with it: passing around a WriteSignal can make it hard to reason about your code. It is not all clear when or how it will change.

---

How can you pass data from a component, across to its sub-component, and then its grand-sub-component(s), great-grand-sub-component(s), e.t.c.

2. By Providing a Context

Example code of using the Context API. This version is actually a variant of Option 1;

```rs
#[component]
pub fn App() -> impl IntoView {
    let (toggled, set_toggled) = create_signal(false);

    // share `set_toggled` with all children of this component
    provide_context(set_toggled);

    view! {
        <p>"Toggled? " {toggled}</p>
        <Layout/>
    }
}

#[component]
pub fn Layout(set_toggled: WriteSignal<bool>) -> impl IntoView {
    view! {
        <header>
            <h1>"My Page"</h1>
        </header>
        <main>
            <Content set_toggled/>
        </main>
    }
}
#[component]
pub fn Content(set_toggled: WriteSignal<bool>) -> impl IntoView {
    view! {
        <div class="content">
            <ButtonD set_toggled/>
        </div>
    }
}


#[component]
pub fn ButtonD() -> impl IntoView {
    // use_context searches up the context tree, hoping to
    // find a `WriteSignal<bool>`
    // in this case, I .expect() because I know I provided it
    let setter = use_context::<WriteSignal<bool>>()
        .expect("to have found the setter provided");

    view! {
        <button
            on:click=move |_| setter.update(|value| *value = !*value)
        >
            "Toggle"
        </button>
    }
}
```

Again, passing a WriteSignal around should be done with caution, as it allows you to mutate state from arbitrary parts of your code. But when done carefully, this can be one of the most effective techniques for global state management in Leptos: simply provide the state at the highest level you’ll need it, and use it wherever you need it lower down.

---

How can a child send notifications about events that occured inside a child-parent back up to the parent?

3. By Using a Callback (callback function) prop

The callback function prop for the sub-component can accept an event, e.g, `MouseEvent` as its argument.

Here is an example code;

```rs
// ** Take note of how the sub-component takes a callback function that responds to a MouseEvent **
#[component]
pub fn ButtonC<F>({{prop_name}}: F) -> impl IntoView
where
    F: Fn(MouseEvent) + 'static,
{
    view! {
        <button on:click={{prop_name}}>
            "Toggle"
        </button>
    }
}

#[component]
pub fn App() -> impl IntoView {
    let (toggled, set_toggled) = create_signal(false);
    view! {
        <p>"Toggled? " {toggled}</p>
        // ** Take note of how a callback function that changes a signal in the parent is passed to a sub-component, and would get fired up ONLY when a 'click' mouse event happens to the child/sub-component, from inside the parent component of course
        <ButtonC {{prop_name}}=move |_| set_toggled.update(|value| *value = !*value)/>
    }
}
```

---

Another way a child can send notifications about events that occured inside a child-parent back up to the parent is:

4. By using an Event Listener

Here is an example code;

```rs
#[component]
pub fn App() -> impl IntoView {
    let (toggled, set_toggled) = create_signal(false);
    view! {
        <p>"Toggled? " {toggled}</p>
        // note the on:click instead of on_click
        // this is the same syntax as an HTML element event listener
        <ButtonC on:click=move |_| set_toggled.update(|value| *value = !*value)/>
    }
}


#[component]
pub fn ButtonC<F>() -> impl IntoView {
    view! {
        <button>"Toggle"</button>
    }
}
```
