# Dynamic CSS Classes & CSS-in-JS

Summary & Action
----------------

Selectors in your website code's CSS are referenced to make experiments work on your website. In order for your experiments to be more resilient to site changes and not break, your website code will need to include static selectors in the CSS that do not change. If those selectors change, experiments will break after code releases.

Please read the below for more detailed info and share this document with your developers to guide them on how to improve your site structure less prone to breakages. We are happy to answer questions that your dev team may have.

* * * * *

Background
----------

Modern websites have increasingly begun to use advanced "application" JavaScript frameworks such a React, Angular, Vue, and so on. These frameworks favor creating reusable "components" which seek to package everything about a particular display into a convenient interface (a component) that can be efficiently reused throughout your site without having to wire up many external dependencies.

These components frequently use what's known as a "**CSS-in-JS**" library. There are several competing libraries that do this (Styled Components and Emotion to name a few), but they tend to operate in a similar fashion. These libraries allow you to embed the "look and feel" style (CSS) of a React/Angular/Vue component directly into the JavaScript code that describes that component. This way all of the form, functionality, and styling are bundled into a nice and tidy self-contained component.

The Problem
-----------

The problem is that these CSS-in-JS libraries typically generate/use **dynamic** css classes on the generated component elements. These are *dynamic* because these identifiers are generated on the fly using randomized labels.

Experimentation using platforms like Optimizely, Convert, VWO, etc. all operate by identifying the parts of the page that you'd like to change using [CSS selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors). These dynamic, randomly-generated identifiers make that job *extremely* difficult and prone to breakages.

### The Good

Some simple and ordinary content might look like this:

```html
<section class="ordinary-section">
  <div class="ordinary-section__header"></div>
  <div class="ordinary-section__content">
    <h2>This is an ordinary Header</h2>
    <p>This is some ordinary content.</p>
  </div>
</section>
```

Now say we want to change this h2 header copy in an experiment. We could use a selector like this:

```css
.ordinary-section__content > h2
```

This is great because it leaves little room for ambiguity and the selector is simple to create and use.

### The Bad

Now if this section were written using CSS-in-JS it might look something like this:

```html
<section class="_2Ug5J">
  <div class="_7hS7u"></div>
  <div class="_9me1H">
    <h2>This is an ordinary Header</h2>
    <p>This is some ordinary content.</p>
  </div>
</section>
```

These odd looking class values like _2Ug5J are dynamically/randomly-generated and will change to another random value whenever the framework decides to change it. This means that we **can't consistently use these identifiers** to reliably identify elements in the page. So instead, we have to get very "creative."

A selector for the same h2 element suddenly looks like this:

```css
section > div:nth-child(2) > h2
```

This selector is much more ambiguous. This selector doesn't clearly say *which* section we're interested in by name, so if there are other section elements in the page it becomes pretty challenging to exactly specify which section this should apply to, and ensure that the changes won't accidentally show up anywhere unintended.

This type of selector is *flimsy* and *brittle*, and is a major reason for experiments taking **longer to build and QA**, and why they **break more easily** and may require continued **maintenance** while they are running.

These flimsy selectors are much more difficult to craft for engineers. It's akin to giving someone directions that read "drive north from the post office until you see the big tree and take a left" instead of just giving them the address. Then imagine someone cuts down that big tree and the instructions don't make sense anymore. In experiment code, this means any little change to the site's markup like adding or removing a seemingly unrelated div element can be catastrophic for these flimsy selectors, breaking the experiment.

### The Ugly

As you can imagine, the example shown above is only a basic sample of something that can grow into quite a messy situation once we're working with real markup with many levels of nesting and complexity.

Here are some ugly real-world examples:

```css
[role="navigation"] > div:nth-of-type(3) > div > div:nth-of-type(2) > div:first-of-type{}
.rate-check-results.form-submitted > div > div > div > div > div > div:first-child{}
.delivery-not-available > div > div > div > div > div > div:nth-of-type(2):not([size]) span{}
header ~ div:nth-of-type(3) > div:not(.delivery-address-box){}
```

You don't have to know CSS to see that these selectors are rather complex, obscure, and difficult to work with. These are just a few examples that I pulled at random from real experiment code (from a variety of clients). We try our best to avoid such workarounds. But, without more anchor points to reference, this is the only way we can build these experiments.

The Solution
------------

The solution is simple! We simply need more "anchor points" within the markup of the site. You can continue to use CSS-in-JS. We just need some additional **static** identifiers to be added to the elements. This may be an extra class (or classes), an id, or some type of data-key attribute.

We don't necessarily need *every single element* to include some kind of static identifier, but these are like signposts on the roadside that we use to give directions. With more static identifiers to leverage, we can more easily write these instructions, the code will be more resilient against minor changes, and generally it'll cost less money and headache to build an experiment.

### Examples

```html
<section class="ordinary-section _2Ug5J"> ... </section>
```

👍 We could use .ordinary-section to reference this element.

```html
<section id="ordinary-section" class="_2Ug5J"> ... </section>
```

👍 We could use #ordinary-section to reference this element.

```html
<section data-label="ordinary-section" class="_2Ug5J"> ... </section>
```

👍 We could use section[data-label="ordinary-section"] to reference this element. This data attribute could use a variety of keys like data-testid, data-section, or whatever is most convenient.

### Rules of thumb

-   Each distinct, separate section of a page should have its own unique identifier at minimum.

    -   The site header might use <header id="site-header">.

    -   The homepage hero section might include hero-section class alongside any dynamic styling classes.

-   It's best if "like" things include the same identifiers, allowing them to be logically grouped.

    -   All of the "add to cart" buttons might include data-cta="Add to cart" or a button-atc class for example.

-   It's a good idea to provide some kind of identifier to the root element of each of your components at minimum.

    -   This way anywhere that the component is used throughout your application, it will be easily identifiable using a simple CSS selector.

-   Be sure to provide a way to differentiate between elements that share visual styling but serve separate functions.

    -   A page might have multiple CTA ("Call To Action") buttons that all visually look the same, but serve different functions. For example, the visitor might have the option to select from three tiers of service: Intro, Pro, or Enterprise. Providing a unique identifier on each of these CTAs would allow us to track which button the visitor selects much more easily.

    -   For pages containing a list of products, it can be helpful to provide context such as the product id, or product category information as data attributes on the product wrapper element.

If you're using React, they have a good writeup about adding classnames: <https://reactjs.org/docs/faq-styling.html>

The Challenge
-------------

The challenge will be in consistently adding these anchor points as a best practice throughout your application. These are simple tweaks to be made within the application, but can have a very significant impact on the success, health, and cost of your testing program.
