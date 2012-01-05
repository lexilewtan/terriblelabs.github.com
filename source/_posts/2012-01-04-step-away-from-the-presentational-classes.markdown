---
layout: post
title: "Step Away from the Presentational Classes"
date: 2012-01-04 15:37
comments: true
categories: [HTML5, CSS3, Sass, Less, Compass]
author: Jeremy Weiskotten
---

A long, long time ago (2004 A.D.) the W3C
[posted a tip](http://www.w3.org/QA/Tips/goodclassnames) for web developers:

{% blockquote %}
Use class with semantics in mind.
{% endblockquote %}

Most web developers apparently didn't
get the memo -- many web sites still use CSS classes that are presentational
in nature, adding no real meaning or value to the HTML document. I think this
is a problem, and it's a problem worth talking more about and making an effort
toward solving.

First, a bit of history to make the distinction between *semantic markup* and
*semantic classes*, just to make sure we're on the same page.

## Semantic markup 

Semantic markup means using meaningful, self-descriptive HTML elements
rather than a bunch of generic divs. The reasons to use semantic markup are
well-reasoned and well-documented, but the argument ultimately boils down to
improving maintainability and flexibility, while better conveying significance
and meaning within the document to agents, including screen readers (for
accessibility) and search engine crawlers (like Googlebot).

I remember when everyone (every. one.) used HTML tables to lay out their web
pages. Nested tables. Nested nested tables. It worked, but it sucked pretty
hard. A movement toward semantic markup took over, led by the likes of
Jeffrey Zeldman, Eric Meyer, and Dan Cederholm.

Tables were relegated to their proper job, presenting tabular data. HTML
markup became more focused on document structure and content. Layout, being a
style concern, was instead described in CSS (which, by the way, we fervently
import from a separate file, and never, ever, hardly-ever specify inline).

By separating layout from our markup, we started building more maintainable,
flexible, accessible, meaningful documents. It sucked a lot less!

But a common problem cropped up, which we affectionately/bitterly call
*divitis* -- using div elements as generic containers for lots of kinds of
things. Sometimes divitis happened because the developer didn't know about
a more semantic alternative:

``` css This garbage is almost as bad as using tables for layout.
<div id="header1"><b>About My Cats</b></div>
```

Sometimes divitis happened because there *wasn't* a more semantic alternative:

``` css
<div id="footer">Copyright 2008.</div>
```
 
[HTML5](http://html5doctor.com/) came along and introduced us to a bunch of
new semantic friends: `header`, `footer`, `nav`, `section`, `aside`, etc.

``` css
<header><h1>About My Cats</h1></header>
<footer>Copyright 2008.</footer>
```

If we have to, we can use tools like [Modernizr](http://www.modernizr.com/)
to support HTML5 in shitty browsers like Internet Explorer (which is shitty).

### Semantic Classes

Remember that tip from the W3C? "Use `class` with semantics in mind."

The W3C recommends against presentational classes. Classes (if any) should
describe *what* an element is, not *how* it should look. Here are some
examples of markup that uses presentational classes, which might look
familiar because they aren't all that uncommon in the real world:

``` html Presentational classes are bad... m'kay?
<p class="orange bold">Your credit card has expired!</p>
<a class="buttonBig green">Sign Up</a>
<nav class="left">...</nav>
```

{% pullquote %}
This is semantic markup, and that's great. But the classes are presentational,
which is not so great. They're considered "presentational" because they
describe how the element is to be styled: position, weight, color, size. The
actual styling is implemented in CSS, but this information doesn't really
belong in the markup.
{" It's like wearing a T-shirt that says "RED" which just so happens to actually be red. "}
{% endpullquote %}

* What if we want that bold, orange text to be italic and yellow? We'd have to change our markup just to change style.
* What if the designer wants to change the green "Sign Up" button to red, and make it extra big? We'd have to change our markup just to change style.
* What if we want to move the navigation elements to the right side of the page, or horizontally across the top? Guess what... We'd have to change our markup just to change style.

We should be able to make most style changes by simply editing our CSS, not
our markup, and we should not have to pollute our markup with presentational
noise. Presentational details are the responsibility of CSS, not HTML!
 
Compare with these versions:

``` html
<p class="warning"> Your credit card has expired!</p>
<a class="primary">Sign Up</a>
<nav class="submenu">...</nav>
```

We've replaced the presentational classes with classes that describe what
each element *is* rather than what it should *look like*. We've completely
separated presentations details from the structure of the document. Now we can
change our design by editing CSS, not our markup. That's awesome!

**But there's still room for improvement...** (This is where you might start to disagree with me.)
 
One of the benefits of CSS classes is that they make it really easy to reuse
a collection of style rules. For example, the popular
[PIE clearfix hack](http://www.positioniseverything.net/easyclearing.html)
looks something like this:

``` css
.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}

.clearfix {display: inline-block;}  /* for IE/Mac */
```

Of course, these rules are only applied to an element if you include the
`clearfix` class in your markup, like this:

``` html
<section id="container" class="clearfix">...</section>
```

If you have a moderately complex layout you might need to use this class
in a bunch of places -- in your markup! That sucks! But CSS doesn't give you
a better mechanism for reuse.

Fortunately, some smart people have solved this problem for us!

## Sass and Less: Mixins

[Sass](http://sass-lang.com) and [Less](http://lesscss.org) are two popular
open source CSS extensions. They add features that make CSS easier to work
with and more powerful, including rule nesting, variables, and *mixins*.
Mixins allow you to reuse a chunk of CSS in different rules.

Sass and Less are pretty comparable, but I'm more familiar with Sass and it's
supported out of the box by new Rails apps, so that will be our focus.

Here's how you define a mixin to implement that zany PIE clearfix hack in Sass
(using the SCSS syntax, which looks a lot like regular CSS):

``` sass
@mixin pie-clearfix {
  display: inline-block; /* for IE/Mac */
 
  &:after {
    content: ".";
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
  }
}
```

Then, to use this mixin in a CSS rule, just `@include` your mixin (along with any
other styles for that rule):

``` sass
section#container {
  @include pie-clearfix;
  background-color: #000;
}
```

Now your markup is completely and hilariously free of any presentation classes:

``` html
<section id="container">...</section>
```

Note that you can also create mixins that take parameters, like this:

``` sass
@mixin border-radius($radius) {
  border-radius: $radius;
  -moz-border-radius: $radius;
  -webkit-border-radius: $radius;
}

nav {
  @include border-radius(8px);
}

section#container {
  @include pie-clearfix;
  @include border-radius(16px);
}
```

#### Example: Compass & Blueprint

The [Compass](http://compass-style.org) framework defines a lot of useful,
reusable, cross-browser compatible mixins. It's definitely worth checking out
if you decide to use Sass. Compass includes some clearfix hacks, gradients,
border radius, and a module that implements the
[Blueprint](http://blueprintcss.org) grid and typography system.

Blueprint defines a bunch of presentational classes, such as `.container` to
define a grid container, `.span-x` to define a row with x columns, `.append-x`
and `.prepend-x` to append or prepend empty columns to a row, and much more.
Presentational classes. Sigh.

Ingeniously, the Compass
[Blueprint module](http://compass-style.org/reference/blueprint/) turns these
classes into reusable mixins, so you can move presentation details into CSS
where it belongs!

``` sass Using Blueprint mixins instead of presentational classes
section#wrapper {
  @include container;
  header, footer {
    @include column(8);
  }
  aside#sidebar {
    @include column(3);
  }
  section#content {
    @include column(5, true);
  }
}
```

#### Example: Less & Twitter Bootstrap

Here's one last example, using the super popular (most-watched project on
GitHub) [Twitter Bootstrap](http://twitter.github.com/bootstrap/)
CSS/JavaScript toolkit (which is built with Less):

``` html
<button class="btn primary">Primary</button>
```

Presentational classes? The nerve! Since Bootstrap is built with Less, you
should be able to use `.btn` and `.primary` as mixins in your .less stylesheet
instead of using them as presentational classes (being a Sass zealot, I
have not tried this).

### Thoughts?

Are you convinced? Have you committed to doing away with presentational
classes in the past, but encountered real-world scenarios where they made
sense? Comments and feedback are welcome!