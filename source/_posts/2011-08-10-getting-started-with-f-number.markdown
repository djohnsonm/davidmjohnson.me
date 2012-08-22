---
layout: post
title: "Getting started with F#"
date: 2011-08-11 15:38
comments: true
categories: [f#, migrated, functional-programming]
---

##What is F#?
F# is a multi-paradigm programming language combining functional, OOP and imperative styles. It is based on the ML variant of languages and is most closely related to OCaml. It is a strongly-typed language that utilizes type inference to resolve data-types at runtime. With the release of Visual Studio 2010, F# is now a first class citizen of the .NET Platform and fully compatible with most .NET libraries (including but not limited to WCF, WPF, XNA). F# is a great example of a specific type of programming called “functional programming.” The functional programming genre seeks to describe more of the “what” and less of the “how” in regards to programming. C# is slowly including more functional aspects into it overtime.

##Origins
Most people don’t know this but C# actually owes a lot to F#. Don Syme, the originator of F#, was responsible for creating the concept of Generics in C# and also spent time working on LINQ. These features tend to be more functional in nature. Generics allow us to see code as data and LINQ is a very declarative way to retrieve and modify data from a variety of sources.

##Predictions
General consensus about functional languages like F# is that they are primarily reserved for the halls of academia (i.e. LISP) but all that is about to change. As demand for multi-core and asynchronous computing increases the future of F# seems bright.

##Why should you care?
{% blockquote Jon Skeet and Tomas Petricek, Real World Functional Programming %}
“Today we are facing new challenges and trends that open the door to functional languages. There has never been a better time to learn them. We need to write programs that process large sets of data and scale to a large number of processors or computers. We want to write programs that can be easily tested. We want to be able to express our logic in a declarative way which expresses results without explicitly specifying execution details–making the code easier to understand and reason about. All of these trends are embodied in functional programming…
{% endblockquote %}

##What F# isn’t
A replacement for C#. While both are very effective langauages in their respective domains and both can overlap in regards to functional and OOP aspects (as well as intermingle via interoperability), C# will most-likely be the first candidate for line-of-business applications due to its adoption levels, performance and pervasiveness in the market. Personally, I believe that F# will be important going forward given the convergence of a few market factors such as data growth, multi-core computing and the desire for elegance in code. Concurrent applications are harder to maintain in a pure object-oriented environment. Race conditions and dealing with thread locking can become very tedious as applications are expected to scale to meet market demand. The functional paradigm does not need to manage state and provides no “side effects”, thus making it much easier to scale applications across multiple cores.

##Who’s using F#?
-Financial Services 
–Algorithmic Trading 
-Statistical analysis 
-Application Development 
-Web Development 

##How Do I Get Started?
I’ll go through some of the basics of F# in this lesson and provide follow-ups to links and books at the end. Let’s get started with our first example in F#, but before we can do that we need to set up our development environment. In Visual Studio it is possible to create a new F# project by hitting “Ctrl-Shift-N” and creating a F# Console Application.
But for our purposes we will just use F# interactive which is a stand-alone command line utility. Great for rapid prototyping and learning the language. So navigate to (C:\Program Files (x86)\Microsoft F#\v4.0\fsi.exe). (Aside: You have to right-click the toolbar and select “paste” in order to insert code).

{% codeblock Simple Multiplication Function lang:fsharp %}
let multiply x y = x * y;
{% endcodeblock %}

This function might seem ridiculous at first glance (it did for me), especially if you’re coming from a pure C# background, but let’s translate it to C# and see if it makes anymore sense.

{% codeblock Simple Multiplication Function in C# lang:csharp %}
public int multiply (int x, int y) 
{
  return x * y;
}
//or
Func<int,int,int> multiply = (x, y) => { return x * y; };
{% endcodeblock %}

##Let
The "Let" keyword might be familiar for those who have worked with LINQ. It might be tempting to think that this keyword is used for assignment. Think of it more as binding to a value rather than assigning a value. Since functions are first class citizens in F# and they can be passed around as parameters and contain return values.

##Val
The “val multiply: int -> int -> int” can be thought of as: A function named “multiply” that takes an integer as a parameter returns a function that takes an integer as a parameter and outputs an integer. Confusing you say? Yes, absolutely. Why didn’t I just say: “A function that takes in two ints and returns and int.” Because of currying.

##Spicy Curry
There is something called “currying” in F# that you should be wary of. Currying is the process of partial function application. This allows us to specify only one parameter of a function to receive a function that takes in the other parameter. Here is an example in code:

{% codeblock Curried Multiplication Function in F# lang:fsharp %}
> let multiply x y = x * y;;
val multiply : int -> int -> int
> let curriedMultiplyX = x 5;;
val curriedMultiplyX : (int -> int)
> curriedMultiplyX 7;;
val it : int = 12
> curriedMultiplyX 9;;
val it : int = 14
{% endcodeblock %}

Notice how I only specify one parameter for the function named “x”? “x 5″ is saying that I want to return a function that is based on the function named “x” but applies the number 5 as a parameter every time basically.

##Visualization
F# can also be utilized for visualiztion. Below is a Windows forms example. Copy and paste it into your F# interactive console.

##Windows Forms Example
In F# Interactive write the following code:

{% codeblock F# Windows Forms Example lang:fsharp %}
open System
open System.Windows.Forms
let form = new Form(Text="Welcome to F# Win Form Style")
let label = new Label(Text = "This is a label!")
form.Controls.Add(label)
form.Show();;
{% endcodeblock %}

As you can see it’s pretty easy to prototype in F# using the interactive console.
{% img https://dl.dropbox.com/u/10021156/FViz1.png %}

{% codeblock Recursive Factorial Function in F# lang:fsharp %}
let rec fac param = if param < 1.0 then 1.0 else param * fac(param - 1.0);;
{% endcodeblock %}

This code calculates the factorial of any number passed in using familiar “if..then” logic. Also, if you are getting strange results, remember that indentation matters in F#, but its really doesn’t inhibit your productivity.

{% codeblock Recursive Factorial Function with Pattern Matching in F# lang:fsharp %}
let rec (!) x =
| 0 -> 1
| _ > n * factorial (n - 1)
{% endcodeblock %}

This code should be pretty self-explanatory. The “rec” prefix means that this function is recursive. The (!) means that the name of this function is “!” (this is not operator overloading but a way to bind symbols to function names). The underscore value (“_”) is used as a placeholder variable for every possible value of n. The pipes are kind of like a switch statement on steroids. The value will navigate down the pipes and return the value “1″ if n is “0″ or “n-1″ for everything else and then multiply by the return value recursively.

Asychronous and Concurrent Programming using CPU bound operations.

Now for the fun stuff.
So let’s take our little factorial example see if we can calculate the factorial of every number from 0 to 51,000 utilizing Parallel Computing. Here’s the code for that. WARNING: If you don’t have a lot of memory then bring it down from 51,000 to something more manageable. 

{% codeblock Async and Parallel Factorial Function lang:fsharp %}
(* Declare our function *)
let rec fac param = if param < 1.0 then 1.0 else param * fac(param - 1.0);;
(* Asynchronous Example *)
let result = Async.Parallel [for i in 0.0..51000.0 -> async { return fac(i) }] |> Run.Synchronously;;
{% endcodeblock %}

I’ll perform the operation on an 8-Core machine with Diagnostics so we can visualize the processor usage better.

{% img https://dl.dropbox.com/u/10021156/blog/8CoreSynch1-1024x121.png %}

{% codeblock Simple Computation Results (Synchronous, non-Parallel) – 41 seconds lang:fsharp %}

open System
open System.Diagnostics
let rec fac x = if x < 1.0 then 1.0 else x * fac(x-1.0)
let watch = new Stopwatch()
watch.Start()
let simpleComputation = [for i in 0.0..51000.0 -> fac(i)]
watch.Stop()
let result = watch.Elapsed.Seconds.ToString()
printfn "%s" result;;
 
(* RESULTS *)
val fac : float -> float
val watch : System.Diagnostics.Stopwatch
val simpleComputation : float list =
  [1.0; 1.0; 2.0; 6.0; 24.0; 120.0; 720.0; 5040.0; 40320.0; 362880.0;
   3628800.0; 39916800.0; 479001600.0; 6227020800.0; 8.71782912e+10;
   1.307674368e+12; 2.092278989e+13; 3.556874281e+14; 6.402373706e+15;...]
val result : string = "41"
{% endcodeblock %}

{% img https://dl.dropbox.com/u/10021156/blog/8CoreAsynParallel-1024x124.png %}

{% codeblock Complex Computation Results (Asychronous, Parallel) – 13 seconds lang:fsharp %}
open System
open System.Diagnostics
let rec fac x = if x < 1.0 then 1.0 else x * fac(x-1.0)
let watch = new Stopwatch()
watch.Start()
let complexComputation = Async.Parallel [for i in 0.0..51000.0 -> async { return fac(i) }] |> Async.RunSynchronously;;
watch.Stop()
let result = watch.Elapsed.Seconds.ToString()
printfn "%s" result;;
 
(* RESULTS *)
val fac : float -> float
val watch : Stopwatch
val complexComputation : float [] =
  [|1.0; 1.0; 2.0; 6.0; 24.0; 120.0; 720.0; 5040.0; 40320.0; 362880.0;
    3628800.0; 39916800.0; 479001600.0; 6227020800.0; 8.71782912e+10;
    1.307674368e+12; 2.092278989e+13; 3.556874281e+14; 6.402373706e+15;
    ...|]
{% endcodeblock %}

Results: Non-parallel version is 3.15 times slower. Parallel version 315% faster!

##Conclusion
There are a lot of things to be excited about in F#. Learning F# will definitely make you a better C# programmer by providing you with more of a functional mindset. I think in the future it would be a fair bet to say that more .NET programmers will begin to inter-operate between these two languages.

##Resource Round-Up
*Interesting F# project to watch: WebSharper
*Check out my F# stock ticker app on my github account. This is a small project that will read information from yahoo’s financial stock history for any given stock, summarize the data using List functions such as “map” and “reduce” and will then output the results to a C# console application. This code was derived from Luca Bolognese’s original talk at PDC on F#. Luca is actually using F# in an algorithmic trading setting now at Credit Suisse.
*Best Getting Started Video Tutorial on F#
