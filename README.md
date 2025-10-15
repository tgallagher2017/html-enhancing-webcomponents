# html-enhancing-webcomponents

WebComponents that enhance HTML instead of build HTML providing greater control of the HTML and CSS. Using Typescript and sometimes LitElement to build the components which can wrap existing HTML to enhance their functionality. A good way to separate the HTML developer from the Javascript developer.


# What is HTML Enhancing?

Most web components encapsulate their related HTML with their functionality, which is fine if you want to create stand alone component libraries.
However, the problem I see with that approach is that Javascript is rending everything, and for others to use your components, you have to provide
some way to access those elements. Further, dealing with the shadow DOM is kind of a hassle.

This might be best expressed in an example which is explained below.


# Why use Typescript?

I personally find Javascript difficult at times trying to keep track of object information. I prefer using a more typed language like Typescript
which I find close enough to Javascript to make it useful without it restricting the Javascript. This may be a hold over from my C/C++ and Java
background.


# Why use LitElement

Web components have lots of bioler plate associated with them and I find that [LitElement](https://lit.dev) helps reduce that bioler plate
stuff with annotations, and best of all, it does not get in the way of the low level HTMLElement functionality as other libraries do.

# HTML Enhancing example

Let's say for example I have a password input field and I want to expose the password as plain text when the user clicks a button.
Here is a way this can be done as a LitElement

```
File: password-exposer.ts

import { customElement } from "lit/decorators.js";
import { html } from "lit-html";

@customElement("password-exposer")
export PasswordExposer extends LitElement {
    @property()
    protected labelText: String = ""

    @property()
    protected toggleText: String = ""

    onTogglePassword(event) {
        let el = this.querySelector("myPwd") as HTMLInputElement
        if (el) {
            if (el.type === 'password') {
                el.type = 'text'
            } else {
                el.type = 'password'
            }
        }
    }

    render() {
        return html '<div>${labelText} <input type="password" name="myPwd" id="myPwd" />'
            + '<button type="button" @click="${this.onTogglePassword}">${toggleText}</button></div>'
            + '<button type="button"">Submit</button>'
    }
}
```

NOTE: of course, there is a typescript build process to convert this to Javascript for the page to read.

```
File: password.html

<html>
  <head>
    <script type="module" src="password-exposer.js
  </head>
  <body>
    Hi, please log in.
    <password-exposer labelText="Please enter password:" toggleText="Toggle password"></password-exposer>
  </body>
</html>
```

The above code seems fine, but the problem is that if I'm someone who knows how to design pages but not that great with 
Javascript, I now have to learn more Javascript and program HTML inside Javascript. I also lose direct access to the 
styling. I also have a hard time changing the 'toggle' to an icon.

Notice to make this component a little generic, the attribute/properties 'labelText' and 'toggleText' are added to the component, but
the Submit button text was is not. This is an example of the Web Component developer not anticipating everything with the rendering
that might need changing by the user. There may be other aspect of this component that need to be added later on due to how things
are rendered.

On the other hand, if I take the enhancing approach, I can put the password field directly into the HTML and I have greater access to
it. The web component changes from being something the generates HTML to something the discovers its assocaited HTML and enhances it.

```
File: password-exposer2.ts

import { customElement, property } from "lit/decorators.js";

@customElement("password-exposer2")
export class PasswordExposer2 extends ShadowlessComponent {

    @property({type: String})
    toggleId: String = '';

    toggleElement: HTMLButtonElement | null = null;

    connectedCallback() {
        this.toggleElement = this.querySelector(`#${this.toggleId}`);
        if (this.toggleElement) {
            this.toggleElement.addEventListener('click', this);
        }
    }

    disconnectedCallback() {
        if (this.toggleElement) {
            this.toggleElement.removeEventListener('click', this);
        }
    }

    handleEvent(event: Event) {
        if (event.type === 'click') {
            this.togglePassword();
        }
    }

    togglePassword() {
        // get the first password child element
        let el = this.querySelector('[type="password"]') as HTMLInputElement;

        if (el) {
            if (el.type === 'password') {
                el.type = 'text';
            } else {
                el.type = 'password';
            }
        }
    }
}
```

```
File: password2.html

<html>
  <head>
    <script type="module" src="password-exposer2.js
  </head>
  <body>
    Hi, please log in.
    <password-exposer2 toggleId="pwdToggle">
      <div>Please enter password: <input type="password" name="myPwd" id="myPwd" />
        <button type="button" id="pwdToggle">Toggle password</button>
      </div>
      <button type="button">Submit</button>
    </password-exposer2>
  </body>
</html>
```

Now, as an HTML developer, I have complete control over how the password and the rest of the HTML appear. Even if the password-exposer2 does
not work properly, the password elements will still appear the way I want. If I wanted to change the toggle from a button to an image, then 
that is easy to do without having to reformat HTML in a Javascript string and rebuilding the component.

This style also benefits from separation of work. HTML/CSS developers can create a page without the password-exposer2, then pass the page off
to the Web Component developer and the HTML gets enhanced.

Of course, password-exposer2 is more Typescript than the first example, with the element query and the addEventListener()/removeEventListener()
lines that LitElement takes care of in the first example with the @click binding. I like to have the events go to the class level and then broker
them from there which makes it easier to remove the listener later. On the other hand, there is no need to have labelText or toggleText as
attribute/properties of the element since the HTML is controlled externally.

With this approach, I find that I spend less time in the Web Component code than I do in the HTML/CSS code.



