---
title: How to Distribute a Rust Binary on NPM
image:
imageMeta:
  attribution:
  attributionLink:
featured: true
authors: 
  - kenneth-larsen
date: Mon Sep 30 2019 20:31:28 GMT+0200 (Central European Summer Time)
tags:
  - learning-rust
---

Recently I published a CLI tool called [baelte](https://github.com/kennethlarsen/baelte). It's written in Rust, and it's aimed at frontend developers who make applications using Svelte. I wanted to write it in Rust as a learning project but also to utilize the performance benefits which Rust provides over Node. 

When it came to building and releasing it, I was without a clue on how to do it. It seemed that I could push it to Cargo, but to me, that was not a good solution. First of all, Cargo seems like a registry for packages used by other Rust packages and not for user faced applications. Secondly, my target group most likely haven't installed Rust and doesn't know what Cargo is. I don't think they should need to know about an entirely different ecosystem to install my tool. Since they are frontend developers, the tool would need to be installable via `npm install`.

But how can I ship a Rust tool on NPM? Let's find out!

## Building

Once my tool was ready to be build and shipped, I looked at what my options were. I wanted something that could run automatically from Github so I shouldn't spend too much time on building locally for several operating systems.

Thankfully, there is a project called [trust](https://github.com/japaric/trust) that saved my day. In the trust repository, they have build configurations ready for you to copy and paste. They make Travis build and test for several operating systems when you create a pull request in your repository.

And even better, when you push a tag to your repository it will trigger a build for several operating systems, create a release on Github and push the binaries as downloads for your users to enjoy.

This is really, really awesome. But, I don't want my users to go to a release page on Github and download the binaries themselves. Trust has a small script that can install the latest release for you:

```bash
curl -LSfs https://japaric.github.io/trust/install.sh | \
    sh -s -- --git kennethlarsen/baelte
```

While this is great, then I still don't want my users to do this since this is out of their normal context.

## Installing via NPM
https://github.com/kennethlarsen/baelte-npm
Frontend developers expect packages and tools to be installable with `npm install`. Since my tool is written in Rust and not JavaScript that makes things difficult. 

I decided to create a JavaScript which execudes the install script previously mentioned:

```js
let exec = require('child_process').exec;

exec('curl -LSfs https://japaric.github.io/trust/install.sh | \
sh -s -- --git kennethlarsen/baelte', (error, stdout, stderr) => {
  console.log(stderr);
});
```

Then in `package.json` I added a `postinstall` like this:

```json
"postinstall": "npm run install",
```

which triggers:

```json
"scripts": {
    "install": "node install.js"
  },
```

And just like that, it is possible to publish your Rust CLI tool to npm. Users can now download my tool with `npm install -g baelte`.
