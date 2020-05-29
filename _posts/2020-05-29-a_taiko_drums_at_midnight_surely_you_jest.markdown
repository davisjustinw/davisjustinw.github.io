---
layout: post
title:      "A Taiko Drums at Midnight. Surely You Jest."
date:       2020-05-29 09:15:50 +0000
permalink:  a_taiko_drums_at_midnight_surely_you_jest
---

## A Case For Testing with Taiko, Create-React-App, and Jest
Today I completed my React final assessment. This post stands between me and graduation. Let's talk about something I should have done during my project.

Writing tests is one of those murky shoulds that we know is good for us. Testing good. Yes, we should do tests.  Like exercise and vegetables. Then kids, life, work deadlines, and global pandemics happen.

It's so easy to power through and just code. It's so easy to believe writing tests will slow progress.


### Decrease Cognitive Load
Yes, we are perfectly capable of coding without the 'crutch' of testing. Why spend time writing tests when you can throw some console.logs in, click around and move on to the next feature? 

Two words, cognitive load. The reason I should have built tests for my React portfolio project.

Without automated tests, every layer of complexity added to an application requires the engineer to imagine what that change may adversely affect. This continues to snowball with complexity and fatigue.

With tests, the engineer can charge forward comforted in the fact that tests will detect unintended consequences of new changes, freeing brain power for the next challenge.

I experimented with unit tests on my first portfolio project. While some tests became obsolete as the project evolved. Some tests endured and saved my bacon more the once. There's just something comforting about red, green, refactor. Funny, I didn't learn my lesson. 

The key I think, is the practice of writing tests develops the instinct to write better tests, code, and ultimately architecture.


### Validations Gone Wrong
My React project, CookTree, helps users share and evolve family recipes. It allows users to create connections with 'users' that don't necessarily login. The user model might require one set of validations upon signup, but a different set when creating connected users without logins.

Early on I established field level validations for signup and moved on. Later I built the ability to add and invite users. In so doing, I shut down model level validations in order to get the new function to work. I wrote myself some comments to come back and flesh out the validations. I noted it in my backlog. Somehow it still slipped my mind.

Fast forward, I got the app to a point where I liked it, recorded my video, and submitted the project. I had some time between submission and my review, so I casually did some refactoring. I had some job coach todos, a busy work schedule and a couple job interviews in that time as well.  

A couple nights before my review I realized there was some functionality I really wanted to add. I dove in, confident that I just had some light refactoring after that. As I dug into that functionality, I started to expose some holes. Validations I swore I had, functionality that I'm pretty sure I had checked off. There's no way I would have let that internal server error go this long. What is happening?  Enter, fatigue and imposter syndrome. I got that functionality going and plugged the holes, but I chewed up quite a bit of time.

On the last night before my review, at 7:00 am, I embarked on my last 'light refactoring'. Things went pretty well until about 9:00 pm. I worked on those user validations and somehow broke something, bad, and in a confusing way. It took me 3 hours and ultimately a git restore to make my project work again.  

There's a special panic to having a broken project at midnight 7 hours before one's review. I didn't complete all the validations I wanted. I pretty sure if I implemented even some light automated testing early on, I would have avoided the heartache.  

So I'm learning some testing. And I'm going to get those validations right.


## Enter Taiko and Jest
Create-React-App comes packaged with the test runner Jest. Taiko is a Javascript based browser automation tool. Taiko and Jest work well together.

I'm going to use Taiko and Jest to set up tests for my login and signup sequences. The signup sequence has those pesky validations I want to fix. For this post we will just discuss setup and the simple login logout sequence.


### Intall Taiko REPL
This allows you to use the Taiko REPL for coding flows. You can run commands that drive the application in browser, then save the sequence off to a Javascript file. Those commands can then be added to a test file that uses Jest. I find this really cool for experimenting with the Taiko API.

`$ npm install -g taiko --unsafe-perm --allow-root`

Some folks can get away with the install command without the --unsafe-perm and --allow-root tags. My machine required those to work.  The flags were recommended in the [Taiko troubleshooting guide](https://docs.taiko.dev/#global-installation)


### Install Taiko in the project.
This threw me for a loop at first.  I tried to include Taiko in my project test but it didn't work. There may be a better way to use a globally installed package in a project.  I'm still learning the subtleties of npm.

In your project directory.


`$ npm install taiko`


### Experiment with the REPL
My goal is to run through the following sequence in the REPL.

  1. Open the browser
  2. Navigate to CookTree running locally
  3. Fill in credentials
  4. Click 'Login'
  5. Open Menu Drawer
  6. Click More Icon
  7. Click Logout


![login](https://live.staticflickr.com/65535/49947943456_a26030f9a2_c.jpg)


```
$ taiko
$ openBrowser()
$ goto('localhost:3000')
```


Pretty straightforward, start up Taiko, open the browser, and navigate. Taiko calls these browser and page actions in their [API](https://docs.taiko.dev/).


```
$ write('one@two.com', into(textBox('Email Address')))
$ write('password', into(textBox('Password')))
$ click(button('Login'))
```


This block writes the text into the inputs and clicks submit.  `write()` and `click()` are page actions.  `into()` is a helper for placing the text. `textBox()` selects for the location. In both `textBox()` selectors, we're telling Taiko to look for inputs with those labels. Taiko has an variety of selectors available in the documentation.


```
$ click(button(toLeftOf('Cook Tree')))
$ click(button(toRightOf('justin')))
$ click('Log out')
```


![logout](https://live.staticflickr.com/65535/49947943426_aec0e7628f_c.jpg)


Weee! These are cool. Taiko calls them proximity selectors. For the first two clicks, we are aiming at icon buttons. Now we could inspect the code and find an id or some other way to use traditional css selectors. But I used Material-ui so there is some freaky DOM elements. So Taiko makes life easy for us to just say hey click that thing next to that thing.

Notice for the last click I didn't identify button. That's because there is only one thing on the screen that says 'Log out'. For the login sequence, there is a title element that says Login.  So if we used just `click('Login')`, Taiko clicks the title.


```
$ .code superCoolTest.js
$ .exit
```


These commands save off successful commands to a javascript file that look like the following. Notice Taiko adds what methods you need in the require statement.


```
const { openBrowser, goto, textBox, into, write, button, click, closeBrowser } = require('taiko');
(async () => {
    try {
        await openBrowser();
        await goto('localhost:3000');
        await write('one@two.com', into(textBox('Email Address')));
        await write('password', into(textBox('Password')));
        await click(button('Login'));
    } catch (error) {
        console.error(error);
    } finally {
        await closeBrowser();
    }
})();
```


## Jest
Now lets build some tests with our automation sequence. Create-react-app comes bundled with Jest. If you look at you scripts in package.json there is a test script. That fires Jest.  

Jest will look for certain [naming conventions](https://create-react-app.dev/docs/running-tests/#filename-conventions). I chose to create folder `__tests__`. Jest will run anything `*.js` in there.

To set up my super simple test, I used [this example](https://github.com/getgauge-examples/jest-taiko) as a starting point.


```
import {
  openBrowser,
  goto,
  textBox,
  text,
  into,
  write,
  button,
  click,
  toLeftOf,
  toRightOf,
  closeBrowser,
  waitFor
} from 'taiko';

// Set a longer timeout than default 5000 milliseconds
// tests that take longer than the timeout fail.
jest.setTimeout(30000)

describe('Login and Logout', () => {

    // headless set to false lets Jest open up the browser
    beforeAll(async () => {
        await openBrowser({ headless: false });
    });

    describe('Login', () => {

      test('navigate to login', async () => {
        await goto('http://localhost:3000');
        await expect(text('Login').exists()).toBeTruthy();
      });

      test('Valid credentials logs into connections', async () => {
        await write('one@two.com', into(textBox('Email Address')));
        await write('password', into(textBox('Password')));
        await click(button('Login'));
        await expect(text('Connetions').exists()).toBeTruthy();
      });

    });

    afterAll(async () => {
        await closeBrowser();
    });

});
```


This code opens a browser navigates to localhost:3000, expecting some text that says 'Login'.  It then fills out the form with valid credentials and expects to navigate to a page that has 'Connections' in the text somewhere.  

I changed Taiko's require statement to an import for consistency with the rest of my project.


![connections](https://live.staticflickr.com/65535/49948233647_9268392cb3_c.jpg)


With the code in the tests folder it's just a matter of starting the application then run:


```
$ npm test
```


Jest will open a browser and run through the script.  It will then listen for changes to the project and re-run. Super cool.


```
$ q
```


Stops Jest.


## That's All
Well Flatiron Community.  It's been fun, you're a special group of folks.

Cheers,

Justin

