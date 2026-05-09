![](images/cover.jpg)

PHP Quick Scripting Reference

![image](images/FM-00.jpg)

Mikael Olsson

![image](images/FM-01.jpg)

**PHP Quick Scripting Reference**

Copyright © 2013 by Mikael Olsson

This work is subject to copyright. All rights are reserved by the Publisher, whether the whole or part of the material is concerned, specifically the rights of translation, reprinting, reuse of illustrations, recitation, broadcasting, reproduction on microfilms or in any other physical way, and transmission or information storage and retrieval, electronic adaptation, computer software, or by similar or dissimilar methodology now known or hereafter developed. Exempted from this legal reservation are brief excerpts in connection with reviews or scholarly analysis or material supplied specifically for the purpose of being entered and executed on a computer system, for exclusive use by the purchaser of the work. Duplication of this publication or parts thereof is permitted only under the provisions of the Copyright Law of the Publisher’s location, in its current version, and permission for use must always be obtained from Springer. Permissions for use may be obtained through RightsLink at the Copyright Clearance Center. Violations are liable to prosecution under the respective Copyright Law.

ISBN-13 (pbk): 978-1-4302-6283-1

ISBN-13 (electronic): 978-1-4302-6284-8

Trademarked names, logos, and images may appear in this book. Rather than use a trademark symbol with every occurrence of a trademarked name, logo, or image we use the names, logos, and images only in an editorial fashion and to the benefit of the trademark owner, with no intention of infringement of the trademark.

The use in this publication of trade names, trademarks, service marks, and similar terms, even if they are not identified as such, is not to be taken as an expression of opinion as to whether or not they are subject to proprietary rights.

While the advice and information in this book are believed to be true and accurate at the date of publication, neither the authors nor the editors nor the publisher can accept any legal responsibility for any errors or omissions that may be made. The publisher makes no warranty, express or implied, with respect to the material contained herein.

President and Publisher: Paul Manning

Lead Editor: Steve Anglin

Development Editor: Douglas Pundick

Technical Reviewer: Jamie Rumbelow

Editorial Board: Steve Anglin, Mark Beckner, Ewan Buckingham, Gary Cornell, Louise Corrigan, Morgan Ertel, Jonathan Gennick, Jonathan Hassell, Robert Hutchinson, Michelle Lowman, James Markham, Matthew Moodie, Jeff Olson, Jeffrey Pepper, Douglas Pundick, Ben Renow-Clarke, Dominic Shakeshaft, Gwenan Spearing, Matt Wade, Tom Welsh

Coordinating Editor: Christine Ricketts

Copy Editor: XXXXX

Compositor: SPi Global

Indexer: SPi Global

Artist: SPi Global

Cover Designer: Anna Ishchenko

Distributed to the book trade worldwide by Springer Science+Business Media New York, 233 Spring Street, 6th Floor, New York, NY 10013\. Phone 1-800-SPRINGER, fax (201) 348-4505, e-mail `orders-ny@springer-sbm.com`, or visit `www.springeronline.com`. Apress Media, LLC is a California LLC and the sole member (owner) is Springer Science + Business Media Finance Inc (SSBM Finance Inc). SSBM Finance Inc is a **Delaware** corporation.

For information on translations, please e-mail `rights@apress.com`, or visit `www.apress.com`.

Apress and friends of ED books may be purchased in bulk for academic, corporate, or promotional use. eBook versions and licenses are also available for most titles. For more information, reference our Special Bulk Sales–eBook Licensing web page at `www.apress.com/bulk-sales`.

Any source code or other supplementary material referenced by the author in this text is available to readers at `www.apress.com`. For detailed information about how to locate your book’s source code, go to `www.apress.com/source-code/`.

Contents at a Glance

[About the Author](#9781430262831_About_the_Author.xhtml)

[Introduction](#9781430262831_Introduction.xhtml)

![image](images/sq1.jpg)[ Chapter 1: Using PHP](#9781430262831_Ch01.xhtml)

![image](images/sq1.jpg)[ Chapter 2: Variables](#9781430262831_Ch02.xhtml)

![image](images/sq1.jpg)[ Chapter 3: Operators](#9781430262831_Ch03.xhtml)

![image](images/sq1.jpg)[ Chapter 4: String](#9781430262831_Ch04.xhtml)

![image](images/sq1.jpg)[ Chapter 5: Arrays](#9781430262831_Ch05.xhtml)

![image](images/sq1.jpg)[ Chapter 6: Conditionals](#9781430262831_Ch06.xhtml)

![image](images/sq1.jpg)[ Chapter 7: Loops](#9781430262831_Ch07.xhtml)

![image](images/sq1.jpg)[ Chapter 8: Functions](#9781430262831_Ch08.xhtml)

![image](images/sq1.jpg)[ Chapter 9: Class](#9781430262831_Ch09.xhtml)

![image](images/sq1.jpg)[ Chapter 10: Inheritance](#9781430262831_Ch10.xhtml)

![image](images/sq1.jpg)[ Chapter 11: Access Levels](#9781430262831_Ch11.xhtml)

![image](images/sq1.jpg)[ Chapter 12: Static](#9781430262831_Ch12.xhtml)

![image](images/sq1.jpg)[ Chapter 13: Constants](#9781430262831_Ch13.xhtml)

![image](images/sq1.jpg)[ Chapter 14: Interface](#9781430262831_Ch14.xhtml)

![image](images/sq1.jpg)[ Chapter 15: Abstract](#9781430262831_Ch15.xhtml)

![image](images/sq1.jpg)[ Chapter 16: Traits](#9781430262831_Ch16.xhtml)

![image](images/sq1.jpg)[ Chapter 17: Importing Files](#9781430262831_Ch17.xhtml)

![image](images/sq1.jpg)[ Chapter 18: Type Hinting](#9781430262831_Ch18.xhtml)

![image](images/sq1.jpg)[ Chapter 19: Type Conversions](#9781430262831_Ch19.xhtml)

![image](images/sq1.jpg)[ Chapter 20: Variable Testing](#9781430262831_Ch20.xhtml)

![image](images/sq1.jpg)[ Chapter 21: Overloading](#9781430262831_Ch21.xhtml)

![image](images/sq1.jpg)[ Chapter 22: Magic Methods](#9781430262831_Ch22.xhtml)

![image](images/sq1.jpg)[ Chapter 23: User Input](#9781430262831_Ch23.xhtml)

![image](images/sq1.jpg)[ Chapter 24: Cookies](#9781430262831_Ch24.xhtml)

![image](images/sq1.jpg)[ Chapter 25: Sessions](#9781430262831_Ch25.xhtml)

![image](images/sq1.jpg)[ Chapter 26: Namespaces](#9781430262831_Ch26.xhtml)

![image](images/sq1.jpg)[ Chapter 27: References](#9781430262831_Ch27.xhtml)

![image](images/sq1.jpg)[ Chapter 28: Advanced Variables](#9781430262831_Ch28.xhtml)

![image](images/sq1.jpg)[ Chapter 29: Error Handling](#9781430262831_Ch29.xhtml)

![image](images/sq1.jpg)[ Chapter 30: Exception Handling](#9781430262831_Ch30.xhtml)

[Index](#9781430262831_Index.xhtml)

Contents

[ About the Author](#9781430262831_About_the_Author.xhtml)

[ Introduction](#9781430262831_Introduction.xhtml)

![images](images/sq1.jpg)[ Chapter 1: Using PHP](#9781430262831_Ch01.xhtml)

[Embedding PHP](#9781430262831_Ch01.xhtml#Sec1)

[Outputting text](#9781430262831_Ch01.xhtml#Sec2)

[Installing a web server](#9781430262831_Ch01.xhtml#Sec3)

[Hello world](#9781430262831_Ch01.xhtml#Sec4)

[Compile and parse](#9781430262831_Ch01.xhtml#Sec5)

[Comments](#9781430262831_Ch01.xhtml#Sec6)

![images](images/sq1.jpg)[ Chapter 2: Variables](#9781430262831_Ch02.xhtml)

[Defining variables](#9781430262831_Ch02.xhtml#Sec1)

[Data types](#9781430262831_Ch02.xhtml#Sec2)

[Integer type](#9781430262831_Ch02.xhtml#Sec3)

[Floating-point type](#9781430262831_Ch02.xhtml#Sec4)

[Bool type](#9781430262831_Ch02.xhtml#Sec5)

[Null type](#9781430262831_Ch02.xhtml#Sec6)

[Default values](#9781430262831_Ch02.xhtml#Sec7)

![images](images/sq1.jpg)[ Chapter 3: Operators](#9781430262831_Ch03.xhtml)

[Arithmetic operators](#9781430262831_Ch03.xhtml#Sec1)

[Assignment operators](#9781430262831_Ch03.xhtml#Sec2)

[Combined assignment operators](#9781430262831_Ch03.xhtml#Sec3)

[Increment and decrement operators](#9781430262831_Ch03.xhtml#Sec4)

[Comparison operators](#9781430262831_Ch03.xhtml#Sec5)

[Logical operators](#9781430262831_Ch03.xhtml#Sec6)

[Bitwise operators](#9781430262831_Ch03.xhtml#Sec7)

[Operator precedence](#9781430262831_Ch03.xhtml#Sec8)

[Additional logical operators](#9781430262831_Ch03.xhtml#Sec9)

![images](images/sq1.jpg)[ Chapter 4: String](#9781430262831_Ch04.xhtml)

[String concatenation](#9781430262831_Ch04.xhtml#Sec1)

[Delimiting strings](#9781430262831_Ch04.xhtml#Sec2)

[Heredoc strings](#9781430262831_Ch04.xhtml#Sec3)

[Nowdoc strings](#9781430262831_Ch04.xhtml#Sec4)

[Escape characters](#9781430262831_Ch04.xhtml#Sec5)

[Character reference](#9781430262831_Ch04.xhtml#Sec6)

[String compare](#9781430262831_Ch04.xhtml#Sec7)

![images](images/sq1.jpg)[ Chapter 5: Arrays](#9781430262831_Ch05.xhtml)

[Numeric arrays](#9781430262831_Ch05.xhtml#Sec1)

[Associative arrays](#9781430262831_Ch05.xhtml#Sec2)

[Mixed arrays](#9781430262831_Ch05.xhtml#Sec3)

[Multi-dimensional arrays](#9781430262831_Ch05.xhtml#Sec4)

![images](images/sq1.jpg)[ Chapter 6: Conditionals](#9781430262831_Ch06.xhtml)

[If statement](#9781430262831_Ch06.xhtml#Sec1)

[Switch statement](#9781430262831_Ch06.xhtml#Sec2)

[Alternative syntax](#9781430262831_Ch06.xhtml#Sec3)

[Mixed modes](#9781430262831_Ch06.xhtml#Sec4)

[Ternary operator](#9781430262831_Ch06.xhtml#Sec5)

![images](images/sq1.jpg)[ Chapter 7: Loops](#9781430262831_Ch07.xhtml)

[While loop](#9781430262831_Ch07.xhtml#Sec1)

[Do-while loop](#9781430262831_Ch07.xhtml#Sec2)

[For loop](#9781430262831_Ch07.xhtml#Sec3)

[Foreach loop](#9781430262831_Ch07.xhtml#Sec4)

[Alternative syntax](#9781430262831_Ch07.xhtml#Sec5)

[Break](#9781430262831_Ch07.xhtml#Sec6)

[Continue](#9781430262831_Ch07.xhtml#Sec7)

[Goto](#9781430262831_Ch07.xhtml#Sec8)

![images](images/sq1.jpg)[ Chapter 8: Functions](#9781430262831_Ch08.xhtml)

[Defining functions](#9781430262831_Ch08.xhtml#Sec1)

[Calling functions](#9781430262831_Ch08.xhtml#Sec2)

[Function parameters](#9781430262831_Ch08.xhtml#Sec3)

[Default parameters](#9781430262831_Ch08.xhtml#Sec4)

[Variable parameter list](#9781430262831_Ch08.xhtml#Sec5)

[Return statement](#9781430262831_Ch08.xhtml#Sec6)

[Scope and lifetime](#9781430262831_Ch08.xhtml#Sec7)

[Anonymous functions](#9781430262831_Ch08.xhtml#Sec8)

[Function overloading](#9781430262831_Ch08.xhtml#Sec9)

[Built-in functions](#9781430262831_Ch08.xhtml#Sec10)

![images](images/sq1.jpg)[ Chapter 9: Class](#9781430262831_Ch09.xhtml)

[Instantiating an object](#9781430262831_Ch09.xhtml#Sec1)

[Accessing object members](#9781430262831_Ch09.xhtml#Sec2)

[Initial property values](#9781430262831_Ch09.xhtml#Sec3)

[Constructor](#9781430262831_Ch09.xhtml#Sec4)

[Destructor](#9781430262831_Ch09.xhtml#Sec5)

[Case sensitivity](#9781430262831_Ch09.xhtml#Sec6)

[Object comparison](#9781430262831_Ch09.xhtml#Sec7)

![images](images/sq1.jpg)[ Chapter 10: Inheritance](#9781430262831_Ch10.xhtml)

[Overriding members](#9781430262831_Ch10.xhtml#Sec1)

[Final keyword](#9781430262831_Ch10.xhtml#Sec2)

[Instanceof operator](#9781430262831_Ch10.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 11: Access Levels](#9781430262831_Ch11.xhtml)

[Private access](#9781430262831_Ch11.xhtml#Sec1)

[Protected access](#9781430262831_Ch11.xhtml#Sec2)

[Public access](#9781430262831_Ch11.xhtml#Sec3)

[Var keyword](#9781430262831_Ch11.xhtml#Sec4)

[Object access](#9781430262831_Ch11.xhtml#Sec5)

[Access level guideline](#9781430262831_Ch11.xhtml#Sec6)

![images](images/sq1.jpg)[ Chapter 12: Static](#9781430262831_Ch12.xhtml)

[Referencing static members](#9781430262831_Ch12.xhtml#Sec1)

[Static variables](#9781430262831_Ch12.xhtml#Sec2)

[Late static bindings](#9781430262831_Ch12.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 13: Constants](#9781430262831_Ch13.xhtml)

[Const](#9781430262831_Ch13.xhtml#Sec1)

[Define](#9781430262831_Ch13.xhtml#Sec2)

[Const and define](#9781430262831_Ch13.xhtml#Sec3)

[Constant guideline](#9781430262831_Ch13.xhtml#Sec4)

[Magic constants](#9781430262831_Ch13.xhtml#Sec5)

![images](images/sq1.jpg)[ Chapter 14: Interface](#9781430262831_Ch14.xhtml)

[Interface signatures](#9781430262831_Ch14.xhtml#Sec1)

[Interface example](#9781430262831_Ch14.xhtml#Sec2)

[Interface usages](#9781430262831_Ch14.xhtml#Sec3)

[Interface guideline](#9781430262831_Ch14.xhtml#Sec4)

![images](images/sq1.jpg)[ Chapter 15: Abstract](#9781430262831_Ch15.xhtml)

[Abstract methods](#9781430262831_Ch15.xhtml#Sec1)

[Abstract example](#9781430262831_Ch15.xhtml#Sec2)

[Abstract classes and interfaces](#9781430262831_Ch15.xhtml#Sec3)

[Abstract guideline](#9781430262831_Ch15.xhtml#Sec4)

![images](images/sq1.jpg)[ Chapter 16: Traits](#9781430262831_Ch16.xhtml)

[Inheritance and traits](#9781430262831_Ch16.xhtml#Sec1)

[Trait guideline](#9781430262831_Ch16.xhtml#Sec2)

![images](images/sq1.jpg)[ Chapter 17: Importing Files](#9781430262831_Ch17.xhtml)

[Include path](#9781430262831_Ch17.xhtml#Sec1)

[Require](#9781430262831_Ch17.xhtml#Sec2)

[Include_once](#9781430262831_Ch17.xhtml#Sec3)

[Require_once](#9781430262831_Ch17.xhtml#Sec4)

[Return](#9781430262831_Ch17.xhtml#Sec5)

[Auto load](#9781430262831_Ch17.xhtml#Sec6)

![images](images/sq1.jpg)[ Chapter 18: Type Hinting](#9781430262831_Ch18.xhtml)

![images](images/sq1.jpg)[ Chapter 19: Type Conversions](#9781430262831_Ch19.xhtml)

[Explicit casts](#9781430262831_Ch19.xhtml#Sec1)

[Settype](#9781430262831_Ch19.xhtml#Sec2)

[Gettype](#9781430262831_Ch19.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 20: Variable Testing](#9781430262831_Ch20.xhtml)

[Isset](#9781430262831_Ch20.xhtml#Sec1)

[Empty](#9781430262831_Ch20.xhtml#Sec2)

[Is_null](#9781430262831_Ch20.xhtml#Sec3)

[Unset](#9781430262831_Ch20.xhtml#Sec4)

[Determining types](#9781430262831_Ch20.xhtml#Sec5)

[Variable information](#9781430262831_Ch20.xhtml#Sec6)

![images](images/sq1.jpg)[ Chapter 21: Overloading](#9781430262831_Ch21.xhtml)

[Property overloading](#9781430262831_Ch21.xhtml#Sec1)

[Method overloading](#9781430262831_Ch21.xhtml#Sec2)

[Isset and unset overloading](#9781430262831_Ch21.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 22: Magic Methods](#9781430262831_Ch22.xhtml)

[Tostring](#9781430262831_Ch22.xhtml#Sec1)

[Invoke](#9781430262831_Ch22.xhtml#Sec2)

[Object serialization](#9781430262831_Ch22.xhtml#Sec3)

[Sleep](#9781430262831_Ch22.xhtml#Sec4)

[Wakeup](#9781430262831_Ch22.xhtml#Sec5)

[Set state](#9781430262831_Ch22.xhtml#Sec6)

[Object cloning](#9781430262831_Ch22.xhtml#Sec7)

![images](images/sq1.jpg)[ Chapter 23: User Input](#9781430262831_Ch23.xhtml)

[HTML form](#9781430262831_Ch23.xhtml#Sec1)

[Sending with post](#9781430262831_Ch23.xhtml#Sec2)

[Sending with get](#9781430262831_Ch23.xhtml#Sec3)

[Request array](#9781430262831_Ch23.xhtml#Sec4)

[Security concerns](#9781430262831_Ch23.xhtml#Sec5)

[Submitting arrays](#9781430262831_Ch23.xhtml#Sec6)

[File uploading](#9781430262831_Ch23.xhtml#Sec7)

[Superglobals](#9781430262831_Ch23.xhtml#Sec8)

![images](images/sq1.jpg)[ Chapter 24: Cookies](#9781430262831_Ch24.xhtml)

[Creating cookies](#9781430262831_Ch24.xhtml#Sec1)

[Cookie array](#9781430262831_Ch24.xhtml#Sec2)

[Deleting cookies](#9781430262831_Ch24.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 25: Sessions](#9781430262831_Ch25.xhtml)

[Starting a session](#9781430262831_Ch25.xhtml#Sec1)

[Session array](#9781430262831_Ch25.xhtml#Sec2)

[Deleting a session](#9781430262831_Ch25.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 26: Namespaces](#9781430262831_Ch26.xhtml)

[Creating namespaces](#9781430262831_Ch26.xhtml#Sec1)

[Nested namespaces](#9781430262831_Ch26.xhtml#Sec2)

[Alternative syntax](#9781430262831_Ch26.xhtml#Sec3)

[Referencing namespaces](#9781430262831_Ch26.xhtml#Sec4)

[Namespace aliases](#9781430262831_Ch26.xhtml#Sec5)

[Namespace keyword](#9781430262831_Ch26.xhtml#Sec6)

[Namespace guideline](#9781430262831_Ch26.xhtml#Sec7)

![images](images/sq1.jpg)[ Chapter 27: References](#9781430262831_Ch27.xhtml)

[Assign by reference](#9781430262831_Ch27.xhtml#Sec1)

[Pass by reference](#9781430262831_Ch27.xhtml#Sec2)

[Return by reference](#9781430262831_Ch27.xhtml#Sec3)

![images](images/sq1.jpg)[ Chapter 28: Advanced Variables](#9781430262831_Ch28.xhtml)

[Curly syntax](#9781430262831_Ch28.xhtml#Sec1)

[Variable variable names](#9781430262831_Ch28.xhtml#Sec2)

[Variable function names](#9781430262831_Ch28.xhtml#Sec3)

[Variable class names](#9781430262831_Ch28.xhtml#Sec4)

![images](images/sq1.jpg)[ Chapter 29: Error Handling](#9781430262831_Ch29.xhtml)

[Correcting errors](#9781430262831_Ch29.xhtml#Sec1)

[Error levels](#9781430262831_Ch29.xhtml#Sec2)

[Error handling environment](#9781430262831_Ch29.xhtml#Sec3)

[Custom error handlers](#9781430262831_Ch29.xhtml#Sec4)

[Raising errors](#9781430262831_Ch29.xhtml#Sec5)

![images](images/sq1.jpg)[ Chapter 30: Exception Handling](#9781430262831_Ch30.xhtml)

[Throwing exceptions](#9781430262831_Ch30.xhtml#Sec1)

[Try-catch statement](#9781430262831_Ch30.xhtml#Sec2)

[Catch block](#9781430262831_Ch30.xhtml#Sec3)

[Finally block](#9781430262831_Ch30.xhtml#Sec4)

[Re-throwing exceptions](#9781430262831_Ch30.xhtml#Sec5)

[Uncaught exception handler](#9781430262831_Ch30.xhtml#Sec6)

[Errors and exceptions](#9781430262831_Ch30.xhtml#Sec7)

[Index](#9781430262831_Index.xhtml)

About the Author

**Mikael Olsson** is a professional web entrepreneur, programmer, and author. He works for an R&D company in Finland where he specializes in software development. In his spare time he writes books and creates websites that summarize various fields of interest. The books he writes are focused on teaching their subject in the most efficient way possible, by explaining only what is relevant and practical without any unnecessary repetition or theory.

About the Technical Reviewer

**Jamie Rumbelow** Jamie has been writing code professionally for years as a freelancer and for even longer as an amateur. He's the author of three books on the PHP framework CodeIgniter, a well-respected and experienced open source contributor and gives talks and workshops on a variety of technology-related subjects. His work has featured in industry journals such as .net magazine and he's worked on projects for small businesses and global charities. He tweets compulsively about politics and programming @jamierumbelow.

Introduction

PHP is a server-side programming language used for creating dynamic websites and interactive web applications. The acronym PHP originally stood for “Personal Home Page,” but as its functionality grew this was changed to “PHP: Hypertext Preprocessor.” This recursive acronym comes from the fact that it takes PHP code as input and produces HTML as output. This means that users do not need to install any software to be able to view PHP generated web pages. All that is required is that the web server has PHP installed in order to interpret the script.

In contrast with HTML sites, PHP sites are dynamically generated. Instead of the site being made up of a large number of static HTML files, a PHP site may consist of only a handful of template files. The template files describe only the structure of the site using PHP code, while the web content is pulled from a database and the style formatting from a Cascading Style Sheet (CSS). This provides a flexible website that allows for site-wide changes from a single location, providing a site that is easy to design, maintain and update with new content.

When creating websites with PHP a Content Management System (CMS) is generally used. A CMS provides a fully integrated platform for website development consisting of a backend and a frontend. The frontend is what visitors see when they arrive to the site, while the backend is where the site may be configured, updated and managed by an administrator. The backend also allows a web developer to change template files and modify plugins, to more extensively customize the functionality and structure of the site. Examples of free PHP-based CMS solutions include WordPress, Joomla, ModX and Drupal, with WordPress being the most popular and accounting for more than half of the CMS market.

The first version of PHP was created by Rasmus Lerdorf and released in 1995\. Since then PHP has evolved greatly from a simple scripting language to a fully featured web programming language. The official implementation is now released by The PHP Group, with PHP 5.5 being the most recent version as of writing. The language may be used free of charge and is open source, allowing developers to extend it for their own use or contribute to its development.

PHP is by far the most popular server-side programming language in use today. It holds a growing 75% market share when compared with other server-side technologies such as ASP.NET, Java, Ruby and Perl. One of the reasons for the widespread adoption of PHP is its platform independence. It can be installed on all major web servers and operating systems and used together with any major database system. Another strong feature of PHP is its simple-to-use syntax based on C and Perl, which is easy to learn for a newcomer but also offers many advanced features for a professional programmer.

CHAPTER 1

![image](images/frontdot.jpg)

Using PHP

To start developing in PHP, create a plain text file with a .php file extension and open it in the editor of your choice – for example Notepad, JEdit, Dreamweaver, Netbeans or PHPEclipse. This PHP file can include any HTML, as well as PHP scripting code. Begin by first entering the following standard HTML elements into the document.

```
<html>
 <head><title>PHP Test</title></head>
 <body></body>
</html>
```

Embedding PHP

PHP code can be embedded anywhere in a web document in one of four different ways. The standard notation is to delimit the code by “`<?php`” and “`?>`”. This is called a PHP code block, or just a PHP block.

```
<?php ... ?>
```

Within a PHP block the engine is said to be in PHP mode, and outside of the block the engine is in HTML mode. In PHP mode everything will be parsed (executed) by the PHP engine, whereas in HTML mode everything will be sent to the generated web page without any execution.

The second notation for switching to PHP mode is a short version of the first where the “php” part is left out. Although this notation is shorter, the longer one is preferable if the PHP code needs to be portable. This is because support for the short delimiter can be disabled in the php.ini configuration file.[¹](#9781430262831_Ch01.xhtml#Fn1)

```
<? ... ?>
```

A third alternative is to embed the PHP code within a HTML script element with the language attribute set to “php”. This alternative is always available, but seldom used.

```
<script language="php">...</script>
```

One remaining style you may encounter is when the script is embedded between ASP tags. This style is disabled by default, but can be enabled from the PHP configuration file.

```
<% ... %>
```

The last closing tag in a script file may be omitted if the file ends in PHP mode.

```
<?php ... ?>
<?php ...
```

Outputting text

Printing text in PHP is done by either typing `echo` or `print` followed by the output. Each statement must end with a semicolon (`;`) in order to separate it from other statements. The semicolon for the last statement in a PHP block is optional.

```
<?php
  echo  "Hello World";
  print "Hello World"
?>
```

Output can also be generated using the “`<?=`” open delimiter. As of PHP 5.4 this syntax is valid, even if the short PHP delimiter is disabled.

```
<?= "Hello World" ?>
<? echo "Hello World" ?>
```

Keep in mind that text output will only be visible on the web page if it is located within the HTML body element.

```
<html>
 <head><title>PHP Test</title></head>
<body>
 <?php echo "Hello World"; ?>
</body>
</html>
```

Installing a web server

To view PHP code in a browser the code first has to be parsed on a web server with the PHP module installed. An easy way to set up a PHP environment is to download and install a distribution of the popular Apache web server called XAMPP,[²](#9781430262831_Ch01.xhtml#Fn2)   which comes pre-installed with PHP, Perl and MySQL. This will allow you to experiment with PHP on your own computer.

After installing the web server point your browser to “`http://localhost`” to make sure that the server is online. It should display the file index.php, which by default is located under “C:\xampp\htdocs\index.php” on a Windows machine. Htdocs is the folder that the Apache web server looks to for files to serve on your domain.

Hello world

Continuing from before, the simple “Hello World” PHP web document should look like this.

```
<html>
 <head><title>PHP Test</title></head>
 <body>
  <?php echo "Hello World"; ?>
 </body>
</html>
```

To view this PHP file parsed into HTML, save it to the web server's htdocs folder (the server's root directory) with a name such as “mypage.php”. Then point your browser to its path, which is “`http://localhost/mypage.php`” for a local web server.

When a request is made for the PHP web page the script is parsed on the server and sent to the browser as only HTML. If the source code for the website is viewed it will not show any of the server-side code that generated the page, only the HTML output.

Compile and parse

PHP is an interpreted language, not a compiled language. Every time a visitor arrives at a PHP website the PHP engine compiles the code and parses it into HTML which is then sent to the visitor. The main advantage of this is that the code can be changed easily without having to recompile and redeploy the website. The main disadvantage is that compiling the code at run-time requires more server resources.

For a small website a lack of server resources is seldom an issue. The time it takes to compile the PHP script is also miniscule compared with other factors, such as the time required to execute database queries. However, for a large web application with lots of traffic the server load from compiling PHP files is likely to be significant. For such a site the script compilation overhead can be removed by precompiling the PHP code. This can be done for example with eAccelerator,[³](#9781430262831_Ch01.xhtml#Fn3)   which caches PHP scripts in their compiled state.

A website that only serves static content (same to all visitors) has another possibility, which is to cache the fully generated HTML pages. This provides all the maintenance benefits of having a dynamic site, with the speed of a static site. One such caching tool is the W3 Total Cache[⁴](#9781430262831_Ch01.xhtml#Fn4) plugin for the WordPress CMS.

Comments

Comments are used to insert notes into the code and will have no effect on the parsing of the script. PHP has the two standard C++ notations for single-line (`//`) and multi-line (`/* */`) comments. The Perl comment notation (`#`) may also be used to make single-line comments.

```
<?php
  // single-line comment
  #  single-line comment
  /* multi-line
     comment */
?>
```

As in HTML, whitespace characters – such as spaces, tabs and comments – are generally ignored by the PHP engine. This allows you a lot of freedom in how to format your code.

[¹](#9781430262831_Ch01.xhtml#_Fn1)`http://www.php.net/manual/en/configuration.file.php`

[²](#9781430262831_Ch01.xhtml#_Fn2)`http://www.apachefriends.org/en/xampp.html`

[³](#9781430262831_Ch01.xhtml#_Fn3)`http://www.eaccelerator.net`

[⁴](#9781430262831_Ch01.xhtml#_Fn4)`http://wordpress.org/extend/plugins/w3-total-cache`

CHAPTER 2

![image](images/frontdot.jpg)

Variables

Variables are used for storing data, such as numbers or strings, so that they can be used multiple times in a script.

Defining variables

A variable starts with a dollar sign (`$`) followed by an *identifier* , which is the name of the variable. A common naming convention for variables is to have each word initially capitalized, except for the first one.

```
$myVar;
```

A value can be assigned to a variable by using the equals sign, or assignment operator (`=`). The variable then becomes *defined* or *initialized* .

```
$myVar = 10;
```

Once a variable has been defined it can be used by referencing the variable’s name. For example, the value of the variable can be printed to the web page by using `echo` followed by the variable’s name.

```
echo $myVar; // "10"
```

Keep in mind that variable names are case sensitive. Names in PHP can include underscore characters and numbers, but they cannot start with a number. They also cannot contain spaces or special characters, and must not be a reserved keyword.

Data types

PHP is a loosely typed language. This means that the type of data that a variable can store is not specified. Instead, a variable’s data type will change automatically to be able to hold the value it is assigned.

```
$myVar = 1;   // int type
$myVar = 1.5; // float type
```

Furthermore, the value of a variable will be evaluated differently depending on the context in which it is used.

```
// Float type evaluated as string type
echo $myVar; // "1.5"
```

Because of these implicit type conversions, knowing the underlying type of a variable is not always necessary. Nevertheless, it is important to have an understanding of the data types PHP works with in the background. These nine types are listed in the table below.

| Data Type | Category | Description |
| --- | --- | --- |
| int | Scalar | Integer |
| float | Scalar | Floating-point number |
| bool | Scalar | Boolean value |
| string | Scalar | Series of characters |
| array | Composite | Collection of values |
| object | Composite | User-defined data type |
| resource | Special | External resource |
| callable | Special | Function or method |
| null | Special | No value |

Integer type

An integer is a whole number. They can be specified in decimal (base 10), hexadecimal (base 16), octal (base 8) or binary (base 2) notation. Hexadecimal numbers are preceded with a “0x”, octal with “0” and binary numbers with “0b”.

```
$myInt = 1234; // decimal number
$myInt = 0b10; // binary number (2 decimal)
$myInt = 0123; // octal number (83 decimal)
$myInt = 0x1A; // hexadecimal number (26 decimal)
```

Integers in PHP are always signed and can therefore store both positive and negative values. The size of an integer depends on the system word size, so on a 32-bit system the largest storable value is 2^(^32-1). If PHP encounters a larger value it will be interpreted as a float instead.

Floating-point type

The float or floating-point type can store real numbers. These can be assigned using either decimal or exponential notation.

```
$myFloat = 1.234;
$myFloat = 3e2; // 3*10^2 = 300
```

The precision of a float is platform dependent. Commonly the 64-bit IEEE format is used, which can hold approximately 14 decimal digits and a maximum decimal value of 1.8x10^(308).

Bool type

The bool type can store a Boolean value, which is a value that can only be either true or false. These values are specified with the `true` and `false` keywords.

```
$myBool = true;
```

Null type

The case-insensitive constant `null` is used to represent a variable with no value. Such a variable is considered to be of the special null data type.

```
$myNull = null; // variable is set to null
```

Just as with other values, the value null will evaluate differently depending on the context in which the variable is used. If evaluated as a bool it becomes false, as a number it becomes zero (`0`), and as a string it becomes an empty string (`""`).

```
$myInt = $myNull + 0;      // numeric context (0)
$myBool = $myNull == true; // bool context    (false)
echo $myNull;              // string context  ("")
```

Default values

In PHP it is possible to use variables that have not been assigned a value. Such undefined variables will then automatically be created with the value null.

```
echo $myUndefined; // variable is set to null
```

Although this behavior is allowed, it is a good coding practice to define variables before they are used, even if the variables are just set to null. As a reminder for this, PHP will issue an error notice when undefined variables are used. Depending on the PHP error reporting settings, this message may or may not be displayed.

```
Notice: Undefined variable: myUndefined in
C:\xampp\htdocs\mypage.php on line 10
```

CHAPTER 3

![image](images/frontdot.jpg)

Operators

Operators are used to operate on values. They can be grouped into five types: arithmetic, assignment, comparison, logical and bitwise operators.

Arithmetic operators

The arithmetic operators include the four basic arithmetic operations, as well as the modulus operator (`%`) which is used to obtain the division remainder.

```
$x = 4 + 2; // 6 // addition
$x = 4 - 2; // 2 // subtraction
$x = 4 * 2; // 8 // multiplication
$x = 4 / 2; // 2 // division
$x = 4 % 2; // 0 // modulus (division remainder)
```

Assignment operators

The second group is the assignment operators. Most importantly, the assignment operator (`=`) itself, which assigns a value to a variable.

Combined assignment operators

A common use of the assignment and arithmetic operators is to operate on a variable and then to save the result back into that same variable. These operations can be shortened with the combined assignment operators.

```
$x = 0;
$x += 5; // $x = $x+5;
$x -= 5; // $x = $x-5;
$x *= 5; // $x = $x*5;
$x /= 5; // $x = $x/5;
$x %= 5; // $x = $x%5;
```

Increment and decrement operators

Another common operation is to increment or decrement a variable by one. This can be simplified with the increment (`++`) and decrement (`−−`) operators.

```
$x++; // $x += 1;
$x−−; // $x -= 1;
```

Both of these operators can be used either before or after a variable.

```
$x++; // post-increment
$x−−; // post-decrement
++$x; // pre-increment
−−$x; // pre-decrement
```

The result on the variable is the same whichever is used. The difference is that the post-operator returns the original value before it changes the variable, while the pre-operator changes the variable first and then returns the value.

```
$x = 5; $y = $x++; // $x=6, $y=5
$x = 5; $y = ++$x; // $x=6, $y=6
```

Comparison operators

The comparison operators compare two values and return either true or false. They are mainly used to specify conditions, which are expressions that evaluate to either true or false.

```
$x = (2 == 3);  // false // equal to
$x = (2 != 3);  // true  // not equal to
$x = (2 <> 3);  // true  // not equal to (alternative)
$x = (2 === 3); // false // identical
$x = (2 !== 3); // true  // not identical
$x = (2 > 3);   // false // greater than
$x = (2 < 3);   // true  // less than
$x = (2 >= 3);  // false // greater than or equal to
$x = (2 <= 3);  // true  // less than or equal to
```

The identical operator (`===`) is used for comparing both the value and data type of the operands. It returns true if both operands have the same value and are of the same type. Likewise, the not identical operator (`!==`) returns true if the operands do not have the same value or are not of the same type. Put another way, the equality operators will perform type conversions, whereas the identical operators will not.

```
$x = (1 ==  "1"); // true  (same value)
$x = (1 === "1"); // false (different types)
```

Logical operators

The logical operators are often used together with the comparison operators. Logical and (`&&`) evaluates to true if both the left and right side are true, and logical or (`||`) evaluates to true if either the left or right side is true. The logical not (`!`) operator is used for inverting a Boolean result. Note that for both “logical and” and “logical or” the right side of the operator will not be evaluated if the result is already determined by the left side.

```
$x = (true && false); // false // logical and
$x = (true || false); // true  // logical or
$x = !(true);         // false // logical not
```

Bitwise operators

The bitwise operators can manipulate binary digits of numbers. For example, the xor operator (`^`) turn on the bits that are set on one side of the operator, but not on both sides.

```
$x = 5 & 4;  // 101 & 100 = 100 (4) // and
$x = 5 | 4;  // 101 | 100 = 101 (5) // or
$x = 5 ^ 4;  // 101 ^ 100 = 001 (1) // xor (exclusive or)
$x = 4 << 1; // 100 << 1  =1000 (8) // left shift
$x = 4 >> 1; // 100 >> 1  =  10 (2) // right shift
$x = ∼4;     // ∼00000100 = 11111011 (-5) // invert
```

These bitwise operators have shorthand assignment operators, just like the arithmetic operators.

```
$x=5; $x &= 4;  // 101 & 100 = 100 (4) // and
$x=5; $x |= 4;  // 101 | 100 = 101 (5) // or
$x=5; $x ^= 4;  // 101 ^ 100 = 001 (1) // xor
$x=5; $x <<= 1; // 101 << 1  =1010 (10)// left shift
$x=5; $x >>= 1; // 101 >> 1  =  10 (2) // right shift
```

Operator precedence

In PHP, expressions are normally evaluated from left to right. However, when an expression contains multiple operators, the precedence of those operators decides the order in which they are evaluated.

![image](images/Table3-1.jpg)

For example, logical and (`&&`) binds weaker than relational operators, which in turn bind weaker than arithmetic operators.

```
$x = 2+3 > 1*4 && 5/5 == 1; // true
```

To make things clearer, parentheses can be used to specify which part of the expression will be evaluated first. Parentheses have the highest precedence of all operators.

```
$x = ((2+3) > (1*4)) && ((5/5) == 1); // true
```

Additional logical operators

In the precedence table make special note of the last three operators: `and`, `or` and `xor`. The `and` and `or` operators work in the same way as the logical `&&` and `||` operators. The only difference is their lower level of precedence.

```
// Same as: $a = (true && false);
$a = true && false; // $a is false

// Same as: ($a = true) and false;
$a = true and false; // $a is true
```

The `xor` operator is a Boolean version of the bitwise `^` operator. It evaluates to true if only one of the operands are true.

```
$a = (true xor true); // false
```

CHAPTER 4

![image](images/frontdot.jpg)

String

A string is a series of characters that can be stored in a variable. In PHP, strings are typically delimited by single quotes.

```
$a = 'Hello';
```

String concatenation

PHP has two string operators. The dot symbol is known as the concatenation operator (`.`) and combines two strings into one. It also has an accompanying assignment operator (`.=`), which appends the right-hand string to the left-hand string variable.

```
$b = $a . ' World'; // Hello World
$a .= ' World';     // Hello World
```

Delimiting strings

PHP strings can be delimited in four different ways. There are two single-line notations: double-quote (`" "`) and single-quote (`' '`). The difference between them is that variables are not parsed in single-quoted strings whereas they are parsed in double-quoted strings.

```
$c = 'World';
echo "Hello $c"; // "Hello World"
echo 'Hello $c'; // "Hello $c"
```

Single-quoted strings tend to be preferred unless parsing is desired, mainly because string parsing has a very small performance overhead. However, double-quoted strings are considered easier to read, which makes the choice more a matter of preference.

In addition to single-quoted and double-quoted strings, there are two multi-line notations: heredoc and nowdoc. These notations are mainly used to include larger blocks of text.

Heredoc strings

The heredoc syntax consists of the `<<<` operator followed by an identifier and a new line. The string is then included followed by a new line containing the identifier in order to close the string. Variables are parsed inside of a heredoc string, just as with double-quoted strings.

```
$s = <<<LABEL
Heredoc (with parsing)
LABEL;
```

Nowdoc strings

The syntax for the nowdoc string is the same as for the heredoc string, except that the initial identifier is enclosed in single-quotes. Variables will not be parsed inside a nowdoc string.

```
$s = <<<'LABEL'
Nowdoc (without parsing)
LABEL;
```

Escape characters

Escape characters are used to write special characters, such as backslashes or double-quotes. A table of the escape characters available in PHP can be seen below.

![image](images/Table4-1.jpg)

For example, line breaks are represented with the escape character “\n” in text.

```
$s = "Hello\nWorld";
```

Note that this character is different from the `<br>` HTML tag, which creates line breaks on web pages.

```
echo "Hello<br>World";
```

When using the single-quote or nowdoc delimiter the only escape characters that work are the backslash (`\\`) and single-quote (`\'`) characters.

```
$s = 'It\'s'; // "It's"
```

Escaping the backslash is only necessary before a single-quote or at the end of the string.

Character reference

Characters within strings can be referenced by specifying the index of the desired character in square brackets after the string variable, starting with zero. This can be used both for accessing and modifying single characters.

```
$s = 'Hello';
$s[0] = 'J';
echo $s; // "Jello"
```

The `strlen` function retrieves the length of the string argument. This can for example be used to change the last character of a string.

```
$s[strlen($s)-1] = 'y';
echo $s; // "Jelly"
```

String compare

The way to compare two strings is simply by using the equal to operator. This will not compare the memory addresses, as in some other languages.

```
$a = 'test';
$b = 'test';
$c = ($a == $b); // true
```

CHAPTER 5

![image](images/frontdot.jpg)

Arrays

An array is used to store a collection of values in a single variable. Arrays in PHP consist of key-value pairs. The key can either be an integer (numeric array), a string (associative array) or a combination of both (mixed array). The value can be any data type.

Numeric arrays

Numeric arrays store each element in the array with a numeric index. An array is created using the `array` constructor. This constructor takes a list of values which are assigned to elements of the array.

```
$a = array(1,2,3);
```

As of PHP 5.4 a shorter syntax is available, where the array constructor is replaced with square brackets.

```
$a = [1,2,3];
```

Once the array is created, its elements can be referenced by placing the index of the desired element in square brackets. Note that the index begins with zero.

```
$a[0] = 1;
$a[1] = 2;
$a[2] = 3;
```

The number of elements in the array is handled automatically. Adding a new element to the array is as easy as assigning a value to it.

```
$a[3] = 4;
```

The index can also be left out to add the value to the end of the array. This syntax will also construct a new array if the variable does not already contain one.

```
$a[] = 5; // $a[4]
```

To retrieve the value of an element in the array the index of that element is specified inside the square brackets.

```
echo "$a[0] $a[1] $a[2] $a[3]"; // "1 2 3 4"
```

Associative arrays

In associative arrays the key is a string instead of a numeric index, which gives the element a name instead of a number. When creating the array the double arrow operator (`=>`) is used to tell which key refers to what value.

```
$b = array('one' => 'a', 'two' => 'b', 'three' => 'c');
```

Elements in associative arrays are referenced using the element names. They cannot be referenced with a numeric index.

```
$b['one']   = 'a';
$b['two']   = 'b';
$b['three'] = 'c';

echo $b['one'] . $b['two'] . $b['three']; // "abc"
```

The double arrow operator can also be used with numeric arrays to decide in which element a value will be placed.

```
$c = array(0 => 0, 1 => 1, 2 => 2);
```

Not all keys need to be specified. If a key is left unspecified the value will be assigned to the element following the largest previously used integer key.

```
$e = array(5 => 5, 6);

```

Mixed arrays

PHP makes no distinction between associative and numerical arrays, and so elements of each can be combined in the same array.

```
$d = array(0 => 1, 'foo' => 'bar');
```

Just be sure to access the elements with the same keys.

```
echo $d[0] . $d['foo']; // "1bar"
```

Multi-dimensional arrays

A multi-dimensional array is an array that contains other arrays. For example, a two-dimensional array can be constructed in the following way.

```
$a = array(array('00', '01'), array('10', '11'));
```

Once created the elements can be modified using two sets of square brackets.

```
$a[0][0] = '00';
$a[0][1] = '01';
$a[1][0] = '10';
$a[1][1] = '11';
```

They are also accessed in the same way.

```
echo $a[0][0] . $a[0][1] . $a[1][0] . $a[1][1];
```

The key can be given a string name to make it into a multi-dimensional associative array, also called a hash table.

```
$b = array('one' => array('00', '01'));
echo $b['one'][0] . $b['one'][1]; // "0001"
```

Multi-dimensional arrays can have more than two dimensions by adding additional sets of square brackets.

```
$c[][][][] = "0000"; // four dimensions
```

CHAPTER 6

![image](images/frontdot.jpg)

Conditionals

Conditional statements are used to execute different code blocks based on different conditions.

If statement

The if statement will only execute if the condition inside the parentheses is evaluated to true. The condition can include any of the comparison and logical operators. In PHP, the condition does not have to be a Boolean expression.

```
if ($x == 1) {
  echo "x is 1";
}
```

To test for other conditions, the if statement can be extended with any number of elseif clauses. Each additional condition will only be tested if all previous conditions are false.

```
elseif ($x == 2) {
  echo "x is 2";
}
```

The if statement can have one else clause at the end, which will execute if all previous conditions are false.

```
else {
  echo "x is something else";
}
```

As for the curly brackets, they can be left out if only a single statement needs to be executed conditionally.

```
if ($x == 1)
  echo "x is 1";
elseif ($x == 2)
  echo "x is 2";
else
  echo "x is something else";
```

Switch statement

The switch statement checks for equality between an integer, float or string and a series of case labels. It then passes execution to the matching case. The statement can contain any number of case clauses and may end with a default label for handling all other cases.

```
switch ($x)
{
  case 1: echo "x is 1"; break;
  case 2: echo "x is 2";  break;
  default: echo "x is something else";
}
```

Note that the statements after each case label are not surrounded by curly brackets. Instead, the statements end with the `break` keyword to break out of the switch. Without the break the execution will fall through to the next case. This can be useful if several cases need to be evaluated in the same way.

Alternative syntax

PHP has an alternative syntax for the conditional statements. In this syntax, the if statement's opening bracket is replaced with a colon, the closing bracket is removed, and the last closing bracket is replaced with the `endif` keyword.

```
if ($x == 1):     echo "x is 1";
elseif ($x == 2): echo "x is 2";
else:             echo "x is something else";
endif;
```

Similarly, the switch statement also has an alternative syntax, which instead uses the `endswitch` keyword to terminate the statement.

```
switch ($x):
  case 1:  echo "x is 1"; break;
  case 2:  echo "x is 2";  break;
  default: echo "x is something else";
endswitch;
```

The alternative syntax is often preferable for longer conditional statements since it then becomes easier to see where those statements end.

Mixed modes

It is possible to switch back to HTML mode in the middle of a code block. This provides another way of writing conditional statements that output text to the web page.

```
<?php if ($x == 1) { ?>
  This will show if $x is 1.
<?php } else { ?>
  Otherwise this will show.
<?php } ?>
```

The alternative syntax may also be used in this way to make the code clearer.

```
<?php if ($x == 1): ?>
  This will show if $x is 1.
<?php else: ?>
  Otherwise this will show.
<?php endif; ?>
```

When outputting HTML and text, particularly larger blocks, this coding style is generally preferred as it makes it easier to distinguish between PHP code and the HTML content that appears on the web page.

Ternary operator

In addition to the if and switch statements there is the ternary operator (`?:`). This operator can replace a single if/else clause. The operator takes three expressions. If the first one is evaluated to true then the second expression is returned, and if it is false, the third one is returned.

```
// Ternary operator expression
$y = ($x == 1) ? 1 : 2;
```

In PHP, this operator cannot just be used as an expression, but also as a statement.

```
// Ternary operator statement
($x == 1) ? $y = 1 : $y = 2;
```

The programming term *expression* refers to code that evaluates to a value, whereas a *statement* is a code segment that ends with a semicolon or a closing curly bracket.

CHAPTER 7

![image](images/frontdot.jpg)

Loops

There are four looping structures in PHP. These are used to execute a specific code block multiple times. Just as with the conditional if statement, the curly brackets for the loops can be left out if there is only one statement in the code block.

While loop

The while loop runs through the code block only if its condition is true, and will continue looping for as long as the condition remains true. Note that the condition is only checked at the beginning of each iteration (loop).

```
$i = 0;
while ($i < 10) { echo $i++; } // 0-9
```

Do-while loop

The do-while loop works in the same way as the while loop, except that it checks the condition after the code block. It will therefore always run through the code block at least once. Bear in mind that this loop ends with a semicolon.

```
$i = 0;
do { echo $i++; } while ($i < 10); // 0-9
```

For loop

The for loop is used to go through a code block a specific number of times. It uses three parameters. The first parameter initializes a counter and is always executed once, before the loop. The second parameter holds the condition for the loop and is checked before each iteration. The third parameter contains the increment of the counter and is executed at the end of each iteration.

```
for ($i = 0; $i < 10; $i++) { echo $i; } // 0-9
```

The for loop has several variations since either one of the parameters can be left out. For example, if the first and third parameters are left out it behaves in the same way as the while loop.

```
for (;$i < 10;) { echo $i++; }
```

The first and third parameters can also be split into several statements using the comma operator (`,`).

```
for ($i = 0, $x = 9; $i < 10; $i++, $x--) {
  echo $x; // 9-0
}
```

The `sizeof` function retrieves the number of elements in an array. Together with the for loop it can be used to iterate through a numeric array.

```
$a = array(1,2,3);

for($i = 0; $i < sizeof($a); $i++) {
  echo $a[$i]; // "123"
}
```

If there is no need to keep track of iterations the foreach loop provides a cleaner syntax. This loop is also necessary for traversing associative arrays.

Foreach loop

The foreach loop provides an easy way to iterate through arrays. At each iteration the next element in the array is assigned to the specified variable (the iterator) and the loop continues to execute until it has gone through the entire array.

```
$a = array (1,2,3);

foreach ($a as $v) {
  echo $v; // "123"
}
```

There is an extension of the foreach loop to also obtain the key’s name or index, by adding a key variable followed by the double arrow operator (`=>`) before the iterator.

```
$a = array ('one' => 1, 'two' => 2, 'three' => 3);

foreach ($a as $k => $v) {
  echo "$k => $v <br>";
}
```

Alternative syntax

As with conditional statements, the brackets in the loops can be rewritten into the alternative syntax with a colon and one of the `endwhile`, `endfor` or `endforeach` keywords.

```
while ($i < 10): echo $i++; endwhile;

for ($i = 0; $i < 10; $i++): echo $i; endfor;

foreach ($a as $v): echo $v; endforeach;
```

The main benefit of this is improved readability, especially for longer loops.

Break

There are two special keywords that can be used inside loops – `break` and `continue`. The `break` keyword ends the execution of a loop structure.

```
for (;;) { break; } // end for
```

It can be given a numeric argument which specifies how many nested looping structures to break out of.

```
$i = 0;
while ($i++ < 10)
{
  for (;;) { break 2; } // end for and while
}
```

Continue

The `continue` keyword can be used within any looping statement to skip the rest of the current loop and continue at the beginning of the next iteration.

```
while ($i++ < 10) { continue; } // start next iteration
```

This keyword can accept an argument for how many enclosing loops it should skip to the end of.

```
$i = 0;
while ($i++ < 10)
{
  for (;;) { continue 2; } // start next while iteration
}
```

In contrast to many other languages, the continue statement also applies to switches, where it behaves the same as `break`. Therefore, to skip an iteration from inside a switch `continue 2` needs to be used.

```
$i = 0;
while ($i++ < 10)
{
  switch ($i)
  {
    case 1: continue 2; // start next while iteration
  }
}
```

Goto

A third jump statement introduced in PHP 5.3 is `goto`, which performs a jump to a specified label. A label is a name followed by a colon (`:`).

```
goto myLabel; // jump to label
myLabel:      // label declaration
```

The target label must be within the same script file and scope. Therefore, goto cannot be used to jump into looping structures, only out of them.

```
loop:
while (!$finished)
{
  // ...
  if ($try_again) goto loop; // restart loop
}
```

In general, the goto statement is often best avoided since it tends to make the flow of execution difficult to follow.

CHAPTER 8

![image](images/frontdot.jpg)

Functions

A function is a reusable code block that will only execute when called.

Defining functions

To create a function the `function` keyword is used followed by a name, a set of parentheses and a code block. The naming convention for functions is the same as for variables – to use a descriptive name with each word initially capitalized, except for the first one.

```
function myFunc()
{
  echo "Hello World";
}
```

A function code block can contain any valid PHP code, including other function definitions.

Calling functions

Once defined a function can be called (invoked) from anywhere on the page, by typing its name followed by a set of parenthesis. Function names are case-insensitive, but it is good practice to use the same casing as they have in their definition.

```
myFunc(); // "Hello World"
```

A function can be called even if the function definition appears further down in the script file.

```
foo(); // ok
function foo() {}
```

An exception to this is where the function is only defined when a certain condition is met. That conditional code must then be executed prior to calling the function.

```
bar(); // error
if (true) { function bar() {} }
bar(); // ok
```

Function parameters

The parentheses that follow the function name are used to pass arguments to the function. To do this the corresponding parameters must first be specified in the function definition in the form of a comma separated list of variables. The parameters can then be used in the function.

```
function myFunc($x,$y)
{
  echo $x . $y;
}
```

With the parameters specified the function can be called with the same number of arguments.

```
myFunc('Hello', ' World'); // "Hello World"
```

To be precise, *parameters* appear in function definitions, while *arguments* appear in function calls. However, the two terms are sometimes used interchangeably.

Default parameters

It is possible to specify default values for parameters by assigning them a value inside the parameter list. Then, if that argument is unspecified when the function is called the default value will be used instead. For this to work as expected it is important that the parameters with default values are declared to the right of those without default values.

```
function myFunc($x, $y = " Earth")
{
  echo $x . $y;
}

myFunc("Hello"); // "Hello Earth"
```

Variable parameter list

A function cannot be called with fewer arguments than is specified in its declaration, but it may be called with more arguments. This allows for the passing of a variable number of arguments, which can then be accessed using a couple of built-in functions. For getting one argument at a time there is the `func_get_arg` function. This function takes a single argument, which is the parameter to be returned starting with zero.

```
function myFunc()
{
  $x = func_get_arg(0);    // get specified argument
  $y = func_get_arg(1);
  $z = func_get_arg(2);
  echo $x . $y . $z;
}

myFunc('Fee', 'Fi', 'Fo'); // "FeeFiFo"
```

There are two more functions related to the argument list. The `func_num_args` function gets the number of arguments passed and `func_get_args` returns an array containing all of the arguments.

```
function myFunc()
{
  $num  = func_num_args();
  $args = func_get_args();
  for ($i = 0; $i < $num; $i++)
    echo $args[$i] . ' ';
}

myFunc('Fee', 'Fi', 'Fo'); // "Fee Fi Fo"
```

Return statement

Return is a jump statement that causes the function to end its execution and return to the location where it was called.

```
function myFunc()
{
  return; // exit function
}
```

It can optionally be given a value to return, in which case it will make the function call evaluate to that value.

```
function myFunc()
{
  return 'Hello'; // exit function and return value
}

echo myFunc();    // "Hello"
```

A function without a return value will be evaluated as null.

Scope and lifetime

Normally, a PHP variable’s scope starts where it is declared and lasts until the end of the page. However, a local function scope is introduced within functions. Any variable used inside a function is by default limited to this local scope. Once the scope of the function ends, the local variable is destroyed.

```
$x = 'Hello';    // global variable

function myFunc()
{
  $y = ' World'; // local variable
}
```

In PHP, trying to access a global variable from a function will not work and will instead create a new local variable. In order to make a global variable accessible the scope of that variable must be extended to the function by declaring it with the `global` keyword.

```
$x = 'Hello';     // global $x

function myFunc()
{
  global $x;      // use global $x in this function
  $x .= ' World'; // change global $x
}

myFunc();
echo $x;          // "Hello World"
```

An alternative way to access variables from the global scope is by using the predefined $GLOBALS array. Note that the variable is specified as a string without the dollar sign.

```
function myFunc()
{
  $GLOBALS['x'] .= ' World'; // change global $x
}
```

In contrast to many other languages, control structures – such as loop and conditional statements – do not have their own variable scope. A variable defined in such a code block will therefore not be destroyed when the code block ends.

```
if(true)
{
  $x = 10; // global $x
}

echo $x;   // "10"
```

In addition to global and local variables PHP has property variables, which will be looked at in the next chapter.

Anonymous functions

PHP 5.3 introduced anonymous functions, which allow functions to be passed as arguments and assigned to variables. An anonymous function is defined like a regular function, except that it has no specified name. The function can be assigned to a variable using the normal assignment syntax, including the semicolon.

```
$say = function($name)
{
  echo "Hello " . $name;
};

$say("World"); // "Hello World"
```

An anonymous function can inherit variables from the scope where it is defined. Such inherited variables are specified with a `use` clause in the function header.

```
$x = 1;
$y = 2;

$callback = function($z) use ($x, $y)
{
  return $x + $y + $z;
};

echo $callback(3); // "6"
```

Function overloading

Unlike many other languages, PHP does not allow a function to be defined multiple times with different parameters. This feature, called function overloading, has the benefit of allowing a function to handle a variety of parameters transparently to the user.

Although PHP does not have function overloading, it is possible to achieve a similar behavior. Since PHP does not have strong typing, a function parameter already accepts any data type. This loose typing – along with default parameter values and variable parameter lists – makes it possible to duplicate the effect of function overloading.

```
// Prints a variable number of parameters
function myprint()
{
  $a = func_get_args();
  foreach ($a as $v) { echo $v; }
}

myprint(1,2,3); // "123"
```

Built-in functions

PHP comes with a large number of built-in functions that are always available, such as string and array handling functions. Other functions depend on what extensions PHP is compiled with, for example the MySQL extension for communicating with MySQL databases. For a complete reference of the built-in PHP functions see the PHP Function Reference.[¹](#9781430262831_Ch08.xhtml#Fn1)

[¹](#9781430262831_Ch08.xhtml#_Fn1)`http://www.php.net/manual/en/funcref.php`

CHAPTER 9

![image](images/frontdot.jpg)

Class

A class is a template used to create objects. To define one the `class` keyword is used followed by a name and a code block. The naming convention for classes is mixed case, meaning that each word should be initially capitalized.

```
class MyRectangle {}
```

The class body can contain properties and methods. Properties are variables that hold the state of the object, while methods are functions which define what the object can do. Properties are also known as fields or attributes in other languages. In PHP, they must have an explicit access level specified. The `public` access level is used below, which gives unrestricted access to the property.

```
class MyRectangle
{
  public $x, $y;
  function newArea($a, $b) { return $a * $b; }
}
```

To access members from inside the class the `$this` pseudo variable is used along with the single arrow operator (`->`). The `$this` variable is a reference to the current instance of the class and can only be used within an object context. Without it `$x` and `$y` would just be seen as local variables.

```
class MyRectangle
{
  public $x, $y;

  function newArea($a, $b)
  {
    return $a * $b;
  }

  function getArea()
  {
    return $this->newArea($this->x, $this->y);
  }
}
```

Instantiating an object

To use a class’s members from outside the enclosing class, an object of the class must first be created. This is done using the `new` keyword, which will create a new object or instance.

```
$r = new MyRectangle(); // object instantiated
```

The object will contain its own set of properties, which can hold values that are different to those of other instances of the class. As with functions, objects of a class may be created even if the class definition appears further down in the script file.

```
$r = new MyDummy(); // ok
class MyDummy {};
```

Accessing object members

In order to access members that belong to an object the single arrow operator (`->`) is needed. This can be used to call methods or to assign values to properties.

```
$r->getArea(10, 2); // 20
$r->x = 5; $r->y = 10;
```

Another way to initialize properties would be to use initial property values.

Initial property values

If a property needs to have an initial value a clean way is to assign the property at the same time as it is declared. This initial value will then be set when the object is created. Assignments of this kind must be a constant expression and cannot, for example, be a variable or a mathematical expression.

```
class MyRectangle
{
  public $x = 5, $y = 10;
}
```

Constructor

A class can have a constructor, which is a special method used to initialize (construct) the object. This method provides a way to initialize properties which is not only limited to constant expressions. In PHP, the constructor starts with two underscores followed by the word “construct”.

```
class MyRectangle
{
  public $x, $y;

  function __construct()
  {
    $this->x = 5;
    $this->y = 10;
    echo "Constructed";
  }
}
```

When a new instance of this class is created the constructor will be called, which in this example sets the properties to the specified values. Note that any initial property values will be set before the constructor is run.

```
$r = new MyRectangle(); // "Constructed"
```

As this constructor takes no arguments the parentheses may optionally be left out.

```
$r = new MyRectangle; // "Constructed"
```

Just as any other method the constructor can have a parameter list. This can be used to make the properties’ initial values dependent on the parameters passed when the object is created.

```
class MyRectangle
{
  public $x, $y;

  function __construct($x, $y)
  {
    $this->x = $x;
    $this->y = $y;
  }
}

$r = new MyRectangle(5,10);
```

Destructor

In addition to the constructor, classes can also have a destructor. This special method starts with two underscores followed by the word “destruct”. It will be called as soon as there are no more references to the object, before the object is destroyed by the PHP garbage collector.

```
class MyRectangle
{
  // ...
  function __destruct() { echo "Destructed"; }
}
```

To test the destructor the `unset` function can be used to manually remove all references to the object.

```
unset($r); // "Destructed"
```

Bear in mind that the object model has been completely rewritten in PHP 5\. Therefore, many features of classes, such as destructors, will not work in earlier versions of the language.

Case sensitivity

Whereas variable names are case-sensitive, class names in PHP are case-insensitive – just as function names, keywords and built-in constructs such as `echo`. This means that a class named “MyClass” can also be referenced as “myclass” or “MYCLASS”.

```
class MyClass {}
$o1 = new myclass(); // ok
$o2 = new MYCLASS(); // ok
```

Object comparison

When using the equal to operator (`==`) on objects these objects are considered equal if the objects are instances of the same class and their properties have the same values and types. In contrast, the identical operator (`===`) returns `true` only if the variables refer to the same instance of the same class.

```
class Flag {
  public $flag = true;
}
$a = new Flag();
$b = new Flag();

$c = ($a == $b);  // true (same values)
$d = ($a === $b); // false (different instances)
```

CHAPTER 10

![image](images/frontdot.jpg)

Inheritance

Inheritance allows a class to acquire the members of another class. In the example below the class Square inherits from Rectangle, specified by the `extends` keyword. Rectangle then becomes the parent class of Square, which in turn becomes a child class of Rectangle. In addition to its own members, Square gains all accessible (non-private) members in Rectangle, including any constructor.

```
// Parent class (base class)
class Rectangle
{
  public $x, $y;
  function __construct($a, $b)
  {
    $this->x = $a;
    $this->y = $b;
  }
}

// Child class (derived class)
class Square extends Rectangle {}
```

When creating an instance of Square two arguments must now be specified, because Square has inherited Rectangle’s constructor.

```
$s = new Square(5,10);
```

The properties inherited from Rectangle can also be accessed from the Square object.

```
$s->x = 5; $s->y = 10;
```

A class in PHP may only inherit from one parent class and the parent must be defined before the child class in the script file.

Overriding members

A member in a child class can redefine a member in its parent class to give it a new implementation. To override an inherited member it just needs to be redeclared with the same name. As shown below, the Square constructor overrides the constructor in Rectangle.

```
class Square extends Rectangle
{
  function __construct($a)
  {
    $this->x = $a;
    $this->y = $a;
  }
}
```

With this new constructor only a single argument is used to create the Square.

```
$s = new Square(5);
```

Because the inherited constructor of Rectangle is overridden, Rectangle’s constructor will no longer be called when the Square object is created. It is up to the developer to call the parent constructor if necessary. This is done by prepending the call with the `parent` keyword and a double colon. The double colon is known as the scope resolution operator (`::`).

```
class Square extends Rectangle
{
  function __construct($a)
  {
    parent::__construct($a,$a);
  }
}
```

The `parent` keyword is an alias for the parent’s class name, which may alternatively be used. In PHP, it is possible to access overridden members that are any number of levels deep in the inheritance hierarchy using this notation.

```
class Square extends Rectangle
{
  function __construct($a)
  {
    Rectangle::__construct($a,$a);
  }
}
```

Like constructors, the parent destructor will not be called implicitly if it is overridden. It too would have to be explicitly called with `parent::__destruct()` from the child destructor.

Final keyword

To stop a child class from overriding a method it can be defined as final. A class itself can also be defined as final to prevent any class from extending it.

```
final class NotExtendable
{
  final function notOverridable() {}
}
```

Instanceof operator

As a safety precaution, you can test to see whether an object can be cast to a specific class by using the `instanceof` operator. This operator returns `true` if the left side object can be cast into the right side type without causing an error. This is `true` when the object is an instance of, or inherits from, the right side class.

```
$s = new Square(5);
$s instanceof Square;    // true
$s instanceof Rectangle; // true
```

CHAPTER 11

![image](images/frontdot.jpg)

Access Levels

Every class member has an accessibility level that determines where the member will be visible. There are three of them available in PHP: public, protected and private.

```
class MyClass
{
  public    $myPublic;    // unrestricted access
  protected $myProtected; // enclosing or child class
  private   $myPrivate;   // enclosing class only
}
```

Private access

All members regardless of access level are accessible in the class in which they are declared, the enclosing class. This is the only place where a private member can be accessed.

```
class MyClass
{
  public    $myPublic    = 'public';
  protected $myProtected = 'protected';
  private   $myPrivate   = 'private';

  function test()
  {
    echo $this->myPublic;    // allowed
    echo $this->myProtected; // allowed
    echo $this->myPrivate;   // allowed
  }
}
```

Unlike properties, methods do not have to have an explicit access level specified. They will default to public access unless set to another level.

Protected access

A protected member can be accessed from inside of child or parent classes as well as from within the enclosing class.

```
class MyChild extends MyClass
{
  function test()
  {

    echo $this->myPublic;    // allowed
    echo $this->myProtected; // allowed
    echo $this->myPrivate;   // inaccessible
  }
}
```

Public access

Public members have unrestricted access. In addition to anywhere a protected member can be accessed, a public member can also be reached through an object variable.

```
$m = new MyClass();
echo $m->myPublic;    // allowed
echo $m->myProtected; // inaccessible
echo $m->myPrivate;   // inaccessible
```

Var keyword

Before PHP 5 the `var` keyword was used to declare properties. In order to maintain backward compatibility this keyword is still usable and gives public access just as the `public` modifier.

```
class MyVars
{
  var $x, $y; // deprecated property declaration
}
```

Object access

In PHP, objects of the same class have access to each other’s private and protected members. This behavior is different from many other programming languages where such access is not allowed.

```
class MyClass
{
  private $myPrivate;

  function setPrivate($obj, $val) {
    $obj->myPrivate = $val; // set private property
  }
}
$a = new MyClass();
$b = new MyClass();
$a->setPrivate($b, 10);
```

Access level guideline

As a guideline, when choosing an access level it is generally best to use the most restrictive level possible. This is because the more places a member can be accessed the more places it can be accessed incorrectly, which makes the code harder to debug. Using restrictive access levels will also make it easier to modify the class without breaking the code for any other developers using that class.

CHAPTER 12

![image](images/frontdot.jpg)

Static

The `static` keyword can be used to declare properties and methods that can be accessed without having to create an instance of the class. Static (class) members only exist in one copy, which belongs to the class itself, whereas instance (non-static) members are created as new copies for each new object.

```
class MyCircle
{
  // Instance members (one per object)
  public $r = 10;
  function getArea() {}

  // Static/class members (only one copy)
  static $pi = 3.14;
  static function newArea($a) {}
}
```

Static methods cannot use instance members since these methods are not part of an instance. They can however use other static members.

Referencing static members

Unlike instance members, static members are not accessed using the single arrow operator (`->`). Instead, in order to reference static members inside a class the member must be prefixed with the `self` keyword followed by the scope resolution operator (`::`). The `self` keyword is an alias for the class name, so alternatively the actual name of the class can be used.

```
static function newArea($a)
{
  return self::$pi * $a * $a;     // ok
  return MyCircle::$pi * $a * $a; // alternative
}
```

This same syntax is used to access static members from an instance method. Note that in contrast to static methods, instance methods can use both static and instance members.

```
function getArea()
{
  return self::newArea($this->$r);
}
```

To access static members from outside the class the name of the class needs to be used followed by the scope resolution operator (`::`).

```
class MyCircle
{
  static $pi = 3.14;

  static function newArea($a)
  {
    return self::$pi * $a * $a;
  }
}

echo MyCircle::$pi;         // "3.14"
echo MyCircle::newArea(10); // "314"
```

The advantage of static members can be seen here in that they can be used without having to create an instance of the class. Methods should therefore be declared static if they perform a generic function independently of instance variables. Likewise, properties should be declared static if there is only need for a single instance of the variable.

Static variables

Local variables can be declared static to make the function remember its value. Such a static variable only exists in the local function’s scope, but it does not lose its value when the function ends. This can for example be used to count the number of times a function is called.

```
function add()
{
  static $val = 0;
  echo $val++;
}

add(); // "0"
add(); // "1"
add(); // "2"
```

The initial value that a static variable is given will only be set once. Keep in mind that static properties and static variables may only be initialized with a constant and not with an expression, such as another variable or a function return value.

Late static bindings

As mentioned before, the `self` keyword is an alias for the class name of the enclosing class. This means that the keyword will refer to its enclosing class even when it is called from the context of a child class.

```
class MyParent
{
  protected static $val = 'parent';

  public static function getVal()
  {
    return self::$val;
  }
}

class MyChild extends MyParent
{
  protected static $val = 'child';
}

echo MyChild::getVal(); // "parent"
```

To get the class reference to evaluate to the actual calling class, the `static` keyword needs to be used instead of the `self` keyword. This feature is called late static bindings and was added in PHP 5.3.

```
class MyParent
{
  protected static $val = 'parent';

  public static function getLateBindingVal()
  {
    return static::$val;
  }
}

class MyChild extends MyParent
{
  protected static $val = 'child';
}

echo MyChild::getLateBindingVal(); // "child"
```

CHAPTER 13

![image](images/frontdot.jpg)

Constants

A constant is a variable with a value that cannot be changed in the script. Such a value must therefore be assigned at the same time as the constant is created. PHP provides two methods for creating constants: the `const` modifier and the `define` function.

Const

The `const` modifier is used to create class constants. Unlike regular properties, class constants do not have an access level specified, as they are always publicly visible. They also do not use the dollar sign parser token (`$`). The naming convention for constants is all-uppercase, with underscores separating each word.

```
class MyCircle
{
  const PI = 3.14;
}
```

Constants must be assigned a value when they are created. Like static properties, a constant may only be initialized with a constant value and not with an expression. Class constants are referenced in the same way as static properties, except that they do not use the dollar sign.

```
echo MyCircle::PI; // "3.14"
```

The `const` modifier may not be applied to local variables or parameters. However, as of PHP 5.3 `const` can be used to create global constants. Such a constant is defined in the global scope and can be accessed anywhere in the script.

```
const PI = 3.14;
echo PI; // "3.14"
```

Define

The `define` function can create both global and local constants, but not class constants. The first argument to this function is the constant’s name and the second is its value.

```
define('DEBUG', 1);
```

Just as constants created with `const`, define constants are used without the dollar sign and their value cannot be modified.

```
echo DEBUG; // "1"
```

Like constants created with const, the value for define may be any scalar data type: integer, float, string or bool. However, unlike const the define function allows an expression to be used in the assignment, such as a variable or the result of a mathematical expression.

```
define('ONE', 1);     // 1
define('TWO', ONE+1); // 2
```

Constant are case-sensitive by default. However, the `define` function takes a third optional argument which may be set to true to create a case-insensitive constant.

```
define('DEBUG', 1, true);
echo debug; // "1"
```

To check whether a constant already exists the `defined` function can be used. This function works both for constants created with `const` and `define`.

```
if (!defined('PI'))
  define('PI', 3.14);
```

Const and define

The `const` modifier creates a compile-time constant and so the compiler will replace all usage of the constant with its value. In contrast, `define` creates a run-time constant which is not set until run-time. This is the reason why define constants may be assigned with expressional values, whereas `const` requires constant values which are known at compile-time.

```
const PI = 3.14;         // compile-time constant
define('BIT_2', 1 << 2); // run-time constant
```

Only `const` may be used for class constants and only `define` for local constants. However, when creating global constants both `const` and `define` are allowed. In these circumstances using `const` is generally preferable. Because `const` is a language construct it reads better than `define` which is a function. Compile-time constants are also slightly faster than run-time constants. The only exception is when the constant is conditionally defined, or an expressional value is required, in which case `define` must be used.

Constant guideline

In general, it is a good idea to create constants instead of variables if their values do not need to be changed. This ensures that the variables will not be changed anywhere in the script by mistake, which in turn helps to prevent bugs.

Magic constants

PHP provides eight predefined constants. These are called magic constants as their values change depending on where they are used.

| Name | Description |
| --- | --- |
| __LINE__ | Current line number of the file. |
| __FILE__ | Full path and filename of the file. |
| __DIR__ | Directory of the file. |
| __FUNCTION__ | Function name. |
| __CLASS__ | Class name including namespace. |
| __TRAIT__ | Trait name including namespace. |
| __METHOD__ | Class method name. |
| __NAMESPACE__ | Current namespace. |

Magic constants are especially useful for debugging purposes. For example, the value of __LINE__ depends on the line on which it appears in the script.

```
if(!isset($var))
{
  echo '$var not set on line ' . __LINE__;
}
```

CHAPTER 14

![image](images/frontdot.jpg)

Interface

An interface specifies methods that classes using the interface must implement. They are defined with the `interface` keyword followed by a name and a code block. Their naming convention is to start with a small “i” and then to have each word initially capitalized.

```
interface iMyInterface {}
```

Interface signatures

The code block for an interface can contain signatures for instance methods. These methods cannot have any implementations. Instead, their bodies are replaced by semicolons. Interface methods must always be public.

```
interface iMyInterface
{
  public function myMethod();
}
```

Additionally, interfaces may define constants. These interface constants behave just as class constants, except that they cannot be overridden.

```
interface iMyInterface
{
  const PI = 3.14;
}
```

An interface may not inherit from a class, but it may inherit from another interface, which effectively combines the interfaces into one.

```
interface i1 {}
interface i2 extends i1 {}
```

Interface example

The example below shows an interface called iComparable, which has a single method named Compare. Note that this method makes use of type hinting to make sure that the method is called with the correct type. This functionality will be covered in a later chapter.

```
interface iComparable
{
  public function compare(iComparable $o);
}
```

The Circle class implements this interface, by using the `implements` keyword after the class name followed by the interface name. If the class also has an extends clause, the implements clause needs to be placed after it. Bear in mind that although a class can only inherit from one parent class it may implement any number of interfaces, by specifying them in a comma separated list.

```
class Circle implements iComparable
{
  public $r;
}
```

Because `Circle` implements `iComparable` it must define the compare() method. For this class the method will return the difference between the circle radiuses. The implemented method must be public, in addition to having the same signature as the method defined in the interface. It may also have more parameters, as long as they are optional.

```
class Circle implements iComparable
{
  public $r;

  public function compare(iComparable $o)
  {
    return $this->r - $o->r;
  }
}
```

Interface usages

Interfaces allow for multiple inheritance of class design without the complications associated with allowing multiple inheritance of functionality. The main benefit of requiring a specific class design can be seen with the iComparable interface, which defines a specific functionality that classes can share. It allows developers to use interface members without having to know the actual type of a class. To illustrate, the example below shows a simple method that takes two iComparable objects and returns the largest one.

```
function largest(iComparable $a, iComparable $b)
{
  return ($a->compare($b) > 0) ? $a : $b;
}
```

This method will work for any two objects of the same type that implement the iComparable interface. It will work regardless of what type the objects are since the method only uses the functionality exposed through that interface.

Interface guideline

An interface provides a design for a class without any implementation. It is a contract by which classes that implement it agree to provide certain functionality. This has two benefits. First, it provides a way to make sure developers implement certain methods. Second, because these classes are guaranteed to have certain methods, they can be used even without knowing the class’s actual type, which allows the code to be more flexible.

CHAPTER 15

![image](images/frontdot.jpg)

Abstract

An abstract class provides a partial implementation that other classes can build upon. When a class is declared as abstract it means that the class can contain incomplete methods that must be implemented in child classes, in addition to normal class members.

Abstract methods

In an abstract class any method can be declared abstract. These methods are then left unimplemented and only their signatures are specified, while their code blocks are replaced by semicolons.

```
abstract class Shape
{
  abstract public function myAbstract();
}
```

Abstract example

To give an example, the class below has two properties and an abstract method.

```
abstract class Shape
{
  private $x = 100, $y = 100;
  abstract public function getArea();
}
```

If a class inherits from this abstract class it is then forced to override the abstract method. The method signature must match, except for the access level which can be made less restricted.

```

 class Rectangle extends Shape
{
  public function getArea()
  {
    return $this->x * $this->y;
  }
}
```

It is not possible to instantiate an abstract class. They serve only as parents for other classes, partly dictating their implementation.

```
$s = new Shape(); // compile-time error
```

However, an abstract class may inherit from a non-abstract (concrete) class.

```
class NonAbstract {}
abstract class MyAbstract extends NonAbstract {}
```

Abstract classes and interfaces

Abstract classes are in many ways similar to interfaces. They can both define member signatures that the deriving classes must implement, and neither one of them can be instantiated. The key differences are first that the abstract class can contain non-abstract members, while the interface cannot. Second, a class can implement any number of interfaces but only inherit from one class, abstract or not.

```
// Defines default functionality and definitions
abstract class Shape
{
  public $x = 100, $y = 100;
  abstract public function getArea();
}
// Class is a Shape
class Rectangle extends Shape { /*...*/ }

// Defines a specific functionality
interface iComparable
{
  function compare();
}
// Class can be compared
class MyClass implements iComparable { /*...*/ }
```

Abstract guideline

An abstract class provides a partially implemented base class that dictates how child classes must behave. They are most useful when child classes share some similarities, but differ in other implementations that child classes are required to define. Just like interfaces, abstract classes are useful constructs in object-oriented programming that help developers follow good coding standards.

CHAPTER 16

![image](images/frontdot.jpg)

Traits

A trait is a group of methods that can be inserted into classes. They were added in PHP 5.4 to enable greater code reuse without the added complexity that comes from allowing multiple-inheritance. Traits are defined with the `trait` keyword followed by a name and a code block. The naming convention is the same as for classes, with each word initially capitalized. The code block may only contain static and instance methods.

```
trait PrintFunctionality
{
  public function myPrint() { echo 'Hello'; }
}
```

Classes that need the functionality that a trait provides can include it with the `use` keyword followed by the trait’s name. The trait’s methods will then behave just as if they had been directly defined in that class.

```
class MyClass
{
  // Insert trait methods
  use PrintFunctionality;
}

$o = new MyClass();
$o->myPrint(); // "Hello"
```

A class may use multiple traits by placing them in a comma-separated list. Similarly, a trait may be composed from one or more other traits.

Inheritance and traits

Trait methods will override inherited methods. Likewise, methods defined in the class will override methods inserted by a trait.

```
class MyParent
{
  public function myPrint() { echo 'Base'; }
}

class MyChild extends MyParent
{
  // Overrides inherited method
  use PrintFunctionality;

  // Overrides trait inserted method
  public function myPrint() { echo 'Child'; }
}

$o = new MyChild();
$o->myPrint(); // "Child"
```

Trait guideline

Single inheritance sometimes forces the developer to make a choice between code reuse and a conceptually clean class hierarchy. To achieve greater code reuse, methods can be moved near the root of the class hierarchy, but then classes start to have methods they do not need which reduces the understandability and maintainability of the code. On the other hand, enforcing conceptual cleanliness in the class hierarchy will often lead to code duplication, which may cause inconsistencies. Traits provide a way to avoid this shortcoming in single inheritance by enabling code reuse that is independent of the class hierarchy.

CHAPTER 17

![image](images/frontdot.jpg)

Importing Files

The same code often needs to be called on multiple pages. This can be done by first placing the code inside a separate file and then including that file using the include statement. This statement takes all the text in the specified file and includes it in the script, just as if the code had been copied to that location. Just like `echo`, `include` is a special language construct and not a function, so parentheses should not be used.

```
<?php
include 'myfile.php';
?>
```

When a file is included parsing changes to HTML mode at the beginning of the target file and resumes PHP mode again at the end. For this reason any code inside the included file that needs to be executed as PHP code must be enclosed within PHP tags.

```
<?php
// myfile.php
?>
```

Include path

An include file can either be specified with a relative path, an absolute path or without a path. A relative file path will be relative to the importing file’s directory, and an absolute file path will include the full file path.

```
// Relative path
include 'myfolder\myfile.php';

// Absolute path
include 'C:\xampp\htdocs\myfile.php';
```

When a relative path or no path is specified, include will first search for the file in the current working directory, which defaults to the directory of the importing script. If the file is not found there, include will check the folders specified by the include_path[¹](#9781430262831_Ch17.xhtml#Fn1) directive defined in php.ini before failing.

```
// No path
include 'myfile.php';
```

In addition to include there are three other language constructs available for importing the content of one file into another: `require`, `include_once` and `require_once`.

Require

The `require` construct includes and evaluates the specified file. It is identical to `include`, except in how it handles failure. When a file import fails `require` will halt the script with an error, whereas `include` will only issue a warning. An import may fail either because the file is not found or because the user running the web server does not have read access to it.

```
require 'myfile.php'; // halt on error
```

Generally it is best to use `require` for any complex PHP application or CMS site. That way the application will not attempt to run in case a key file is missing. For less critical code segments and simple PHP websites `include` may suffice, in which case PHP will go on and show the output even if the included file is missing.

Include_once

The `include_once` statement behaves like `include`, except that if the specified file has already been included it will not be included again.

```
include_once 'myfile.php'; // include only once
```

Require_once

The `require_once` statement works like `require`, but will not import a file if it has already been imported before.

```
require_once 'myfile.php'; // require only once
```

The `include_once` and `require_once` statements may be used instead of `include` and `require` in cases where the same file might be imported more than once during a particular execution of a script. This avoids errors caused by, for example, function and class redefinitions.

Return

It is possible to execute a return statement inside an imported file. This will stop the execution and return to the script that called the file import.

```
<?php
// myimport.php
return 'OK';
?>
```

If a return value is specified, the import statement will evaluate to that value just as a normal function.

```
<?php
// myfile.php
if ((include 'myimport.php') == 'OK')
  echo 'OK';
?>
```

Auto load

For large web applications the number of includes required in every script may be substantial. This can be avoided by defining an `__autoload` function. This function is automatically invoked when an undefined class or interface is used in order to try to load that definition. It takes one parameter, which is the name of the class or interface that PHP is looking for.

```
function __autoload($class_name){
  include $class_name . '.php';
}

// Attempt to auto include MyClass.php
$obj = new MyClass();
```

A good coding practice to follow when writing object-oriented applications is to have one source file for every class definition and to name the file according to the class name. Following this convention the `__autoload` function above will be able to load the class, provided that it is in the same folder as the script file that needed it.

```
<?php
// myclass.php
class MyClass {}
?>
```

If the file is located in a subfolder the class name can include underscore characters to symbolize this. The underscore characters would then need to be converted into directory separators in the `__autoload` function.

[¹](#9781430262831_Ch17.xhtml#_Fn1)`http://www.php.net/manual/en/ini.core.php#ini.include-path`

CHAPTER 18

![image](images/frontdot.jpg)

Type Hinting

PHP relies on the proper documentation of functions for developers to know what arguments a function can take. To simplify this PHP 5 introduced type hinting, which allows a function to specify the type of arguments it accepts. Allowed types include classes, interfaces and the pseudo types `array` and `callable`.

| Name | Description |
| --- | --- |
| class name | Argument must be an object of this class or a child. |
| interface name | Argument must be an object implementing this interface. |
| `array` | Argument must be an array. |
| `callable` | Argument must be callable as a function. |

A type hint is set by prefixing the parameter with the type in the function signature. Below is an example using the `array` pseudo type introduced in PHP 5.1.

```
function myprint(array $a)
{
  foreach ($a as $v) { echo $v; }
}

myprint( array(1,2,3) ); // "123"
```

Failing to satisfy the type hint results in a fatal error. This makes it easier to detect when an invalid argument is used.

```
myprint('Test'); // error
```

The `callable` pseudo type was added in PHP 5.4\. With this type hint in place the argument must be a callable function, method or object. Language constructs such as `echo` are not allowed, but anonymous functions may be used as in the example below.

```
function mycall(callable $callback, $data)
{
  $callback($data);
}

$say = function($myString) { echo $myString; };
mycall( $say, 'Hi' ); // "Hi";
```

Type hints cannot be used with scalar types, such as bool, int or string. The pseudo types used by built-in functions, such as mixed and number, are also not allowed. Furthermore, there is no way to hint a function’s return value.

CHAPTER 19

![image](images/frontdot.jpg)

Type Conversions

PHP will automatically convert a variable’s data type as necessary given the context in which it is used. For this reason explicit type conversions are seldom required. Nonetheless, the type of a variable or expression may be changed by performing an explicit type cast.

Explicit casts

An explicit cast is performed by placing the desired data type in parentheses before the variable or value that is to be evaluated. In the example below, the explicit cast forces the bool variable to be evaluated as an int.

```
$myBool = false;
$myInt = (int)$myBool; // 0
```

One use for explicit casts can be seen when the bool variable is sent as output to the page. Due to automatic type conversions the false value becomes an empty string and is therefore not displayed. By first converting it to an integer, the value false will show up as zero instead.

```
echo $myBool;      // ""
echo (int)$myBool; // "0"
```

Allowed casts are listed in the table below.

| Name | Description |
| --- | --- |
| (int), (integer) | Cast to int |
| (bool), (boolean) | Cast to bool |
| (float), (double), (real) | Cast to float |
| (string) | Cast to string |
| (array) | Cast to array |
| (object) | Cast to object |
| (unset) | Cast to null |

To give some examples, the array cast will convert a scalar type to an array with a single element. It performs the same function as using the array constructor.

```
$myInt = 10;
$myArr = (array)$myInt;
$myArr = array($myInt); // same as above
echo $myArr[0];         // "10"
```

If a scalar type such as int is cast to object it will become an instance of the built-in class `stdClass`. The value of the variable will be stored in a property of this class called `scalar`.

```
$myObj = (object)$myInt;
echo $myObj->scalar; // "10"
```

The unset cast makes the variable evaluate to null. Despite its name, it does not actually unset the variable. The cast merely exists for the sake of completeness, as null is considered a data type.

```
$myNull = (unset)$myInt;
$myNull = null; // same as above
```

Settype

An explicit cast does not change the type of the variable it precedes, only how it is evaluated in that expression. To change the type of a variable the `settype` function can be used, which takes two arguments. The first is the variable to be converted and the second is the data type given as a string.

```
$myVar = 1.2;
settype($myVar, 'int'); // convert variable to int
```

Alternatively, a type conversion can be performed by storing the result of an explicit cast back into the same variable.

```
$myVar = 1.2;
$myVar = (int)$myVar; // 1
```

Gettype

Related to `settype` is the `gettype` function, which returns the type of the supplied argument as a human-readable string.

```
$myBool = true;
echo gettype($myBool); // "boolean"
```

CHAPTER 20

![image](images/frontdot.jpg)

Variable Testing

PHP has a number of built-in constructs that can be used to test the value of a variable. All these functions return a bool value.

Isset

The `isset` language construct returns true if the variable has been assigned a value other than null.

```
isset($a); // false

$a = 10;
isset($a); // true

$a = null;
isset($a); // false
```

Empty

The `empty` construct checks whether the specified variable has an empty value – such as null, 0, false or an empty string – and returns true if that is the case. It also returns true if the variable does not exist.

```
empty($b); // true

$b = false;
empty($b); // true
```

Is_null

The `is_null` construct can be used to test whether a variable is set to null.

```
$c = null;
is_null($c); // true

$c = 10;
is_null($c);  // false
```

If the variable does not exist `is_null` will also return true, but with an error notice as it is not supposed to be used with uninitialized variables.

```
is_null($d); // true (undefined variable notice)
```

Unset

Another language construct that is useful to know about is `unset`, which deletes a variable from the current scope.

```
$e = 10;
unset($e); // delete $e
```

References may be deleted with `unset` without destroying the variable content.

```
$var = 'my value';
$ref = &$var;

unset($ref); // delete only $ref
```

When a global variable is made accessible in a function with the `global` keyword, this code actually creates a local reference to the global variable in the $GLOBALS array. For this reason, attempting to unset a global variable in a function will only delete the local reference. To delete the global variable from a function scope the unset has to be made directly on the $GLOBALS array.

```
function myUnset()
{
  // Make $o a reference to $GLOBALS['o']
  global $o;

  // Remove the reference variable
  unset($o);

  // Remove the global variable
  unset($GLOBALS['o']);
}
```

Unsetting a variable is slightly different than setting the variable to null. When a variable is set to null the variable still exists, but the variable content it held is immediately freed. In contrast, unsetting a variable deletes the variable, but the memory will still be considered to be in use until the garbage collector clears it. Performance issues aside, using `unset` is recommended as it makes the code’s intent clearer.

```
$var = null; // free memory
unset($var); // delete variable
```

Keep in mind that most of the time it is not necessary to manually unset variables, as the PHP garbage collector will automatically delete variables when they go out of scope. However, if a server performs very memory intensive tasks then unsetting those variables manually will allow the server to handle a greater number of simultaneous requests before running out of memory.

Determining types

PHP has several useful functions for determining the type of a variable. These functions can be seen in the table below.

| Name | Description |
| --- | --- |
| is_array() | True if variable is an array. |
| is_bool() | True if variable is a bool. |
| is_callable() | True if variable can be called as a function. |
| is_float(), is_double(), is_real() | True if variable is a float. |
| is_int(), is_integer(), is_long() | True if variable is an integer. |
| is_null() | True if variable is set to null. |
| is_numeric() | True if variable is a number or numeric string. |
| is_scalar() | True if variable is an int, float, string or bool. |
| is_object() | True if variable is an object. |
| is_resource() | True if variable is a resource. |
| is_string() | True if variable is a string. |

To give an example, the `is_numeric` function will return true if the argument contains either a number or a string that can be evaluated to a number.

```
is_numeric(10.5);   // true  (float)
is_numeric('33');   // true  (numeric string)
is_numeric('text'); // false (non-numeric string)
```

Variable information

PHP has three built-in functions for retrieving information about variables: `print_r`, `var_dump` and `var_export`. The `print_r` function displays the value of a variable in a human-readable way. It is useful for debugging purposes.

```
$a = array('one', 'two', 'three');
print_r($a);
```

The code above will produce the following output.

```
Array ( [0] => one [1] => two [2] => three )
```

Similar to `print_r` is `var_dump`, which in addition to values also displays data types and sizes. Calling `var_dump($a)` will show this output.

```
array(3) {
  [0]=> string(3) "one"
  [1]=> string(3) "two"
  [2]=> string(5) "three"
}
```

Finally, there is the `var_export` function, which prints variable information in a style that can be used as PHP code. The output for `var_export($a)` can be seen below. Note the trailing comma after the last element which is allowed.

```
array ( 0 => 'one', 1 => 'two', 2 => 'three', )
```

The `var_export` function, along with `print_r`, accepts an optional Boolean second argument. When set to true, the function will return the output instead of printing it. This gives `var_export` further uses, such as being combined with the language construct `eval` . This construct takes a string argument and evaluates it as PHP code.

```
eval('$b = ' . var_export($a, true) . ';');
```

The ability to execute arbitrary code with `eval` is a powerful feature that should be used with care. It should not be used to execute any user provided data, at least not without proper validation, as this represents a security risk. Another reason why the use of `eval` is discouraged is because similarly to `goto` it makes the execution of code more difficult to follow.

CHAPTER 21

![image](images/frontdot.jpg)

Overloading

Overloading in PHP provides the ability to add object members at runtime. This is done by having the class implement the overloading methods `__get`, `__set`, `__call` and `__callStatic`. Bear in mind that the meaning of overloading in PHP is different from many other languages.

Property overloading

The `__get` and `__set` methods provide a convenient way to implement getter and setter methods, which are methods that are often used to safely read and write to properties. These overloading methods are invoked when using properties that are inaccessible, either because they are not defined in the class or because they are unavailable from the current scope. In the example below, the `__set` method adds any inaccessible properties to the `$data` array and `__get` safely retrieves the elements.

```
class MyProperties
{
  private $data = array();

  public function __set($name, $value)
  {
    $this->data[$name] = $value;
  }

  public function __get($name)
  {
    if (array_key_exists($name, $this->data))
      return $this->data[$name];
  }
}
```

When setting the value of an inaccessible property `__set` is called with the name of the property and the value as its arguments. Similarly, when accessing an inaccessible property `__get` is called with the property name as its argument.

```
$obj = new MyProperties();

$obj->a = 1;  // __set called
echo $obj->a; // __get called
```

Method overloading

There are two methods for handling calls to inaccessible methods of a class – `__call` and `__callStatic`. The `__call` method is invoked for instance method calls.

```
class MyClass
{
  public function __call($name, $args)
  {
    echo "Calling $name $args[0]";
  }
}

// "Calling myTest in object context"
(new MyClass())->myTest('in object context');
```

The first argument to `__call` is the name of the method being called and the second is a numeric array containing the parameters passed to the method. These arguments are the same for the `__callStatic` method, which handles calls to inaccessible static methods.

```
class MyClass
{
  public static function __callStatic($name, $args)
  {
    echo "Calling $name $args[0]";
  }
}

// "Calling myTest in static context"
MyClass::myTest('in static context');
```

Isset and unset overloading

The built-in constructs `isset`, `empty` and `unset` will only work on explicitly defined properties, not overloaded ones. This functionality can be added to a class by overloading the `__isset` and `__unset` methods.

```
class MyClass
{
  private $data = array();

  public function __set($name, $value) {
    $this->data[$name] = $value;
  }
  public function __get($name) {
    if (array_key_exists($name, $this->data))
      return $this->data[$name];
  }

  public function __isset($name) {
    return isset($this->data[$name]);
  }

  public function __unset($name) {
    unset( $this->data[$name] );
  }
}
```

The `__isset` method will be invoked when `isset` is called on an inaccessible property.

```
$obj = new MyClass();
$obj->name = "Joe";

isset($obj->name); // true
isset($obj->age);  // false
```

When `unset` is called on an inaccessible property the `__unset` method will handle that call.

```
unset($obj->name); // delete property
isset($obj->name); // false
```

The `empty` construct will only work on overloaded properties if both `__isset` and `__get` are implemented. If the result from `__isset` is false the `empty` construct will return true. If on the other hand `__isset` returns true then `empty` will retrieve the property with `__get` and evaluate if it has a value considered to be empty.

```
empty($obj->name); // false
empty($obj->age);  // true
```

CHAPTER 22

![image](images/frontdot.jpg)

Magic Methods

There are a number of methods that can be implemented in a class for the purpose of being called internally by the PHP engine. These are known as magic methods and they are easy to recognize as they all start with two underscores. Listed in the table below are the magic methods that have been discussed so far.

| Name | Description |
| --- | --- |
| __construct(. . .) | Called when creating a new instance. |
| __destruct() | Called when object has no references left. |
| __call($name, $array) | Called when invoking inaccessible methods in an object context. |
| __callStatic($name, $array) | Called when invoking inaccessible methods in a static context. |
| __get($name) | Called when reading data from inaccessible properties. |
| __set($name, $value) | Called when writing data to inaccessible properties. |
| __isset($string) | Called when `isset` or `empty` is used on inaccessible properties. |
| __unset($string) | Called when `unset` is used on inaccessible properties. |

In addition to these, there are six more magic methods that like the others can be implemented in classes to provide certain functionalities.

| Name | Description |
| --- | --- |
| __toString() | Called for object to string conversions. |
| __invoke(. . .) | Called for object to function conversions. |
| __sleep() | Called by `serialize`. Performs cleanup tasks and returns an array of variables to be serialized. |
| __wakeup() | Called by `unserialize` to reconstruct the object. |
| __set_state($array) | Called by `var_export`. The method must be static and its argument contains the exported properties. |
| __clone() | Called after object has been cloned. |

Tostring

When an object is used in a context where a string is expected, the PHP engine will search for a method named `__toString` to retrieve a string representation of the object.

```
class MyClass
{
  public function __toString()
  {
    return 'Instance of ' . __CLASS__;
  }
}

$obj = new MyClass();
echo $obj; // "Instance of MyClass"
```

It is not possible to define how an object will behave when evaluated as types other than string.

Invoke

The `__invoke` method allows an object to be treated as a function. Any arguments provided when the object is called will be used as the `__invoke` function’s arguments.

```
class MyClass
{
  public function __invoke($arg)
  {
    echo $arg;
  }
}

$obj = new MyClass();
$obj('Test'); // "Test"
```

Object serialization

Serialization is the process of converting data into a string format. This is useful for storing objects in databases or files. In PHP, the built-in `serialize` function performs this object-to-string conversion and `unserialize` converts the string back into the original object. The `serialize` function handles all types, except for the resource type, which for example is used to hold database connections and file handlers. Consider the following simple database class.

```
class MyConnection
{
  public $link, $server, $user, $pass;

  public function connect()
  {
    $this->link = mysql_connect($this->server,
                                $this->user,
                                $this->pass);
  }
}
```

When this class is serialized the database connection will be lost and the resource type variable `$link` holding the connection will be stored as null.

```
$obj = new MyConnection();
// ...

$bin = serialize($obj);   // serialize object
$obj = unserialize($bin); // restore object
```

To get greater control over how object data is serialized and unserialized the `__sleep` and `__wakeup` methods may be implemented by this class.

Sleep

The `__sleep` method is called by `serialize` and needs to return an array containing the properties that will be serialized. This array must not include private or protected properties, as serialize will not be able to access them. The method may also perform cleanup tasks before the serialization occurs, such as committing any pending data to storage mediums.

```
public function __sleep()
{
  return array('server', 'user', 'pass');
}
```

Note that the properties are returned to `serialize` in string form. The resource type pointer `$link` is not included in the array as it cannot be serialized. To reestablish the database connection the `__wakeup` method can be used.

Wakeup

Calling `unserialize` on the serialized object will invoke the `__wakeup` method in order to restore the object. It accepts no arguments and does not need to return any value. It is used for re-establishing resource-type variables and for performing other initializing tasks that may need to be done after the object has been unserialized. In this example, it reestablishes the MySQL database connection.

```
public function __wakeup()
{
  if(isset($this->server, $this->user, $this->pass))
    $this->connect();
}
```

Note that the `isset` construct is called here with multiple arguments, in which case it will only return true if all parameters are set.

Set state

The `var_export` function retrieves variable information that is to be usable as valid PHP code. In the example below this function is used on an object.

```
class Fruit
{
  public $name = 'Lemon';
}

$export = var_export(new Fruit(), true);
```

Since an object is a complex type there is no generic syntax for constructing it along with its members. Instead, `var_export` creates the following string.

```
Fruit::__set_state(array( 'name' => 'Lemon', ))
```

This string relies on a static `__set_state` method being defined for the object in order to construct it. As seen above, the `__set_state` method takes an associative array containing key-value pairs of each of the object’s properties, including private and protected members.

```
static function __set_state(array $array)
{
  $tmp = new Fruit();
  $tmp->name = $array['name'];
  return $tmp;
}
```

With this method defined in the Fruit class the exported string can now be parsed with the `eval` construct to create an identical object.

```
eval('$MyFruit = ' . $export . ';');
```

Object cloning

Assigning an object to a variable will only create a new reference to the same object. In order to copy an object, the `clone` operator can be used.

```
class Fruit {}

$f1 = new Fruit();
$f2 = $f1;       // copy object reference
$f3 = clone $f1; // copy object
```

When an object is cloned its properties will be copied over to the new object. However, any child objects it may contain will not be cloned, so they will be shared between the copies. This is where the `__clone` method comes in. It will be called on the cloned copy after the cloning is done and can be used to clone any child objects.

```
class Apple {}

class FruitBasket
{
  public $apple;

  function __construct() { $apple = new Apple(); }

  function __clone()
  {
    $this->apple = clone $this->apple;
  }
}
```

CHAPTER 23

![image](images/frontdot.jpg)

User Input

When an HTML form is submitted to a PHP page the data becomes available to that script.

HTML form

An HTML form has two required attributes: action and method. The action attribute specified the script to which the form data is passed. For example, the following form submits one input property called `myString` to the script file MyPage.php.

```
<html>
<body>
  <form action="MyPage.php" method="post">
    <input type="text" name="myString" />
    <input type="submit" />
  </form>
</body>
</html>
```

The other required attribute of the form element specifies the sending method, which may be either get or post.

Sending with post

If the form is sent using the post method the data will be available through the `$_POST` array. The names of the properties will be the keys in that associative array. Data sent with the post method is not visible on the URL of the page, but this also means that the state of the page cannot be saved by, for example, bookmarking the page.

```
echo $_POST['myString'];
```

Sending with get

The alternative to post is to send the form data with the get method and to retrieve it using the `$_GET` array. The variables are then displayed in the address bar, which effectively maintains the state of the page if it is bookmarked and revisited.

```
echo $_GET['myString'];
```

Because the data is contained in the address bar this means that variables cannot only be passed through HTML forms but also through HTML links. The `$_GET` array can then be used to change the state of the page accordingly. This provides one way of passing variables from one page to another.

```
<a href="MyPage.php?myString=Foo+Bar">link</a>
```

Request array

If it does not matter whether the post or get method was used to send the data the  `$_REQUEST` array can be used. This array typically contains the `$_GET` and `$_POST` arrays, but may also contain the `$_COOKIE` array.

```
echo $_REQUEST['myString']; // "Foo Bar"
```

The content of the `$_REQUEST` array can be set in the PHP configuration file[¹](#9781430262831_Ch23.xhtml#Fn1) and varies between PHP distributions. Due to security concerns, the `$_COOKIE` array is usually not included.

Security concerns

Any user-provided data can be manipulated and should therefore be validated and sanitized before being used. Validation means that you make sure the data is in the form you expect, in terms of data type, range and content. For example, the following code validates an email address.

```
if(!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL))
  echo "Invalid email address";
```

Sanitizing is when you disable potentially malicious code in the user input. This is done by escaping the code according to the rules of the language where the input is to be used. For example, if the data will be sent to a database it needs to be sanitized with the `mysql_real_escape_string` function to disable any embedded SQL code.

```
// Sanitize for database use
$name = mysql_real_escape_string($_POST['name']);

// Execute SQL command
$sql = "SELECT * FROM users WHERE user='" . $name . "'";
$result = mysql_query($sql);
```

When user supplied data is to be output to the web page as text the `htmlspecialchars` function should be used. It will disable any HTML markup so the user input is displayed but not interpreted.

```
// Sanitize for web page use
echo htmlspecialchars($_POST['comment']);
```

Submitting arrays

Form data can be grouped into arrays by including array square brackets after the variable names in the form. This works for all form input elements, including `<input>`, `<select>` and `<textarea>`.

```
<input type="text" name="myArr[]" />
<input type="text" name="myArr[]" />
```

The elements may also be assigned their own array keys.

```
<input type="text" name="myArr[name]" />
```

Once submitted, the array will be available for use in the script.

```
$val1 = $_POST['myArr'][0];
$val2 = $_POST['myArr'][1];
$name = $_POST['myArr']['name'];
```

The form `<select>` element has an attribute for allowing multiple items to be selected from the list.

```
<select name="myArr[]" size="3" multiple="true">
  <option value="apple">Apple</option>
  <option value="orange">Orange</option>
  <option value="pear">Pear</option>
</select>
```

When this multi-select element is included in a form, the array brackets become necessary for retrieving the selected values in the script.

```
foreach ($_POST['myArr'] as $item)
  echo $item . ' '; // ex "apple orange pear"
```

File uploading

The HTML form provides a file input type that allows files to be uploaded to the server. For file uploading to work the form’s optional enctype attribute must be set to “multipart/form-data”, as in the example below.

```
<form action="MyPage.php" method="post"
      enctype="multipart/form-data">
  <input name="myfile" type="file" />
  <input type="submit" value="Upload" />
</form>
```

Information about the uploaded file is stored in the `$_FILES` array. The keys of this associative array are seen in the following table.

| Name | Description |
| --- | --- |
| name | Original name of uploaded file. |
| tmp_name | Path to temporary server copy. |
| type | Mime type of the file. |
| size | File size in bytes. |
| error | Error code. |

A received file is only temporarily stored on the server. If it is not saved by the script it will be deleted. A simple example of how to save the file is given below. The example checks the error code to make sure that the file was successfully received, and if so moves the file out of the temporary folder to save it. In practice, you would also want to examine the file size and type in order to determine whether the file is to be kept.

```
$dest = 'upload\\' . basename($_FILES['myfile']['name']);
$file = $_FILES['myfile']['tmp_name'];
$err  = $_FILES['myfile']['error'];

if($err == 0 && move_uploaded_file($file, $dest))
  echo 'File successfully uploaded';
```

Two new functions are seen in this example. The `move_uploaded_file` function checks to ensure the first argument contains a valid upload file, and if so it moves it to the path and renames it to the filename, specified by the second argument. The specified folder must already exist and if the function succeeds in moving the file it returns true. The other new function is `basename`. It returns the filename component of a path, including the file extension.

Superglobals

As seen in this chapter there are a number of built-in associate arrays that make external data available to PHP scripts. These arrays are known as superglobals, because they are automatically available in every scope. There are a total of nine superglobals in PHP, each of which is described briefly below.

| Name | Description |
| --- | --- |
| $GLOBALS | Contains all global variables, including other superglobals. |
| $_GET | Contains variables sent via a HTTP GET request. |
| $_POST | Contains variables sent via a HTTP POST request. |
| $_FILES | Contains variables sent via a HTTP POST file upload. |
| $_COOKIE | Contains variables sent via HTTP cookies. |
| $_SESSION | Contains variables stored in a user’s session. |
| $_REQUEST | Contains $_GET, $_POST and possibly $_COOKIE variables. |
| $_SERVER | Contains information about the web server and the request made to it. |
| $_ENV | Contains all environment variables set by the web server. |

The content of the variables `$_GET, $_POST, $_COOKIE, $_SERVER` and *$_ENV* is included in the output generated by the `phpinfo` function. This function will also display the general settings of the PHP configuration file, php.ini, along with other information regarding PHP.

```
phpinfo(); // display PHP information
```

[¹](#9781430262831_Ch23.xhtml#_Fn1)`http://www.php.net/manual/en/ini.core.php#ini.request-order`

CHAPTER 24

![image](images/frontdot.jpg)

Cookies

A cookie is a small file kept on the client’s computer that can be used to store data relating to that user.

Creating cookies

To create a cookie the `setcookie` function is used. This function must be called before any output is sent to the browser. It has three mandatory parameters that contain the name, value and expiration date of the cookie.

```
setcookie("lastvisit", date("H:i:s"), time() + 60*60);
```

The value is here set with the `date` function, which returns a string formatted according to the specified format string. The expiration date is measured in seconds and is usually set relative to the current time in seconds retrieved through the `time` function. In this example, the cookie expires after one hour.

Cookie array

Once the cookie has been set for a user this cookie will be sent along the next time that user views the page and can then be accessed through the $_COOKIE array.

```
if (isset($_COOKIE['lastvisit']))
  echo "Last visit: " . $_COOKIE['lastvisit'];
```

Deleting cookies

A cookie can be deleted manually by recreating that same cookie with an old expiration date. It will then be removed when the browser is closed.

```
setcookie("lastvisit", 0, 0);
```

CHAPTER 25

![image](images/frontdot.jpg)

Sessions

A session provides a way to make variables accessible across multiple web pages. Unlike cookies, session data is stored on the server.

Starting a session

To begin a session the `session_start` function is used. This function must appear before any output is send to the web page.

```
<?php session_start(); ?>
```

The `session_start` function will set a cookie on the client’s computer containing an id used to associate the client with the session. If the client already has an ongoing session the function will resume that session instead of starting a new one.

Session array

With the session started the $_SESSION array can be used to store session data as well as retrieve it. As an example, the page view count can be stored with the code below. The first time the page is viewed the session element is initialized to one.

```
if(isset($_SESSION['views']))
  $_SESSION['views'] += 1;
else
  $_SESSION['views'] = 1;
```

This element can now be retrieved from any page on the domain as long as `session_start` is called on the top of that page.

```
echo 'Views: ' . $_SESSION['views'];
```

Deleting a session

A session is guaranteed to last until the user leaves the website. Then the garbage collector is free to delete that session. To manually remove a session variable the `unset` function can be used, and for removing all session variables there is the `session_destroy` function.

```
unset($_SESSION['views']); // destroy session variable
session_destroy();         // destroy session
```

CHAPTER 26

![image](images/frontdot.jpg)

Namespaces

Namespaces provide a way to avoid naming conflicts and to group namespace members into a hierarchy. Any code may be contained within a namespace, but only four code constructs are affected: classes, interfaces, functions and constants.

Creating namespaces

A construct that is not included in a namespace belongs to the global namespace.

```
// Global code/namespace
class MyClass {}
```

To assign the construct to another namespace a namespace directive is defined. Any code constructs below the namespace directive will belong to that namespace. The naming convention for namespaces is all lowercase.

```
namespace my;

// Belongs to my namespace
class MyClass {}
```

A script file containing namespaced code must declare the namespace at the top of the file before any other PHP or HTML code.

```
<?php
namespace my;
class MyClass {}
?>
<html><body></body></html>
```

Nested namespaces

Namespaces can be nested any number of levels deep to further define the namespace hierarchy. Like directories and files in Windows, namespaces and their members are separated with a backslash character.

```
namespace my\sub;
class MyClass {} // my\sub\MyClass
```

Alternative syntax

Alternatively, namespaces may be defined with the bracketed syntax commonly used in other programming languages. Just as with the regular syntax, no text or code may exist outside of the namespace.

```
<?php
namespace my
{
  class MyClass {}
?>
<html><body></body></html>
<?php }?>
```

Multiple namespaces can be declared in the same file, although this is not considered good coding practice. If global code is to be combined with namespaced code then the bracketed syntax must be used. The global code is then enclosed in an unnamed namespace block.

```
// Namespaced code
namespace my
{
  const PI = 3.14;
}

// Global code
namespace
{
  echo my\PI; // "3.14"
}
```

Unlike other PHP constructs, the same namespace may be defined in more than one file. This allows namespace content to be split up across multiple files.

Referencing namespaces

A namespace member can be referred to in three ways: fully qualified, qualified and unqualified. The fully qualified name can always be used. It consists of the global prefix operator (`\`) followed by the namespace path and the member. The global prefix operator indicates that the path is relative to the global namespace.

```
namespace my
{
  class MyClass {}
}

namespace other
{
  // Fully qualified name
  $obj = new \my\MyClass();
}
```

The qualified name includes the namespace path, but not the global prefix operator. It can therefore only be used if the wanted member is defined in a namespace below the current namespace in the hierarchy.

```
namespace my
{
  class MyClass {}
}

namespace
{
  // Qualified name
  $obj = new my\MyClass();
}
```

The member name alone, or unqualified name, may only be used within the namespace that defines the member.

```
namespace my
{
  class MyClass {}

  // Unqualified name
  $obj = new MyClass();
}
```

Unqualified class and interface names will only resolve to the current namespace. In contrast, if an unqualified function or constant does not exist in the current namespace, they will try to resolve to any global function or constant by the same name.

```
namespace
{
  function myPrint() { echo 'global'; }
}
namespace my
{
  // Falls back to global namespace
  myPrint(); // "global"
}
```

Alternatively, the global prefix operator can be used to explicitly refer to the global member. This would be necessary if the namespace contained a function with the same name.

```
namespace my
{
  function myPrint() { echo 'my'; }

  // Call function from global namespace
  \myPrint(); // "global"
  myPrint();  // "my"
}
```

Namespace aliases

Aliases can be created to shorten qualified names in order to improve the readability of the source code. The names for classes, interfaces and namespaces can be shortened, whereas function and constant aliases are not supported. An alias is defined with a use directive, which must be placed directly after the namespace name.

```
namespace my;
class MyClass {}

namespace foo;
use my\MyClass as MyAlias;
$obj = new MyAlias();
```

With the bracketed syntax, any use directives are placed before the opening curly bracket.

```
namespace foo;
use my\MyClass as MyAlias;
{
  $obj = new MyAlias();
}
```

The as clause may optionally be left out to import the member under its current name.

```
namespace foo;

// Same as my\MyClass as MyClass
use \my\MyClass;
$obj = new MyClass();
```

It is not possible to mass-import the members of another namespace. However, there is a syntactical shortcut for importing multiple members in the same use statement.

```
namespace foo;
use my\Class1 as C1, my\Class2 as C2;
```

Keep in mind that aliases only apply to the script file that defines them. An imported file will therefore not inherit the parent file’s aliases.

Namespace keyword

The `namespace` keyword can be used as a constant that will evaluate to the current namespace, or an empty string in global code. It may be used to explicitly refer to the current namespace.

```
namespace my\name
{
  function myPrint() { echo 'Hi'; }
}
namespace my
{
  namespace\name\myPrint(); // "Hi"
  name\myPrint();           // same as above
}
```

Namespace guideline

As the number of components involved in a web application grow, so too increases the potential for name clashes. One solution for this is to prefix names with the name of the component. However, this creates long names which reduce the readability of the source code. For this reason PHP 5.3 introduced namespaces, which allow developers to group code for each component into separate named containers.

CHAPTER 27

![image](images/frontdot.jpg)

References

A reference is an alias which allows two different variables to write to the same value. There are three operations that can be performed with references: assign by reference, pass by reference and return by reference.

Assign by reference

A reference is assigned by placing an ampersand (`&`) before the variable that is to be bound.

```
$x = 5;
$r = &$x; // r is a reference to x
$s =& $x; // alternative syntax
```

The reference then becomes an alias for that variable and can be used exactly as if it was the original variable.

```
$r = 10; // assign value to $r/$x
echo $x; // "10"
```

Pass by reference

In PHP, function arguments are by default passed by value. This means that a local copy of the variable is passed, so if the copy is changed it will not affect the original variable.

```
function myFunc($x) { $x .= ' World'; }

$x = 'Hello';
myFunc($x); // value of x is passed
echo $x;    // "Hello"
```

To allow a function to modify an argument it must be passed by reference. This is done by adding an ampersand before the parameter’s name in the function definition.

```
function myFunc(&$x) { $x .= ' World'; }

$x = 'Hello';
myFunc($x); // reference to x is passed
echo $x;    // "Hello World"
```

Object variables are also by default passed by value. However, what is actually passed is a pointer to the object data, not the data itself. Therefore, changes to the object’s members will affect the original object, but replacing the object variable with the assignment operator will only create a local variable.

```
class MyClass { public $x = 1; }

function modifyVal($o)
{
  $o->x = 5;
  $o = new MyClass(); // new local object
}

$o = new MyClass();
modifyVal($o);        // pointer to object is passed
echo $o->x;           // "5"
```

In contrast, when an object variable is passed by reference it is not only possible to change its properties, but also to replace the entire object and have the change propagate back to the original object variable.

```
class MyClass { public $x = 1; }

function modifyRef(&$o)
{
  $o->x = 5;
  $o = new MyClass(); // new object
}

$o = new MyClass();
modifyRef($o);        // reference to object is passed
echo $o->x;           // "1"
```

Return by reference

A variable can be assigned a reference from a function by having that function return by reference. The syntax for returning a reference is to place the ampersand before the function name. In contrast to pass by reference, the ampersand is also used when calling the function in order to bind the reference.

```
class MyClass
{
  public $val = 10;

  function &getVal()
  {
    return $this->val;
  }
}

$obj = new MyClass();
$myVal = &$obj->getVal();
```

Bear in mind that references should not be used merely for performance reasons, as the PHP engine will take care of such optimizations on its own. Only use references when you have a need for reference-type behavior.

CHAPTER 28

![image](images/frontdot.jpg)

Advanced Variables

In addition to being a container for data, PHP variables have some other features that will be examined in this chapter. These are features that are not commonly used but are good to know about.

Curly syntax

A variable name can be explicitly specified by enclosing it in curly brackets. This is known as curly or complex syntax. To illustrate, the following code will output the variable even though it appears in the middle of a word.

```
$fruit = 'Apple';
echo "Two {$fruit}s"; // "Two Apples"
```

More importantly, the curly syntax is useful for forming variable names out of expressions. Consider the following code which uses the curly syntax to construct names for three variables.

```
for ($i = 1; $i <= 3; $i++)
  ${'x'.$i} = $i;

echo "$x1 $x2 $x3"; // "1 2 3"
```

The curly syntax is required here because the expression needs to be evaluated in order to form a valid variable name. If the expression has only a single variable the curly brackets are not needed.

```
for ($i = 'a'; $i <= 'c'; $i++)
  $$i = $i;

echo "$a $b $c"; // "a b c"
```

This syntax is known as a variable variable in PHP.

Variable variable names

A variable variable is a variable whose name can be changed through code. As an example, consider the following regular variable.

```
$a = 'foo';
```

This variable’s value can be used as a variable name, by placing an additional dollar sign before it.

```
$$a = 'bar';
```

The value of `$a`, which is "foo", now becomes an alternative name for the variable `$$a`.

```
echo $foo; // "bar"
echo $$a;  // "bar"
```

An example usage for this would be to generate variables from an array.

```
$arr = array('a' => 'Foo', 'b' => 'Bar');

foreach ($arr as $key => $value)
{
  $$key = $value;
}

echo "$a $b"; // "Foo Bar"
```

Variable function names

By placing parentheses after a variable its value will be evaluated as the name for a function.

```
function myPrint($s) { echo $s; }

$func = 'myPrint';
$func('Hello'); // "Hello"
```

This behavior will not work with built-in language constructs, such as `echo`.

```
echo('Hello');  // "Hello"

$func = 'echo';
$func('Hello'); // error
```

Variable class names

Similar to variable function names, classes can be referenced using string variables. This functionality was introduced in PHP 5.3.

```
class MyClass {}

$classname = 'MyClass';
$obj = new $classname();
```

The mechanism of accessing code entities via strings and string variables also works for members of a class or instance.

```
class MyClass
{
  public $myProperty = 10;
}

$obj = new MyClass();
echo $obj->{'myProperty'}; // "10"
```

CHAPTER 29

![image](images/frontdot.jpg)

Error Handling

An error is a mistake in the code that the developer needs to fix. When an error occurs in PHP the default behavior is to display the error message in the browser. This message includes the filename, line number and error description in order to help the developer correct the problem.

While compile and parse errors are typically easy to detect and fix, run-time errors can be harder to find as they may only occur in certain situations and for reasons beyond the developer’s control. Consider the following code that attempts to open a file for reading using the `fopen` function.

```
$handle = fopen('myfile.txt', 'r');
```

It relies on the assumption that the requested file will always be there. If, for any reason, the file is not there or is otherwise inaccessible the function will generate an error.

```
Warning: fopen(myfile.txt):
failed to open stream: No such file or directory in
C:\xampp\htdocs\mypage.php on line 2
```

Once an error has been detected it should be corrected, even if it only occurs in exceptional situations.

Correcting errors

There are two ways to correct this error. The first way is to check to make sure that the file can be read before attempting to open it. PHP conveniently provides the `is_readable` function for this task, which returns true if the specified file exists and is readable.

```
if (is_readable('myfile.txt'))
  $handle = fopen('myfile.txt', 'r');
```

The second way is to use the error control operator (`@`). When prepended to an expression this operator will suppress any error messages that might be generated by that expression. Either way works to remove the warning.

```
$handle = @fopen('myfile.txt', 'r');
```

To determine if the file was opened successfully the return value needs to be examined. Looking at the documentation,[¹](#9781430262831_Ch29.xhtml#Fn1)   you can find that `fopen` returns false on error.

```
if ($handle === false)
{
  echo 'File not found.';
}
```

If this is not the case then the content of the file can be read with the `fread` function. This function reads the number of bytes specified in the second argument from the file handle given in the first argument.

```
else
{
  // Read the content of the whole file
  $content = fread($handle, filesize('myfile.txt'));

  // Close the file handler
  fclose($handle);
}
```

Once the file handler is no longer needed it is good practice to close it with `fclose` , although PHP will also automatically close the file after the script has finished.

Error levels

PHP provides several built-in constants for describing different error levels. The table below includes some of the more important ones.

| Name | Description |
| --- | --- |
| E_ERROR | Fatal run-time error. Execution is halted. |
| E_WARNING | Non-fatal run-time error. |
| E_NOTICE | Run-time notice about possible error. |
| E_USER_ERROR | Fatal user-generated error. |
| E_USER_WARNING | Non-fatal user-generated warning. |
| E_USER_NOTICE | User-generated notice. |
| E_COMPILE_ERROR | Fatal compile-time error. |
| E_PARSE | Compile-time parsing error. |
| E_STRICT | Suggested change to ensure forward compatibility. |
| E_ALL | All errors, except E_STRICT prior to PHP 5.4. |

The first three of these levels represent run-time errors generated by the PHP engine. Some examples of operations that trigger these errors are given below.

```
// E_NOTICE – Use of unassigned variable
$a = $x;

// E_WARNING – Missing file
$b = fopen('missing.txt', 'r');

// E_ERROR – Missing function
$c = missing();
```

Error handling environment

PHP provides a few configuration directives for setting up the error handling environment. The `error_reporting` function sets which errors PHP will report through the internal error handler. The error level constants have bit mask values. This allows them to be combined and subtracted using bitwise operators, as shown here.

```
error_reporting(E_ALL | ∼E_NOTICE); // all but E_NOTICE
```

The error reporting level can also be changed permanently in php.ini. The default value found in php.ini varies between servers, but for an XAMPP server it is set to display all error messages. This is a good setting to have during development and it can be set programmatically by placing the following line of code at the start of the script. Note that E_STRICT is added, as this error level was not included in E_ALL until PHP 5.4.

```
// During development
error_reporting(E_ALL | E_STRICT);
```

When the web app goes live raw error messages should be hidden away from users. This can be done with the `display_errors` directive. It determines whether errors are printed to the web page by the internal error handler. The default value is to print them, but when the web site is live it is a good idea to hide any potential raw error messages.

```
// During production
ini_set('display_errors','0');
```

Another directive related to the error handling environment is the `log_errors` directive. It sets whether error messages will be recorded in the server's error log. This directive is disabled by default, but it is a good idea to enable it during development to keep track of errors.

```
// During development
ini_set('log_errors','1');
```

The `ini_set` function sets the value of a configuration option. Alternatively, these options can all be permanently set in the php.ini configuration file instead of in the script files.

Custom error handlers

The internal error handler can be overridden with a custom error handler. This is the preferred method for handling errors as it allows you to abstract the raw errors with friendly, custom error messages to the end users.

A custom error handler is defined using the `set_error_handler` function. This function accepts two arguments: a callback function that will be called when the error is raised, and optionally the error levels that the function will handle.

```
set_error_handler('myError', E_ALL | E_STRICT);
```

If no error levels are specified the error handler will be set to handle all errors, including E_STRICT. However, a user-defined error handler will only actually be able to handle run-time errors, and only run-time errors other than E_ERROR. Keep in mind that changes to the `error_reporting` setting will not affect the custom error handler, only the internal one.

The callback function requires two parameters: the error level and error description. Optional parameters include the file name, line number and error context, which is an array containing every variable in the scope the error was triggered in.

```
function myError($errlvl, $errdesc, $errfile, $errline)
{
  switch($errlvl)
  {
    case E_USER_ERROR:
      error_log("Error: $errdesc", 1, 'me@example.com');
      require_once('my_error_page.php');
      return true;
  }
  return false;
}
```

This example function handles errors of level E_USER_ERROR. When such an error occurs an email is sent to the specified address and a custom error page is displayed. By returning false from the function for other errors they will be handled by the internal error handler instead.

Raising errors

PHP provides the `trigger_error` function for raising errors. It has one required argument, the error message, and a second optional argument specifying the error level. The error level must be one of the three E_USER levels, with E_USER_NOTICE being the default level.

```
if( !isset($myVar) )
  trigger_error('$myVar not set'); // E_USER_NOTICE
```

Triggering errors is useful when you have a custom error handler in place, allowing you to combine the handling of both custom errors and errors raised by PHP.

[¹](#9781430262831_Ch29.xhtml#_Fn1)`http://www.php.net/manual/en/function.fopen.php`

CHAPTER 30

![image](images/frontdot.jpg)

Exception Handling

PHP 5 introduced exceptions, a built-in mechanism for handling program failures within the context in which they occur. Unlike errors, which generally need to be fixed by the developer, exceptions are handled by the script. They represent an irregular run-time situation that should have been expected as a possibility, and which the script should be able to handle on its own.

Throwing exceptions

When a situation occurs that a function cannot recover from it can generate an exception to signal to the caller that the function has failed. This is done using the `throw` keyword followed by a new instance of the Exception class or a child class of `Exception`, such as `LogicException`.[¹](#9781430262831_Ch30.xhtml#Fn1)

```
function invert($x)
{
  if ($x == 0)
    throw new LogicException('Division by zero');

  return 1 / $x;
}
```

Try-catch statement

To handle an exception it must be caught using a try-catch statement. This statement consists of a `try` block containing the code that may cause the exception, and one or more catch clauses.

```
try
{
  $div = invert(0);
}
catch (LogicException $e) {}
```

If the `try` block successfully executes, the program will then continue running after the try-catch statement. However, if an exception occurs the execution will then be passed to the first `catch` block able to handle that exception type.

Catch block

In the previous example, the catch block is set to handle the built-in `LogicException` type. If the code in the `try` block could throw more kinds of exceptions, multiple `catch` blocks can be used, allowing different exceptions to be handled in different ways.

```
catch (LogicException $e) {}
catch (RuntimeException $e) {}
// ...
```

To catch a more specific exception the `catch` block needs to be placed before more general exceptions. For example, the `LogicException` inherits from `Exception`, so the `LogicException` needs to be caught first.

```
catch (LogicException $e) {}
catch (Exception $e) {}
```

The `catch` clause defines an exception object. This object can be used to obtain more information about the exception, such as a description of the exception using the `getMessage` method.

```
catch (LogicException $e)
{
  echo $e->getMessage(); // "Division by zero"
}
```

Finally block

PHP 5.5 introduced the `finally` block, which can be added as the last clause in a try-catch statement. This block is used to clean up resources allocated in the `try` block and will always execute whether or not there is an exception.

```
$resource = myopen();
try { myuse($resource); }
catch(Exception $e) {}
finally { myfree($resource); }
```

Re-throwing exceptions

Sometimes an exception cannot be handled where it is first caught. It can then be re-thrown using the `throw` keyword followed by the exception object.

```
try { $div = invert(0); }
catch (LogicException $e) { throw $e; }
```

The exception will then propagate up the caller stack until it is caught by another try-catch statement. If the exception is never caught it will become an error of level E_ERROR which halts the script, unless an uncaught exception handler has been defined.

Uncaught exception handler

The `set_exception_handler` function allows any uncaught exceptions to be caught. It takes a single argument, which is the callback function that will be raised for such an event.

```
set_exception_handler('myException');
```

The callback function only needs one parameter, the exception object that was thrown.

```
function myException($e)
{
  $file = 'exceptionlog.txt';
  file_put_contents($file,$e->getMessage(),FILE_APPEND);
  require_once('my_error_page.php');
  exit;
}
```

As this exception handler is called outside the context where the exception occurred, recovering from the exception would be difficult. Instead, this example handler writes the exception to a log file and displays an error page. To stop further execution of the script the built-in `exit` construct is used. It is synonymous with the `die` construct and optionally takes a string argument that will be printed before the script is halted.

Errors and exceptions

Whereas exceptions are thrown with the intention of being handled by the script, errors are generated for the purpose of informing the developer that there is a mistake in the code. When it comes to problems that occur at run-time, the exception mechanism is generally considered superior. However, since it was not introduced until PHP 5 all internal functions still use the error mechanism. For user-defined functions, the developer is free to choose either mechanism. Keep in mind that errors cannot be caught by try-catch statements. Likewise, exceptions do not trigger error handlers.

[¹](#9781430262831_Ch30.xhtml#_Fn1)`http://www.php.net/manual/en/spl.exceptions.php`

Index

![images](images/square1.jpg)  A

[abstract](#9781430262831_Ch15.xhtml#cXXX.97)

[Access levels](#9781430262831_Ch11.xhtml#cXXX.81)

[Anonymous functions](#9781430262831_Ch08.xhtml#cXXX.64)

[Arrays](#9781430262831_Ch05.xhtml#cXXX.35)

[assignment operator (=)](#9781430262831_Ch03.xhtml#cXXX.22)

[Associative arrays](#9781430262831_Ch05.xhtml#cXXX.37)

[__auto load](#9781430262831_Ch17.xhtml#cXXX.65a)

![images](images/square1.jpg)  B

[bool](#9781430262831_Ch02.xhtml#cXXX.17)

[break](#9781430262831_Ch06.xhtml#cXXX.42)

![images](images/square1.jpg)  C

[catch](#9781430262831_Ch30.xhtml#cXXX.154)

[class](#9781430262831_Ch09.xhtml#cXXX.65)

[class member](#9781430262831_Ch12.xhtml#cXXX.88)

[__clone](#9781430262831_Ch22.xhtml#cXXX.123)

[comma operator (,)](#9781430262831_Ch07.xhtml#cXXX.52)

[Comments](#9781430262831_Ch01.xhtml#cXXX.8)

[Compile](#9781430262831_Ch01.xhtml#cXXX.6)

[concatenation operator (.)](#9781430262831_Ch04.xhtml#cXXX.31)

[Conditionals](#9781430262831_Ch06.xhtml#cXXX.39)

[const](#9781430262831_Ch13.xhtml#cXXX.94)

[Constants](#9781430262831_Ch13.xhtml#cXXX.93)

[Constructor](#9781430262831_Ch09.xhtml#cXXX.72)

[continue](#9781430262831_Ch07.xhtml#cXXX.56)

[Cookies](#9781430262831_Ch24.xhtml#cXXX.131)

![images](images/square1.jpg)  D

[Data types](#9781430262831_Ch02.xhtml#cXXX.13)

[decrement operator (  )](#9781430262831_Ch03.xhtml#cXXX.24)

[Default parameters](#9781430262831_Ch08.xhtml#cXXX.59)

[Default values](#9781430262831_Ch02.xhtml#cXXX.19)

[define](#9781430262831_Ch02.xhtml#cXXX.11)

[defined](#9781430262831_Ch13.xhtml#cXXX.95)

[Destructor](#9781430262831_Ch09.xhtml#cXXX.73)

[die](#9781430262831_Ch30.xhtml#cXXX.158)

[display_errors](#9781430262831_Ch29.xhtml#cXXX.147)

[double arrow operator (=>)](#9781430262831_Ch07.xhtml#cXXX.55)

[do-while](#9781430262831_Ch07.xhtml#cXXX.50)

![images](images/square1.jpg)  E

[echo](#9781430262831_Ch01.xhtml#cXXX.3)

[empty](#9781430262831_Ch20.xhtml#cXXX.111)

[Enclosing class](#9781430262831_Ch11.xhtml#cXXX.83)

[error control operator (@)](#9781430262831_Ch29.xhtml#cXXX.143)

[Error handling](#9781430262831_Ch29.xhtml#cXXX.140)

[error_reporting](#9781430262831_Ch29.xhtml#cXXX.146)

[Escape characters](#9781430262831_Ch04.xhtml#cXXX.34)

[eval](#9781430262831_Ch20.xhtml#cXXX.114)

[exit](#9781430262831_Ch30.xhtml#cXXX.157)

[Explicit cast](#9781430262831_Ch19.xhtml#cXXX.106)

[Expression](#9781430262831_Ch06.xhtml#cXXX.45)

[extends](#9781430262831_Ch10.xhtml#cXXX.76)

![images](images/square1.jpg)  F

[false](#9781430262831_Ch03.xhtml#cXXX.26)

[fclose](#9781430262831_Ch29.xhtml#cXXX.145)

[$_FILES](#9781430262831_Ch23.xhtml#cXXX.128)

[final](#9781430262831_Ch10.xhtml#cXXX.79)

[finally](#9781430262831_Ch30.xhtml#cXXX.155)

[float](#9781430262831_Ch02.xhtml#cXXX.16)

[fopen](#9781430262831_Ch29.xhtml#cXXX.141)

[for](#9781430262831_Ch07.xhtml#cXXX.51)

[foreach](#9781430262831_Ch07.xhtml#cXXX.54)

[fread](#9781430262831_Ch29.xhtml#cXXX.144)

[Functions](#9781430262831_Ch08.xhtml#cXXX.58)

![images](images/square1.jpg)  G

[Garbage collector](#9781430262831_Ch09.xhtml#cXXX.74)

[$_GET](#9781430262831_Ch23.xhtml#cXXX.126)

[gettype](#9781430262831_Ch19.xhtml#cXXX.108)

[global](#9781430262831_Ch08.xhtml#cXXX.62)

[global prefix operator (\)](#9781430262831_Ch26.xhtml#cXXX.137)

[$GLOBALS](#9781430262831_Ch08.xhtml#cXXX.63)

[goto](#9781430262831_Ch07.xhtml#cXXX.57)

![images](images/square1.jpg)  H

[heredoc](#9781430262831_Ch04.xhtml#cXXX.32)

[HTML mode](#9781430262831_Ch01.xhtml#cXXX.2)

![images](images/square1.jpg)  I, J, K

[Identifier](#9781430262831_Ch02.xhtml#cXXX.10)

[if](#9781430262831_Ch06.xhtml#cXXX.40)

[include](#9781430262831_Ch17.xhtml#cXXX.99)

[include_once](#9781430262831_Ch17.xhtml#cXXX.101)

[increment operator (++)](#9781430262831_Ch03.xhtml#cXXX.23)

[Inheritance](#9781430262831_Ch10.xhtml#cXXX.75)

[ini_set](#9781430262831_Ch29.xhtml#cXXX.149)

[Initialize](#9781430262831_Ch02.xhtml#cXXX.12)

[Instance member](#9781430262831_Ch12.xhtml#cXXX.89)

[instanceof](#9781430262831_Ch10.xhtml#cXXX.80)

[Integer](#9781430262831_Ch02.xhtml#cXXX.15)

[interface](#9781430262831_Ch14.xhtml#cXXX.96)

[__invoke](#9781430262831_Ch22.xhtml#cXXX.118)

[is_null](#9781430262831_Ch20.xhtml#cXXX.112)

[is_readable](#9781430262831_Ch29.xhtml#cXXX.142)

[isset](#9781430262831_Ch20.xhtml#cXXX.110)

[Iteration](#9781430262831_Ch07.xhtml#cXXX.49)

![images](images/square1.jpg)  L

[Late static bindings](#9781430262831_Ch12.xhtml#cXXX.92)

[log_errors](#9781430262831_Ch29.xhtml#cXXX.148)

[logical and (&&)](#9781430262831_Ch03.xhtml#cXXX.27)

[logical not (!)](#9781430262831_Ch03.xhtml#cXXX.29)

[logical or (||)](#9781430262831_Ch03.xhtml#cXXX.28)

[Loops](#9781430262831_Ch07.xhtml#cXXX.47)

[Loose typing](#9781430262831_Ch02.xhtml#cXXX.14)

![images](images/square1.jpg)  M

[Magic methods](#9781430262831_Ch22.xhtml#cXXX.116)

[Method](#9781430262831_Ch09.xhtml#cXXX.68)

[Mixed modes](#9781430262831_Ch06.xhtml#cXXX.43)

[modulus operator (%)](#9781430262831_Ch03.xhtml#cXXX.21)

[Multi-dimensional arrays](#9781430262831_Ch05.xhtml#cXXX.38)

![images](images/square1.jpg)  N

[namespace](#9781430262831_Ch26.xhtml#cXXX.138)

[new](#9781430262831_Ch09.xhtml#cXXX.71)

[nowdoc](#9781430262831_Ch04.xhtml#cXXX.33)

[null](#9781430262831_Ch02.xhtml#cXXX.18)

[Numeric arrays](#9781430262831_Ch05.xhtml#cXXX.36)

![images](images/square1.jpg)  O

[Object](#9781430262831_Ch09.xhtml#cXXX.66)

[Operators](#9781430262831_Ch03.xhtml#cXXX.20)

[Overloading](#9781430262831_Ch21.xhtml#cXXX.115)

[Overriding](#9781430262831_Ch10.xhtml#cXXX.77)

![images](images/square1.jpg)  P, Q

[Parse](#9781430262831_Ch01.xhtml#cXXX.7)

[phpinfo](#9781430262831_Ch23.xhtml#cXXX.130)

[PHP mode](#9781430262831_Ch01.xhtml#cXXX.1)

[$_POST](#9781430262831_Ch23.xhtml#cXXX.125)

[print](#9781430262831_Ch01.xhtml#cXXX.4)

[private](#9781430262831_Ch11.xhtml#cXXX.82)

[property](#9781430262831_Ch09.xhtml#cXXX.67)

[protected](#9781430262831_Ch11.xhtml#cXXX.84)

[public](#9781430262831_Ch11.xhtml#cXXX.85)

![images](images/square1.jpg)  R

[References](#9781430262831_Ch27.xhtml#cXXX.139)

[$_REQUEST](#9781430262831_Ch23.xhtml#cXXX.127)

[require](#9781430262831_Ch17.xhtml#cXXX.100)

[require_once](#9781430262831_Ch17.xhtml#cXXX.102)

[Return](#9781430262831_Ch08.xhtml#cXXX.60)

![images](images/square1.jpg)  S

[Scope](#9781430262831_Ch08.xhtml#cXXX.61)

[scope resolution operator (::)](#9781430262831_Ch10.xhtml#cXXX.40a)

[self](#9781430262831_Ch12.xhtml#cXXX.90)

[semicolon (;)](#9781430262831_Ch01.xhtml#cXXX.2a)

[session(s)](#9781430262831_Ch25.xhtml#cXXX.95a)

[session_destroy](#9781430262831_Ch25.xhtml#cXXX.135)

[session_start](#9781430262831_Ch25.xhtml#cXXX.134)

[setcookie](#9781430262831_Ch24.xhtml#cXXX.132)

[set_error_handler](#9781430262831_Ch29.xhtml#cXXX.150)

[set_exception_handler](#9781430262831_Ch30.xhtml#cXXX.156)

[__set_state](#9781430262831_Ch22.xhtml#cXXX.121)

[settype](#9781430262831_Ch19.xhtml#cXXX.107)

[single arrow operator (−>)](#9781430262831_Ch09.xhtml#cXXX.35a)

[sizeof](#9781430262831_Ch07.xhtml#cXXX.53)

[__sleep](#9781430262831_Ch22.xhtml#cXXX.119)

[Statement](#9781430262831_Ch06.xhtml#cXXX.46)

[static](#9781430262831_Ch12.xhtml#cXXX.87)

[static variables](#9781430262831_Ch12.xhtml#cXXX.91)

[String](#9781430262831_Ch04.xhtml#cXXX.30)

[Superglobals](#9781430262831_Ch23.xhtml#cXXX.129)

[switch](#9781430262831_Ch06.xhtml#cXXX.41)

![images](images/square1.jpg)  T

[ternary operator (?:)](#9781430262831_Ch06.xhtml#cXXX.23a)

[$this](#9781430262831_Ch09.xhtml#cXXX.69)

[throw](#9781430262831_Ch30.xhtml#cXXX.152)

[__toString](#9781430262831_Ch22.xhtml#cXXX.117)

[Traits](#9781430262831_Ch16.xhtml#cXXX.98)

[trigger_error](#9781430262831_Ch29.xhtml#cXXX.151)

[true](#9781430262831_Ch03.xhtml#cXXX.25)

[try](#9781430262831_Ch30.xhtml#cXXX.153)

[Type conversions](#9781430262831_Ch19.xhtml#cXXX.105)

[Type hinting](#9781430262831_Ch18.xhtml#cXXX.104)

![images](images/square1.jpg)  U

[unset](#9781430262831_Ch20.xhtml#cXXX.113)

[User input](#9781430262831_Ch23.xhtml#cXXX.124)

![images](images/square1.jpg)  V

[var](#9781430262831_Ch11.xhtml#cXXX.86)

[Variable](#9781430262831_Ch02.xhtml#cXXX.9)

[Variable testing](#9781430262831_Ch20.xhtml#cXXX.109)

![images](images/square1.jpg)  W, X, Y, Z

[__wakeup](#9781430262831_Ch22.xhtml#cXXX.120)

[while](#9781430262831_Ch07.xhtml#cXXX.48)