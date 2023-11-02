<h4>
REACTIVITY::
WORKING WITH SIGNALS (THE DIFFERENT SIGNAL METHODS)
</h4>

There Are Four Basic Signal Operations:

- `.get()`
- `.with()`
- `.update()`
- `.set()`

1. The signal getters:

- `.get()` clones the current value of the signal and tracks any future changes to the value reactively.

Example code:

```rs
let (count, set_count) = create_signal(0);
view! {
    <p>"The count is: " {count.get()}</p>
    // OR
    <p>"The count is: " {count()}</p>
}
```

- `.with()` takes a function, which receives the current value of the signal by reference (&T), and then tracks any future changes

Example code:

```rs
let (count, set_count) = create_signal(0);
view! {
    <p>"The count is: " {count.with(|n| n)}</p> // n is a reference, as in, &T.
    // To make .with() analogous with .get(), do this
    <p>"The count is: " {count.with(|n| n.clone())}</p>
}
```

2. The setters:

- `.update()` takes a function, which recieves a mutable reference to the current value of the signal (&mut T), and notifies any subsribers that they need to update. (`.update()` doesn't return the value returned by the closure, but you can use `.try_update()` if you need to; for example, if you're removing an item from a `Vec<_>` and want the removed item.)

Example code:

```rs
let (count, set_count) = create_signal(0);
view! {
    <button on:click=move |_| {set_count.update(|n| *n * 2)}>
      "doubler button"
    </button>
}
```

- `.set()` replaces the current value of the signal with an entirely new value, and notifies any subscribers that they need to update.

Example code:

```rs
let (count, set_count) = create_signal(0);
view! {
    <button on:click=move |_| {set_count.set(1)}>
      "reset button"
    </button>
    // OR
    <button on:click=move |_| {set_count(1)}>

    // To make .set() analogous with .update(), do this
    <button on:click=move |_| {set_count.update(|n| *n = 1)}>
}
```

---

The Efficient Signal Methods.

It is much efficient to use `.with()`, and `.update()` in general, especially for large signal values.

Examine this `.with()`&`.update()` code snippet example for reference;

```rs
let (names, set_names) = create_signal(Vec::new());
if names.with(|names| names.is_empty()) {
    set_names.update(|names| names.push("Alice".to_string()));
}
```

Or this better alternate `.with()`&`.update()` code snippet;

```rs
let (names, set_names) = create_signal(Vec::new());
if names.with(Vec::is_empty) {
    set_names.update(|names| names.push("Alice".to_string()));
}
```

---

3. Access Multiple Signals In One Go.

You have macros like `with!`, `update!`, `with_value`, and `update_value!` to acccess multiple signals and perform various operations with them in one go.

Here is a `.with!` code example snippet;

```rs
let name = move || with!(|first, middle, last| format!("{first} {middle} {last}"));
```

The above code snippet example is a replacement for this much longer version;

```rs
let name = move || {
    first.with(|first| {
        middle.with(|middle| last.with(|last| format!("{first} {middle} {last}")))
    })
};
```

Making signals depend on each other. (Changing A Signal Based On A Change That Happens To Another Signal's Value)

There are two major ways to make signals depend on each other.

- using a derived signal
- using a memo

Demos:

4. Signal B is a function of signal A (i.e, signal B depends on signal A), you can create either a derived signal or memo from signal A like this;

```rs
// signal A
let (count, set_count) = create_signal(1);
// anytime 'count' changes, 'derived_signal_double_count' would also change
let derived_signal_double_count =  move || count() * 2;
// anytime 'count' changes, 'memoized_double_count' would also change. Open this link (https://docs.rs/leptos/latest/leptos/fn.create_memo.html) to understand the difference between a momoized signal and a derived signal, or read 'Reactivity - leptos-analysis-2'
let memoized_double_count = create_memo(move |_| count() * 2);
```

<h6>N.B:</h6>

Take note that `create_memo` memoizes (stores) a state value so that it can be accessed repeatedly without be re-computed. This is very helpful if the computation of the signal's value is heavy. `create_memo` will only re-compute a signal's value, if the signal's value is changed.

5. Signal C is a function of signal A and some other signal B (i.e, signal C depends on signals A, and B), you can create either a derived signal or memo from signals A and B like this;

```rs
// signal A
let (first_name, set_first_name) = create_signal("Bridget".to_string());
// signal B
let (last_name, set_last_name) = create_signal("Jones".to_string());
// derived signal C
let full_name = move || format!("{} {}", first_name(), last_name());
// memoized signal C
let full_name = create_memo(move |_| format!("{} {}", first_name(), last_name()));
```

6. Signal A and signal B are independent signals, but sometimes updated at the same time.
   Imagine this example being great for reseting two or more signals to a value at the same time.

Code example;

```rs
let (age, set_age) = create_signal(32);
let (favorite_number, set_favorite_number) = create_signal(42);
// use this to handle a click on a `Clear` button
let clear_handler = move |_| {
  set_age(0);
  set_favorite_number(0);
};
```
