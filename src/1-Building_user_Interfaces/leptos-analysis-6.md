<h4>
BUILDING USER INTERFACES::
ERROR HANDLING
</h4>

You can use Leptos' built-in component `<ErrorBoundary/>` to cater to displaying error messages if an unexpected input has been provided.

`<ErrorBoundary/>` works almost similarly to `<Show/>` conditional render component. Here is how `<ErrorBoundary>` works;
If everything is `Ok(_)` - it renders its children. But if there's an `Err(_)` rendered among those children, it will trigger the `<ErrorBoundary/>`'s `fallback`

Here is an example code of `<ErrorBoundary/>` in action;

```rs
use leptos::*;

#[component]
fn App() -> impl IntoView {
    let (value, set_value) = create_signal(Ok(0));

    // when input changes, try to parse a number from the input
    let on_input = move |ev| set_value(event_target_value(&ev).parse::<i32>());

    view! {
        <h1>"Error Handling"</h1>
        <label>
            "Type a number (or something that's not a number!)"
            <input type="number" on:input=on_input/>
            // If an `Err(_) had been rendered inside the <ErrorBoundary/>,
            // the fallback will be displayed. Otherwise, the children of the
            // <ErrorBoundary/> will be displayed.
            <ErrorBoundary
                // the fallback receives a signal containing current errors
                fallback=|errors| view! {
                    <div class="error">
                        <p>"Not a number! Errors: "</p>
                        // we can render a list of errors
                        // as strings, if we'd like
                        <ul>
                            {move || errors.get()
                                .into_iter()
                                .map(|(_, e)| view! { <li>{e.to_string()}</li>})
                                .collect::<Vec<_>>()
                            }
                        </ul>
                    </div>
                }
            >
                <p>
                    "You entered "
                    // because `value` is `Result<i32, _>`,
                    // it will render the `i32` if it is `Ok`,
                    // and render nothing and trigger the error boundary
                    // if it is `Err`. It's a signal, so this will dynamically
                    // update when `value` changes
                    <strong>{value}</strong> // ** If this cannot be evaluated into its expected type, 'ErrorBoundary' would render its 'fallback' value
                </p>
            </ErrorBoundary>
        </label>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```
