## ... If JavaScript is your first language.

I probably understood closures in JavaScript for a good six months or so before I was confidently able to convince myself that this was the case. I think this is because they’re usually discussed as some advanced, exotic, weird feature that only the real pros understand. Now that I’m a little more confident with my JavaScript understanding, I think that this belief come primarily from JavaScript as a Second Language (JSL) developers, who came to JavaScript after already having a solid understanding of another programming language. From their perspective, I can definitely accept that closures are a bit exotic, a bit weird, a little crazy maybe. However, if you’re new to programming, and have chosen to start with JavaScript (not a bad plan imho), I don’t think you’ll find them that scary once you get to know them. If you have a decent understanding of the basics of JavaScript (variables, objects, functions, scope), you probably already understand all the concepts that make closures so powerful, it’s just a matter of putting it all together.

When you get right down to it, closures aren’t so much a feature of JavaScript, as they are a consequence of several other basic features of the language. To a JSL developer, these other basic features are unusual, therefore the consequence seems unusual. If you’re untainted by the paradigms of other languages, however, the consequence might even seem natural and obvious. Let’s start from the beginning. I suggest you try running these examples in your console just to get used to what you’re expecting to happen.

## Scope in JavaScript

A quick review on scope.

JavaScript has global variables, like this:

	var value = "cheese";
    
This variable is defined _globally_, so I can get that same value back anywhere I please. It’s visible to code in functions, loops, and conditionals so long as it doesn’t get re-defined anywhere.

    var myVar = “cheese”;
    
    function say() {
     return “Say “ + myVar;
    }
    
    say(); // returns “Say cheese”
    
With me so far? Simple, right?

## Scope in Functions in JavaScript

JavaScript has _function scope_, meaning that variables declared inside a function are visible anywhere inside that function. Basically, a variable defined inside a function behaves the same way as a global variable, except that it’s only defined within the walls of the function. This means that the variable is visible to code in functions, loops, and conditionals inside the function, so long as it doesn’t get re-defined.

    function say() {
     var myVar = “cheese”;
     if (2 > 1) {
       return “Say “ + myVar;
     }
    }
    
    say(); // returns “Say cheese”
    
If we try to get the value of `myVar` outside of the `say` function, we get an error:

		console.log(myVar); // Uncaught ReferenceError: value is not defined

Obvious, right? Let’s try another one. What happens if we put one function inside another function?

    function say() {
     var myVar = “cheese”;
     function nested() {
       console.log(“Say “ + myVar);
     }
     nested();
    }
    
    say(); // “Say cheese”
    
What just happened? We made a function `say`, in which we define a variable `myVar`. Then, we made a function `nested` inside of `say`, which logs `myVar` out to the console. Then we called our `nested` function. The `nested` function logs the value of `myVar` as “cheese”, which is what we defined it as in the parent `say` function.

Still not so bad, right? Not terribly useful yet, but we’ll get there.

## Functions that make functions

Here’s where things start to get interesting. In our last example, we called the `nested` function from inside the `say` function, right after we defined it. That seems kind of silly. What if we had a function that returned another function, instead of returning a value? Let’s see how that effects variable scope:

    function builder() {
     var house = “224 Victor”;
     return function() {
       return “I live at “ + house;
     }
    }
    
    var address = builder();
    
    address(); // I live at 224 Victor

So this output might seem obvious at first glance, but if you think about it, something a bit unexpected (and pretty cool) is happening here. When we assign the variable `address` to the output of the `builder` function, we called the function, and it returned a function. Now that we’ve assigned our inner function to a variable, the parent function has returned and is no longer in control. It’s returned. Finished.

BUT (and here’s the crazy part), the inner function (which is now attached to `address`) still has access to `builder`’s scope. Pretty wacky. Pretty exotic. That’s closure. Is it obvious yet that this could be super useful? Probably not. Stay with me.

So let’s talk about this variable `house`. It’s inside a function, so it’s hidden from the global scope. That’s a good thing. The function assigned to `address` came from the function in which `house` is defined, meaning that it has access to the value of `house`. This is also a good thing. Now we’ll see why:

## Using Closures

Let’s start with one of the simplest and most obvious examples: generating a unique ID.

Let’s say we need to assign some IDs to something. Sheep maybe. We need to give each of the sheep in our flock a unique ID. Maybe those IDs are sequential numbers starting at 100. How would we code this?

    var id = 100;
    
    function createSheepId() {
     id++;
     return id;
    }
    
    createSheepId(); // 101
    createSheepId();//102

Cool, this is doing what we want. BUT there’s a pretty huge problem with this. `id` is a global variable. Global variables are very bad. Our `id` variable could be modified from anywhere else in our program, or even by a malicious hacker or sheep thief who wants to lower our sheep count so we don’t notice they’re missing. Or something.

Using what we know about functions and scope in JavaScript, we can make a function that hides our `id` variable from the rest of the program, and returns a function that has privileged access to this variable. Then we just assign a variable to that returned function, call it, and we'll get our unique ID. 

    function createIdCreator() {
     var id = 100;
     return function() {
      id++;
      return id;
     }
    }
    
    var createSheepId = createIdCreator();
    
    createSheepId(); // 101
    createSheepId(); // 102
    
Pretty cool right? Our syntax looks a bit clunky though, and it’s pretty crappy to have to assign our function to a new variable, and also `createIdCreator` is a terrible name. There’s a nicer way to do this, using a common pattern called an Immediately Invoked Funcion Expression (IIFE). An IIFE is just a (nameless) function that has a pair of round brackets immediately following its definition, which calls the function right after defining it. It looks like this:

    (function(){
     return 1 + 1;
    }()); // 2
    
It's also called a Self Executing Anonymous Function, because it executes itself. And is anonymous (nameless). 

Why does this function have parentheses around it? Because otherwise it’d be a syntax error, that’s why. If you’re really curious, [read more about IIFEs](http://benalman.com/news/2010/11/immediately-invoked-function-expression/).

So anyways, back to sheep. We can use an IIFE to avoid having to create a function-maker function, by assigning a variable to the result of invoking our IIFE. If that sounds confusing, observe an example:

    var greeting = (function(){
     return function() {
      alert(“ohai”);
     }
    }());
    
    greeting(); // alerts "ohai"
    
Okay let’s think about this for a second. What does `greeting` refer to? The IIFE? Not quite. `thing` refers to the result of invoking that anonymous function. In this case, since our IIFE just returns another function, `greeting` refers to the function being returned (the one that alerts “ohai”). The fun thing about this pattern is that we can do more than just return a function: we can create some variables inside our IIFE that won’t be visible to the outside world:

    var greeting = (function(){
     var person = “Mark”;
     return function() {
       alert(“ohai “ + person);
     }
    }());
    
    greeting(); // “ohai Mark”

We can use this pattern to make our sheep counter a little bit less clunky:

    var createSheepId = (function() {
     var id = 100;
     return function() {
       id++;
       return id;
     }
    }());
    
    var id1 = createSheepId(); // 101
    var id2 = createSheepId(); // 102
    
The resulting `createSheepId` function has privilegd access to the `id` variable, even though the anonymous function that defined it has already returned. This variable persists, but can only be modified by the returned function. 

Very useful. Very powerful.

## Module Pattern

So useful and powerful, in fact, that it’s the basis of a whole JavaScript design pattern referred to as the "module" pattern.

Let’s say we wanted to be able to reset our sheep ID generator. Maybe we have a new herd. Or we want to let our neighbour count their sheep. Something like that. Anyways, we want our sheep ID program to be able to create new IDs and reset the ID creator back to 100. Since both functions will need access to the same variable, we’ll need to wrap them both in the same closure. Since we’ll need our IIFE to return more than one function, we’ll have it return an object with two methods.

    var sheepId = (function() {
      var id = 100;
      return {
       newId: function() {
         id++;
         return id;
       },
       resetId: function() {
         id = 100;
       }
     }
    }());

The variable `sheepId` is still the result of calling the IIFE, but this time that is an object instead of a function. To call these functions, we just use the method dot notation:

    sheepId.newId(); // 101
    sheepId.newId(); // 102
    sheepId.resetId(); 
    sheepId.newId(); // 101