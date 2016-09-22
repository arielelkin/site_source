---
layout: page
title: "Why I'm not a React Native Developer"
date: 2016-09-21 18:00
comments: false
sharing: true
footer: true
---

Many people are currently assessing React Native as a _platform_ to develop their next mobile app on. This is no trivial matter. Switching your software development platform involves a high setup cost and will profoundly impact your daily programming workflow. It is also one of the costliest decisions to reverse after anything substantial has been built. 

Perhaps more importantly, your software development platforms also shape you as a software engineer. A software development platform encourages (or forces) the use of one language over another, prioritises certain architectures over others, requires using specific tools and workflows, and marries you to an entire ecosystem and its developer community. 

Facebook wants you to switch:

![Is Facebook's intention with React Native to wholly replace native mobile development as we know it today? Yes](https://i.imgur.com/GWXQehS.png)

And the React Native Team's sustained and painstaking efforts match the ambition. The've produced a software development platform so substantial that it can legitimately be considered a replacement for our traditional Xcode/Swift/ObjC stack. 

Would it be a viable replacement? The blog posts I've read about React Native provide a rather cursory assessment of it. None has assessed its advantages _and_ disadvantages with the depth you'd expect from someone trying to persuade you to change your software development platform. 

Having spent several months using React Native, I have found that it is neither a platform I would develop in, nor a platform I would recommend the use of. This article proposes to provide a more thorough evaluation of the pros and cons of switching from Swift development to React Native, and will argue against the switch.

# Pros

## Declarative style

One thing I've found delightful about working with React Native was the declarative style in which the UI is programmed. In the React way of doing things, UI is _a function of_ state and properties, whereas in Cocoa Touch, UI is imperatively written.

The following example will clarify what I mean. Suppose our UI needs to include a small square on the top left side of the screen, this small square should be red if, say, the user is connected and green if the user is not connected. 

Here's how we typically do it on iOS: 

```swift
class ViewController: UIViewController {

  let indicatorView = ConnectivityIndicatorView()

  override func viewDidLoad() {
    super.viewDidLoad()

    let indicatorViewFrame = CGRect(origin: CGPoint.zero, size: CGSize(width: 20, height: 20))
    indicatorView.frame = indicatorViewFrame
    view.addSubview(indicatorView)
  }
}


class ConnectivityIndicatorView: UIView {

  var isConnected: Bool = false {
    didSet {
      if isConnected {
        backgroundColor = .green
      } else {
        backgroundColor = .red
      }
    }
  }

  override func didMoveToSuperview() {
    super.didMoveToSuperview()

    let weAreConnected: Bool = arc4random()%10 > 4
    self.isConnected = weAreConnected
  }
}
```

In the imperative style, you specify all the steps required to update the UI. We need to listen for changes in `isConnected` and update the view accordingly. We tell iOS _how_ to compute state.

Compare this with React's declarative way:

> Simply express how your app should look at any given point in time, and React will automatically manage all UI updates when your underlying data changes.



{% raw %}
```
class Root extends Component {  
  render() {
    return (
      <ConnectivityIndicatorView />
    );
  }
}

class ConnectivityIndicatorView extends Component {

  constructor() {
    super()
    this.state = {
      isConnected: false
    }
  }

  componentDidMount() {

    var weAreConnected = Math.floor(Math.random() * 10) > 5;

    if (weAreConnected === true) {
      this.setState({
        isConnected: true
      })
    } 
    else {
      this.setState({
        isConnected: false
      })
    }
  }


  render() {
    var color;
    if (this.state.isConnected) {
      color = 'green'
    }
    else {
      color = 'red'
    }
    return (
      
      <View style={{'backgroundColor': color, 'width': 20, 'height': 20}}>
      </View>
    )
  }
}
```
{% endraw %}

React's declarative style makes you _describe_ your UI with React within the view's `render()` method. The React framework ensures that any changes in state trigger re-rendering. Changes in your data (i.e. when `backgroundColor` changes) automatically trigger a change in the UI. 

This effectively does away with you manually updating views in response to changes in the model. Whether or not you like being relieved of that responsibility, React does a very good job at ensuring that the updates are carried out according to your descriptions, and you no longer have to worry about rolling out and maintaining a property setter for that `isConnected` variable. All you update is the state. 

You'll also notice that you end up reasoning of UI elements as if they were functions rather than instances of classes. They take in state and render a UIKit object. The purpose of a `Component` is to return itself in the state it's supposed to be, when requested. 

I found it to be a useful way to think about UI. And it's a good evolution from MVC, with the View being just responsible for displaying itself and not for managing data, a welcome departure from `UIViewController` and its turning a blind eye to incestuous relationships between V and C. 


## Faster iterations

React Native's appeal isn't just intellectual, the framework will appeal to the pragmatically minded too. 

When you program in React Native, the framework will create a local server that serves the JavaScript files you're working on. You build the app _once_, run it on the iOS Simulator or a device, and React Native ensures that any changes you make in the JavaScript are reflected in the app. 

You're given two options:

* Live Reloading will reload the app every time you edit and save one of its files. Basically saves you from switching to the Simulator and pressing `⌘ + R`. 
* Hot Reloading doesn't reload the entire app, it only reloads the file you just edited. If, say, you're working on the UI of a table view cell deep inside a navigation stack, the change will be immediately reflected in the cell you see in the Simulator, you won't have to navigate from the startup screen to the cell to see the changes. And the component will stay in the state it's already in. It's a WYSIWYG programming experience, a luxury that Xcode has never afforded you for your apps. 

I remember having to add a debug method in AppDelegate for every ViewController I'd want the app to show on startup, so that I wouldn't have to manually navigate down to it every time I'd run the app to see the changes I had just coded in. Hot Reload now makes me feel like a caveman.

React Native's feedback loop is _bewitchingly_ low. It takes less than one or two seconds between you saving a file and seeing the change in your app. That's easily ten times less than the typical Build and Run cycle we're used to in Xcode. 

This allows over-the-air code updates. Any changes in your JavaScript code can be instantly pushed to your users while the app is in production. 


## Cross-platform

Remember that graceless and awkward conversation? 

>"Great app! Does it run on Android? [...] Oh. And when do you plan to release an Android version? [...] Oh."

Your codebase can now create an app that can run on millions of additional devices, and you've increased your outreach by several orders of magnitude. 

And I can attest that the same codebase will reliably produce the same-looking and same-acting app on both platforms. 

Being able to produce an Android incarnation of your app from the same codebase, graphically and functionally equivalent to its iOS counterpart, is empowering. 

The obvious advantages of any cross-platform frameworks are always compelling: market development, unified codebase, and a unified skill set required to maintain the app's codebase. 


# Cons

## Uncertain roadmap

A major concern with using React Native is the lack of long-term commitment for the project. 

If this project were a mere plug-and-play _component_ of my app, like a networking library or an SVG to `CGPath` renderer, its long-term survival and maintenance is a secondary concern of mine; for if its developers abandon it, or if its pace of development is unsatisfactory – sad as that may be – I can replace it with a roughly equivalent library or take up maintenance myself. It might be a big deal, but in any case it won't be a huge deal; I make sure to have loose enough couplings with all my third-party libraries/Cocoapods to ensure that the whole project doesn't collapse if one library stagnates or goes missing. 

But React Native is not a plug-and-play Cocoapod, it's not a mere SDK, it's not a mere library, it's an entire software development platform. Saying my app has a "tight coupling" to it would be an understatement, my app is _entirely dependent_ on it. If Facebook stops maintaining React Native, my app will stagnate, and there won't be a "React Native replacement" at my disposal. If I want to carry on development of it myself, I'll have to get familiar with the the React Native source, _as well as_ the React.js codebase, the React Native CLI tools, and `JavaScriptCore`. Will the community ensure the project survives? Maybe – and if it does it probably will not be at the pace that we're used to. 

On the GitHub repository you'll see a new React Native release at a healthy pace of roughly once every two weeks. Not bad for a software development platform that targets two separate and complex software development platforms. Promising as that may be today, Facebook hasn't made _any_ long-term commitments to maintaining React Native for a sustained period of time. The company hasn't provided _any_ guarantees that it _won't_ pull the plug on the project – not just for the foreseeable future, but for the lifetime of _your app_. In other words, you currently don't have any guarantees it'll _ever_ be compatible with iOS 11, or 12. 

> Here's to the next year!

Ends an official React Native blog post from April 2016. What about the ones after? Facebook is _worryingly_ silent about that.

> We will publish plans quite soon - Konstantin

Said the React Native team, back in December 2015. Those plans haven't yet been published. The results of Facebook's long-term cost-benefit analysis of React Native, if there ever was one, sound less awesome than you thought. 

Again, we're not talking about an individual Cocoapod, we're talking about _the only platform your code can run on_. It's the kind of thing you do need very-long-term visibility on, and you don't have any.



## Patently daunting

Alongside its permissive BSD-style license, React Native ships with Facebook's **Additional Grant of Patent Rights**, Version 2. Facebook's motives for including this file aren't clear, nor is the file itself. 

It's a bipolar document. Right after the document grants you a "perpetual, worldwide, royalty-free, non-exclusive, irrevocable" license to use React Native, we are served with the following clause:

> The license granted hereunder will terminate, automatically and without notice, if you (or any of your subsidiaries, corporate affiliates or agents) initiate directly or indirectly, or take a direct financial interest in, any Patent Assertion: (i) against Facebook or any of its subsidiaries or corporate affiliates, (ii) against any party if such Patent Assertion arises in whole or in part from any software, technology, product or service of Facebook or any of its subsidiaries or corporate affiliates, or (iii) against any party relating to the Software.

I'm not a lawyer, so I've asked a lawyer to help me clarify this (and if you're not a lawyer you should too). According to him, that clause boils down to this: if I initiate _any_ lawsuit alleging patent infringement against Facebook, my license to use React Native would be immediately terminated. (I've quoted his more detailed breakdown in the Appendix below.)

In practice, this seems like a deterrent to me suing Facebook over patents. By implication, it also seems to give Facebook a good deal of breathing room to infringe _my_ patents ("nice quantum physics technology you have here, would be a shame if something bad were to happen to your app"). 

You may find the above interpretation pessimistic. But you should be pessimistic when evaluating software licenses and patents. We assess the efficiency of our algorithms in terms of how well they perform in the worst _possible_ scenario. I think that's how we ought to evaluate software licenses and patents too – for exactly the same reasons. 

A linear search algorithm will perform optimally in scenarios where the target value happens to be the first in the list, but that fact _alone_ isn't sufficient to persuade me of using linear search algorithms in any given scenario. _If it is possible_ for the target value to be the last in the list, my linear search algorithm's average performance will be very poor. 

Likewise, the fact that Facebook happens not to be a serial IP infringer _today_ reassures me little _if it is possible_ for Facebook to become a serial IP infringer tomorrow and punish any retaliation from my side by revoking my license to use React Native. If Facebook were to infringe my IP tomorrow (however unrelated to my software it is) and I sue them for it, I have given them the right to pull the plug on my app. 

What I'm jeopardising here is both _the platform_ my app depends on _as well as_ my intellectual property. 

iOS apps enter the App Store _entirely_ at Apple's discretion. I don't want to multiply that uneasy feeling by two. 

### Patently silent

Is the interpretation above correct? Is that worst-case scenario a possibility? Do I have a good reason to be worried? Do I have a good reason **not** to be worried? 

In various GitHub issues and forums posts, developers have claimed that their legal departments have advised them _not_ to use React or React Native precisely because of this clause. Matters remain deplorably ambiguous for developers who don't have a legal department at hand. 

More worryingly, Facebook hasn't made any substantial efforts to clarify matters. A 2015 post on the Facebook Open Source blog acknowledges this "confusion" and cheerfully announced clarifications to its Additional Grant of Patent Rights. More than five related GitHub issues later, the fog hasn't cleared. 

Various Facebook developers have responded to the issues, attempting to reassure developers while making it clear that whatever they wrote was not an assurance of anything. One of them even offered links to discussions of the issue on Reddit and Hacker News – unhelpfully, and rather misleadingly. 

This has turned into an exercise in hermeneutics and speculation over a question of genuine importance: Can Facebook revoke its React Native license? If so, under what conditions? 





## Javascript

A crucial downside to switching to React Native from Swift is the **technical regress**: you have to adopt and use JavaScript, a language that's 

* technically defective
* unsafe
* slowly evolving

Allow me to argue why. 

All the examples of JavaScript code hereafter will be of valid JavaScript (ES2016) code


### Javascript's inadequacy

One of my favourite bumper stickers proclaims:

> Safety is no accident

At first sight, the wit in this quip lies in its double meaning. We are offered a definition of safety as the absence of accidents _and_ the suggestion that it is the product of a set of safeguards. 

Is safety the product of the safeguards baked into your car, such as seat belts and airbags, or of your safe driving?

The answer, of course, is _both in equal measure_.

Drivers should prefer cars that have the widest amount of safeguards built into them. Not because they make driving _easier_, but because they decrease your chances of having a _preventable_ accident. 

Likewise, a programming language should offer safeguards against programmer error. 

The fact that millions of drivers productively drive cars without wearing a seatbelt isn't a good argument for cars with no seat belt. Similarly, the fact that millions of JavaScript developers productively use an inherently unsafe language isn't a good argument for the use of unsafe languages. 

The importance of safety in a programming language is one that I appreciated as the tools for iOS development evolved. 

When Automatic Reference Counting came to Objective-C, you had the option to switch it off for your iOS project. Why was it a bad idea to switch off ARC? Because the compiler could now perform an evaluation of the lifetime of your objects that was more accurate than yours. "The compiler is smarter than you" was the mantra, and that was certainly the case at least as far as reference counting was concerned. I remember how satisfied I was to realise that my `EXC_BAD_ACCESS` runtime crashes were decimated as a result. 

Objective-C gives you the option of setting the type of a variable to `id`, which stood for "any type whatsoever". But it is bad practice to use it for a variable whose type you know because it opens the door to runtime crashes – _that the compiler could prevent_. If it's a problem that the compiler can solve, you let the compiler solve it – and tackle interesting problems instead. 

You'll remember `unrecognized selector sent to instance` crashes. You were calling a method on an object that didn't respond to it. Type error. Third of all my bugs. 

Unsurprisingly, one of my first reactions after using Swift was "this was made by someone who got tired of preventable runtime crashes in Objective-C".

Swift is safe. The compiler won't let you pass in an `Int` to a function that expects a `String`. In fact, if the compiler isn't able to infer a type, you have to explicitly set it. 

But JavaScript lacks these safeguards against programmer error, making _preventable_ runtime crashes and _preventable_ programmer errors part of your _routine_.


#### Type errors

The language does not enforce type of variables and parameters to functions:

Any variable can be anything at any time:

```javascript
var j = null

var j = function foo() {
  return j
}

var j = 23

var j = 'hello'

console.log(j) //hello
```

JavaScript makes you believe that it has some notion of types and class, with keywords like `class`, `typeof`, and `instanceof`:

```javascript
class Person {
  //nobody is immortal:
  isImmortal() {
    return false
  }
}

var p = new Person()

console.log(typeof p) //object
console.log(p instanceof Person) //true

console.log(p.isImmortal()) //false

p.isImmortal = function() {return true}

console.log(p.isImmortal()) // true
console.log(p instanceof Person) //true
```

Here we see the following:

* JavaScript's notion of "class", "type", and "instance" is completely different from the majority of the programming world.
* types serve no useful purpose in JavaScript, as it's too easy to _unreliably_ define a type.

Remember those `unrecognized selector sent to instance` crashes? Thought they were no longer part of your life? Here's their incarnation in React Native:

![this.foo is not a function](https://i.imgur.com/J5ugzMI.png)


#### Lack of optionals

A very large amount of bugs in Objective-C code (and in many other older programming languages) are due to programmers inadvertently calling methods on objects that are `nil`. 

In the world of React Native and JavaScript, this error is common: 

![screenshot showing null is not an object error](https://i.imgur.com/vAJFl7X.png)

And preventable, both in theory and in practice. 

Swift did away with this problem by implementing optionals, which force you to take required `nil` checks if you know that an object can be `nil`. 



#### Lack of function signature

In JavaScript, functions don't have a return type, you don't know what can be returned by the function, or if it returns anything at all.

```javascript
var foo = 'im a number'

function divideByFour(number) {
  return number / 4
}

console.log(divideByFour(foo)) //NaN
```

Let's make this even more fun. In JavaScript, any expression can be evaluated at any time by any function. Consider this example using two functions from the JavaScript standard library; `map`, which is supposedly the equivalent of Swift's `map`, and `parseInt`, which parses a string into an integer. 

```javascript
var foo = ["1", "2", "3"].map(parseInt)

console.log(foo) //[1, NaN, NaN]
```

This messed up result is due to the fact that `parseInt` takes two parameters `(val, radix)` but `map` passes it 3 parameters `(currentValue, index, array)`. Still totally legal JavaScript (a language that some people consider viable for functional programming). 


#### Immutability

JavaScript's support for immutability is very poor. 

There is a `const` operator, which helps ensure primitives don't get mutated. But anything that's not a primitive is as malleable as jell-o: 

```javascript
const array = [3, 6]
array[32] = 9
console.log(array) // [3, 6, 32: 9]


const foo = function() {
  this.number = 42
}
const bar = foo
bar.number = 999
console.log(foo.number) // 999
```

Unique copies? Nope. Any unique object could be modified by any part of your app at any time. Good luck multithreading. Indeed,

> Much of what makes application development difficult is tracking mutation and maintaining state. 

Says... Facebook, in the documentation of Immutable.js. (A framework designed to bring immutable data structures to JavaScript.)

But here's how you enforce immutability in React.

![](https://i.imgur.com/SMvOwOA.png)

You politely ask the developers to not mutate state. Yes, that's a screenshot from the React documentation. 


#### You can't trust arrays

You thought arrays were a "systematic arrangement of similar objects, usually in rows and columns"? Think again. 

```javascript
var array = [0,1,2]

array["hello"] = "watsup"

array[6] = 999

for (var index in array) {
  console.log(array[index]) //0,1,2,999,watsup
}

for (var index of array) {
  console.log(index) //0,1,2,undefined,999
}

array.forEach(function(value, key, array) {
  console.log(value) //0,1,2,999
})

console.log(array[5]) //undefined
```

JavaScript arrays have more in common with ordinary JavaScript objects than with what we'd call an array. Their lack of precise sequentiality and mutability doesn't make them well-behaved.

#### Poor error handling

In JavaScript, you can have *any* function throw a runtime error or exception, without warning. 

```javascript
function tenDividedBy(number) {
  if (number == 0) {
    throw "can't divide by zero"
  }
  return 10 / number
}

console.log(tenDividedBy(0))
```

You can throw _anything_ you want as an exception; a string, a `Date`, a function, etc. There is no mechanism for you to mark a function as potentially crashing your teammate's code or for specifying how to process an exception. Use `if` statements instead, say the docs:

>It’s best to leave exceptions as a last line of defense, for handling the exceptional errors you can’t anticipate, and to manage anticipated errors with control flow statements.



#### No support for decimals

Given that most decimal fractions cannot be accurately represented as binary fractions in hardware, many programming languages (including JavaScript and Swift) will often produce mathematically incorrect decimal arithmetic:

```javascript
console.log(0.1 + 0.2) //0.30000000000000004
console.log(0.1 + 0.2 === 0.3) //false
```

Which is why other languages' standard libraries often have support for decimal fractions (in Swift, for instance, we can use `Decimal`). In JavaScript, you have to resort to using a third-party library – or write your own.

#### Dodgy maths 

Remember to be careful with primary school arithmetic too. JavaScript has a complicated relationship with `0` and things that are not numbers. 


```javascript
var a = 0
var b = -0

console.log(a === b) // true

console.log(1/a === 1/b) // false


var x = Math.sqrt(-2)

console.log(x === NaN) //false

console.log(isNaN(x)) //true

console.log(isNaN('i like pumpkins')) //true
```


#### Unsafe initialisation

JavaScript allows for objects to be left in an inconsistent state right after being created, as their properties don't need to be initialised.


```javascript
class Rectangle {
  constructor (width, height) {
    this.width = width
    this.height = height
  }
  area () {
    return width * height
  }
}

class Square extends Rectangle {
  constructor(){
    super() 
  }
}

var myRect = new Square()

console.log(myRect.area()) // ReferenceError: width is not defined
```

#### Optional curly braces after an `if`

Curly braces after an `if` statement are optional:

```javascript
if (1 > 2) 
  console.log("ha"); console.log("he")
  console.log("hi")
  console.log("ho")

/*prints: 
  he
  hi
  ho
  */
```

To add that taste of adventure to your control flow statements. 


#### Ambiguous curly braces

Unless programmer intent can be very accurately inferred, a language shouldn't let curly braces be optional. 

```javascript
function foo() {
  return
  {
    {a: 4}
  }
}

console.log(foo()) //undefined
```


#### Switch fallthrough

If you forget to include a `break` statement in a clause of a `switch` statement, you fall through. Also, a `swift` statement doesn't carry out checks for case exhaustivity.

```javascript
var j = 32

switch (j) {
   case 32:
    console.log('spot on')
   case 0:
    console.log('zero')   
}

/* prints:

  spot on
  zero
*/
```

#### What's nothing? 

```javascript
var foo;
foo === null; //false 
foo === undefined; //true
```

Yes, something that's not `null` could be nothing for all intents and purposes. Not a helpful distinction.

And what if you want to know whether a variable contains any data? Well, for your checks to be exhaustive, you need to check for both `null` and `undefined`:

```javascript
if (foo !== null && foo !== undefined) {
  shootMyself()
}
```

#### Poor expressivity

* No enums. Let alone enums with associated types. Good luck reliably representing state. 
* No `guard` statement.
* No generics.
* No `where` statement to increase expressivity of control flow statements.


#### Exceedingly slow evolution

Behold, ES2016 brought these new features to JavaScript:

1. The `includes` method for arrays. 

It checks if an array contains a specified value. Here's how you use it:

```javascript
var array = [1, 2, {"hello": 32}]

console.log(array.includes({"hello": 32})) // false


var foo = 32

var bar = foo 

var otherArray = [bar, 23]

console.log(otherArray.includes(foo)) // true
```

Try not to. 


<ol start="2">
  <li>The <b>**</b> operator</li>
</ol>
 
To be used for exponentiation. `a ** b` is shorthand for `Math.pow(x, y)`. 

Consider this: Python's `**` operator was in Python 1. Ruby's `**` operator has been in Ruby for more than 20 years. 

So it took JavaScript 20 years to add a basic arithmetic operator and a quite limited membership-checking method for arrays, as a year's worth of improvements. 



#### Flow to the rescue!

Flow is Facebook's answer to many of the above grievances. It's a static type-checker for JavaScript, capable of inferring and tracking the types of the variables in your code, and alert you of impending doom. 

Recall my example above about the problems that arise from lack of function signature in JavaScript (the `divideByFour` function which expects a number but is fed a string). Here's how Flow would deal with it:


```javascript
// @flow
var foo = 'im a number'

function divideByFour(number) {
  return number / 4
}

console.log(divideByFour(foo))

/* console output: 

fooman.js:8
  8: console.log(divideByFour(foo))
                 ^^^^^^^^^^^^^^^^^ function call
  5:   return number / 4
              ^^^^^^ string. This type is incompatible with
  5:   return number / 4
              ^^^^^^^^^^ number

Found 1 error
*/
```

This fixes many of the problems that occur from JavaScript's lack of enforceable function signature.  

It can also produce the equivalent of generic arrays:

```javascript
// @flow
class J {
}

var b = function() {
  return 2
}

var array = [1, new J(), b]

function giveMeAnArrayOfNumbers(numbers: Array<number>) {}

giveMeAnArrayOfNumbers(array)

/* console output:

fooman.js:13
 13: giveMeAnArrayOfNumbers(array)
     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ function call
  9: var array = [1, new J(), b]
                         ^ J. This type is incompatible with
 11: function giveMeAnArrayOfNumbers(numbers: Array<number>) {}
                                                    ^^^^^^ number

fooman.js:13
 13: giveMeAnArrayOfNumbers(array)
     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ function call
  9: var array = [1, new J(), b]
                              ^ function. This type is incompatible with
 11: function giveMeAnArrayOfNumbers(numbers: Array<number>) {}
                                                    ^^^^^^ number
Found 2 errors

*/

```

Flow deals very well with nullability, and will warn you if a value you'd expect not to be `null` might be `null`:

```javascript
// @flow
var j = null

function greet(name) {
  console.log("hello " + name)
}

greet(j)

/* console output:
fooman.js:6
  6:   console.log("hello " + name)
                              ^^^^ null. This type cannot be added to
  6:   console.log("hello " + name)
                   ^^^^^^^^^^^^^^^ string


Found 1 error
*/
```

If `j` is supposed to be a `string`, but could be `null` at any point in time, Flow encourages us to annotate it accordingly. It is thus able to complain if we don't carry out null-checks before feeding it to anything that needs it to be a non-null `string`:

```javascript
// @flow

var j: ?string = null

function greet(name: string) {
  console.log("hello " + name)
}

if (j != null) {
  greet(j)
}
/* console output:
No errors!
*/ 
```

Flow's features go positively beyond type-checking and annotation, and add support for new constructs. One of them is literal types, which we can use to build, for example, enums. 

```javascript
// @flow
type CompassPoint =
  | "North"
  | "South";

class Compass {
  direction: CompassPoint;
  degrees: number 

  constructor(degrees: number) {
    this.degrees = degrees
  }
  direction(): CompassPoint {
    if (this.degrees > 270 || this.degrees < 90) {
      return "North"  
    }
    else {
      return "South"
    }
  }
}
```

And Flow will complain if `direction` returns anything that's not `"North"` or `"South"`. 

Another construct I find useful is Union types, which constrain a value to be one and only one of a predefined set of types. Here's an example from the Flow docs:

```javascript
// @flow
type U = number | string;
var x: U = 1;
x = "two";
```

Flow is great friends with React Native, as long as you help it help you. The Flow docs offer a good example of how what happens when you correctly annotate the `proptypes` of your component:

```jsx
const Greeter = React.createClass({
  propTypes: {
    name: React.PropTypes.string.isRequired,
  },
  render() {
    return <p>Hello, {this.props.name}!</p>;
  },
});

<Greeter />; // Missing `name`
<Greeter name={null} />; // `name` should be a string
<Greeter name="World" />; // "Hello, World!"
```

The comments indicate where the Flow warnings appear, adequately reprimanding you for not fulfilling your own contract. 

Flow's an altogether powerful tool. Here are some examples of the useful things its command-line interface lets you do:

*  `suggest         Shows type annotation suggestions for given files`
*  `type-at-pos     Shows the type at a given file and position`
*  `get-def         Gets the definition location of a variable or property`


#### Flow's like flossing

Is JavaScript fixed now? No. 

Impressive as an engineering effort Flow may be, it remains a superset of JavaScript and thus forces you to build on inherently weak foundations. Hailing Flow (or TypeScript, for that matter) as any kind of remedy for JavaScript's faults reminds me of Swamp Castle and its King (from Monty Python and the Holy Grail):

![](https://i.imgur.com/yxap6gp.png)

> When I first came here, this was all swamp. Everyone said I was daft to build a castle on a swamp, but I built it all the same, just to show them. It sank into the swamp. So I built a second one. That sank into the swamp. So I built a third. That burned down, fell over, then sank into the swamp. But the fourth one stayed up. And that's what you're going to get, lad, the strongest castle in all of England.

The fact that things can be built on unsafe foundations does not make the foundations any safer, nor does it make the process any more efficient. And our insisting in perpetrating this exercise makes us lose sight of its preposterousness.

Perhaps more revealingly, JavaScript supersets, linters, or static analyzers are really palliative measures to mitigate the fact that you're dealing with a platform that doesn't give you a choice of a safer language – if it did, you wouldn't have to resort to using them. 

There is another fundamental problem with these palliative measures. Safeguards, like laws, are worth very little if they're not actually enforced by the authorities and respected by the community. 

Flow doesn't stop you from building and running React Native apps with code bound to generate runtime crashes. And that is a basic safety requirement for a programming language: if it's a preventable error, the language should _actively prevent_ it, it should stop me from writing and running unsafe code _by default_, not as an afterthought. 

It just _should not be possible_ for my teammate to write functions that are ambiguous as to their return value, it _should not be possible_ for me to call an method on an object that doesn't respond to it, it _should not be possible_ for someone to not define `proptypes` on a component and me having to manually point out during code review that `proptypes` should have been defined. 

Flow, like unit testing and flossing, has the curse of being beneficial _and_ optional _and_ wearisome. It's in your next year's resolutions. 

And that's the reality: how many public `.js` files on GitHub use flow (i.e. contain `@flow` to signal they're Flow-checkable) ? Roughly 1,400,000 out of roughly 80,000,000 public `.js` files. That's... less than 2% public usage of a tool to write safer JavaScript. 

On a more relevant note, out of the hundred React Native repositories over at `awesome-react-native`, I wasn't able to find one that made any significant use of Flow and its type annotations. Nor was I able to find a single React Native tutorial mentioning (let alone advising you) the use of Flow. 


### The Javascript Ecosystem: balls and chains

JavaScript's deficiencies seem to impress everyone except JavaScript developers, for whom the aspects of JavaScript that I outlined above are not awful warts, they're "quirks" or "gotchas" that _you_, not your language, have to be on the lookout for. 

This is because JavaScript developers also don't believe that JavaScript is inadequate. 

![that's just like your opinion, man](https://i.imgur.com/x6jGooV.jpg)

The language has no support for immutability? Let's make a library. The language has no support for typing? Let's make a library. The language has no support for decimals? Let's make a library. The language doesn't allow safe functional programming? Let's make a library. Language has no support for nullability? Let's make a library. 

Or... given that you acknowledge that those features are fundamentally important, you could just... switch to a language that supports them out of the box? (I don't know, I'm just throwing ideas out there.)

To my mind, there is a generalised obstinate refusal to see JavaScript as inherently deficient and therefore needing _replacement_ rather than _polishing_. The resulting proliferation of grafts and crutches is seen as signs of a vibrant ecosystem, whereas what it really shows is that the language is lacking in fundamentally important functionality. 

The situation is illustrated very well by this cartoon:

![](https://i.imgur.com/qPlBUp5.png)


"Freedom from digging" is simply choosing a language that has built-in support for the things you need. This digging is not a good use of energy. JavaScript is not naturally conducive to good software engineering also for this reason: it forces you to develop and depend on things that other languages offer by default. 


#### Chains

JavaScript has bigger fish to fry. It's a language that needs to cater to the needs of literally _billions_ of internet users, who may or may not update their browser or website. That cripples the development of the language. 

Remember `typeof(null) === 'object'` ? Well, there was a proposal to change the type of `null` to `null`. But

> it turned out that it broke a lot of existing sites. In the spirit of One JavaScript this is not feasible.

And the proposal was rejected. As of ES06, `null` is still `object`. 

JavaScript's evolution, which caters to the needs of: 

* millions of different outdated browser users
* a set of disparate browser vendors
* billions of websites and their respective developers

is admirably democratic. But on the other hand, it will be much slower than that of other languages, and it probably won't prioritise developer ergonomics as much as  portability. 


#### Wider angles

From a historical perspective, the pattern we're seeing here, of a seemingly deficient language gaining popularity and being repudiated by the incumbents of the platforms it wants to take over, is poignant and familiar. 

Come sit by the fire (of the past flame wars) and I'll tell you a story. 

Back in 1994, Richard Stallman wrote to the newsgroup `comp.lang.tcl` an article unambiguously titled "Why you should not use Tcl". Stallman was sourly unforgiving of Tcl's shortcomings as a programming language, and denounced what he saw as Tcl's unfitness for purpose. The parallels with my own grievances against JavaScript are spectacular: 

> The principal lesson of Emacs is that a language for extensions should not be a mere "extension language". It should be a real programming language, designed for writing and maintaining substantial programs. Because people will want to do that! [...] Tcl was not designed to be a serious programming language.  It was designed to be a "scripting language", on the assumption that a "scripting language" need not try to be a real programming language.
> So Tcl doesn't have the capabilities of one.  It lacks arrays; it lacks structures from which you can make linked lists.  It fakes having numbers, which works, but has to be slow.  Tcl is ok for writing small programs, but when you push it beyond that, it becomes insufficient.

His article effectively triggered the Tcl War of 1994. Of all the responses he received, I found John Ousterhout's (who created Tcl) the most imposing:

> Language designers love to argue about why this language or that language _must_ be better or worse a priori, but none of these arguments really matter a lot. Ultimately all language issues get settled when users vote with their feet. If Tcl makes people more productive then they will use it; when some other language comes along that is better (or if it is here already), then people will switch to that language. This is The Law, and it is good. 

Ousterhout's counter-attack strikes on many of the fronts I've been active in, so his artillery is worth taking a closer look at.

(Some readers will notice Ousterhout argues circularly: "Why do people switch to a language? Because it is better. Why is it better? Because many people switched to it." His argument deserves a more generous interpretation nevertheless, not least because we often see this line of reasoning in many other arguments for any specific tech stacks, such as, you guessed it, React Native.)

According to Ousterhout, programming languages have features that can make them better _a priori_ (i.e. purely intrinsic characteristics such as programming paradigm, syntax, standard library, or portability) and features that can make them better _a posteriori_ (i.e. widespread adoption by many developers). It doesn't matter that Language A is _a priori_ better than Language B, if Language A isn't _a posteriori_ better than Language B.

In other words, adoption trumps technical superiority (regardless of how language designers define superiority). 

Ousterhout's argument fits on a record cover:

![](https://i.imgur.com/sSQXwte.jpg)

The more fans a singer has, the better he is. The point is that popularity, whether it's quantified by fans or hit records, is a good criterion for evaluating singers. 

But the fact that a language enjoys widespread adoption and can boast a sparkling app showcase should not influence your judgement as to its merits. 

A gallery of complex or nice-looking apps built using Technology X only means that Technology X _may_ be used to build complex or nice-looking apps; but not that it's an optimal tool for the job. 

Dozens of sandcastle competitions across the world show you the beautiful structures you can build using just sand and water. That may warrant the assertion that sand is a great toy, but sandcastle competitions alone shouldn't be enough to convince you that you should build your own house out of sand. Yet many are convinced. 

The King of Swamp Castle is the king of many Hackers. 

![nice sandcastle](https://i.imgur.com/hTEOskq.jpg)

Similarly, popularity is an unreliable criterion for evaluating programming languages. For a pretty basic reason: popularity may very well be the result of vendor lock-in (like being the only programming language that can be run in a browser), bandwagon effects, or legacy codebases. 

The intrinsic characteristics of a programming language (such as its safeguards against programmer error, standard library, portability, etc.) are what you should base your criteria on. They are the basis for productivity gains. 

In fact, many things lead to negative productivity gains. For example: 

* lack of safeguards to prevent you making avoidable errors
* a tiny standard library, unfit for most purposes
* syntax and semantics that don't let you express your intent clearly
* a slow pace of development
* All the above (JavaScript)

## Dependencies

React Native has a total of 648 dependencies. 

![648 total dependencies](https://i.imgur.com/g8B9S7a.png)

Not particularly surprising, as dependency chains can be long in the world of `npm`, React Native's package manager. 

This showcases the comradeship of open source: your app is built on more than 600 people's sustained efforts. 

This is also a pitfall: you depend on 648 volunteers to keep maintaining their library for the lifetime of your app, without any formal commitment. 

Will their licenses stay compatible with your software? Hopefully.

And will they all implement best practices in security? Or can you tolerate 648 separate potential security risks? 


## Better alternatives

If cross-platform development is what drew you to React Native, consider your options. 

React Native faces strong competition from two established cross-platform development platforms: Xamarin and Appcelerator. 

Both Xamarin and Appcelerator offer support for iOS, Android, _and_ Windows Phone. They both have:

* more comprehensive APIs
* more mature IDEs
* better documentation
* clearer and friendlier licensing
* equal (if not better) performance

Xamarin development is in C#, which is usually less bug-prone and more expressive than JavaScript. If you believe in JavaScript, Appcelerator development is in JavaScript. 

Compared to React Native, both Xamarin and Appcelerator have better prospects at longevity. Appcelerator (the maker of Titanium) (runs on apps installed on 350 million devices) was acquired in January 2016, and Xamarin (used by more than one hundred Fortune 500 companies) was acquired in February 2016 (by Microsoft). Both had previously raised more than $80M in equity funding. 

Projects are more likely to survive and thrive in this red ocean of cross-platform frameworks (I recently counted more than ten in active development), if they're backed by companies whose _business_ is to supply me with a reliable software development platform. 

(An honorable mention goes to Flutter, a cross-platform development platform (iOS and Android) being written by Google. It uses a language that's safer and more expressive than JavaScript (Dart), it helps you follow principles of Material design, and produces natively compiled code. Unlike Xamarin and Appcelerator, it's open-source. It's currently not production-ready, but it's promising nonetheless.)


# Conclusion

Good software development platforms have four essential features. 

* Portability – can target more than one platform. 
* Productivity – the maturity of its IDE and other development tools, documentation, and expressivity of its development language.
* Safety – the extent to which the platform prevents you making mistakes.
* Longevity – how likely the platform will be around for the lifetime of your app.

Even though our industry (_still_) has no established open standard to measure these, I'd like to offer my evaluation based on my experience and research. 

React Native's strengths in terms of portability and productivity are outweighed by its shortfalls in safety, the uncertainty about its long-term roadmap, and a daunting patent license:

![React Native Radar Chart](https://i.imgur.com/TgMsSwu.png)

Mature platforms such as Xamarin and Appcelerator offer the best portability (with support for Windows phone), their mature development tools and choice of programming language will also offer gains in productivity and safety, and the fact that they are the core product of well-funded companies offer good assurances of longevity:

![Xamarin/Appcelerator Radar Chart](https://i.imgur.com/ag2OzXC.png)

Swift development is a winner in terms of safety, longevity, and productivity. Its portability is relatively low but not negligible: it does not allow you to build an Android version of your app, but it allows you to build a macOS version, and lets you write the app's server backend:

![Swift Radar Chart](https://i.imgur.com/syAGZym.png)

And that is why I am not a React Native developer.



# Acknowledgements

My thanks to Rami Chowdhury and Alkis Papadakis for their helpful feedback on early version of this article. 

# Changelog

You can track this article's edits [here](https://github.com/arielelkin/arielelkin.github.io/commits/master/articles/why-im-not-a-react-native-developer.html).

# Comments

Please join the [discussion of this article on Hacker News](https://news.ycombinator.com/item?id=12549590).


# References

* [We are the Team working on React Native AMA](https://www.reddit.com/r/IAmA/comments/3wyb3m/we_are_the_team_working_on_react_native_ask_us/cxzvfir)
* [How to pitch React Native to Developers](https://medium.com/@dschmidt1992/how-to-pitch-react-native-to-developers-dcf092cb4614#.j3qrkbby6)
* [Classic Programmer paintings: “Functional programmer listens to JavaScript claims”](http://classicprogrammerpaintings.com/post/146305408264/functional-programmer-listens-to-javascript)
* [React Native at Artsy](http://artsy.github.io/blog/2016/08/15/React-Native-at-Artsy/)
* [React Native: A Year in Review](https://code.facebook.com/posts/597378980427792/react-native-a-year-in-review/)
* [Hacker News Discussion: Node.js is one of the worst things to happen to the software industry](https://news.ycombinator.com/item?id=12338365)
* [npm Meltdown Security Concerns](https://ponyfoo.com/articles/npm-meltdown-security-concerns)
* [https://blog.addjam.com/react-native-its-not-all-sugar-and-spice-cb5d6b25eae9#.gy4fxnpxz](React Native: It’s not all sugar and spice)
* [The King of Swamp Castle, from _Monty Python and the Holy Grail_](https://youtu.be/aNaXdLWt17A)
* [The Three Big Lies About JavaScript](https://medium.com/javascript-non-grata/the-three-big-lies-about-javascript-e227cabe3beb#.mpvix0q96)
* [Review: Xamarin in 2015 - Data Corruption and Swift are Not Helping](http://www.whitneyland.com/2015/07/xamarin-review-2015.html)
* [Product or Process?](http://robnapier.net/product-or-process)
* [StackOverflow question: Difference between strongly and weakly typed languages?](http://stackoverflow.com/q/17072179/1072846)
* [Let’s Make A Webpage In 2016](https://medium.com/friendship-dot-js/let-s-make-a-webpage-in-2016-55a673ac791c#.5igpj6hz3)
* [Why JavaScript is the Future of Programming](https://www.sitepoint.com/why-javascript-is-the-future-of-programming/)
* [Using React Native: One Year Later](https://discord.engineering/react-native-deep-dive-91fd5e949933)
* [Redmonk Language Rankings 2016](http://redmonk.com/sogrady/2016/07/20/language-rankings-6-16/?)
* [Seth Godin: This is Broken](https://www.ted.com/talks/seth_godin_this_is_broken_1)
* [_Argumentum ad populum_](https://en.wikipedia.org/wiki/Argumentum_ad_populum)
* [Hacker News Discussion: Why C++ sucks (2016 edition)](https://news.ycombinator.com/item?id=11147031)
* [React Native it's not all sugar and spice](https://blog.addjam.com/react-native-its-not-all-sugar-and-spice-cb5d6b25eae9#.s2sc2bgna)
* [ECMAScript Wiki: harmony:typeof_null](http://wiki.ecmascript.org/doku.php?id=harmony:typeof_null)
* [Cube Drone: Relentless Persistence](http://cube-drone.com/comics/c/relentless-persistence)
* [Wat](https://www.destroyallsoftware.com/talks/wat)
* [YourLanguageSucks](https://wiki.theory.org/YourLanguageSucks)
* [Defective C++](http://yosefk.com/c++fqa/defective.html)
* [JavaScript ES6 Features](http://es6-features.org/)
* [awesome-react-native GitHub repository](https://github.com/jondot/awesome-react-native)
* [Types, by Gary Berhnhardt](https://gist.github.com/garybernhardt/122909856b570c5c457a6cd674795a9c?)
* [All you wanted to know about types but were afraid to ask, by Joseph Abrahamson](https://github.com/tel/old-blog/blob/master/_posts/2014-07-08-all_you_wanted_to_know_about_types_but_were_afraid_to_ask.md)
* [Stallman vs. Ousterhout](https://groups.google.com/forum/#!msg/comp.lang.tcl/7JXGt-Uxqag/3JBTj5I43yAJ)
* [Ousterhout vs. Stallman](https://groups.google.com/d/msg/comp.lang.tcl/7JXGt-Uxqag/vQNLEgvjmWsJ)
* [Visualisation of React Native's npm dependencies](http://npm.anvaka.com/#/view/2d/react-native)
* [Updating Our Open Source Patent Grant](https://code.facebook.com/posts/1639473982937255/updating-our-open-source-patent-grant/). Blog post from Facebook's _Open Source Blog_. 
* ["Legal Department did not allow use of React"](https://discuss.reactjs.org/t/legal-department-did-not-allow-use-of-react/3309/7). Forum post from the _React Discussion Forum_.
* [License clarification #7293](https://github.com/facebook/react/issues/7293). Issue opened on the React repository. 
* [Reconsider restrictions in patents file #402](https://github.com/facebook/react-native/issues/402). Issue opened on the React Native repository. 
* [Please reconsider restrictions in PATENTS (Additional Grant of Patent Rights Version 2) #8952](https://github.com/facebook/react-native/issues/8952)
* [Update PATENTS #3554](https://github.com/facebook/react/pull/3554). Ambitious pull request on the React repository. 
* [clarification on PATENTS grants and termination clauses #3617](https://github.com/facebook/react/issues/3617). Issue opened on the React repository.

### Appendix: A more detailed breakdown of React Native's Patent issues

From [Greg McMullen](https://twitter.com/gmcmullen).

```
# None of this is legal advice! Just my reading of the license. If you want advice specific to you, see your own lawyer.

# If you sue Facebook for patent infringement and any of these three conditions are met, the license is terminated.

IF 
You (or any of your subsidiaries, corporate affiliates or agents) initiate directly or indirectly, or take a direct financial interest in, any Patent Assertion

  (i) against Facebook or any of its subsidiaries or corporate affiliates,

  OR
  (ii) against any party if such Patent Assertion arises in whole or in part from any software, technology, product or 
        service of Facebook or any of its subsidiaries or corporate affiliates, 

  OR
  (iii) against any party relating to the Software:


THEN
The license granted hereunder will terminate, automatically and without notice.



# If Facebook sues you for patent infringement, you can make a patent infringement counter-claim against them without the license being terminated.

IF
1) Facebook or any of its subsidiaries or corporate affiliates files a lawsuit alleging patent infringement against you in the first instance, 

AND 
2) you respond by filing a patent infringement counterclaim in that lawsuit against that party that is unrelated to the Software, 

THEN
The license will not terminate due to such counterclaim.

# Just to be clear, the license also won't terminate if Facebook sues you and you don't make a counterclaim
```

