<h4>
BUILDING USER INTERFACES::
COMPONENTS, SUB-COMPONENT PROPS (Part 1).
</h4>

See Part 2 [here](./leptos-analysis-8.md)

1. Specifying a component prop for a sub-component looks like this;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}(
    {{the_name_of_prop_1}}: {{its_data_type}} // e.g, progress: ReadSignal<i32>
) -> impl IntoView {
    view! {
        <progress
          max="50"
          value={{the_name_of_prop_1}}
        />
    }
}

// And here is how you would use such component with a property
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);
    view! {
        <{{TheNameOfTheSubComponent}} {{the_name_of_prop_1}}=count/>
    }
}
```

---

Optional Sub-component Prop

2. Specifying an optional component prop for a sub-component looks like this;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}(
    // You need the '#[prop(optional)]' annotation
    #[prop(optional)]
    {{the_name_of_optional_prop_1}}: {{its_data_type}}
) -> impl IntoView {
    view! {
        <progress
          max="14"
          value={{the_name_optional_prop_1}}
        />
    }
}

// And here is how you would use such component with a property
#[component]
fn App() -> impl IntoView {
    view! {
        <{{TheNameOfTheSubComponent}}/> // notice how {{the_name_of_optional_prop_1}} was entirely skipped!
    }
}
```

---

Optional Sub-component Prop With Default Value

3. Specifying an optional component with a default value when skipped for a sub-component;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}(
    // You need the '#[prop(optional)]' annotation
    #[prop(default = 20)]
    {{the_name_of_optional_prop_1}}: {{its_data_type}}
) -> impl IntoView {
    view! {
        <progress
          max={{the_name_of_optional_prop_1}}
          value="14"
        />
    }
}

// And here is how you would use such component with a property
#[component]
fn App() -> impl IntoView {
    view! {
        <{{TheNameOfTheSubComponent}}/> // ** notice how {{the_name_of_optional_prop_1}} was entirely skipped!
    }
}
```

---

Generic Sub-component Prop

4. Specifying a generic sub-component prop lets you use a value that is of a certain data-type such as, and i32, or use a closure that returns a value of the same data-type (i32).

5. Here is a (trait-style) demonstration of specifying a generic sub-component prop;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}<F>(
    {{the_name_of_prop_1}}: F
) -> impl IntoView
where
  F: Fn() -> i32 + 'static, // "F: impl F() -> i32 + 'static " not allowed in Leptos yet
{
    view! {
        <progress
          max="14"
          value={{the_name_of_prop_1}}
        />
    }
}

// Usage:
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0); // count is an i32 type of value
    let double_count = move || { count() * 2 }; // double_count is closure that returns an i32 type of value
    view! {
        <{{TheNameOfTheSubComponent}} {{the_name_of_prop_1}}=count/> // direct value of type i32
        <{{TheNameOfTheSubComponent}} {{the_name_of_prop_1}}=double_count> // closure that evaluates to a value of type i32
    }
}
```

6. Here is an (attribute-style) demonstration of specifying a generic sub-component prop;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}(
    #[prop(into)]
    {{the_name_of_prop_1}}: Signal<i32>
) -> impl IntoView
{
    view! {
        <progress
          max="14"
          value={{the_name_of_prop_1}}
        />
    }
}

// Usage:
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0); // count is an i32 type of value
    let double_count = move || { count() * 2 }; // double_count is closure that returns an i32 type of value
    view! {
        <{{TheNameOfTheSubComponent}} {{the_name_of_prop_1}}=count/> // direct value of type i32
        // ** Take Note Here **
        <{{TheNameOfTheSubComponent}} {{the_name_of_prop_1}}=Signal::derive(double_count)/> // closure that evaluates to a value of type i32
    }
}
```

---

Optional Generic Sub-component Prop

7. Usually, you can't specify optional generic props for a sub-component, but there is a workaround for this, which is `Option<Box<dyn Fn() -> {{DataType}}>>`.

Example code;

```rs
#[component]
fn {{TheNameOfTheSubComponent}}(
    {{the_name_of_optional_generic_prop_1}}: Option:<Box<dyn Fn() -> i32>>, // change i32 to your desired data-type
) -> impl IntoView {

  {{the_name_of_optional_generic_prop_1}}.map( |{{optional_generic_prop_1_value}}| {
    view! {
        <progress
          max="14"
          value={{optional_generic_prop_1_value}}
        />
    }
  })
}

// Usage:
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(3); // count is an i32 type of value
    let double_count = move || { count() * 2 }; // double_count is closure that returns an i32 type of value
    view! {
        // ** providing no optional generic prop
        <{{TheNameOfTheSubComponent}}/>
        // ** providing the optional generic prop as an i32 type
        <{{TheNameOfTheSubComponent}} {{the_name_of_optional_generic_prop_1}}=count/>
        // ** providing the optional generic optional prop as a double type
        <{{TheNameOfTheSubComponent}} {{the_name_of_optional_generic_prop_1}}=double_count/>
    }
}
```

---

Documenting Components

8. Here is a demonstration of how you would document components. It is very simple. You do so with three slashes, as in, `///`.

```rs
/// Shows progress toward a goal.
#[component]
fn ProgressBar(
    /// The maximum value of the progress bar.
    #[prop(default = 100)]
    max: u16,
    /// How much progress should be displayed.
    #[prop(into)]
    progress: Signal<i32>,
) -> impl IntoView {
    /* ...the body of your component... */
}
```

---

9. Dispalying Data Directly Without Any Additional Wrapping

To do so, you would need to use attribute `#[component(transparent)]` when writing such component that needs to display data directly without any additional wrapping.

"Note: In general, you should not need to use transparent components unless you are creating custom wrapping components that fall into one of these two categories

- Creating wrappers around `<Suspense/>` or `<Transition/>`, which return a transparent suspense structure to integrate with SSR and hydration properly.

- Refactoring `<Route/>` definitions for `leptos_router` out into separate components, because `<Route/>` is a transparent component that returns a `RouteDefinition` struct rather than a view.

---

_Sefini_
