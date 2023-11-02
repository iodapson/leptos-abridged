<h4>
REACTIVITY::
RESPONDING TO CHANGES WITH 'create_effect'
</h4>

You can decide to research the docs for broader information and methods beyond what's covered in this entire page here - https://docs.rs/leptos_reactive/latest/leptos_reactive/index.html

---

ABOUT 'create_effect'

Without effects, signals can change within the reactive system but never observed in a way that interacts with the outside world.
Without signals, effects run once but never again, as there's no observable value to subscribe to.

Why Do Effects (created by `create_effect()`) Exist

They exist (only) to synchronize the reactive system with the non-reactive world outside it.

`create_effect` - takes a function as its argument. It immediately runs the function, and if you access any reactive signal inside that function, it registers the fact that the effect depends on that signal with the reactive runtime.
Whenever one of the signals that the effect depends on (i.e, subscribes to) changes, the effect runs again.

---

1. Here is how you create a `create_effect` in leptos

`create_effect` example code;

```rs
let (a, set_a) = create_signal(0);
let (b, set_b) = create_signal(0);

create_effect(move |_| {
  // immediately prints "Value: 0" and subscribes to `a`
  log::debug!("Value: {}", a());
});
```

<h6>N.B:</h6>

By default, effects do not run on the server. This means you can call browser-specific APIs within the effect function without causing issues. If you need an effect to run on the server, use `create_isomorphic_effect`.

2. Here is another `create_effect` example code;

```rs
let (first, set_first) = create_signal(String::new());
let (last, set_last) = create_signal(String::new());
let (use_last, set_use_last) = create_signal(true);

// this will add the name to the log
// any time one of the source signals changes
create_effect(move |_| {
    log(
        // 'first()' and 'last()' will be auto-subscribed to if 'use_last()' evaluates to true, and will be dynamically removed from its subscribe-list when false. Leptos' effects are that smart!
        if use_last() {
            format!("{} {}", first(), last())
        } else {
            first()
        },
    )
});
```

3. Effects That Can Run Both On the Client and Server - `create_isomorphic_effect`.

Here is an example code;

```rs
let (a, set_a) = create_signal(0);

// ✅ use effects to interact between reactive state and the outside world
create_isomorphic_effect(move |_| {
  // immediately prints "Value: 0" and subscribes to `a`
  log::debug!("Value: {}", a.get());
});

set_a.set(1);
// ✅ because it's subscribed to `a` and its value is changed here, the effect reruns and prints "Value: 1"

```

3. Cancellable Effects

There's an effect creation function called `watch`. Like `create_resource`, `watch` takes a first argument, which is reactively tracked, and a second, which is not. Whenever a reactive value in its "deps" argument is changed, the "callback" is run. `watch` returns a function that can be called to stop tracking the dependencies.

Example code;

```rs
let (num, set_num) = create_signal(0);

let stop = watch(
    // num will be reactively tracked, depended on, subscribed to. The "deps" closure -
    move || num.get(),
    // prev_num will not be reactively tracked, the "callback"
    move |num, prev_num, _| {
        log::debug!("Number: {}; Prev: {:?}", num, prev_num);
    },
    // "immediate" boolean
    false,
);

set_num.set(1); // > "Number: 1; Prev: Some(0)"

stop(); // stop watching / tracking / depending-on /subcription

set_num.set(2); // (nothing happens)
```
