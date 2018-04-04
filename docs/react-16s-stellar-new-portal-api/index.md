## The Portal API

React 16 landed with a helpful [new API](https://reactjs.org/docs/portals.html) called _portals_, which is a first-class way of rendering children into a DOM node outside of the parent component's hierarchy. Before 16, you had to provide your own solution for breaking outside of this hierarchy. Now it's just as simple as calling:

```
ReactDOM.createPortal(<Component>, domNode)
```

Yup, just pass a `<Component>` you'd like to render and _any_ valid HTML DOM node that you'd like React to render the component into.


### So what, who cares? 🤔 Why is this helpful?

The main reason the Portal API is useful is to escape inherited styles of parent DOM nodes -- think `z-index`, `overflow` and `position` -- and provide a way to render "outside" of the current component hierarchy. Before Portals, this was somewhat tricky...

As you may know, data in React naturally flows down from parent to child, so it's completely normal to end up with nested components that are logically related to their ancestors. But, we may find ourselves in a situation where the parent DOM element has some styles that impact the child elements and the only way to avoid these effects is to move that component outside of the parent.

Seems simple enough — but what if the state and context of the child are closely coupled with the parent component? Well, then you'll have to start communicating between the two. This can also be accomplished in a number of ways such as with prop functions, using actions from a state management library like Redux, or even using a third party portal implementation.

All solutions have drawbacks: Prop callbacks can get old really fast if you've "drilled" too deep, state libraries can be overly complex for a simple React project and add an extra dependency, and third party portals obviously aren't built in.

Enter the magic 🔮 of native Portals! Instead of moving the component, we can "transport" it through a portal to render into a DOM node of our choosing. We'll still be rendering in the same context and have access to all the parent props/state/data we may need, but we'll also be able to escape any parent styles that were proving problematic for our design. 

## Demo Time!

I've "thrown" together a few examples to illustrate how the portal API can simplify the relocation or transportation of DOM markup without resorting to prop functions, Redux, or other libraries.

### Escaping Hidden Overflow

Let's create a simple artboard that a user can interact with:
  
- Users can scroll up/down/left/right within the window to see the artboard.
- Users can click anywhere on the artboard to show a contextual menu.
- Users can select a menu item to add a shape to the artboard.

Cool, so with those parameters, I've created a very simple React app that allows just for that. It's only a few components — most of them stateless:

- `<Window>` will represent the faux OS X window and will render children. 
- `<Artboard>` which will include _all_ of our stateful interactions.
- `<ContextMenu>` to display menu options to the user.

It's important to note that the `<Window>` has some CSS to hide overflow in my use case that was absolutely necessary to have nifty rounded corners 😜. 

Here's what it looks like. Be sure to click around on the artboard to see how the menu is cut-off by the window. Then enable the portal to fix the overflow issue and render to the root of the document.

<iframe height='670' scrolling='no' title='Portal for escaping hidden overflow' src='//codepen.io/jscottsmith/embed/36f1d12b6b54e0131bfac956c2b35d01/?height=524&theme-id=8020&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jscottsmith/pen/36f1d12b6b54e0131bfac956c2b35d01/'>Portal for escaping hidden overflow</a> by J Scott Smith (<a href='https://codepen.io/jscottsmith'>@jscottsmith</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

**How the Portal Works**

The button to toggle the portal on and off illustrates the issue of overflow and why the portal is helpful in this case. When the portal is enabled React is directed to render my `<ContextMenu>` outside of the `<Window>`. Specifically, it is told to render it into the body of the HTML document at `<div id="context-menu"></div>`. This allows it to appear as a global UI element while still remaining logically nested within the component hierarchy.

Check out what the `ContextMenu` is returning and you'll see a call to `createPortal` where the argument `menu` is the markup, and `portalEl` is the `#context-menu` DOM element (the ternary exists so I can toggle it on and off in the demo):

```
return portalEl ? ReactDOM.createPortal(menu, portalEl) : menu;
```

So the portal in this example allows us to keep the `<ContextMenu>` close to the relevant application state which the `<Artboard>` is managing and avoids us having to move it to the root of our application and devise a way of communicating between the two. No prop functions or Redux to pass around state, just a native portal to transport our DOM markup to where it's needed.

### Escaping Positioning

Sometimes the necessary positioning of a parent can cause limitations when positioning a child. Another perfect use case for a Portal.

Let's make a chat bot that allows us to send messages using emoji with shortcodes like `:robot:` 🤖 or `:smile:` 😀. Here's the main things it will do:

- Allow users to type into a message box that automatically searches for potential emoji shortcodes.
- Display a menu of those potentially matching emoji to select an insert into the message. 
- Allow tab to autocomplete with the first matching emoji.
- Send and display a message in the chat.

Problems the Portal solves:

- Allows us to escape the absolute positioning of the `<TextInput>` and position the menu relative to the `<Chat>` window, without relocating the component outside the parent.

This time, since I'm not going to to be positioning the element globally, I won't use an element at the document body like before. Instead I'll use a ref function `ref={ref => (this.header = ref)}` to get a reference to the DOM element, and I'll pass another function `getRef={() => this.header}` to my `<TextInput>` that retrieves the necessary ref. This allows me to get the DOM element that will contain the portal. Since this DOM node will be managed by React we should make sure it will always be available when needed and not potentially unmounted. See the render method of `<App>`.

```jsx
<header ref={ref => (this.header = ref)} /> // the portal el
<Chat comments={this.state.comments} />
<TextInput
    handleSubmit={this.handleSubmit}
    getRef={() => this.header} // this allows us to retrieve the ref for creating the portal
/>
```

Now that we have a reference to the element, we can create the portal. I've set up my `<TextInput>` component to show the emoji menu when a potential shortcode is found. Once that state is `true`, we create and render into our portal:

```jsx
<div className="text-input">
    <textarea />
    {showEmoji &&
        ReactDOM.createPortal(
            <EmojiMenu />,
            this.props.getRef() // the header element that exists outside this component
        )}
</div>
```

That's really it. Again, it's incredibly simple to setup and now we're rendering part of our component markup into another DOM node (this time a node managed by React) outside of the parent.

Here's the Demo — type in the text area, insert a `:` and the shortcode name to see the menu pop up.

<iframe height='670' scrolling='no' title='Chat Window' src='//codepen.io/jscottsmith/embed/544df53610280169a398240b971f1f70/?height=670&theme-id=8020&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jscottsmith/pen/544df53610280169a398240b971f1f70/'>Chat Window</a> by J Scott Smith (<a href='https://codepen.io/jscottsmith'>@jscottsmith</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

As you can see in the demo, a menu will pop up at the top of the window even though our parent component is located within another component.

## Portal Event Bubbling

One interesting and sometimes unanticipated behavior of portals is that event bubbling behaves as though the event is bubbling through the React component's hierarchy and _not_ the DOM hierarchy. Here's a quick example of what this means.

A simple app with portal structure:

```jsx
<App onClick={appClicks}>
    <Portal>
        <button onClick={buttonClicks}>
            Click me
        </button>
    </Portal>
</App>
```

Here's the markup `<App>` will produce:

```html
<div id="app"><div>
<div id="portal">
    <button>Click me<button>
</div>
```

Notice that the `#portal` isn't a child of `#app`, so we wouldn't expect an event from the portal to bubble to the app.

However, when clicking the `<button>` inside the portal, the event _will_ bubble up through the portal into the `<App>`, and call any click handlers along the way. This can feel odd since the HTML hierarchy doesn't represent our React component hierarchy, thus the bubbling up to the App would seem odd. But these are _synthetic_ events native to React, not actual events in the DOM. To prevent this we can simply call `event.stopPropagation()` on the button click handler. Just don't forget about this behavior as there's the potential of introducing minor bugs.

Depending on the complexity of what you are doing with portals, you may encounter more issues with this bubbling. There's some interesting discussion taking place by React team members regarding [portal bubbling](https://github.com/facebook/react/issues/11387).

Alright, back to the demos!

## Portal Event Demo

To further illustrate bubbling in a fun way, I've setup some examples.

In each, I'll recursively render a component into itself that captures and runs an animation on click events. The first demo renders with React normally so we get a deeply nested tree of HTML. The second renders into a portal that is attached to the body of the HTML document, so we end up with a flat HTML structure.

Here's the recursive component that renders itself and a given component, until the depth is `0`:

```jsx
const Recursive = ({ children, depth, component: Component }) => (
    depth > 0 &&
        <Component depth={depth - 1}>
            <Recursive depth={depth - 1} component={Component} />
        </Component>
);
```

and the same but with portals:

```jsx
const RecursivePortal = ({ children, depth, component: Component }) => (
    depth > 0 && ReactDOM.createPortal(
        <Component depth={depth - 1}>
            <RecursivePortal depth={depth - 1} component={Component} />
        </Component>, 
        document.body
    ) 
);
```

In the `<Component>` that is rendered recursively, we'll capture click events by attaching an `onClick={event => handleClick(depth, event)}` that adds a class after a delay (based on depth) to the element to show when it receives an event.

In each, you can click an element and see the event propagate up the _React_ hierarchy. I've also synthetically delayed the animation of the event by increasing the delay with each capture. The real event happens instantaneously, but this far is more interesting to look at. 😉

It's also cool to check out the HTML tree vs the React tree using Chrome dev tools. I've included links to the actual HTML pages so that you can inspect them to see these differences.

Bubbling in a normal component hierarchy (React structure mirrors HTML):

```html
React structure                     HTML structure
---------------                     --------------

<Component>                         <div>            
└── <Component>                     └── <div>        
    └── <Component>                     └── <div>    
        └── <Component>                     └── <div>
```

Demo:

<iframe height='941' scrolling='no' title='Event Bubbling React' src='//codepen.io/jscottsmith/embed/2b3e1a30d955abb75aa8fbb740e8d87f/?height=941&theme-id=8020&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jscottsmith/pen/2b3e1a30d955abb75aa8fbb740e8d87f/'>Event Bubbling React</a> by J Scott Smith (<a href='https://codepen.io/jscottsmith'>@jscottsmith</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

[Debug View](https://s.codepen.io/jscottsmith/debug/2b3e1a30d955abb75aa8fbb740e8d87f)

Bubbling with portals (React structure does not reflect HTML):

```html
React structure                     HTML structure
---------------                     --------------

<Portal>                            <div>  
└── <Portal>                        <div>      
    └── <Portal>                    <div>    
        └── <Portal>                <div>
```

<iframe height='536' scrolling='no' title='Event Bubbling React Portals' src='//codepen.io/jscottsmith/embed/37674e2ece7e0ab898341eecfeb3ef2b/?height=536&theme-id=8020&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jscottsmith/pen/37674e2ece7e0ab898341eecfeb3ef2b/'>Event Bubbling React Portals</a> by J Scott Smith (<a href='https://codepen.io/jscottsmith'>@jscottsmith</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

[Debug View](https://s.codepen.io/jscottsmith/debug/37674e2ece7e0ab898341eecfeb3ef2b)

## Portal To Another Window

Since portals can be created with any valid DOM element, we can also use them to transport part of our application to a whole new browser window. Credit to David Gilbertson for pointing this out in this [article](https://hackernoon.com/using-a-react-16-portal-to-do-something-cool-2a2d627b0202). I strongly suggest you read for a nice breakdown of what's happening. 

So when I first read that you could use a portal to render into a new window I couldn't think of a use case in which it would make sense. But recently I came across an almost perfect reason to do so. I rebuilt a dumbed down version to illustrate the issue and how the portal helped solve the UX problem.

Here's what our _Window Portal_ Messenger app will do:

- Allow users to type a message and their name into some inputs.
- Allow messages to be saved and new ones to be created.
- Users can open a link to their saved messages to share with others.

The key problem is that when users "export" a message, we must perform some async work before we can open a link to their content, like uploading to AWS. In the demo, I've setup a `uploadMarkup` function that takes a few seconds to resolve so simulate this.

```js
async function createHtml(message) {
    const markup = await uploadMarkup(message);

    return new Promise((resolve, reject) => {
        resolve(markup);
    });
}
```

So here's where you'll run into some fun gotchas. 

If a user is opening a link but the link isn't actually ready, we have to wait to open the window -- but if we _do_ wait and try and open a URL later after the async actions have completed, we will be blocked by the browser's pop-up blocker. This is because even though the action to open a link was triggered by a user, the trusted event context is lost during async functions.

Now let's revisit this idea with portals.

Once again, a user opens a link to their saved message, and but this time we do open a new window immediately -- but it's a blank window. And while our async functions are running, we'll use React portals to render the status of that upload to the new window. Finally, once our functions have completed, we can reload the window to the requested link.

This solution offers a really nice user experience because when opening an external link, we expect that to happen immediately, which it will but we can also display that status while work is being completed to create the link.

So, here's the demo. Save a message and open a link to see the portal in action:

<iframe height='724' scrolling='no' title='Window Portal' src='//codepen.io/jscottsmith/embed/7ff654307d2ea47b86e8a80f418ae3c5/?height=724&theme-id=32757&default-tab=result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jscottsmith/pen/7ff654307d2ea47b86e8a80f418ae3c5/'>Window Portal</a> by J Scott Smith (<a href='https://codepen.io/jscottsmith'>@jscottsmith</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

[Debug View](https://s.codepen.io/jscottsmith/debug/7ff654307d2ea47b86e8a80f418ae3c5)

Since this app is a bit more complex component-wise, I won't go over the source much. But if you're interested in looking it over I'd suggest heading to this [github repo](https://github.com/jscottsmith/portal-demos) which is home to all of these demos.

## Recap and Takeaways

Portals should be a really be a helpful API for developing React apps. So just to reiterate some points:

- It's simple to use, but a powerful API.
- Transports markup outside of parent hierarchy without needing prop function, Redux, or third party solutions.
- Escape inherited or parent styles like overflow, position, and z-index.
- Keep component state and logic close to what it controls, but relocate DOM markup where needed.
- Remember: event bubbling is synthetic and will propagate up the component hierarchy. 

Hopefully these demos provide a nice look at how you might use portals in your own React application. Again, if you wanna look at the source, it's all on [GitHub](https://github.com/jscottsmith/portal-demos). 
