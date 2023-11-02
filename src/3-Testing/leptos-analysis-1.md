<h4>
TESTING
</h4>

_NOTE: You can find examples to testing Leptos code from the 'examples' directory of the Leptos repository here - https://github.com/leptos-rs/leptos/tree/main/examples_

1. Test Business Logic With Ordinary Rust Tests

In many cases, it makes sense to pull the logic out of your components and test it separately. For some simple componenents, ther's no particular logic to test, but for many it's worth using a testable wrapping type and implementing the logic in ordinary Rust `impl` blocks.

Example code;

```rs
pub struct Todos(Vec<Todo>);

impl Todos {
    pub fn num_remaining(&self) -> usize {
        todos.iter().filter(|todo| !todo.completed).sum()
    }
}

#[cfg(test)]
mod tests {
    #[test]
    fn test_remaining() {
        // ...
    }
}

#[component]
pub fn TodoApp() -> impl IntoView {
    let (todos, set_todos) = create_signal(Todos(vec![Todo { /* ... */ }]));
    // ✅ this has a test associated with it
    let num_remaining = move || todos.with(Todos::num_remaining);
}
```

In general, the less of your logic is wrapped into your components themselves, the more idiomatic your code with feel and the easier it will be to test.

2. Test Components With End-to-End (e2e) Testing

- `wasm-bindgen-test` with counter example;

```rs
#[wasm_bindgen_test]
fn clear() {
    let document = leptos::document();
    let test_wrapper = document.create_element("section").unwrap();
    let _ = document.body().unwrap().append_child(&test_wrapper);

    mount_to(
        test_wrapper.clone().unchecked_into(),
        || view! { <SimpleCounter initial_value=10 step=1/> },
    );

    let div = test_wrapper.query_selector("div").unwrap().unwrap();
    let clear = test_wrapper
        .query_selector("button")
        .unwrap()
        .unwrap()
        .unchecked_into::<web_sys::HtmlElement>();

    clear.click();

assert_eq!(
    div.outer_html(),
    // here we spawn a mini reactive system to render the test case
    run_scope(create_runtime(), || {
        // it's as if we're creating it with a value of 0, right?
        let (value, set_value) = create_signal(0);

        // we can remove the event listeners because they're not rendered to HTML
        view! {
            <div>
                <button>"Clear"</button>
                <button>"-1"</button>
                <span>"Value: " {value} "!"</span>
                <button>"+1"</button>
            </div>
        }
        // the view returned an HtmlElement<Div>, which is a smart pointer for
        // a DOM element. So we can still just call .outer_html()
        .outer_html()
    })
);
}
```

- `wasm-bindgen-test` with counters_stable.

This more developed test suite uses a system of fixtures to refactor the manual DOM manipulation of the `counter` tests and easily test a wide range of cases.

```rs
use super::*;
use crate::counters_page as ui;
use pretty_assertions::assert_eq;

#[wasm_bindgen_test]
fn should_increase_the_total_count() {
    // Given
    ui::view_counters();
    ui::add_counter();

    // When
    ui::increment_counter(1);
    ui::increment_counter(1);
    ui::increment_counter(1);

    // Then
    assert_eq!(ui::total(), 3);
}
```

- "Playwright" with counters_stable.

These tests use the common JavaScript testing tool Playwright to run end-to-end tests on the same example, using a library and testing approach familiar to many who have done frontedn development before.

Example code;

```rs
import { test, expect } from "@playwright/test";
import { CountersPage } from "./fixtures/counters_page";

test.describe("Increment Count", () => {
  test("should increase the total count", async ({ page }) => {
    const ui = new CountersPage(page);
    await ui.goto();
    await ui.addCounter();

    await ui.incrementCount();
    await ui.incrementCount();
    await ui.incrementCount();

    await expect(ui.total).toHaveText("3");
  });
});

```
