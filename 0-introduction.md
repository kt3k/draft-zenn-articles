# 0. Introduction

If your have no idea what programming is, then this book is for you! If you're an experienced programmer, interested in Deno specific topics, I recommend you should read the official manual of Deno at https://deno.land/manual.

In this book, we explain how to use Deno, and how to do programming with Deno. We don't assume any knowledge about programming.

## What is Deno

Deno is a kind of programming languages. There are many programming languages out there like C, C++, Java, Python, Go, Rust, etc. Deno is one of them.

Among them, Deno is easy to learn, easy to start with, but still powerful enough to do tough works like web server. Deno is a very much suitable one for starting the programming with.

I said Deno is a kind of programming language, but that's technically a little inaccurate. The language which Deno handles is called JavaScript and TypeScript, and Deno is an engine to run these languages.

## History of JavaScript, Node.js, and Deno

JavaScript was invented as a launguage which ran in a browser in 1995. It wasn't used very much at that time because people didn't see the needs of such thing, didn't understand what it could do.

After almost a decade, Google invented GMail and Google Maps. These are signifant applications which ran in browser without any installation. These 2 apps use JavaScript as their cores. People now understood the potential of JavaScript, and started investing much time and money to it.

In 2008, Google invented Google Chrome, a new web browser, and V8 engine, a very fast JavaScript engine.

Next year Ryan Dahl invented Node.js, which was the JavaScript engine, which utilizes V8 engine at its core and ran in the servers, not in the browsers. This was a big hit. A huge amount of software were written using Node.js. The registry of Node.js programs, which was called npm registry, had the largest number of packages among all the programming langauges. Despite the enormous success of Node.js, Ryan stepped down as a leader of the project in 2012.

Deno was invented again by Ryan Dahl in 2018. Though Node.js got significant popularity, it had a few flaws like modules, security models, etc. Deno was an improvement of these shortcomings of Node.js.
