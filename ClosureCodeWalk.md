# Introduction #

In this code walk you will learn how to create simple page using Closure based UI library that looks as below.

![http://lh4.ggpht.com/_dS5nq8R6miE/TUkhSTcFsGI/AAAAAAAAEIc/ihsiQSV7kYQ/tv.ui.Decoration%20codelab%20-%20Google%20Docs.jpg](http://lh4.ggpht.com/_dS5nq8R6miE/TUkhSTcFsGI/AAAAAAAAEIc/ihsiQSV7kYQ/tv.ui.Decoration%20codelab%20-%20Google%20Docs.jpg)

First [download archive with simple HTML for this codewalk](https://code.google.com/p/gtv-ui-lib/downloads/detail?name=codewalk-start.zip) and take a look at it. All it does is:
  * bootstraps Closure Library along with TV UI library code in HEAD element,
  * imports necessary components using goog.require(),
  * contains basic CSS styles that defines the page,
  * contains HTML only for part of above page, rest of it is commented as it will be explained later.


If you would have some problems, you can [download final result of the codewalk](https://code.google.com/p/gtv-ui-lib/downloads/detail?name=codewalk-final.zip).

# What we want to do? #

Now let’s take a closer look at page logic that we want to create.

![http://lh3.ggpht.com/_dS5nq8R6miE/TUkhSddEcOI/AAAAAAAAEIY/nsdYDFBusIo/s800/tv.ui.Decorationcodelab.png](http://lh3.ggpht.com/_dS5nq8R6miE/TUkhSddEcOI/AAAAAAAAEIY/nsdYDFBusIo/s800/tv.ui.Decorationcodelab.png)

Toolkit does not construct any HTML on the fly. It’s designed to decorate existing HTML in order to make components out of it and add dynamics to it. Decoration is based on conventions by adding special CSS classes to HTML elements (all of those have “tv-” prefix). Let’s take a look at already downloaded example.

```
<!-- Top level container is declared as vertical container. Note that
     additional styles can be applied for customization. -->
<div class="tv-container-vertical main-container">
  <!-- Declares horizontal container that holds two buttons. -->
  <div class="tv-container-horizontal buttons-container">
    <!-- “alert button” and “go to google.com” -->
  </div>
  <!-- Declares vertical container that holds shelves. -->
  <div class="tv-container-vertical shelves">
    <!-- “shelves” with images -->
  </div>
</div>
```

# First horizontal container #

Let’s take a look at first horizontal container that holds two buttons.

```
<div class="tv-container-horizontal buttons-container">
  <!-- Declares that following element will be button. Buttons have
       ACTION events that you can hook up. -->
  <div class="tv-button alert-button">Alert button</div>
  <!-- Declares a special case of button that is a Link. Note that it
       contains href attribute with link to http://google.com. Toolkit
       will recognize it and when using will click on it or press ENTER
       when it’s focused, it will redirect to google.com. -->
  <div class="tv-link goto-link" href="http://google.com">
     Go to google.com
  </div>
</div>
```

As you can see components are plain HTML using mainly DIV elements. You can use other elements but DIVs are in most cases the best choice.

```
<div class="tv-container-vertical shelves">
  <!-- First container, 5 items, no scrolling, no animation. -->
  <div class="tv-container-horizontal movie-cover-container
              movie-covers-1">
    <!-- Single movie cover is declared as Component. This is lowest -->
    <!--  lowest level in hierarchy component in toolkit. -->
    <div class="tv-component movie-cover">
      <img src="http://g2.gstatic.com/.../93951_aa.jpg">
    </div>
    <!-- [... 4 other tv-components omitted ...] -->
  </div>
</div>
```

# Fire up toolkit #

Now go ahead and fire up toolkit. Uncomment SCRIPT element on the bottom of the page.

```
// DecorationHandler facilitates post-decoration initialization of
// components.
//  * Components can be matched by element id and CSS class.
//  * Multiple handlers can be called for decorated component.
var decorateHandler = new tv.ui.DecorateHandler();
// Capturing first component to be focused. We want it to be button.
var firstNavItem;
decorateHandler.addClassHandler('tv-button', function(button) {
  firstNavItem = button;
});
// Attaching on ACTION handler to button.
decorateHandler.addClassHandler('alert-button', function(button) {
  goog.events.listen(button, tv.ui.Button.EventType.ACTION,
      function() {
        alert('Button clicked.');
      });
});

// Decorating whole document's body and adding dynamics to it.
tv.ui.decorate(document.body, decorateHandler.getHandler());
// Giving focus to first button on the page.
tv.ui.Document.getInstance().setFocusedComponent(firstNavItem);
```

Refresh the page and you will be able to navigate through the page using keyboard and mouse. Play with it:
  * press ENTER on “alert button”,
  * click on “Go to google.com”,
  * use arrow keys to navigate.


Note that:
If component is focused, a tv-component-focused CSS class is being added to it. In CSS styles declares on top of the page, it’s declared to add blue box shadow.
Fifth element of first movie covers shelf is partially visible.

# Scrolling container #

Let’s fix the issue with last partially visible element in the container. Uncomment the content under `<!-- Second container … -->`.

```
<div class="tv-container-horizontal movie-cover-container
            movie-covers-2">
  <!-- If in the container there is element decorated with CSS class
       tv-container-start-scroll, scrolling behaviour will be added
       to the container. -->
  <div class="tv-container-start-scroll">
   <!-- [... the same as in first container ...] -->
  </div>
</div>
```

Adding child element with tv-container-start-scroll is a convention that triggers a specific scrolling behavior in container. With this class applied selected child will always be positioned at start of scrolling window.

# Scrolling animated container #

Adding animation is easy when using CSS3 animations. Uncomment third shelf.

```
<div class="tv-container-horizontal movie-cover-container
            movie-covers-3">
  <div class="tv-container-start-scroll animation">
     <!-- [... the same as in first container ...] -->
  </div>
</div>
```

It differs from second shelf with only additional animation CSS class added to scrolling container. It’s defined as -webkit-transition: all 200ms ease. That means that all CSS properties will be animated for 200ms with ease function. Refresh the page and see the animations in the third container.

You can modify scrolling behavior. For instance selected child in the container can be set to be middle one. It’s more pleasant for the user when using horizontal containers. Uncomment the fourth container.

```
<div class="tv-container-horizontal movie-cover-container
            movie-covers-3">
  <div class="tv-container-middle-scroll animation">
     <!-- [... 10 movie covers ...] -->
  </div>
</div>
```

Vertical container can have scrolling behaviour in the same way as horizontal containers. In our example height of the screen can be too small to display all shelves. Uncomment following line

```
<div class="tv-container-vertical shelves">
      <!-- <div class="tv-container-start-scroll animation"> -->
```

and enclosing `</div>` at the bottom. After page refresh you will notice that shelves container is also scrolling with animation.