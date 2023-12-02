<h3 style="text-align:center;">Component Properties, Documenting Components, Displaying Data Without Any Additional Wrapping (Part 1) </h3>

See Part 2 of specifying component properties in the context of passing properties down from a parent component to a a sub-component, one or more sub-component layers deep [here](./leptos-analysis-8.md)

---

#### [Required Component Properties](#a---component-properties),

#### [Optional Component Properties](#b---optional-component-properties),

#### [Optional Component Properties With Default Values](#c---optional-component-props-with-default-values)

#### [Required Generic Component Properties](#d---required-generic-component-props)

#### [Optional Generic Component Properties](#e---optional-generic-component-props)

#### [Documenting Components](#f---documenting-components)

#### [Displaying Data Without Any Additional Wrapping](#g---displaying-data-directly-without-any-additional-wrapping)

---

##### A - Component Properties

- You can specify that your component require properties to be passed in to it when rendered in the `view!{...}` of another component (its parent component), or perhaps nested inside a child HTML element, e.g, `<div></div>` of another component, as its descendant.

- The syntax to specify a component's property or props/properties looks like this:

```rs
use leptos::*;

#[component]
fn {{TheNameOfYourComponent}}(
    {{a_prop_name}}: {{its_data_type}},
    {{another_prop_name}}: i32 // i32, for example
) -> impl IntoView {

    ...

}
```

<details>
<summary>Usage Example</summary>

Module `leptos_project/src/lib.rs`:

```rs
use leptos::*;

#[component]
fn SimpleProgressBar(
    current_progress: u32
) -> impl IntoView {

    view! {
        <progress
          max="50"
          value=current_progress
        />
    }
}
```

Module `leptos_project/src/main.rs`:

```rs
// use the custom component 'SimpleProgressBar'
use leptos_project::SimpleProgressBar;
// OR
// use super::SimpleProgressBar;
use leptos::*;

/* And here is how you would render SimpleProgressBar such that you
   provide it its required property 'current_progress'.
   In this example, 'current_progress' is given a signal value
*/
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);
    view! {
        <div>
          <p>"The current progress is at:"</p>
          <SimpleProgressBar current_progress=count/>
        </div>
    }
}
```

</details>

##### B - Optional Component Properties

- Specifying an optional component prop for component looks like this:

```rs
#[component]
fn {{TheNameOfYourComponent}}(
    /* You need the '#[prop(optional)]' annotation to make
       any property optional
    */
    #[prop(optional)]
    {{the_name_of_the_optional_prop}}: {{its_data_type}}

) -> impl IntoView {

    view! {
        <progress
          max="14"
          value={{the_name_of_the_optional_prop}}
        />
    }
}

// And here is how you could use a component with an optional property
#[component]
fn App() -> impl IntoView {
    view! {
        <{{TheNameOfYourComponent}}/>
    }
}
```

> You can provide a value for an optional property too like this: <div><code><{{TheNameOfYourComponent}} the_name_of_the_optional_prop={{a_signal_or_value}} /></code></div>

##### C - Optional Component Props With Default Values

- Specifying an optional component with a default value when skipped for a component looks like this:

```rs
#[component]
fn {{TheNameOfYourComponent}}(
    // You need the '#[prop(optional)]' annotation
    #[prop(default = 20)]
    {{the_name_of_the_optional_prop}}: {{its_data_type}}
) -> impl IntoView {
    view! {
        <progress
          max={{the_name_of_the_optional_prop}}
          value="14"
        />
    }
}

// And here is how you would use such component with a property
#[component]
fn App() -> impl IntoView {
    view! {
        <{{TheNameOfTheSubComponent}}/>
        /* notice how {{the_name_of_the_optional_prop}} was
           entirely skipped!
        */
    }
}
```

##### D - Required Generic Component Props

- Specifying a required generic sub-component prop lets you require a property of a simple concrete optioned data-type such as `Option<i32>`, `Option<bool>`, or perhaps even a property which is itself a closure that returns a value of a certain data-type, e.g, an `i32`, just like your good' ol derived signals (for example; `move || count() *2`).

- You could create a generic property for a component either by

  - requiring the component prop's data type to implement `Fn`, as shown below
  - or by annotating the target property with attribute `#[prop(into)]`, as shown further below

- Below is a (traity-style) code demonstration of specifying a required generic component prop:

```rs
#[component]
fn {{TheNameOfYourComponent}}<F>(
    {{the_name_of_the_prop}}: F
) -> impl IntoView

where
  F: Fn() -> i32 + 'static,
  /* Please note:
     " F: impl F() -> i32 + 'static " is not allowed
     in Leptos just yet
  */

{

    view! {
        <progress
          max="14"
          value={{the_name_of_the_prop}}
        />
    }

}

// Usage:
#[component]
fn App() -> impl IntoView {


    let (count, set_count) = create_signal(0); // ReadSignal<i32>
    // 'double_count' is a derived signal that returns an i32
    let double_count = move || { count() * 2 };

    view! {
        <{{TheNameOfYourComponent}} {{the_name_of_the_prop}}=count/>

        <{{TheNameOfYourComponent}}
          {{the_name_of_the_prop}}=double_count
        />
    }
}
```

> In the above code demo, notice that you can pass a prop of type `ReadSignal<T>`, e.g. `ReadSignal<i32>` to a component that requires a prop of type `Fn() -> i32 + 'static` just fine, as long as the type `T` in the `ReadSignal<T>` matches what the `Fn` returns.

- Here is an (attribute-style) code demonstration of specifying a required generic component prop:

```rs
#[component]
fn {{TheNameOfYourComponent}}(
    #[prop(into)]
    {{the_name_of_the_prop}}: Signal<i32>
) -> impl IntoView
{
    view! {
        <progress
          max="14"
          value={{the_name_of_the_prop}}
        />
    }
}

// Usage:
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(0);
    let double_count = move || { count() * 2 };
    view! {
        <{{TheNameOfYourComponent}} {{the_name_of_the_prop}}=count/>
        // Take note here
        <{{TheNameOfYourComponent}}
          {{the_name_of_the_prop}}=Signal::derive(double_count)/>
    }
}
```

##### E - Optional Generic Component Props

- Usually, you can't specify optional generic props for a sub-component, but there is a workaround for this, which is `Option<Box<dyn Fn() -> {{DataType}}>>`.

- Optional generic component prop example code:

```rs
#[component]
fn {{TheNameOfYourComponent}}(
    {{the_name_of_the_optional_generic_prop}}:
      Option:<Box<dyn Fn() -> i32>>
) -> impl IntoView {

  {{the_name_of_the_optional_generic_prop}}.map(
      |{{the_optional_generic_prop_value}}| {
          view! {
              <progress
                max="14"
                value={{the_optional_generic_prop_value}}
              />
          }
      }
  )

}

// Usage:
#[component]
fn App() -> impl IntoView {
    let (count, set_count) = create_signal(3);
    let double_count = move || { count() * 2 };

    view! {
        /* providing no prop value for the component's optional
           generic prop
        */
        <{{TheNameOfYourComponent}}/>

        /* providing a ReadSignal<i32> as the component's optional
           generic prop' value
        */
        <{{TheNameOfYourComponent}}
          {{the_name_of_optional_generic_prop_1}}=count
        />

        /* providing a derived-signal as the component's optional
           generic prop' value
        */
        <{{TheNameOfYourComponent}}
          {{the_name_of_optional_generic_prop_1}}=double_count
        />
    }
}
```

##### F - Documenting Components

- You document a component with three trailing forward-slashes, as in, `///`, the a newline before your component's definition. Here is a demonstration below:

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

##### G - Displaying Data Directly Without Any Additional Wrapping

- `#[component(transparent)]` is the attribute that you need when writing a component that needs to display data directly without going through any transformation to `impl IntoView`.

- In general, you would not need to use transparent components unless, you are creating custom wrapping components that fall into one of these two categories:

  - Creating wrappers around `<Suspense/>` or `<Transition/>`, which return a transparent suspense structure to integrate with SSR and hydration properly.

  - Refactoring `<Route/>` definitions for `leptos_router` out into separate components, because `<Route/>` is a transparent component that returns a `RouteDefinition` struct rather than a view. "
