# ```delegatesFocus``` flag proposal

To fix focusability and focus navigation order issue ([W3C bug 25473](https://www.w3.org/Bugs/Public/show_bug.cgi?id=25473), [Chromium issue 380445](https://code.google.com/p/chromium/issues/detail?id=380445), introduce ```delegatesFocus``` flag which belongs to shadow root, to address some shortcomings that
the default focus navigation behavior ([defined in the Shadow DOM spec](https://w3c.github.io/webcomponents/spec/shadow/#focus-navigation) has.

- Fix tab navigation ordering issue when shadow host has ```tabindex``` attribute
- Makes shadow root-backed host focusable
- Changes focusing behavior when mouse click or `focus()` within shadow root

## IDL

```WebIDL
partial interface ShadowRoot {
  readonly boolean delegatesFocus;
}

partial dictionary ShadowRootInit {
  boolean delegatesFocus;
}
```

- `delegatesFocus` is a read-only boolean property on a shadow root, which indicates focus activity (tab navigation, mouse click, focus()) on its shadow host will be delegated to its shadow.  This attribute is set via ```ShadowRootInit``` dictionary when ```createShadowRoot``` is called.  This attribute is immutable during the lifetime of its associated shadow root.


## Details


If a shadow host delegates focus (its shadow root was created with `delegatesFocus=true`), the following behavior is expected.


1. TAB navigation order<br>
If tabindex is 0 or positive and the containing shadow tree has one or more focusable elements, forward tab navigation will skip focus on the host itself and forwards focus to the first focusable element, then to the second, …, to the last focusable element until focus blurs to the next focusable element within the same treescope as the host element. For backward tab navigation, the last focusable element gets focus first, and after the first focusable element, until focus blurs to the previous focusable element in the same treescope as the host element. This behavior applies recursively on a shadow host in a shadow tree.
If the host element has no focusable element under it, tab navigation just skips the host and its shadow tree.
In the case of `tabindex="-1"`, the whole shadow tree is skipped for the tab navigation. See discussion below for whether or not user agent should skip the whole subtree or not.

2. `focus()` method behavior<br>
Invoking focus() method on the host element will delegate the focus the first focusable element in its subtree. This applies recursively for nested shadow trees. If the shadow root doesn’t contain any focusable element, the host itself gets focus.

3. [`autofocus` attribute](https://html.spec.whatwg.org/multipage/forms.html#autofocusing-a-form-control:-the-autofocus-attribute)<br>
If autofocus attribute is specified, the focus is forwarded like focus() method when the page load finishes.

4. Response to mouse click<br>
If focusable area within the shadow tree is clicked, the element gets focus. If non-focusable area is clicked, the shadow host gets focus. This is analogous to when `<div>` with `tabindex` attribute gets focus when its content is clicked. The difference is, that the nearest focusable shadow host up in the tree gets focus, skipping non-focusable shadow hosts.

5. CSS `:focus` pseudo-class<br>
A selector like `#host:focus` matches when focus is in any of its descendent shadow tree.

6. `document.activeElement` and `ShadowRoot.activeElement`<br>
These don’t change. The shadow host becomes activeElement when an element in its shadow tree has focus.


If you want to control which element gets first focus when forward or backward navigation comes to the component, you can use `tabindex` attribute for each element within the component.


## Use cases

- When a web author wants to create his/her own one with multiple focusable fields using combination of Shadow DOM and Custom Elements


## Demo

1. [<date-input> component](https://takayoshikochi.github.io/tabindex-focus-navigation-explainer/demo/date-input.html)

You see `<date-input>` `<input type=date>` fields.  The former is built with web components (as a polyme element), the latter is native implementation.
