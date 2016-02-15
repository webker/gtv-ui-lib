# Writing custom component #

## Checking your reasons ##

TV UI library designed in a way most tasks could be accomplished without writing your own component. Do so only if your goal is code reuse. Otherwise you probably doing something wrong.

## Extending base class ##

Choose a base class to extend. All TV UI classes are designed with extensibility in mind, so feel free to choose the one that best suits your needs. That said, a usual candidate is `tv.ui.Component`.

```
// Provide a class name for your component.
goog.provide('acme.Button');

// Require base class.
goog.require('tv.ui.Component');

/**
 * @constructor
 * @extends {tv.ui.Component}
 */
acme.Button = function() {
  // Don’t forget to call a constructor of base class.
  goog.base(this);

  // ...rest of initialization code...
};
// Actually inherit your class from base.
goog.inherits(acme.Button, tv.ui.Component);
```

## Registering decorator ##

`tv.ui.decorate` function won’t know about your class unless you bind CSS class name with a constructor using `tv.ui.registerDecorator` function.

```
/**
 * @type {string} CSS class that triggers decoration. 
 */
acme.Button.CLASS = 'acme-button';

tv.ui.registerDecorator(acme.Button, acme.Button.CLASS);
```

Since CSS doesn’t support namespaces it’s easy to trigger decoration accidentally, especially when creating components with such generic names as button. Thus it is recommended to use unique product-specific prefix in name of CSS class (`acme-...` in example above).

## Handling events ##

If your component needs to handle an event, first check whether one of base classes isn’t already listening to it. Look for `@protected` methods, which have `on...` prefix, this is standard convention for event handlers in TV UI library. For example, `tv.ui.Component` handles several events through `onFocus`, `onBlur`, `onKey` and `onMouseDown` methods.

```
/**
 * @inheritDoc
 */
acme.Button.prototype.onMouseDown = function(event) {
  // Unless you want to completely override behavior, 
  // call a handler from base class.
  goog.base(this, 'onMouseDown', event);

  // ...rest of handler code...

  // You should call stopPropagation if event was handled successfully.
  // Otherwise event will bubble towards top of component hierarchy.
  event.stopPropagation();
};
```

Beware, listening to event yourself instead of overriding existing handler could cause hard-to-catch race condition between handlers. That said, if none of base classes has a handler you need, you should implement your own.

A good place to subscribe to an event is `decorate` method.

```
/** @inheritDoc */
acme.Button.prototype.decorate = function(element) {
  goog.base(this, 'decorate', element);

  // Use of goog.events.EventHandler is preferred to goog.events.listen
  // as it eliminates the need of binding ‘this’ to handler method.
  this.getEventHandler().listen(
      element, goog.events.EventType.CLICK, this.onClick);
};

/**
 * Handles click event.
 * @param {goog.events.Event} event Click event.
 * @protected
 */
acme.Button.prototype.onClick = function(event) {
  // ...
};
```

## Defining new events ##

Start defining new type of event by creating nested string enumeration named `EventType`. Since all event names in application share the same namespace, use `goog.events.getUniqueId` function to avoid event name collision.

```
/**
 * Event types dispatched by button.
 * @enum {string}
 */
acme.Button.EventType = {
  /**
   * Dispatched when button is clicked or Enter is pressed when button is
   * focused.
   */
  ACTION: goog.events.getUniqueId('action'),

  // ...other event types...
};
```

Subscribe to newly defined event in `decorate` method to make sure that subclasses could handle an event before external listeners.

```
acme.Button.prototype.decorate = function(element) {
  // ...

  this.getEventHandler().listen(
      this, acme.Button.EventType.ACTION, this.onAction);
};

/**
 * Handles action event.
 * To be used in subclasses.
 * @param {goog.events.Event} event Action event.
 * @protected
 */
acme.Button.prototype.onAction = goog.nullFunction;
```

Since `tv.ui.Component` extends `goog.events.EventTarget`, dispatching event is easy.

```
this.dispatchEvent(acme.Button.EventType.ACTION);
```

If event needs additional data, define a class for event and dispatch an instance of this class.

```
var event = new acme.ActionEvent();
event.type = acme.Button.EventType.ACTION;
event.data = data;
this.dispatchEvent(event);
```

If you leave `event.target` uninitialized, it will be conveniently set to `this` by `dispatchEvent` method.

## Defining new properties and CSS classes ##

It is highly recommended to design components that are configurable from HTML to the extent possible. HTML (as opposed to JavaScript) is declarative and makes UI layout much easier to create and maintain.

When adding new property to your component, consider spending some time to make it accessible from HTML by means of CSS classes. This will greatly benefit your class clients. Start from creating nested string enumeration named `Class`, then define CSS classes for newly added properties. Note that property name is prefixed with name of a class.

```
/**
 * CSS classes that control and reflect look and feel of the component.
 * @enum {string}
 */
acme.Button.Class = {
  /**
   * Applied to root DOM element of fancy buttons.
   * @see #isFancy
   */
  FANCY: 'acme-button-fancy',

  // ...rest of CSS class names...
};
```

There are two ways of implementing getter and setter methods for property. First relies on storing the property state in CSS.

```
/** @return {boolean} Whether button is fancy. */
acme.Button.prototype.isFancy = function() {
  return goog.dom.classes.has(this.getElement(), acme.Button.Class.FANCY);
};

/** @param {boolean} fancy Whether button is fancy. */
acme.Button.prototype.setFancy = function(fancy) {
  goog.dom.classes.enable(this.getElement(), acme.Button.Class.FANCY, fancy);
};
```

Second method merely reads and updates CSS when needed.

```
acme.Button.prototype.fancy_ = false;

/** @inheritDoc */
acme.Button.prototype.decorate = function(element) {
  // ...

  this.fancy_ = goog.dom.classes.has(element, acme.Button.Class.FANCY);
};

/** @return {boolean} Whether button is fancy. */
acme.Button.prototype.isFancy = function() {
  return this.fancy_;
};

/** @param {boolean} fancy Whether button is fancy. */
acme.Button.prototype.setFancy = function(fancy) {
  this.fancy_ = fancy;
  goog.dom.classes.enable(this.getElement(), acme.Button.Class.FANCY, fancy);
};
```

First method is concise, less error-prone and needs less memory, second may be faster if your code reads property a lot.

When implementing boolean properties, choose reasonable defaults. For example, `tv.ui.Component` by default is visible and not focused (one can say, most components are visible and not focused). Thus tv-component-hidden is added only when component is hidden. There is no tv-component-shown class, it is assumed by default.

If default value of boolean property is not obvious, use explicit classes for both states but behave reasonably if value is omitted. Consider a container whose orientation is defined by two classes: acme-container-horizontal and acme-container-vertical. If you make vertical a default orientation, class clients can ignore the value when it’s not important, for example when container only has one child. On the other hand, class clients doesn’t have to remember default and can mark their containers explicitly as horizontal.

As boolean is just a particular case of enumeration with two members, same recommendations apply to properties of enumerated types.

Of course there are more types than just booleans and enumerations so it can appear that making a property available in HTML is impractical or even impossible. However, an inspirational example should be given to show declarative abilities of HTML.

```
<div class=”tv-container”>
  <div class=”tv-component”></div>
  <div class=”tv-component tv-container-selected-child”></div>
</div>
```

Sole `tv-container-selected-child` class in above HTML is equivalent to following JavaScript:

```
container.setSelectedChild(container.getChildren()[1]);
```

As you can see, even properties of type `tv.ui.Component` could be set from HTML.

## Optimizing rendering ##

Why bother, you say? See good explanation at http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/.

If your component use computation-intensive DOM methods such as `offsetHeight`, which will force layout and CSS style calculation in browser, it makes sense to postpone component rendering so that cumulative effect of changing component’s properties is calculated only once.
To achieve that, call inherited `scheduleRender` method when component state changes and put computation-intensive calls into overriden `render` method.

```
/** @param {number} minWidth Minimum width. */
acme.Button.prototype.setMinWidth = function(minWidth) {
  this.minWidth_ = minWidth;
  this.scheduleRender();
};

/** @param {number} minHeight Minimum height. */
acme.Button.prototype.setMinHeight = function(minHeight) {
  this.minHeight_ = minHeight;
  this.scheduleRender();
};

/** @inheritDoc */
acme.Button.prototype.render = function() {
  goog.base(this, ‘render’);

  // Use this.minWidth_ and this.minHeight_ to update component’s DOM.
};
```

If `setMinWidth` and `setMinHeight` methods are called within the same callback, passed to `tv.ui.postponeRender` function, `render` method will be called once, right after callback exits.

```
tv.ui.postponeRender(function() {
  button.setMinWidth(100); // calls button.scheduleRender()
  button.setMinHeight(100); // calls button.scheduleRender()
}); // calls button.render()
```

If `setMinWidth` and `setMinHeight` methods are called outside of `tv.ui.postponeRender` function, `scheduleRender` is equivalent to immediate call of `render` method.

```
button.setMinWidth(100); // calls button.scheduleRender() which calls button.render()
button.setMinHeight(100); // calls button.scheduleRender() which calls button.render()
```

## Examples ##

It is good idea to explore how TV UI components are written before implementing your own. Recommended reading is `tv.ui.Button` which is almost a perfect and concise example of custom component, and `tv.ui.Container` which use rendering optimization to improve it’s performance manyfold.