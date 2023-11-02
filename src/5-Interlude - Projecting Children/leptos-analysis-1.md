<h4>
INTERLUDE: PROJECTING CHILDREN
</h4>

You would sometimes find that you need to be able to project components as childrens to deeper sub-components.

1. Use the `store_value` primitive to project children. This essentially stores a value in the reactive system, handling ownership off to the framework in exchange for a reference that is, like signals, `Copy` and `'static`, which we can access or modify through certain methods.

Example code;

```rs
pub fn LoggedIn<F, IV>(fallback: F, children: ChildrenFn) -> impl IntoView
where
    F: Fn() -> IV + 'static,
    IV: IntoView,
{
    let fallback = store_value(fallback);
    let children = store_value(children);
    view! {
        <Suspense
            fallback=|| ()
        >
            <Show
                when=|| todo!()
                fallback=move || fallback.with_value(|fallback| fallback())
            >
                {children.with_value(|children| children())}
            </Show>
        </Suspense>
    }
}
```

At the top level, we store both `fallback` and `children` in the reactive scope owned by `LoggedIn`. Now we can simply move those references down through other layers into the `<Show/>` component and call them there.

_Note that this works because `<Show/>` and `<Susspense/>` only need an immutable refernce to their children (which `.with_value` can give it), not ownership._

2. In other cases, you may need to project owned props through a function that takes `ChildrenFn` and therfore needs to be called more than once. In this case, you may find the `clone`: helper in the `view` macro helpful.

Consider this code example

```rs
#[component]
pub fn App() -> impl IntoView {
    let name = "Alice".to_string();
    view! {
        <Outer>
            <Inner>
                <Inmost name=name.clone()/>
            </Inner>
        </Outer>
    }
}

#[component]
pub fn Outer(ChildrenFn) -> impl IntoView {
    children()
}

#[component]
pub fn Inner(ChildrenFn) -> impl IntoView {
    children()
}

#[component]
pub fn Inmost(ng) -> impl IntoView {
    view! {
        <p>{name}</p>
    }
}

// You should not name=name.clone(), as this gives the error;
// connot move out of `name`, a captured variable in an `Fn` closure.
// use the `clone:name` syntax instead;
view! {
    <Outer>
      <Inner clone:name>
        <Inmost name=name.clone()/>
    </Outer>
}
```

These issues can be a little tricky to understand or debug because of the opacity of the `view` macro. But in general, they can always be solved.
