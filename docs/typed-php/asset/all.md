![](images/title_page.jpg)

![Typed PHP](images/title_page.jpg)

## Typed PHP

### Stronger Types For Cleaner Code

#### Christopher Pitt

This book is for sale at [http://leanpub.com/typedphp](http://leanpub.com/typedphp)

This version was published on 2014-06-23

![publisher's logo](images/leanpub-logo.png)

*   *   *   *   *

This is a [Leanpub](http://leanpub.com) book. Leanpub empowers authors and publishers with the Lean Publishing process. [Lean Publishing](http://leanpub.com/manifesto) is the act of publishing an in-progress ebook using lightweight tools and many iterations to get reader feedback, pivot until you have the right book and build traction once you do.

*   *   *   *   *

© 2013 - 2014 Christopher Pitt

## Table of Contents

*   [Introduction](#chap00.xhtml#leanpub-auto-introduction)
    *   [Why Write This Book](#chap01.xhtml#leanpub-auto-why-write-this-book)
    *   [Who This Book Is For](#chap02.xhtml#leanpub-auto-who-this-book-is-for)
    *   [Procedural / Object Oriented](#chap03.xhtml#leanpub-auto-procedural--object-oriented)
        *   [Procedural Programming](#chap03.xhtml#leanpub-auto-procedural-programming)
        *   [Object Oriented Programming](#chap03.xhtml#leanpub-auto-object-oriented-programming)
        *   [Which is the best?](#chap03.xhtml#leanpub-auto-which-is-the-best)
        *   [Which is PHP?](#chap03.xhtml#leanpub-auto-which-is-php)
    *   [Native Function Inconsistencies](#chap04.xhtml#leanpub-auto-native-function-inconsistencies)
        *   [Sporadic Underscores](#chap04.xhtml#leanpub-auto-sporadic-underscores)
        *   [Sporadic Abbreviation](#chap04.xhtml#leanpub-auto-sporadic-abbreviation)
        *   [Inconsistent Argument Order](#chap04.xhtml#leanpub-auto-inconsistent-argument-order)
        *   [Regular Expression / Strings](#chap04.xhtml#leanpub-auto-regular-expression--strings)
        *   [Nouns / Verbs](#chap04.xhtml#leanpub-auto-nouns--verbs)
        *   [Strange Return Values](#chap04.xhtml#leanpub-auto-strange-return-values)
    *   [Conclusion](#chap05.xhtml#leanpub-auto-conclusion)
*   [Structure](#chap06.xhtml#leanpub-auto-structure)
    *   [Boxing](#chap07.xhtml#leanpub-auto-boxing)
    *   [Resolving Types](#chap08.xhtml#leanpub-auto-resolving-types)
        *   [Scalar Types](#chap08.xhtml#leanpub-auto-scalar-types)
        *   [Functions](#chap08.xhtml#leanpub-auto-functions)
        *   [Classes](#chap08.xhtml#leanpub-auto-classes)
        *   [Regular Expressions](#chap08.xhtml#leanpub-auto-regular-expressions)
    *   [Namespace Functions](#chap09.xhtml#leanpub-auto-namespace-functions)
        *   [Composer Autoload](#chap09.xhtml#leanpub-auto-composer-autoload)
        *   [Future Goodies](#chap09.xhtml#leanpub-auto-future-goodies)
    *   [Conclusion](#chap10.xhtml#leanpub-auto-conclusion-1)
*   [Extensions](#chap11.xhtml#leanpub-auto-extensions)
    *   [Vagrant + Phansible](#chap12.xhtml#leanpub-auto-vagrant--phansible)
        *   [Installing](#chap12.xhtml#leanpub-auto-installing)
        *   [Provisioning](#chap12.xhtml#leanpub-auto-provisioning)
        *   [Vagrant Commands](#chap12.xhtml#leanpub-auto-vagrant-commands)
    *   [SPL Types](#chap13.xhtml#leanpub-auto-spl-types)
        *   [Installing SPL Types](#chap13.xhtml#leanpub-auto-installing-spl-types)
        *   [Using SPL Types](#chap13.xhtml#leanpub-auto-using-spl-types)
    *   [Scalar Objects](#chap14.xhtml#leanpub-auto-scalar-objects)
        *   [Installing Scalar Objects](#chap14.xhtml#leanpub-auto-installing-scalar-objects)
        *   [Using Scalar Objects](#chap14.xhtml#leanpub-auto-using-scalar-objects)
    *   [Zephir](#chap15.xhtml#leanpub-auto-zephir)
        *   [Installing Zephir](#chap15.xhtml#leanpub-auto-installing-zephir)
        *   [Using Zephir](#chap15.xhtml#leanpub-auto-using-zephir)
    *   [Conclusion](#chap16.xhtml#leanpub-auto-conclusion-2)
*   [Design](#chap17.xhtml#leanpub-auto-design)
    *   [Which Method To Use](#chap18.xhtml#leanpub-auto-which-method-to-use)
        *   [Namespace Methods](#chap18.xhtml#leanpub-auto-namespace-methods)
        *   [Scalar Objects / SPL Types](#chap18.xhtml#leanpub-auto-scalar-objects--spl-types)
        *   [Zephir](#chap18.xhtml#leanpub-auto-zephir-1)
    *   [How To Structure The Types](#chap19.xhtml#leanpub-auto-how-to-structure-the-types)
        *   [Resolving Types](#chap19.xhtml#leanpub-auto-resolving-types-1)
        *   [Chaining](#chap19.xhtml#leanpub-auto-chaining)
        *   [Combining Number Types](#chap19.xhtml#leanpub-auto-combining-number-types)
    *   [How To Test](#chap20.xhtml#leanpub-auto-how-to-test)
        *   [PHPUnit](#chap20.xhtml#leanpub-auto-phpunit)
        *   [What Should Be Tested?](#chap20.xhtml#leanpub-auto-what-should-be-tested)
        *   [When Should Tests Be Written?](#chap20.xhtml#leanpub-auto-when-should-tests-be-written)
        *   [Recommendations](#chap20.xhtml#leanpub-auto-recommendations)
    *   [How To Package](#chap21.xhtml#leanpub-auto-how-to-package)
        *   [Make A `Readme` File](#chap21.xhtml#leanpub-auto-make-a-readme-file)
        *   [A Few Examples](#chap21.xhtml#leanpub-auto-a-few-examples)
        *   [Testing Instructions](#chap21.xhtml#leanpub-auto-testing-instructions)
        *   [Installation Instructions](#chap21.xhtml#leanpub-auto-installation-instructions)
        *   [License](#chap21.xhtml#leanpub-auto-license)
        *   [Contribution Guidelines](#chap21.xhtml#leanpub-auto-contribution-guidelines)
    *   [Conclusion](#chap22.xhtml#leanpub-auto-conclusion-3)
*   [Designing For The Future](#chap23.xhtml#leanpub-auto-designing-for-the-future)
*   [Appendices](#chap24.xhtml#leanpub-auto-appendices)
    *   [PHP Implementation](#chap25.xhtml#leanpub-auto-php-implementation)
    *   [Zephir Implementation](#chap26.xhtml#leanpub-auto-zephir-implementation)
    *   [JavaScript Implementation](#chap27.xhtml#leanpub-auto-javascript-implementation)

## Introduction

### Why Write This Book

I decided to write this book after spending a great deal of time working towards libraries that simplify the methods used when working with strings, numbers, arrays etc. They’re called Scalar Types, because PHP treats them differently to objects. They have no properties, no methods.

PHP has a rich history and a dominant place on the web. It has achieved much despite language inconsistencies and difficulties, particularly when it comes to these scalar types. Bjarne Stroustrup once said: “There are only two kinds of languages: the ones people complain about and the ones nobody uses”. PHP is one of those languages that *everybody* uses, yet that’s often seen as a good reason to ignore the bad parts and just get stuff done.

I’m all for getting stuff done, and to that end I have used Plain Ol’ PHP for many years. It’s always bugged me how procedural PHP is, in an ecosystem of OOP libraries and frameworks. So I decided to take a deeper look at building a stronger type system on top of PHP.

That’s the goal of this book. We look at using *standard* PHP libraries. We look at using user-land libraries. We look at using extensions and cross-compilers. All this is works towards creating a set of reusable tools which unify and ease the scalar types of PHP.

### Who This Book Is For

This book assumes you have working knowledge of PHP. That means you understand the basics of programming, and have already employed them to write PHP code.

You don’t need to know how to set up a PHP stack - we will cover how that can easily be done (using VirtualBox, Vagrant and Phansible).

You also need to have an open mind. Many of the concepts, covered by this book, are experimental and hardly any of them are commonplace. That’s not to say that you can’t use these techniques in production applications, but it’s up to you to decide if your architecture will benefit from third-party extensions (in addition to PHP core).

Finally, you should have access to a decent Internet connection. The examples in this book work best inside a Vagrant virtual machine. If you’re unfamiliar with the term, don’t worry. We’ll talk about it later, but it’s basically software that installed development environments. That installation happens often and benefits from a fast connection.

### Procedural / Object Oriented

Procedural and Object Oriented are different programming styles which approach program execution in different ways. It’s good to understand how they work, and how they define the state of scalar types.

#### Procedural Programming

Procedural Programming describes a top-down approach to program execution. That is; Procedural programs consist of a list of steps for the interpreter to take (from top to bottom).

In pseudo-code, a Procedural image resize program may resemble:

1.  start execution
2.  then store a file reference returned by a `open_file` function
3.  then store a modified image returned by a `resize_image` function
4.  then close the open file
5.  then store a file reference returned by a `open_file` function
6.  then write the modified image data to the second open file
7.  then close the open file
8.  then empty the modified image variable
9.  end execution

Procedural programs can call on functions (as described in the example) and can define functions. The currently executing line position can be modified by things like looping constructs and goto-like constructs, but for the most part the program is framed in a set of instructions.

#### Object Oriented Programming

Object Oriented Programming describes program execution as the interaction and description of entities (or objects). These objects can have any combination of properties (another name for owned variables) and methods (another name for owned functions).

The code within these owned functions are still executed from top to bottom, but the main idea is that the program depends on inter-object communication for it’s coherence.

In pseudo-code, an Object-Oriented image resize program may resemble:

1.  start execution
2.  then create a file object
3.  then create an image resize object
4.  then pass the file object to the image resize object
5.  then create another file object
6.  then write the result of the image resize object’s `resize` method to the second file object
7.  then close the second file object
8.  then destroy the second file object
9.  then destroy the image resize object
10.  then destroy the first file object
11.  end execution

#### Which is the best?

…is the wrong question to ask. Both have their strengths and weaknesses. Object-Oriented Programming mostly leads to more code than Procedural Programming. Object-oriented Programming lends itself to better separation between concerns. Stated another way - it’s easier to think of objects and their behaviours than to think of the whole flow of a program.

Ultimately, Procedural code can be as clean as Object-Oriented code.

“Sometimes, the elegant implementation is just a function. Not a method. Not a class. Not a framework. Just a function.” - John Carmack

#### Which is PHP?

PHP is very procedural. That’s how Rasmus built it (back in the proverbial day) and that’s how it has mostly stayed. This is apparent, when it comes to how scalar variables are handled and manipulated. The PHP types are not objects. They don’t have methods or properties. If you want to do something to a PHP scalar variable (string, int, float, bool…), you pass it to a function.

Before PHP 5.3, there were no namespaces. As a result - all of these type-specific methods were added to the global namespace. They are available everywhere, and (as we’ve seen) they are inconsistent.

This makes scalar type code ugly code.

### Native Function Inconsistencies

PHP is often decried because of the inconsistencies in the native functions. PHP is often praised because of the quality of the documentation. These are related! The documentation has evolved so well because native PHP functions are inconsistent.

This section is going to make me sound like a PHP-hater. That couldn’t be further from the truth! I love PHP and I’m committed to using it and helping others use it. To know what we’re building, we have to know what we’re trying to avoid building. That’s the point of what’s to follow.

#### Sporadic Underscores

*   `parse_str`
*   `printf`
*   `str_pad`
*   `strcmp`
*   `strip_tags`
*   `stripslashes`

These functions are described at **[http://php.net/manual/en/ref.strings.php](http://php.net/manual/en/ref.strings.php)**

The full list contains 98 functions, 30 of which use one or more underscores. Sometimes functions clearly composed of multiple full words (like `setlocale`) don’t have underscores. Sometimes functions which do almost exactly the same things (`strlen` vs. `str_word_count`) are handled differently.

#### Sporadic Abbreviation

*   `addslashes`
*   `chr`
*   `htmlentities`
*   `lcfirst`
*   `number_format`
*   `stroll`

Most of the string functions use abbreviations of some kind. Applied consistently, this wouldn’t be a problem. However, the inclusion of a few non-abbreviated functions makes the string API difficult to memorise, which often means a round-trip to the documentation.

#### Inconsistent Argument Order

*   `array_key_exists($needle, $haystack)`
*   `stripos($haystack , $needle)`

The natural order of arguments differs depending on whether you are working with array functions or string functions. Rasmus explains this as a result of keeping as close to the underlying C libraries as possible. The problem with this explanation is that it means nothing to developers who have never worked with C, and just want to work with PHP.

It’s helpful to remember that the array methods are needle/haystack and the string methods are haystack/needle.

#### Regular Expression / Strings

*   `preg_filter`
*   `str_replace`
*   `preg_match`
*   `strstr`
*   `preg_split`
*   `explode`

In PHP, Regular Expressions are represented as strings but handled with a completely different set of functions. That’s just strange. While I can accept that Regular Expressions have a similar representation to strings, and that the functions they are given to (in the C libraries) expect to work with strings, developers shouldn’t need to account for this.

PHP assumes a string is a Regular Expression, if it starts and ends with recognisable delimiters. You can learn more about that at **[http://www.php.net/manual/en/regexp.reference.delimiters.php](http://www.php.net/manual/en/regexp.reference.delimiters.php)**.

Alternatively, Regular Expressions should have their own representation which clearly separates them from strings. An example of a language which already has this, is JavaScript:

```
1 "abc".replace("b", "123"); // "abc" becomes "a123c"
2 "def".replace(/[e]/, "456"); // "def" becomes "d456f"
```

These languages make clear the distinction between strings and Regular Expressions.

#### Nouns / Verbs

*   `echo`
*   `htmlentities`
*   `lcfirst`
*   `md5`
*   `parse_str`
*   `soundedex`

Some of the native functions are verbs (like `echo` and `parse_str`) while others are nouns (like `htmlentities` and `soundex`). This makes it tricky to reason about what the method is doing.

#### Strange Return Values

Many of the native functions return multiple types. In the case of `strstr` a string is returned if it is matched, else the function returns false.

In contrast to this, the `preg_match` function returns 1 if the pattern is found, 0 if it is not and false if an error occurred. This makes for a slew of type checking before any of the return values can be used for their intended purpose.

### Conclusion

PHP is a great language.

But if you’ve worked much with PHP, you will either have grown to ignore the sad state of PHP’s scalar type handling, or been frustrated at the lack of good alternatives.

And for most small-to-medium sized projects, adding a revised type-system is unnecessary. Yet if you learned how to make (or even just use) a well-built type system, wouldn’t it make sense to use it in large projects? Or in any projects you cared enough about?

This area of PHP often chases developers into prettier languages. Don’t be one of those developers! Learn how to write cleaner code, by using a clean abstraction.

## Structure

### Boxing

Boxing is a term given to the practice of wrapping (enclosing) data within a class, so that behaviour can be added to the wrapped data. Let’s look at an example:

```
 1 <?php
 2 
 3 class StringBox
 4 {
 5   /**
 6    * @var string
 7    */
 8   protected $data;
 9 
10   /**
11    * @param string $data
12    */
13   public function __construct($data)
14   {
15     $this->data = $data;
16   }
17 
18   /**
19    * @return string
20    */
21   public function toString()
22   {
23     return (string) $this->data;
24   }
25 
26   /**
27    * @return string
28    */
29   public function toUpperCase()
30   {
31     return strtoupper($this->data);
32   }
33 
34   /**
35    * @return string
36    */
37   public function toLowerCase()
38   {
39     return strtolower($this->data);
40   }
41 
42   /**
43    * @param string $needle
44    * @param mixed  $offset
45    *
46    * @return int
47    */
48   public function getIndexOf($needle, $offset = null)
49   {
50     $index = strpos($this->data, $needle, $offset);
51 
52     if ($index === false) {
53       return -1;
54     }
55 
56     return $index;
57   }
58 }
59 
60 $box = new StringBox("Hello World");
61 
62 print $box->toString();          // "hello world"
63 print $box->toUpperCase();       // "HELLO WORLD"
64 print $box->toLowerCase();       // "hello world"
65 print $box->getIndexOf("foo");   // -1
66 print $box->getIndexOf("World"); // 6
```

This approach is fully supported by PHP, without any additional extensions or dependencies. It’s a simple concept, if you think about it. You’re using protected properties and exposing a public API for setting them, getting them and manipulating them.

The difficulty with it is that it leads to a lot more code than just using the native functions. You need to define wrappers. You need to define setters and getters. You have to use them every time you want to put *native* types in and get *native* types out. What do I mean by that last statement?

```
1 $helloBox = new StringBox("Hello");
2 $worldBox = new StringBox("World");
3 
4 $helloWorldBox = new StringBox(
5   $helloBox->toString() . " " . $worldBox->toString()
6 );
```

This quickly becomes unwieldy. It’s also more memory intensive, and slower than just using the native functions and types.

PHP allows the use of a method called `__toString`. When a class defines this method (to return a string), and an instance of the class is used in an operation that expects a string; the `__toString` method is automatically called.

This isn’t enough for us to simulate an extensible type system, or this would be a short book.

### Resolving Types

PHP is a weakly typed language. That is, variables can be declared without specifying a type. Their type can be changed at any point. They can be coerced into different types on demand.

If we want to implement stronger type-handling, we need to be able to identify the type of a variable. PHP provides a number of functions which help with this, but they have a fe issues to overcome…

#### Scalar Types

The `gettype` function returns the data type of a variable. It can identify the following types:

*   `"boolean"`
*   `"integer"`
*   `"double"`
*   `"string"`
*   `"array"`
*   `"object"`
*   `"resource"`
*   `"NULL"`

If none of these types correctly describes the variable, `gettype` will return `"unknown type"`. Again we see some strange inconsistencies. Firstly, floats are identified as `"double"` and null values are returned as `"NULL"` (uppercase, in contrast to every other type identifier).

We can abstract around this function, with something resembling the following:

```
 1 <?php
 2 
 3 function type($variable) {
 4   $type = gettype($variable);
 5 
 6   switch ($type) {
 7 
 8     case "integer":
 9     case "double":
10       return "number";
11 
12     case "NULL":
13       return "null";
14 
15   }
16 
17   return $type;
18 }
19 
20 print type(0);    // "number"
21 print type(.0);   // "number"
22 print type(null); // "null"
```

What about when the variable contains a callback? In that case, we can use…

#### Functions

The `is_callable` function checks to see if something is a callable function, whether it is a string or a anonymous function.

```
 1 <?php
 2 
 3 print is_callable("is_callable"); // true
 4 print is_callable(function(){});  // true
 5 print is_callable(null);          // false
 6 
 7 class Foo
 8 {
 9   public function bar()
10   {
11 
12   }
13 
14   public function identify()
15   {
16     return is_callable([$this, "bar"]);
17   }
18 }
19 
20 $foo = new Foo();
21 
22 print $foo->identify(); // true
```

Technically, `is_callable` identifies a valid argument to any callback-accepting function in PHP. The following are all valid for these kinds of functions:

*   An actual function (`function(){}`)
*   An array of context and method name (`[$this, "bar"]`)
*   The name of a function, as a string (`"is_callable"`)

Due to how loose this method is, you need to be careful when trying to identify strings and functions in the same resolver function. If you’re using `is_string` and `is_callable` at the same time, your results may differ depending on the order in which these are called.

For example:

```
1 $variable = "is_callable";
2 
3 if (is_string($variable)) {
4   die("variable is a string");
5 }
6 
7 if (is_callable($variable)) {
8   die("variable is callable");
9 }
```

The script will terminate with the string `"variable is a string"`, because it is a string. But it’s also the name of a callable function, so swapping the conditional statements will make the script terminate in `"variable is callable"`.

If you want to get more specific, then it’s helpful to know that `gettype(function(){})` will return `"object"`. In that case, we can do this:

```
1 <?php
2 
3 function is_function($variable) {
4   return is_callable($variable) and gettype($variable) === "object";
5 }
6 
7 print is_function(function(){});  //true
8 print is_function("is_function"); //false
9 print is_function(new stdClass);  //false
```

#### Classes

Sometimes what we want to do is identify the type of object we’re working with. `gettype` won’t tell us much more than that the variable is an object. We’ll have to combine that with the `get_class` function:

```
 1 <?php
 2 
 3 function type($variable) {
 4   $type = gettype($variable);
 5 
 6   switch ($type) {
 7 
 8     case "integer":
 9     case "double":
10       return "number";
11 
12     case "NULL":
13       return "null";
14 
15     case "object":
16       return get_class($variable);
17 
18   }
19 
20   return $type;
21 }
22 
23 print type("foo"); // "string"
24 
25 class Foo
26 {
27 
28 }
29 
30 $foo = new Foo();
31 
32 print type($foo);         // "Foo"
33 print type(function(){}); // "Closure"
```

We could further normalise `"Closure"` to something like `"function"`; if we wanted to treat it with the same gravity as the other types.

#### Regular Expressions

Since PHP has multiple sets of string functions (one for plain strings and a few for regular expressions), it would be great if we had a way to differentiate between things that look like regular expressions and things that don’t.

It’s important to differentiate between definitively identifying regular expressions and identifying things that *look like*. Just because something looks like a regular expression doesn’t mean that it is. Nor does it mean that the intention of the author was for it to be a regular expression.

The PHP documentation (at [http://www.php.net/manual/en/intro.pcre.php](http://www.php.net/manual/en/intro.pcre.php)) describes expressions as [a string] enclosed in delimiters. These delimiters can be any non-alphanumeric that aren’t also a backslash or null byte.

We can cover a large majority of cases, using the following:

```
 1 <?php
 2 
 3 function is_regex($variable) {
 4   return @preg_match($regex, "") !== false
 5     && preg_last_error() == PREG_NO_ERROR;
 6 }
 7 
 8 print is_regex("/^.*$/"); // true
 9 print is_regex("/hello world/"); // true
10 print is_regex("/hello world/i"); // true
11 print is_regex("hello world"); // false
12 print is_regex("\\hello world\\"); // false
13 print is_regex("\\x00foo\\x00"); // false
14 print is_regex("1foo1"); // false
15 print is_regex("afooa"); // false
```

The `preg_match` function returns a `1` for a match, `0` for no match and `false` if an error occurred. Assuming `preg_match` returns false, the error could be for any number of reasons (like `PREG_INTERNAL_ERROR` or `PREG_BAD_UTF8_ERROR`). So then we use the `preg_last_error` function to rule out all other errors.

Using error suppression (`@`) is generally a bad idea. There are very few instances when using it is considered ok. This is one of them!

### Namespace Functions

I’ve spoke a bit about how all these global functions are bad, and there’s a really simple alternative: namespace functions. You’ve probably used namespaces, for classes, but the same constructs work well to isolate functions:

```
 1 <?php
 2 
 3 namespace Type\String {
 4   function length($string) {
 5     return strlen($string);
 6   }
 7 }
 8 
 9 namespace {
10   print Type\String\length("Hello World");
11 }
```

#### Composer Autoload

This kind of function definition doesn’t follow normal autoload patterns. In order to have Composer autoload these kinds of files is to explicitly define which files should be loaded:

```
1 {
2   "autoload" : {
3     "files" : [
4       "namespace-functions.php"
5     ]
6   }
7 }
```

Following this, we’ll have to dump the old autoloader, with:

```
1 $ composer dump-autoload
2 
3 Generating autoload files
```

The files (containing namespace functions) will now be automatically loaded, wherever the Composer autoloader is used.

#### Future Goodies

PHP 5.6 supports the `use function` construct, to import functions from other namespaces. It’s currently in beta (at the time of writing), but this feature is right up our proverbial alley.

In the meantime, it’s possible to partially import functions:

```
1 use Type\String;
2 
3 print String\length("Hello World");
```

This is especially useful for deeply-nested namespace designs. Still; it’s great to be able to omit even a little of the full namespace. The result is clean(er), non-colliding procedural code.

### Conclusion

Even today, it’s possible to create a cleaner scalar type system/abstraction. We don’t need special extensions, or compilation steps. Just a little bit of work will clean the code right up.

Having said that, extensions can go a long way towards making our lives easier, while still allowing the level of customisation and unity that we are hoping to achieve. We’ll look at a few of these in the following chapter…

## Extensions

### Vagrant + Phansible

Many of the libraries, we will be working with, require a bit of special installation. Instead of labouring away at a guide for each operating system, we’re going to look at using Vagrant for our development environment.

Vagrant (in case you haven’t heard of it yet) is a programatic interface for managing virtual machines. If you’ve ever set up a virtual machine, installed the operating system, installed the development tools, you will know how much time it takes to do well. This process can be automated with Vagrant.

Vagrant depends on underlying virtualisation providers and provisioners. We’ll look at using VirtualBox as the virtualisation provider and Ansible as the provisioner.

#### Installing

To get VirtualBox installed, go to [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads) and download the installer for your operating system.

![](images/virtualbox-download.png) ![](images/virtualbox-install.png)

Once you have downloaded and installed VirtualBox, you should be able to install Vagrant. Go to [http://www.vagrantup.com/downloads.html](http://www.vagrantup.com/downloads.html) and download the installer for your operating system.

![](images/vagrant-download.png) ![](images/vagrant-install.png)

We’ll also need to install Ansible so we can use play books to provision the virtual machine. Go to [http://docs.ansible.com/intro_installation.html](http://docs.ansible.com/intro_installation.html) and download the installer for your operating system.

![](images/ansible-download.png)

#### Provisioning

Provisioning is just another word for a set of instructions that tell Vagrant which dependencies to install for you. There are many kinds of Vagrant provisioners, but the only one we will use is called Ansible.

We’ll use [http://phansible.com](http://phansible.com) to do most of the heavy-lifting.

![](images/phansible.png)

Set the following options:

1.  Operating System: Ubuntu Precise Pangolin 64
2.  Webserver: Nginx + PHP5-FPM
3.  PHP: 5.5
4.  Composer: Enabled
5.  PHP Modules: php-pear php5-cli php5-common

When you click “Generate”, you’ll start downloading an archive of files. Extract these into a working directory and start up Terminal.

These files are instruction files (Ansible-specific YAML syntax) which describe which dependencies Vagrant must automatically install. You shouldn’t need to change them to get the PHP stack working, but feel free to familiarise yourself with what they are doing.

To start the virtual machine, run the following command:

```
1 $ vagrant up
2 
3 Bringing machine 'default' up with 'virtualbox' provider...
4 ==> default: Importing base box 'precise64'...
5 ==> default: Matching MAC address for NAT networking...
6 ==> default: Setting the name of the VM: Default
7 ==> default: Clearing any previously set network interfaces...
```

You may be asked to provide an administrator password, as part of setting up the virtual machine. This will allow the NFS shared folders to be set up.

Once the virtual machine is initialised, you can log into it with:

```
1 $ vagrant ssh
2 
3 Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)
4 
5  * Documentation:  https://help.ubuntu.com/
6 Welcome to your Vagrant-built virtual machine.
7 Last login: Tue May 27 11:16:05 2014 from 10.0.2.2
8 
9 vagrant@precise64:~$
```

You can also check the installed version of PHP (and that the CLI was installed correctly) with:

```
1 $ php -v
2 
3 PHP 5.5.12-2+deb.sury.org~precise+1 (cli) (built: May 12 2014 13:46:35)
4 Copyright (c) 1997-2014 The PHP Group
5 Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
6     with Zend OPcache v7.0.4-dev, Copyright (c) 1999-2014, by Zend Technologies
```

#### Vagrant Commands

There are a few Vagrant commands you’re likely to use often:

```
1 $ vagrant up
```

This command will start the Vagrant virtual machine, and run any outstanding provisioning. That means the first time you run this command, it will take longer boot (depending on the complexity of your provisioning scripts).

```
1 $ vagrant halt
```

This command will shut the virtual machine down. It will try to gracefully shut the machine down.

```
1 $ vagrant destroy
```

This command will remove the virtual machine and clean up settings applied when it was created. If you break something inside the visual machine, and you want to *reset* it to the default state, you’ll want to run this command followed by `vagrant up`.

```
1 $ vagrant ssh
```

This command will take you inside the virtual machine, just as if you were connecting to a remote server. Inside the virtual machine, you can run any of the commands usually supported by the guest operating system, including executing PHP scripts against the packages installed on the virtual machine.

[http://phansible.com](http://phansible.com) is the brain-child of [Erika Heidi](https://twitter.com/erikaheidi). She is also the author of [Vagrant Cookbook](https://leanpub.com/vagrantcookbook). I highly recommend reading this book, if you have any questions or simply want to know more about Vagrant!

### SPL Types

SPL (or Standard PHP Library) is a library of additional types to augment those native to core PHP. There are some popular classes (like `LogicException` and `ArrayObject`). Some of the SPL ships with *standard* PHP installations. The parts we’re going to look at shortly do not…

You can find these mystery libraries at **http://www.php.net/manual/en/book.spl-types.php**

This section assumes you’re using the Vagrant box explained earlier. If not, please set that up first. These commands are Linux-specific and depend on the pre-installed modules explained earlier.

#### Installing SPL Types

To install them, run the following commands:

```
1 $ sudo apt-get install libpcre3-dev php5-dev
2 
3 Reading package lists... Done
4 Building dependency tree
5 Reading state information... Done
6 The following extra packages will be installed:
7   autoconf automake autotools-dev build-essential...
```

```
1 $ sudo pecl install SPL_Types
2 
3 downloading SPL_Types-0.4.0.tgz ...
4 Starting to download SPL_Types-0.4.0.tgz (8,388 bytes)
5 .....done: 8,388 bytes
6 6 source files, building
7 running: phpize...
```

These two commands install the pre-requisites for compiling PECL extensions. PECL is a repository just like PEAR. If you’ve heard of neither, then don’t worry. All you need to know is that the SPL Types are hosted here, so to install them we need to be able to compile PECL extensions.

```
1 $ sudo bash -c "echo extension=spl_types.so >> /etc/php5/cli/php.ini"
```

This command appends `extension=spl_types.so` to the `php.ini` file (as per the installation instructions).

```
1 $ sudo service php5-fpm restart
2 
3 php5-fpm stop/waiting
4 php5-fpm start/running, process...
```

This command restarts PHP-FPM - the process that interprets PHP command line instructions and Nginx web requests. These commands should have installed the SPL Types, but just to be sure, run the following command:

```
1 $ php -i | grep SPL_Types
2 SPL_Types
```

If you see that `SPL_Types` line, then you should be good to go!

#### Using SPL Types

Let’s look at a few examples of how these classes can be used:

```
 1 <?php
 2 
 3 class NumberType extends SplFloat
 4 {
 5   /**
 6    * @return float
 7    */
 8   public function toInteger()
 9   {
10     return round($this);
11   }
12 
13   /**
14    * @return string
15    */
16   public function toString()
17   {
18     return (string) $this;
19   }
20 }
21 
22 $number = new NumberType(13.86);
23 
24 print $number->toInteger(); // 14
25 print $number->toString();  // "13.86"
26 
27 print (float) $number + 1.00; // 14.86
28 print $number * 12;           // 156
```

The `toInteger` and `toString` methods do similar things to the box classes. The magic happens when we do basic arithmetic with the $number object. SPL Types are automatically unboxed when they are used in arithmetic expressions, or cast or concatenated. Any operator that would normally with with a scalar type will work with the corresponding SPL Type.

Here’s another example:

```
 1 <?php
 2 
 3 class StringType extends SplString
 4 {
 5   /**
 6    * @param int   $start
 7    * @param mixed $length
 8    *
 9    * @return StringType
10    */
11   public function slice($start = 0, $length = null)
12   {
13     if ($length === null) {
14       return new static(substr($this, $start));
15     }
16 
17     return new static(substr($this, $start, $length));
18   }
19 }
20 
21 $string = new StringType("Hello World");
22 
23 print $string->slice(6); // "World"
```

We can design our types so that they return new instances. This gives us a simple *chaining* interface.

Be careful when assuming the return type of native PHP functions. Be sure to check them before the call to `new static()` or you may encounter fatal errors.

### Scalar Objects

[Nikita Popov](https://twitter.com/nikita_ppv) is a prolific contributor to PHP (both core and user-land). He’s made libraries such as [PHP-Parser](https://github.com/nikic/PHP-Parser) which is used all over the web, and in popular frameworks (like [Laravel](http://laravel.com)). He’s championed many multiple RFCs, which have become parts of core PHP.

He’s also created a custom extension which allows the registration of custom type handlers. You can find it at [https://github.com/nikic/scalar_objects](https://github.com/nikic/scalar_objects).

We’re going to install and use this module to get even closer to our ideal type handling situation…

This section assumes you’re using the Vagrant box explained earlier. If not, please set that up first. These commands are Linux-specific and depend on the pre-installed modules explained earlier.

#### Installing Scalar Objects

First up, we need to install the Git command-line tool:

```
1 $ sudo apt-get install git
2 
3 Reading package lists... Done
4 Building dependency tree
5 Reading state information... Done
6 The following extra packages will be installed:
7   git-man liberror-perl...
```

Following this, we can clone and build the extension:

```
1 $ git clone https://github.com/nikic/scalar_objects.git
2 
3 Cloning into 'scalar_objects'...
4 remote: Reusing existing pack: 213, done.
5 remote: Total 213 (delta 0), reused 0 (delta 0)
6 Receiving objects: 100% (213/213), 75.36 KiB, done.
7 Resolving deltas: 100% (112/112), done.
```

```
1 $ cd scalar_objects && phpize && ./configure && make && sudo make install
```

These commands will the Scalar Objects extension, but we still need to add it to the configuration: `$ sudo bash -c "echo extension=scalar_objects.so >> /etc/php5/cli/php.ini"`

This command resembles the one we used to install the SPL Types. We’re essentially doing the same thing, so we need to restart PHP5-FPM:

```
1 $ sudo service php5-fpm restart
2 
3 php5-fpm stop/waiting
4 php5-fpm start/running, process...
```

This should complete the process of installing the Scalar Objects extension, but we can make sure it’s working by running the following command:

```
1 $ php -i | grep scalar
2 
3 scalar_objects
4 scalar-objects support => enabled
```

#### Using Scalar Objects

The new extension adds a method we can use to register these type handlers. This is how you we can use it:

```
 1 <?php
 2 
 3 class StringHandler
 4 {
 5   /**
 6    * @param int   $start
 7    * @param mixed $length
 8    *
 9    * @return StringType
10    */
11   public function slice($start = 0, $length = null)
12   {
13     if ($length === null) {
14       return substr($this, $start);
15     }
16 
17     return substr($this, $start, $length);
18   }
19 }
20 
21 register_primitive_type_handler("string", "StringHandler");
22 
23 $string = "Hello World";
24 
25 print $string->slice(6); // "World"
```

This is easier than boxing scalar types, as we don’t have to pull *native* types out of class instances. This is easier than SPL Types, as we don’t have to put *native* types into class instances.

There are seven supported types:

*   `null`
*   `bool`
*   `int`
*   `float`
*   `string`
*   `array`
*   `resource`

You may be wondering whether this extension plays nicely with SPL Types. The answer is *probably not*. You shouldn’t mix these extensions and since the Scalar Objects extension does everything SPL Types you shouldn’t need both.

### Zephir

Zephir is a framework for writing compilable and installable PHP extensions, using a PHP superset language sharing some similarity with C code. Zephir isn’t strictly a PHP extension, nor are Zephir libraries written in true PHP.

It’s part of the same collective from which the Phalcon framework comes, and Phalcon is itself a PHP extension. This should get interesting!

#### Installing Zephir

Zephir requires a few libraries, in order to compile correctly. We can install these with:

```
1 $ sudo apt-get install git gcc make re2c php5 php5-json php5-dev libpcre3-dev
```

Next, we need to install the JSON-C library (which Zephir uses to compile extensions):

```
1 $ git clone https://github.com/json-c/json-c.git && cd json-c && sh autogen.sh\
2  && ./configure && make && sudo make install
```

These commands will clone the JSON-C repository, configure and compile it. Finally, we need to install Zephir:

```
1 $ git clone https://github.com/phalcon/zephir && cd zephir && ./install -c
```

That should have installed a usable version of Zephir. You can check that it’s working by heading into the clone folder and running:

```
1 $ zephir version
```

#### Using Zephir

Using Zephir is relatively simple (considering the work that actually goes on). Let’s begin by initialising a new extension skeleton project:

```
1 $ zephir init type
```

This will create a skeleton project folder in the current working directory. Navigate into the new `type` directory and run:

```
1 $ ls -la
2 
3 ext/ type/ config.json
```

Extension classes go in the `type` folder (it’s specific to the name of the extension, which we gave the `init` command). Make a find in there, called `StringType.zep`, and open that file in your editor.

The Zephir syntax is quite similar to PHP, but with a twist of C style. You can find a reasonable amount of documentation at [http://www.zephir-lang.com/index.html](http://www.zephir-lang.com/index.html).

Create the following class:

```
 1 namespace Type;
 2 
 3 class StringType
 4 {
 5   protected data;
 6 
 7   public function __construct(var data)
 8   {
 9     let this->data = data;
10   }
11 
12   public function length()
13   {
14     return strlen(this->data);
15   }
16 }
```

Other than the missing `$` symbols, and the `var`/`let` keywords added, this is pretty understandable. Save the file and (from the base extension folder) run:

```
 1 $ zephir build
 2 
 3 Compiling...
 4 /bin/bash /vagrant/zephir/type/ext/libtool --mode=compile gcc  -I. -I/vagrant/\
 5 zephir/type/ext -DPHP_ATOM_INC -I/vagrant/zephir/type/ext/include -I/vagrant/z\
 6 ephir/type/ext/main -I/vagrant/zephir/type/ext -I/usr/include/php5 -I/usr/incl\
 7 ude/php5/main -I/usr/include/php5/TSRM -I/usr/include/php5/Zend -I/usr/include\
 8 /php5/ext -I/usr/include/php5/ext/date/lib  -DHAVE_CONFIG_H  -O2 -fvisibility=\
 9 hidden -Wparentheses -flto   -c /vagrant/zephir/type/ext/type/stringtype.zep.c\
10  -o type/stringtype.lo...
```

Zephir cross-compiles the extension class files to Plain Ol’ C, and adds the class loading code. The end of the build output should look something like:

```
1 Installing...
2 Extension installed!
3 Add extension=type.so to your php.ini
4 Don't forget to restart your web server
```

…So we need to add the extension to the `php.ini` file:

```
1 $ sudo bash -c "echo extension=type.so >> /etc/php5/cli/php.ini"
```

This will have installed the Type extension we just created. We can check that it’s installed by running:

```
1 $ php -i | grep "type => enabled"
2 
3 type => enabled
```

If you see that line returned, you know the extension is installed, and ready to go.

Using this new extension is as simple as:

```
1 <?php
2 
3 $string = new Type\StringType("Hello World");
4 
5 print $string->length();
```

The namespace and class reside completely within the compiled extension file. Zephir extensions can use pre-existing core and extension namespaces/classes. The can be used by Plain Ol’ PHP code (provided the extension is registered by the time it’s used).

Zephir extensions can even override core functions, with better-performing versions.

### Conclusion

Extensions make our lives significantly easier, by handling things like boxing and unboxing for us. They let us create better-performing code (as in the case of Zephir) and stricter types (as in the case of SPL Types).

We don’t have to use these to make a cleaner system. If we do, we can expect to have a much strong type system, without the hard work that library-only code expects of us.

## Design

### Which Method To Use

We’ve had a look at a number of methods and extensions which can help us to abstract away the inconsistent type handling PHP presents to us. Part of creating this abstraction is deciding on which methods and/or extensions to use.

#### Namespace Methods

Generally, we should avoid any method that would expose a significant amount of functions in the global scope. We don’t want to create any more clutter than there already is, and being able to use our abstraction, in addition to the standard stuff, is definitely beneficial.

> We should endeavour to have all our code reside in namespaces.

Furthermore, we should implement our code in such a way that it can be used both in procedural environments and object-oriented environments. Stated differently; we should contain the business logic within functions and call those functions (so that they can be used in procedural code) from within an object-oriented framework (so that scalar types can be used like objects).

> We should build our logic in functions and add those functions (as methods) to scalar type objects.

#### Scalar Objects / SPL Types

This means we will want to use the namespace functions we covered earlier, and that we will also want to use either the scalar objects or the SPL Types as an Object-Oriented basis for our type system.

I mentioned that the Scalar Objects and SPL Types extensions shouldn’t be used together. That’s because they do the same thing. It’s not possible to proxy function calls (in the way I’ve just described) without repeating the whole Object-Oriented side of things for each extension.

This is due to the fact that the Scalar Objects extension automatically boxes variables while the SPL Types extension expects developers to box their own variables. It’s the different between `return new static(substr($this, $offset, $length));` and `return substr($this, $offset, $length);`

While I love the Scalar Objects extension, I am inclined to suggest with the SPL Types extension. The main reason is that it doesn’t interfere with how PHP handles scalar types. This means we will need to box variables, but we’ll also enjoy the benefits of state (object-like scalar variables) and automatic unboxing.

#### Zephir

As cool as Zephir is, it’s not PHP. That means any developers you want to work on your code, will need to know or learn another language. It also increases the time between coding, testing and shipping.

So, for the remainder of the book, I will show code that is built on top of SPL Types and not translated into Zephir extensions.

The appendices [will] include a PHP and a Zephir implementation. I don’t want the implementation to be a focus of the book, but I will enjoy making it!

### How To Structure The Types

We’ve already decided to use namespaced functions as the basis, so we need to decide how to use these from within classes.

#### Resolving Types

We’ll often need to resolve types, within the type methods. Any arguments could potentially be the wrong type. It makes sense for the type resolution/conversion functions to be in their own namespace:

*   `Type\isString`
*   `Type\isStringObject`
*   `Type\isExpression`
*   …
*   `Type\toStringObject`
*   `Type\toExpression`
*   …

We should proxy to the creation methods:

```
 1 class StringObject extends SPLString
 2 {
 3   public function trim($mask = "\t\n\r\0\x0B")
 4   {
 5     $isString       = Type\isString($mask);
 6     $isStringObject = Type\isStringObject($mask);
 7 
 8     if ($isString or $isStringObject) {
 9       if (Type\isExpression($mask)) {
10         $raw = Type\String\trimWithExpression(
11           $this,
12           $mask
13         );
14       } else {
15         $raw = Type\String\trimWithString(
16           $this,
17           $mask
18         );
19       }
20 
21       return Type\toStringObject($raw);
22     }
23 
24     throw new LogicException("mask is not a string");
25   }
26 }
```

#### Chaining

One of the benefits of the object-oriented approach we’re taking is that we can chain calls on the types. You may have noticed how this is implemented (from the previous example), but in-case you didn’t:

```
1 return Type\toStringObject($raw);
```

The manipulation methods should return plain PHP types - because we want people to be able to use the types interchangeably. So it falls to the classes (proxies) to wrap plain PHP types within the SPL Types.

#### Combining Number Types

I’ve already alluded to my desire of a single number type, and that’s exactly what I want to achieve here. For that reason, we’ll completely ignore the `SPLNumber` class in favour of `SPLFloat`.

The reason is simple - numbers should be able to handle decimal points, without a separate set of methods or additional mental overhead. This will lead to additional casting in numeric operations, but that’s not the end of the world.

### How To Test

Testing is an essential part of supplanting the native type handling system. The good news is that it will be easy to do!

#### PHPUnit

PHPUnit is a unit-testing library which makes the process of testing small units of code super easy:

```
 1 class StringObjectTest extends PHPUnit_Framework_TestCase
 2 {
 3   /** @test */
 4   public function trimWorksWithStrings()
 5   {
 6     $subject = Type\toStringObject("Hello World...");
 7 
 8     $this->assertEquals(
 9       "Hello World",
10       $object->trim(".")
11     );
12   }
13 }
```

This kind of testing can be done with a number of testing frameworks. At the end of the day, as long as you are writing tests, whichever testing framework you use is up to you.

#### What Should Be Tested?

The short answer is: everything.

We’re aiming to build something solid, and it’s pretty low-level. That means we need to be sure things continue to work as expected. It’s not even that hard when you consider how simple the methods are that we are going to make.

We should aim to have good coverage of the namespaced functions, and a few tests to ensure that these are correctly called from the (proxy) classes.

#### When Should Tests Be Written?

This is really up to you. Maybe you want to write your tests first, and follow the red-green-refactor cycle. Maybe you want to write all of your library code first, and then make sure everything works as you expected it to (by writing tests for the library code).

The important thing is to write tests.

#### Recommendations

I highly recommend the following books:

*   [Clean Code, by Robert C. Martin](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
*   [The Grumpy Programmer’s PHPUnit Cookbook, by Chris Hartjes](https://leanpub.com/grumpy-phpunit)

### How To Package

If you want your code to be used (by anyone else), you’d be wise to package it in such a way that others will want to use it.

#### Make A `Readme` File

It sounds simple, but you’d be surprised how few developers actually do this! When other developers stumble across your repository (on Github/BitBucket/what-have-you), they won’t be impressed to see a bare-bones directory structure and nothing else.

Make a `Readme` file, and include the following…

#### A Few Examples

Show what your library can do. There are few things more frustrating than being enticed (to learn more about a library) by some marketing, only to have to figure it out from tests and/or source-code.

Those things are excellent places to learn, but they must be sought out, whereas a few examples (in your `Readme` file) are easy to find. Include examples covering the major aspects of your library.

In our case, this means an example (or two) about the procedural code underpinning the library. It means an example (or two) about the object-oriented wrappers, which provide fluent interfaces to traditionally procedural actions. It means providing an example (or two) about who to subclass the types for further customisation.

It’s not hard to make these - they can even be extracted from your unit tests…

#### Testing Instructions

Developers want to know how solid your code is. Sure you’re using [SemVer](http://semver.org), but how much test coverage do you have? Do your tests pass? Can I trust you?

The easiest want to answer these questions is to write the unit tests, and then provide instructions for how they can be run. You don’t require anything elaborate:

```
1 $ composer install && phpunit
```

That’s how you might be running your unit tests, and it’s easy to tell others how to run them.

#### Installation Instructions

Ok - you’ve convinced someone to use your [hopefully] well-tested library. The examples provide its value. So how do they install your library?

This is where you make a `composer.json` file and include Composer installation instructions. This will require a round-trip to [Packagist](https://packagist.org), but you’ll be all the better for it.

Then just a simple set of instructions is all you need add:

```
1 $ composer require "vendor/library:1.0.0"
```

#### License

Include an open-source license. Something friendly like [MIT](http://opensource.org/licenses/MIT) will do nicely. Specify this in your `Readme` file, and include a clearly-named file (like `LICENSE` or `license.md`) which contains the full license.

#### Contribution Guidelines

This is optional, but greatly increases the chance that other developers will send compatible pull-requests. If you are fussy about code style (and you should be), then be sure to tell people how you want their submissions to look.

### Conclusion

Building a new type system is only partly about code. There’s a lot of design consideration that goes into well-crafted libraries. Take the time to decide on a reasonable structure - one that allows consumers the most flexibility.

Test well. Package well. Be clear.

## Designing For The Future

This chapter is incomplete.

## Appendices

### PHP Implementation

This section is incomplete.

### Zephir Implementation

This section is incomplete.

### JavaScript Implementation

This section is incomplete.