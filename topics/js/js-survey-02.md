---
layout: page
title: Java Survey 2 
permalink: /js-survey-02/
---


 - So you can augment any object or any array simply by assigning new properties to it, which is really nice. 
 - You can also augment all of the types by changing the initial prototypes. 
 - So for example, you can add a new method to every string. 
 You can add a new method to every number, which is a really, really nice way of being able to improve the quality of the language. 
 You can make it more suitable to the kinds of things that you're doing. 
 So all of these types, object, array, function, number, string, and Boolean have prototype properties that you can add new methods to. 
 So let me show you an example. 
 -  Virtually every language has a trim, function which will take a string and remove whitespace from the ends. 
 -  That's a really useful thing to do, particularly to input fields where users might put whitespace, but you don't want to have that going all the way through your system. 
 -  So trim is really a nice way of doing that. 
 -  Unfortunately, JavaScript didn't think to put in a trim function. 
 -  But that's OK. 
 -  Because it's really easy to make one. 
 -  So what you do is you first want to see if there already is trim function. 
 -  Because in the future, there might be one. 
 -  If there isn't, then go to the string.prototype.trim. 
 -  And put a function there. 
 -  In this case, I'm going to have a function which calls the strings replace method with that regular expression and that result parameter. 
 -  We'll talk more about regular expressions later. 
 -  And that'll have the effect of removing the whitespace and returning just to the clean string. 
 -  That's really nice.
 -  Another useful function to have would be supplant-- a supplant method in which I could have a string which would contain variables. 
 -  And I could have those variables replaced. 
 -  So for example, I've got this template that looks like an HTML template with border, last, and first variables in it. 
 -  And I've got a data object, which also contains first, last, and border. 
 -  And what I'd like to do is say, take these values, and replace those variables in the template. 
 -  That would be a really useful thing to do. 
 -  So I could write something like that, where I could assign to enter HTML, the result of supplanting the data on the template. 
 -  That would be really useful. 
 -  Unfortunately, there isn't a supplant method. 
 -  But fortunately, it's just as easy to make. 
 -  So we'll have supplant. 
 -  Again, it's going to call the strings replace method with a regular expression, this time passing in a function as a second parameter, which will look to see if the value which is being matched in the pattern matches a member of the object. 
 -  And if it does, it will do the replacement. 
 -  What is passed to the function is the things that were matched. 
 -  So A would be everything that's matched. 
 -  B would be the first parenthesized thing that was matched. 
 -  In that case, A would contain the curly braces, and B would not. 
 -  So we'd then take the B. 
 -  Look it up. 
 -  If we've got a string, then we'll replace it. 
 -  So in this language, you can have objects inheriting from other objects, which is good. 
 -  But you can't really inherit from an array. 
 -  You can't inherit the array's properties. 
 -  You can inherit the array's prototype. 
 -  But the thing you get back is an object that doesn't have the trickiness in it that make arrays interesting, particularly with the length property. 
 -  But what you can do is you can augment an array. 
 -  You can take any array, and stick a function on it. 
 -  Because arrays in this language are really objects-- and because objects can take any kind of name, not just numbers-- you can stick those on two arrays, as well. 
 -  So you can add any methods you want to the array. 
 -  And then you can also augment all of the arrays at once by changing array.prototype. 
 -  Yeah so, do I recommend any libraries for doing this sort of augmentation? Most of the AJAX libraries are properly cautious in making these modifications with the exception of the prototype library, which is dangerously reckless in the way it makes these modifications. 
 -  So your mileage may vary. 
 -  So since typeof doesn't tell you if something is an object or an array, how can you tell? There are a couple of things you can do. 
 -  One is you can ask it if value.constructor equals the array function. 
 -  You can also ask if value is an instance of the array function. 
 -  Unfortunately, neither of those work if the array value came from another frame. 
 -  So if you have IFrames and you're passing functions values from one frame to another, each frame has a different array function. 
 -  And so, they will not recognize each other. 
 -  In the future, there will be an isArray method built into arrays. 
 -  But it's not there yet. 
 -  JavaScript has an eval function. 
     -  That's a global function. 
     -  It gives you access to the compiler. 
     -  It will compile an expression, and execute it, and the result and. 
     -  That can be very, very powerful. 
     -  In fact, it's what browsers use to convert strings into actions. 
     -  When a browser sees an event handler or something in a script tag, it calls eval on it internally. 
     -  That's why it's there. It is also the most misused feature in the language. 
     -  When people are avoiding learning the language, very often, what they'll do is learn a little piece of it and then eval and synthesize what other language they don't know out of that misuse of the eval function. 
     -  If you ever find yourself wanting to use eval, really good chance you're using it incorrectly. 
     -  And you should step away from the computer, and try to figure out what you're doing wrong. 
     -  There are a couple of good uses for it. 
     -  So I can't say never use it. 
     -  Yeah, I can. Never use it. 
 -  There's also a function constructor, which is the thing eval uses to make functions. 
 -  It can take a string and compile it. 
 -  And occasionally, it's useful. 
 -  For example, if you have an AJAX application that's doing incremental loading of JavaScript, you could have it take source bodies, and ship them over. 
 -  And then you can turn them into functions using this. 
 -  Otherwise, it's really just a variation on eval and should also probably be avoided. 
 -  Java has both int and integer, which are two types which both range over the same values and have different levels of performance and inconvenience. 
 -  JavaScript copied that pattern to no good advantage. 
 -  Because JavaScript didn't need it. 
 -  But it has these wrappers anyway. 
 -  And I highly recommend you avoid them. 
 -  They're kind of seductive for people who are coming from Java. 
 -  Because they're familiar with seeing things like new string. 
 -  But don't. 
 -  It means something different. 
 -  And in fact, if you say new Boolean and pass it false, you'll get something whose value is false, but it will be truthy. 
 -  Because it's an object and not a Boolean. 
 -  So just stay away from these. 
 -  They're confusing. 
 -  You can directly modify individual objects to give them just the characteristics you want. 
 -  And you don't need to create new classes to do that. 
 -  We can then use the new object as the prototype for lots of other new objects, each of which can then be independently augmented. 
 -  That turns out to be a really powerful pattern. 
 -  So we're going to talk a lot more about this today-- working with the grain. 
 -  There are classical patterns that are possible in this language. 
 -  But they're less effective than the prototypal patterns or other patterns that we're going to see. 
 -  Formal classes are not needed for reuse or extension. 
 -  I've been talking a bit about globals in this language there is a thing called the global object. 
 -  It is a language that dares not speak its name. 
 -  And by that I mean it's not actually represented in the language. 
 -  There's no way of saying directly the global object. 
 -  Although sometimes, as we've found out, this points to it-- sometimes disastrously. 
 -  So knowing that, it's possible to create your own variable called global and assign the reference of this to it. 
 -  If you really need to get at the global object, generally you want to avoid all global variables, because they're dangerous. 
 -  It turns out, in the browser-- and we'll talk more about this this afternoon-- the window object in the browser is JavaScript's global object.
 -  Again, this is one of these places where standards merged. 
 -  But there is no standard which actually says that's how it works. 
 -  And global variables are evil. 
 -  Functions within an application can very easily clobber each other. 
 -  And cooperating applications can clobber each other. 
 -  Anytime you have two things which unexpectedly are going after the same variable, there's a good chance for hazard. 
 -  So use of global namespaces must be must be minimized. 
 -  And the language provides some way of doing that. 
 -  One particular danger is the implied global. 
 -  Any variable which is not properly declared is assumed to be global by default. 
 -  This is really, really bad. 
 -  It makes it really easy for people who do not know or care about encapsulation to be productive. 
 -  But it also makes applications much less reliable. 
 -  And one of the most common cases of this-- talking about it just a moment ago. 
 -  You have a for statement that's looping over i. 
 -  And you forget to declare i as a variable. 
 -  Any variable within any function in which you don't explicitly say, this is a bar, it is assumed to be global. 
 -  And so if you do this in two places in your program, your program is going to fail in ways that are virtually impossible to debug. 
 -  So the best answer to that is never rely on implied globals. 
 - Declare everything. 
 -  So JavaScript gives us some tools for dealing with this. 
-  One is that every object is a separate namespace. 
-  Within an object, its use of keys is completely distinct from all other objects. 
-  So we can use an object to encapsulate our namespace so that it doesn't interfere with other people. 
-  So for example, if I have an application, I can put all of my application in one global variable. 
-  And that way, the likelihood that I'm going to interfere with other things that might be on my page, such as ads or AJAX libraries, is significantly reduced. 
-  And I can do that by simply declaring a global variable and assigning an object to it. 
-  I will then use that as the root for my application. 
-  We can also use function scope. 
-  Since anything declared within a function is not visible outside of the function, that gives us a nice thing to make modules with. 
-  So we can use an anonymous function to wrap our applications. 
-  So say, we've got a trivial application or a trivial widget that runs as part of our application. 
-  And I want to make an object which has two methods, getPoser and showPoser. 
-  And I want those two functions to be able to share private information so that they can work together on it. 
-  And I don't want that information to be going in a global variable. 
-  Because then it could be tampered with by other applications. 
-  So I could use this pattern. 
-  I've got a function. 
-  And I'm defining my common variables and my common functions. 
- And then I'm returning an object literal containing the two methods. 
- These methods, these functions-- because they're defined inside of that outer function-- will have access to all that privacy stuff. 
- So even after the outer function returns, they will continue to enjoy those shared secrets, which is really, really good. 
- And the most interesting thing about this pattern is I'm assigning to trivia not a function but the result of executing that function. 
- And I wrap though the whole invocation in commas to make it clear that this is an expression and not a function value that's being stored in there. 
- And using this pattern, I can make applications which can be quite large and magnificent containing only a single global variable. 
- These parents, one there and in front of function, are not syntactically meaningful in this case. 
- They're just a visual clue to the reader that there is something more going on here than assigning a method to that property. 
- The outer parents are for the benefit of the developer, not for the machine. 
- Syntactically, there is no ambiguity. 
- The interesting thing about this function is that it returns a value. 
- And it's the value that it returns that gets assigned to the trivia. 
- Do I recommend this? Absolutely. 
- This is a really good pattern in this language. 
- And I'll show you lots of variations on this which are also very interesting. 
- So thinking about type, this language trades type safety for dynamism. 
- There are a lot of people who come to this, particularly from Java and other strongly-typed languages, and get really uncomfortable. 
- Because you don't declare types. 
- And so there is no type checking involved. 
- The promise of type checking is that there is a large class of errors that the compiler can find for you early if you declare all the types. 
- And there is some comfort in that. 
- Because the longer it takes to discover an error, the more it costs. 
- And so we want to get the program to be as bug free as possible as early as possible. 
- And type checking helps. 
- So then you come to JavaScript. 
- And there is no type checking.
- It's like, ah, how can I have any confidence that anything is ever going to work? 
- What I've found in my practice is that, the sorts of errors that a type checker can find, I find right away. 
- The amount of testing you have to do is about the same. 
- Just having a type system doesn't mean that you don't have to test.
- You still have to test a lot. 
- You have to be really rigorous in the testing. 
- And the type errors just fall out really quickly. 
- The errors that keep you up at night are not detectable by a type system. 
- And you're always going to have those. And that's a fact of life. 
- The other half of this trade-off is that, because you don't have to specify types, it's easier to program. 
- You never have to cast. 
- You never have to work around the type system. 
- The values are all there. 
- And they all work right all the time. 
- And so you can develop more quickly. 
- And your programs tend to be smaller. 
- And so it's a trade-off. 
- But it's a good trade-off in this language. 
- You think about, why do we have types at all? What's really happening? There is a protection thing. 
- And there's also the reuse thing. 
- But in JavaScript, we've got lots of other reuse patterns that are quite sophisticated. 
- So you could argue about, are these trade-offs good or bad? And ultimately, it doesn't matter. 
- If you're using JavaScript, you have to learn to live with a loosely-typed language. 
- Just to complete our survey of the language, JavaScript has a date function, which was based on the Java date class, neither of which were Y2K ready. 
- I guess they didn't see it coming. 
- But it's been fixed now. 
- And we're proceeding bravely into the 21st century. 
- We've got **regular expressions** in this language, which allows a form of pattern matching. 
    - A regular expression a literal is enclosed in slashes. 
    - This is an example of a regular expression that matches regular expressions-- or at least, that's what I claim. 
    - There's no way you can look at that and have any confidence that it's matching anything useful. 
    - What is that accepting? 
    - What is that rejecting? 
    - Nobody knows. 
    - It's a bizarre notation. 
    - It's really difficult to read. 
    - One thing that frightens me is that regular expressions are commonly used for validation for securing systems. 
    - How can you have any confidence that you're keeping the bad guy out when you're writing tenses that look like that? It's just horrible. 
    - But they're really popular. So that's good. It's good to be popular. 
- The language definition says nothing about **threads**. 
    - There are some JavaScript processors, like the SpiderMonkey processor from Mozilla, which provides some thread support. 
    - Most application environments that use JavaScript, such as browsers, do not. 
    - And that's a really good thing, because threads are evil. 
    - And I believe they should never be used at the application level. 
    - Because they're just too difficult to reason about. 
    - So JavaScript doesn't have them. 
    - And I think that's good.