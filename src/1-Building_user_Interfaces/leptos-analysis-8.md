<h4>
BUILDING USER INTERFACES::
COMPONENT CHILDREN (PASSING COMPONENT[S] TO ANOTHER COMPONENT AS PROPS)
</h4>

Pass A Component To Another Component

In leptos, there are two ways to go about passing a component into another component. They are:

1. `render_prop`: This is a special property that takes a function that return a view.

Here is an example code;

```rs
#[component]
pub fn TakesRenderProp<F>(
    render_prop: F
) -> impl IntoView
where
    F: Fn() -> IV,
    IV: IntoView,
{
    view! {
        <h2>"Render Prop"</h2>
        {render_prop()}
    }
}

// usage:
pub fn App() -> IntoView {
    view! {
        <TakesRenderProp render_prop=|| view! {
            <p>"Hi, there!"</p>
        }>
    }
}
```

`render_prop` is a function, so you can call it to generate the appropriate view, as done in the sample code above.

2. `Children`: This is a special component property that includes anything you pass a child to the component (with the `children` property).

Here is an example code;

```rs
#[component]
pub fn TakesChildren(children: Children) -> IntoView {
    view! {
        <h2>"Children"</h2>
        {children()}
    }
}

// usage:
pub fn App() -> IntoView {
    <TakesChildren>
      "Some text"
      <span>"A span"</span>
    </TakesChildren>
}
```

`children` is a function, so you can call it to generate the appropriate view, as done in the sample code above.

<h6>Please note:</h6>

- _`ChildrenFn` can be used if you need to call `children` more than once. And `ChildrenMut` is available if you not only need to call `children` more than once, but also actually mutate its value._

- _`Children` in particular, is an alias for `Box<dyn FnOnce() -> Fragment>`. And the `Fragment` type is basically a way of wrapping `Vec<View>`. You can insert it anywhere into your view._

3. You can use `render_props` and `Children` together inside the same component.

Here is an example code;

```rs
#[component]
pub fn TakesRenderPropAndChildren<F, IV>(
    /// Takes a function (type F) that returns anything that can be
    /// converted into a View (type IV)
    render_prop: F,
    /// `children` takes the `Children` type
    children: Children,
) -> impl IntoView
where
    F: Fn() -> IV,
    IV: IntoView,
{
    view! {
        <h2>"Render Prop"</h2>
        {render_prop()}

        <h2>"Children"</h2>
        {children()}
    }
}

// usage:
view! {
    <TakesRenderPropAndChildren render_prop=|| view! { <p>"Hi, there!"</p> }>
        // these get passed to `children`
        "Some text"
        <span>"A span"</span>
    </TakesRenderPropAndChildren>
}
```

---

Iterating/Manipulating Children, or ChildrenFn, or ChildrenMut

Here is an example code;

```rs
#[component]
pub fn WrapsChildren(children: Children) -> impl IntoView {
    // Fragment has `nodes` field that contains a Vec<View>
    let children = children()
        .nodes
        .into_iter()
        .map(|child| view! { <li>{child}</li> })
        .collect_view();

    view! {
        <ul>{children}</ul>
    }
}

// usage:
pub fn App() -> IntoView {
    view! {
    <WrapsChildren>
        "A"
        "B"
        "C"
    </WrapsChildren>
}
}
```
