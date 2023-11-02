<h4>
ROUTER::
DEFINING `< Routes/>`
NESTED ROUTING
</h4>

---

A router is the answer to the question, "Give this URL, what should appear on the page?"

- A URL consists of many parts. For example, the URL `https://my-cool-blog.com/blog/search?q=Search#resuts` consists of

  - a sheme: `https`
  - a domain: `my-cool-blog.com`
  - a path: `/blog/search`
  - a query (or search): `?q=Search`
  - a hash: `#result`

- The Leptos Router works with the path and query (`blog/search/?=Search`). Given this piece of the URL, what should the app render on the page?

- A Router also helps with layouts in some cases, which you are about to see.

- For most major applications, most major changes in the state of the app should be reflected in the URL. If you copy and paste the URL and open it in another tab, you should find yourself more or less in the same place.
  In this sense, the router is really at the heart of the global state management for your application. More than anything else, it drives what is displayed on the page.
  The router handles most of this work for you by mapping the current loaction to particular components.

---

1. To use a `Router` in your leptos code, import the relevant types from the `leptos_router` package to your code as dependencies using either syntax;

```rs
use leptos_router::{Route, RouteProps, Router, RouterProps, Routes, RoutesProps};
```

OR

```
use leptos_router::*;
```

2. Routing behavior is provided by the `<Router/>` component. This should usually be somewhere near the root of your application, the rest of the app.

3. You shouldn’t try to use multiple `<Router/>`s in your app. Remember that the router drives global state: if you have multiple routers, which one decides what to do when the URL changes?

4. The `<Routes/>` component is where you define all the routes to which a user can navigate in your application. Each possible route is defined by a `<Route/>` component.

5. Everything outside `<Routes/>` will be present on every page, so you leave things like a navigation bar or many outside a `<Routes/>`.

6. Here is an example code that uses a `<Router/>` and `<Routes/>`, and a couple `<Route/>`;

```rs
use leptos::*;
use leptos_router::*;

#[component]
pub fn App() -> impl IntoView {
  view! {
    <Router>
      <nav>
        /* ... */
      </nav>
      <main>
        // all our routes will appear inside <main>
        <Routes>
          /* ... */
          <Route path="/" view=Home/>
          <Route path="/users" view=Users/>
          <Route path="/users/:id" view=UserProfile/>
          <Route path="/*any" view=|| view! { <h1>"Not Found"</h1> }/>
        </Routes>
      </main>
    </Router>
  }
}
```

7. `<Route/>` takes a _path_ and a _view_. When the current location matches path, the view will be created and displayed. The `path` can include;

- a static path (`/user`),
- dynamic, named parameters beginning with a colon (`/:id`),
- and/or a wildcard beginning with an asterisk (`/user/*any`)

8. view takes a `Fn() -> impl IntoView`. If a component has no props, it can be passed directly into the `view`. In this case, `view=Home` is just a shorthand for `|| view! { <Home/> }`.

---

CONDITIONAL ROUTES

9. Examine this conditional code for inspiration;

```rs
// ✅ do this instead!
view! {
  <Routes>
    // parent route
    <Route path="/" view=move || {
      view! {
        // only show the outlet if data have loaded
        <Show when=|| is_loaded() fallback=|| view! { <p>"Loading"</p> }>
          <Outlet/>
        </Show>
      }
    }>
      // nested child route
      <Route path="/" view=Home/>
    </Route>
  </Routes>
}
```

---

NESTED ROUTING

10. Example nested routing code (with `<Outlet/>` specified);

```rs
<Routes>
  <Route path="/users" view=Users> // * specify '<Outlet/> inside 'Users'
  // ... <Outlet/> is how you tell a parent component where to render any nested components inside its view
    <Route path=":id" view=UserProfile/>
    <Route path="" view=NoUser/>
  </Route>
</Routes>
```

_Now:_

- if you go to path `/users/3`, the path matches `<Users/>` and `<UserProfile/>`.
- if you go to `/users`, the path matches `<Users/>` and `<NoUser/>`.

_Each 'path' can match mutiple routes, as in, each URL can render views provided by multiple `Route/>` components nested inside it, at the same time, on the same page. You can imagine it being useful for something like a contacts app for example._

11. Another nested routing code example;

```rs
<Routes>
  <Route path="/contacts" view=ContactList> // specify <Outlet/> inside 'ContactList' to state where its children will be nested, i.e, the view for its "" and ":id" paths below
    <Route path=":id" view=ContactInfo>
      <Route path="" view=EmailAndPhone/>
      <Route path="address" view=Address/>
      <Route path="messages" view=Messages/>
    </Route>
    <Route path="" view=|| view! {
      <p>"Select a contact to view more info."</p>
    }/>
  </Route>
</Routes>
```

PARAMS AND QUERIES

12. Accessing params and queries is pretty simple with a couple of hooks.

- `use_query` (typed) or `use_query_map` (untyped, flexible, and recommended).
- `use_params` (typed) or `use_params_map` (untyped, flexible, and recommended).

13. It is recommended that you use `use_query_map` and `use_params_map` for ease, but in case that you're interested in using thier counterpart, then visit this URL (Parmas and Queries more info)[https://leptos-rs.github.io/leptos/router/18_params_and_queries.html].

_Look at item 21 (the routing comprehensive demo code) for inspiration of how you might use a `use_param_map` for example._

---

THE `<A/>` COMPONENT

_N.B: Note that `<A/>` is basically an `<a>` with some upgrades which you will soon see._

14. The `Router` will only try to do a client-side navigation both for `<a>` and `<A>` when it's pretty sure that it can handle it (on the client-side).

15. Both `<a>` and `<A/>` will do client-side navigation unless these circumstances occur;

- the click event (to a link) has had `prevent_default()` called on it.
- the `Meta`, `Alt`, `Ctrl`, or `Shift` keys were held during a click.
- the `<a>` or `<A/>` has a `target` or `download` attributes, or `rel="external"`.
- the link has a different origin from the current location. For this use `<a rel="external">`.

16. Here are `<A/>`'s upgrades over `<a>`, hence you should choose `<A/>` over `<a>` under the following circumstances;

- you need this form of quick, short relative nested-route for path `/path/:id`, as `<A href="1"`, instead of `<a href="/path/1"`.
- you need `aria-current` attribute to `page` auto-set for you.

---

NAVIGATING PROGRAMMATICALLY

17. Using links and forms to navigate is the best solution for accessibility and graceful degradation, hence, you would mostly use either `<a>` and `<form>`, or their upgraded counterparts - `<A/>` and `<Form/>`. Bear this in mind.

18. On the occassion that you want to navigate programmatically (as, in call a function that can and will navigate to a new page) though, you the `use_navigate` function.

19. `use_navigate` when used, should be called on the client. It does nothing when during server rendering.

20. Here is an example code snippet of `use_navigate`;

```rs
let navigate = leptos_router::use_navigate();
navigate("/to-some-link", Default::default());
```

_N.B: Its (required) second argument is a set of `NavigateOptions` - (NavigateOption)[https://docs.rs/leptos_router/latest/leptos_router/struct.NavigateOptions.html], which has the following construction function, as in, default implmentations;_

```rs
impl Default for NavigateOptions {
    fn default() -> Self {
        Self {
            resolve: true,
            replace: false,
            scroll: true,
            state: State(None),
        }
    }
}
```

For accessibility reasons, Do not programmatically navigate to a link like this;

```rs
<button on:click=move |_| navigate(/* ... */)>
```

Again, any `on:click` that navigates should be an `<a>`, for reasons of accessibility.

---

ROUTING COMPREHENSIVE DEMO CODE

21. Examine the following comprehensive routing code for some routing guide and inspiration;

```rs
use leptos::*;
use leptos_router::*;

#[component]
fn App() -> impl IntoView {
    view! {
        <Router>
            <h1>"Contact App"</h1>
            // this <nav> will show on every routes,
            // because it's outside the <Routes/>
            // note: we can just use normal <a> tags
            // and the router will use client-side navigation
            <nav>
                <h2>"Navigation"</h2>
                <a href="/">"Home"</a>
                <a href="/contacts">"Contacts"</a>
            </nav>
            <main>
                <Routes>
                    // / just has an un-nested "Home"
                    <Route path="/" view=|| view! {
                        <h3>"Home"</h3>
                    }/>
                    // /contacts has nested routes
                    <Route
                        path="/contacts"
                        view=ContactList
                      >
                        // if no id specified, fall back
                        <Route path=":id" view=ContactInfo>
                            <Route path="" view=|| view! {
                                <div class="tab">
                                    "(Contact Info)"
                                </div>
                            }/>
                            <Route path="conversations" view=|| view! {
                                <div class="tab">
                                    "(Conversations)"
                                </div>
                            }/>
                        </Route>
                        // if no id specified, fall back
                        <Route path="" view=|| view! {
                            <div class="select-user">
                                "Select a user to view contact info."
                            </div>
                        }/>
                    </Route>
                </Routes>
            </main>
        </Router>
    }
}

#[component]
fn ContactList() -> impl IntoView {
    view! {
        <div class="contact-list">
            // here's our contact list component itself
            <div class="contact-list-contacts">
                <h3>"Contacts"</h3>
                <A href="alice">"Alice"</A>
                <A href="bob">"Bob"</A>
                <A href="steve">"Steve"</A>
            </div>

            // <Outlet/> will show the nested child route
            // we can position this outlet wherever we want
            // within the layout
            <Outlet/>
        </div>
    }
}

#[component]
fn ContactInfo() -> impl IntoView {
    // we can access the :id param reactively with `use_params_map`
    let params = use_params_map();
    let id = move || params.with(|params| params.get("id").cloned().unwrap_or_default());

    // imagine we're loading data from an API here
    let name = move || match id().as_str() {
        "alice" => "Alice",
        "bob" => "Bob",
        "steve" => "Steve",
        _ => "User not found.",
    };

    view! {
        <div class="contact-info">
            <h4>{name}</h4>
            <div class="tabs">
                <A href="" exact=true>"Contact Info"</A>
                <A href="conversations">"Conversations"</A>
            </div>

            // <Outlet/> here is the tabs that are nested
            // underneath the /contacts/:id route
            <Outlet/>
        </div>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```
