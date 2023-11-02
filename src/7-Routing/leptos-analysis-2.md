<h4>
LEPTOS ROUTING::
`< FORM />`
</h4>

THE `<Form/>` COMPONENT

1. The router provides a `<Form>` component, which works like the HTML `<form>` element, but uses client-side navigations instead of full page reloads.

2. `<Form/>` works with both `GET` and `POST` requests. With `method="GET"`, it will navigate to the URL encoded in the form data. With `method="POST"` it will make a `POST` request and handle the server’s response.

3. `<Form/>` provides the basis for some components like `<ActionForm/>` and `<MultiActionForm/>` that we’ll see in later chapters. But it also enables some powerful patterns of its own.

4. Here is an example `<Form/>` code for searching with a URL query (as a signal) tied to an (uncontrolled) input element as its source;

```rs
async fn fetch_results() {
    // some async function to fetch our search results
}

#[component]
pub fn FormExample() -> impl IntoView {
    // reactive access to URL query strings
    let query = use_query_map();
    // search stored as ?q=
    let search = move || query().get("q").cloned().unwrap_or_default();
    // a resource driven by the search string
    let search_results = create_resource(search, fetch_results);

    view! {
        <Form method="GET" action="">
            <input type="search" name="search" value=search/>
            <input type="submit"/>
        </Form>
        <Transition fallback=move || ()>
            /* render search results */
        </Transition>
    }
}
```

In the above code sample,

- whenever the `submit` type input is clicked, the `<Form/>` will navigate to `?q={search}`, but because this navigation is done on the client side, there's no page flicker or reload.
- because `search` is the source signal for the `search_results` resource, this triggers `search_results` to reload its resource.
- the `<Transition/>` continues displaying the current search results until the new ones have loaded, and when they complete, it switches to displaying the new result until yet another one reloads (again).

5. Here is an example `<Form/>` code for searching with a URL query (as a signal) tied to an (controlled) input element as its source;

```rs
async fn fetch_results() {
    // some async function to fetch our search results
}

#[component]
pub fn FormExample() -> impl IntoView {
    // reactive access to URL query strings
    let query = use_query_map();
    // search stored as ?q=
    let search = move || query().get("q").cloned().unwrap_or_default();
    // a resource driven by the search string
    let search_results = create_resource(search, fetch_results);

    view! {
        <Form method="GET" action="">
            // pay attention to the input here
            <input type="search"
              name="search"
              value=search
              oninput="this.form.requestSubmit()"
            />
        </Form>
        <Transition fallback=move || ()>
            /* render search results */
        </Transition>
    }
}
```

In the above sample code;

- The `submit` type input has been removed in favor of a input with HTML attribut `oninput` that does its own submission, such that, it reloads data each time its received input changes.
- the input's `oninput`'s `this.form` gives the form element (in this case `<Form/>`) that the input is attached to.
- the input's `oninput`'s `this.form.requestSubmit()` is the JavaScript function that actually fires the `submit` event on the `<form>`, which is caught by the `<Form/>`, just as if you had clicked a submit button.

6. Here is a bigger code example of using `<Form/>`;

```rs
use leptos::*;
use leptos_router::*;

#[component]
fn App() -> impl IntoView {
    view! {
        <Router>
            <h1><code>"<Form/>"</code></h1>
            <main>
                <Routes>
                    <Route path="" view=FormExample/>
                </Routes>
            </main>
        </Router>
    }
}

#[component]
pub fn FormExample() -> impl IntoView {
    // reactive access to URL query
    let query = use_query_map();
    let name = move || query().get("name").cloned().unwrap_or_default();
    let number = move || query().get("number").cloned().unwrap_or_default();
    let select = move || query().get("select").cloned().unwrap_or_default();

    view! {
        // read out the URL query strings
        <table>
            <tr>
                <td><code>"name"</code></td>
                <td>{name}</td>
            </tr>
            <tr>
                <td><code>"number"</code></td>
                <td>{number}</td>
            </tr>
            <tr>
                <td><code>"select"</code></td>
                <td>{select}</td>
            </tr>
        </table>

        // <Form/> will navigate whenever submitted
        <h2>"Manual Submission"</h2>
        <Form method="GET" action="">
            // input names determine query string key
            <input type="text" name="name" value=name/>
            <input type="number" name="number" value=number/>
            <select name="select">
                // `selected` will set which starts as selected
                <option selected=move || select() == "A">
                    "A"
                </option>
                <option selected=move || select() == "B">
                    "B"
                </option>
                <option selected=move || select() == "C">
                    "C"
                </option>
            </select>
            // submitting should cause a client-side
            // navigation, not a full reload
            <input type="submit"/>
        </Form>

        // This <Form/> uses some JavaScript to submit
        // on every input
        <h2>"Automatic Submission"</h2>
        <Form method="GET" action="">
            <input
                type="text"
                name="name"
                value=name
                // this oninput attribute will cause the
                // form to submit on every input to the field
                oninput="this.form.requestSubmit()"
            />
            <input
                type="number"
                name="number"
                value=number
                oninput="this.form.requestSubmit()"
            />
            <select name="select"
                onchange="this.form.requestSubmit()"
            >
                <option selected=move || select() == "A">
                    "A"
                </option>
                <option selected=move || select() == "B">
                    "B"
                </option>
                <option selected=move || select() == "C">
                    "C"
                </option>
            </select>
            // submitting should cause a client-side
            // navigation, not a full reload
            <input type="submit"/>
        </Form>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```
