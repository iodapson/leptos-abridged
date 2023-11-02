<h4>
BUILDING USER INTERFACES::
FORMS AND INPUTS
</h4>

Uncontrolled Inputs (Useful for Form submissions)

Here is an example code of an uncontrolled input;

```rs
// You need to import leptos::ev::SubmitEvent
use leptos::{ev::SubmitEvent, *};

#[component]
fn {{TheNameOfTheFormComponent}}() -> impl IntoView {

  // import the type for <input>
  use leptos::html::Input;

  // create a signal companion for the target uncontrolled input
  let (input_signal_1, set_input_signal_1) = create_signal("The Uncontrolled input's initial value".to_string());
  // create a NodeRef<Input> to store a reference to the target uncontrolled input
  let input_element_1: NodeRef<Input> = create_node_ref();
  // store the <input>'s value inside signal (name, set_name) on event 'submit'
  let on_submit = move |event: SubmitEvent| {
    // stop the page from reloading!
    event.prevent_default();
    // extract the uncontrolled input's current value
    let extracted_value = input_element_1().expect("<input> to exist").value();
    // assign new 'value' to the 'input_element
    set_input_signal_1(extracted_value);
  }


  // Create an uncontrolled input
  view! {
    <form on:submit=on_submit>
      // the input below is the target uncontrolled input
      <input type="text" value=input_signal_1 node_ref=input_element_1/>
      // create the submit input
      <input type="submit" value="Submit"/>
    </form>
    <p>"Name is: " {name}</p>
  }
}

#[component]
fn App() -> impl IntoView {
    view! {
        <h2>"Uncontrolled Component"</h2>
        <UncontrolledComponent/>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```

---

Controlled Inputs (Useful when you want to user to see current changes in realtime)

Here is an example code of a controlled input

```rs
use leptos::*;

#[component]
fn {{TheNameOfTheControlledInputComponent}}() -> impl IntoView {
  // create a signal to hold the controlled input 'name' value
  let (name, set_name) = create_signal("Controlled input initial value".to_string())

  // render the input that would be controlled by signal
  view! {
    <input
      type="text"
      on:input=move |event| {
        set_name(event_target_value(&event));
      }
      // the 'prop' syntax lets you update a DOM property
      // tl;dr: use prop:value for form inputs
      prop:value=name
    />
  }
}

#[component]
fn App() -> impl IntoView {
    view! {
        <h2>"Controlled Component"</h2>
        <ControlledComponent/>
    }
}

fn main() {
    leptos::mount_to_body(App)
}
```
