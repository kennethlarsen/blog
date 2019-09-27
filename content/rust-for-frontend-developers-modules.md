---
title: 'Rust for Frontend Developers: Modules'
image: ''
authors:
  - kenneth-larsen
date: '2019-04-30T08:43:53.000Z'
tags:
  - learning-rust
excerpt: 'Learning Rust has been great, but there are some things that are still weird to me.'
---
Not that long ago I decided to start learning Rust. While it has a lot of useful
resources online and a very friendly community there's still things that are
weird to me. That's because Rust is a very different mental model than the
frontend mental model I'm used to. 


That means that things that are obvious to a lot of people are not obvious to
me. So I'll try to document some of these things from the perspective of a
frontend developer. This time on using modules.


--------------------------------------------------------------------------------

With Javascript using imports or modules of any kind is fairly straight forward.
If you have a very basic HTML page with a couple of JavaScript files you can add
them all with a script tag. 

If you have a more fancy setup you can import JavaScript files (or modules) like
this:

```js
require('cool-module')
```
or even 

```js
import { function } from 'cool-module'
```

If you use a framework like Ember.js there's a standard of separating code that
also goes beyond these module imports. Components are isolated chunks of code
that can be invoked like this: 

```js
<CoolComponent @arg="wow" />
```

The point is that there is a lot of ways of separating code on the frontend to
make sure your codebase is manageable. 

## Modules with Rust
While trying to make my Rust clone of [Ember-CLI](https://github.com/kennethlarsen/rember-cli) I quickly discovered that the 
`main()` function was getting out of hand. I wanted to split the code into
separate files to make sure that I could maintain all of the features I wanted
to add.

There are a few things to keep in mind here.

## Where's my npm install?
The good news is that there is an equivalent system in place for distributing
and using packages.  In the world of Rust that's called [Crates](https://crates.io/). Crates is the registry of the Rust community. To consume
packages from this registry or publishing you have to use the cargo package
manager. 

If you want a package installed globally like `npm install -g cool-module`  then
you'll run `cargo install cool-module`.

But what if you want to install a package for your project? I'm honestly not
sure if there is an equivalent to `npm install --save-dev cool-module` since all
documentation is proposing something different. 

The documented approach is to add the package name and version to the project's 
`Cargo.toml`. This file is the equivalent of `package.json`. Then when you build
your project it will fetch the dependencies. 

Now you have to remember to import it where you need it like this:

```rust
use package_name::module_name{function_name}.
```

## Separating Concerns
Back to my main issue. I wanted to split out the code into different modules
that I could import when I needed it.

My project is currently like this:

 * `main.rs` containing the basics to run the CLI tool and process input from the
   user.
 * `new.rs` containing everything related to the `new` command for generating new
   Ember projects.
 * `utils.rs` for utility-like function such as creating a progress bar and
   replacing placeholder values with user inputted names.

First I saw that it was possible to use the same import style from the package
manager but for local files. I wanted to use the `create_new_application()`
function from `new.rs`.

In `main.rs` I tried to use `rember::new::{create_new_application}`

But quickly got an error saying that `rember::new` wasn't found.

It turns out that these internal modules have to be declared first. Since I was
building a CLI tool they had to go in a file named `lib.rs` like this:

```rust
mod new;
```

This made the `new.rs` available for me to use in `main.rs`.

Now I had another problem. I wanted to use a function from `utils.rs` in `new.rs` 
and just assumed that I had to use the exact same approach. But now I got the
error with module not found.

I still don't quite understand why but it looks like that for modules not
imported in `main.rs` they have to use `super` instead of the package name `rember`.
So in `new.rs` it looks like this:

```rust
use super::utils::{create_progress_bar};
```

instead of

```rust 
use rember::utils::{create_progress_bar};
```
