+++
date = "2015-02-18"
draft = false
title = "Geb 101: Simple Scripts"
slug = "geb-101-simple-scripts"
tags = ["groovy", "geb", "scripting", "dev"]
author = "Miguel de la Cruz"
+++

These days I’m in between projects, so I decided to automate a task related to a web interface and a tedious list shaped as an HTML form. This automation would imply two processes: parsing an HTML page and filling some fields in another page, plus some small and simple processing.

I had two options, to use HTTP directly through [HTTP Builder](https://github.com/jgritman/httpbuilder) or something like that or to code various small scripts with Geb. Of course, I chose the latter.

### A little bit of context

From its webpage:

> Geb is a browser automation solution.
>
> It brings together the power of WebDriver, the elegance of jQuery content selection, the robustness of Page Object
> modelling and the expressiveness of the Groovy language.

With a simple script, I can have a Firefox or Google Chrome webdriver (among others) up and running quickly, use
jQuery-like selectors to traverse and modify the webpage DOM and process its information with all the power of Groovy.

And best of all, the dependencies are managed by [Grape](http://groovy.codehaus.org/Grape), so the only thing I need in
any machine to run this scripts is Groovy itself.

### Requirements

To run this examples, the only requirements needed are the **Firefox Web Browser** and the **Groovy Runtime**, version
2.X or greater.

### Dependencies and Hello World

First of all, we need to add the dependencies to our script.

```groovy
@Grab(group='org.gebish', module='geb-core', version='0.10.0')
@Grab(group='commons-logging', module='commons-logging', version='1.2')
@Grab(group='org.seleniumhq.selenium', module='selenium-firefox-driver', version='2.43.0')
```

Usually, the `commons-logging` dependency is transitive to the other two and should be resolved automaticaly, but Groovy
is unable to find the transitive version, so we put the one we found on [Maven Central](http://search.maven.org/)
explicitly and problem solved.

When used inside a script, Geb works with a simple DSL notation. Our *hello world* script is going to drive the browser
to the google webpage and fill the form with "Hello Geb!".

```groovy
@Grab(group='org.gebish', module='geb-core', version='0.10.0')
@Grab(group='commons-logging', module='commons-logging', version='1.2')
@Grab(group='org.seleniumhq.selenium', module='selenium-firefox-driver', version='2.43.0')
import geb.Browser


Browser.drive {
    go "http://google.com"

    $("#gbqfq").value("Hello Geb!")
}
```

At this point, you can go to your terminal and run:

```sh
groovy HelloWorld.groovy
```

### Selecting content

Geb uses jQuery-like selectors to access the content of the page through the `$` function. This selectors always return
a `geb.navigator.Navigator` object that represents one or many elements on the page. This function can be used through
the `find` alias.

The signature of the `$` function is

```groovy
$([CSS SELECTOR], [INDEX OR RANGE], [ATTRIBUTE / TEXT MATCHERS])
```

being all the arguments optional, so the following calls are valid

```groovy
$("#content h1")
$("#content h1", 0)
$(0)
$("div", title: "my-title")
$(name: "password-input")
```

We can select again over a `geb.navigator.Navigator` object, calling `$` or `find` again

```groovy
// <div class="a">
//   <p class="b par" data="validateable">Something</p>
// </div>
// <div class="a">
//   <p class="b par" data="validateable">Something more</p>
// </div>

$(".a").find(".b")
assert $(".a").find(".b") == $(".a").$(".b")
```

This selectors are iterable too

```groovy
// <p>Something</p>
// <p>Something more</p>

$("p").collect {
    it.text()
}
//=> ['Something', 'Something more']
```

they expose the following special methods: `tag()`, `text()` and `classes()`

```groovy
// <p class="b par">Something</p>
// <p class="b">Something more</p>

def el = $(".b", 0)
el.tag()
//=> 'p'
el.text()
//=> 'Something'
el.classes()
//=> ['b', 'par']
```

and all their attributes are exposed throught `@attribute`

```groovy
// <p class="b par" data="validateable">Something</p>

def el = $(".b", 0)
el.@data
//=> 'validateable'
```

### Handling forms

Geb provides us with tools to manage forms and input fields. Input fields have the method `value()` that we can use to
both get the current field value or set our own. Given this code

```groovy
// <form action="/" method="POST">
//   <input type="text" name="username" />
//   <input type="password" name="password" />
// </form>

$("form", name: "password").value()
//=> my username
$("form", name: "password").value("another username")
```

We can use value on elements of type input, select or textarea. If we try to set a value on a different type of element,
we will get an `UnableToSetElementException` exception.

To ease working with forms, we can use shortcuts on the `Navigator` objects to access the form fields by their name

```groovy
// <form action="/" method="POST">
//   <input type="text" name="username" />
//   <input type="password" name="password" />
// </form>

assert $("form").username == "my username"
$("form").username = "another username"
$("form").password = "mysupersecret"
```

Special inputs like textarea, select, checkbox, etc. are accessed each one in its particular way. You can find each case
explained in detail in the [setting values](http://www.gebish.org/manual/current/navigator.html#setting_values) section
of the documentation.

### Moving around

Among all the possible interactions that we have in Geb, one of the most important ones is the **click**. Navigator
objects have a `click()` method, that simply clicks on the **first element** of the navigator

```groovy
$("a").click()
```

Geb has tools for waiting to some conditions to happen or some content to appear on the page. This is useful when
following links, when interacting with the page or when working with timers.

The `waitFor()` method allows us to wait a certain amount of time

```groovy
waitFor(10)
waitFor(10, 0.5)  // wait up to 10 seconds, with half a second between retries
```

and to wait for some closure to return a true object

```groovy
waitFor { title.contains("Search Results") }
```

To finally close the browser, we use the `quit()` method of the DSL

### Geb and Javascript

If we use Geb with the DSL, we have a `js` object to both interact with the data and execute code in Javascript.

#### Accessing variables

Any Javascript global varialbe is accesible by name

```groovy
// <html>
//   <script type="text/javascript">
//     var aVariable = 1;
//   </script>
// <body>
// </body>
// </html>

assert js.aVariable == 1
```

we can even access nested variables

```groovy
assert js."document.title" == "My Webpage"
```

#### Executing code

You can execute arbitrary javascript code through the `js` object

```groovy
assert js.exec('return 1 + 1;') == 2
```

if you want to send arguments to the javascript snippet, they are the first in the `exec()` signature

```groovy
assert js.exec(1, 2, 'arguments[0] + arguments[1];') == 3
```

### Beyond the script

Geb is awesome to write simple and powerful scripts, but its power doesn’t stop there. If we need bigger and more
complicated logic, Geb provides us with the `Driver` and `Page` classes and a lot of abstractions way more structured
than a DSL to use.

Reach [the manual](http://www.gebish.org/manual/current/) or the
[API documentation](http://www.gebish.org/manual/current/api/) if you want to find more and have fun with Groovy and
Geb.
