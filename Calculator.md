# Calculator Language

> A compiler is just a staged definitional interpreter. What's the problem?

Don't worry about the above quote. Let's design a programming language, and write a compiler for it!

## Design

What should the language look like? 

JSON. To be precise, the source code of the program is a JSON file. By doing so, we do not need to parse the program from a textual format - it is already done. Even though parsing makes your program look nice (because it is not in JSON), and parsing is deep and useful knowledge, we will completely skip parsing: it is very complex, and if we talk about it, we will spend most of our time talking about it, leaving little time for everything else. 

OK. What feature should it have?

Almost none. The more features a language has, the harder it is to design and implement. To ease such a task, we will be doing, essentially, agile development. We will design and implement a simple language, refactor the code, add more features to the language, and repeat. This way, the learning curve will be smoother, and you get to see the conflict between different programming language features, and the conflict between performance and expressivity, as the language evolves.

## Language Definition

Our language is so simple that your (non-scientific) calculator will laugh at it. It has exactly three features: number (an integer), plus (add two numbers), and multiply (time two numbers).

Now we have to specify the JSON format for our program.

For a number x, the format is `{type: "Literal", value: x}`.
For a + b, the format is `{type: "Plus", left: a, right: b}`.
For a * b, the format is `{type: "Multiply", left: a, right: b}`.

As an example, (1 + 2) * (3 + 4) is written as

    {type: "Multiply", 
      left: {type: "Plus", 
        left: {type: "Literal", value: 1}, 
        right: {type: "Literal", value: 2}},
      right: {type: "Plus",
        left: {type: "Literal", value: 3}, 
        right: {type: "Literal", value: 4}}}

What a pain in the ass. Even though we don't have a parser, we can still write a pretty printer: it outputs the code in a prettier, actually human-readable format.

## Pretty Print

We will write it in Java, cause everyone knows Java. I am gonna code it with you, step by step. This is like pair programming.
    
    static String prettyPrint(JSONObject json) {  
      throw new RuntimeException("...");  
    };

We start by getting the 'type' field from the JSON, and see which case it is:
    
    static String prettyPrint(JSONObject j) {  
      String type = j.getString("type");
      return switch (type) { 
        case "Literal" -> throw new RuntimeException("...");  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected: " + type);  
    };  

Literal is easy - we just print the value: 

        case "Literal" -> String.valueOf(j.getInt("value"));  

Unlike Literal, Plus and Multiply contain more JSON as children, in the left and right fields, which we have to handle by calling prettyPrint() again - recursion! 

        case "Plus" -> "(" + prettyPrint(j.getJSONObject("left")) +  
          "+" + prettyPrint(j.getJSONObject("right")) + ")";  
        case "Multiply" -> "(" + prettyPrint(j.getJSONObject("left")) +
          "*" + prettyPrint(j.getJSONObject("right")) + ")";  

Now calling prettyPrint() on our example return "((1+2)*(3+4))". Look pretty neat.

## Evaluate

Moving on to the evaluator, which takes a JSON, and returns an Int:

    static int evaluate(JSONObject j) {  
      throw new RuntimeException("...");    
    }

Get the type and match on it...

    static int evaluate(JSONObject j) {  
      String type = j.getString("type");  
      return switch (type) {  
        case "Literal" -> throw new RuntimeException("...");  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected value: " + type);  
      };  
    }

Handle the base case (no recursion, simple)...

        case "Literal" -> j.getInt("value");  

Handle the recursive case (recurse into the children, and combine the result)...

        case "Plus" -> evaluate(j.getJSONObject("left")) + evaluate(j.getJSONObject("right"));  
        case "Multiply" -> evaluate(j.getJSONObject("left")) * evaluate(j.getJSONObject("right"));  

If you compare evaluate() to prettyPrint(), you will find that they are almost the same! The biggest difference is that one of them has + as an operator that adds two Int, and one of them has + as "+" the string. 

This is not a coincidence. Think about how we can process program in our calculator language:
 - Given a program in the JSON format, we must retrieve its type so we can work on it
 - For the base case, we can immediately return the correct value
 - For the recursive case, we have to handle the children by recursing

Without any of the three steps, the program is incorrect, and cannot be fixed without essentially re-introducing it. This pattern is thus universal: you will see it again, a lot.

## Meta-Language, Object-Language, and the Definitional Interpreter
Let's step back and reflect on what we had done: in less than 10 lines of code, we had implemented an Interpreter! What is the magic? Before I answer this question, some terminology: when we use a program X to manipulate another program Y, the language X is in, is called the metalanguage, and Y's language is the objectlanguage.
A definitional interpreter is when we use metalanguage's feature to implement the corresponding objectlanguage's feature. In this case, evaluate() is a definitional interpreter, because we use Java's Int to implement Calculator's Int. Definitional interpreters are known for their simplicity because they essentially do nothing.

However, despite doing nothing, they are very useful. The most straightforward and popular use of a definitional interpreter is to give new syntax or to provide eval(), for already-existing features, also known as inventing a novel programming language. We could take a definitional interpreter, and add a new language feature that we actually implement. We could also take a definitional interpreter, add some print to every case, and now we had a tracing debugger. Or, as the quote at the beginning of the chapter dictate, a compiler is also a definitional interpreter but slightly modified. Essentially, a definitional interpreter is a solid foundation, allowing you to make a bunch of modifications to it, to get all sorts of useful things.

## Refactoring
Manipulating JSON is hard! I have to remember the name and the type of everything, and that sure is tiresome! I am gonna refactor the JSON (JavaScript object) into Java Objects since we are writing Java not JavaScript.

    static class Expr {}  
  
    static class Lit extends Expr {  
      int val;  
  
      Lit(int val) {this.val = val;}  
    }  
  
    static class Plus extends Expr {  
      Expr left, right;  
  
      Plus(Expr left, Expr right) {  
        this.left = left;  
        this.right = right;  
      }  
    }  
  
    static class Mult extends Expr { // ...

The class definitions.

    static Expr mkLit(int val) {return new Lit(val);}  
  
    static Expr mkPlus(Expr left, Expr right) {return new Plus(left, right);}  
  
    static Expr mkMult(Expr left, Expr right) {return new Mult(left, right);}

Some shorthands.

    mkMult(mkPlus(mkLit(1), mkLit(2)), mkPlus(mkLit(3), mkLit(4)));

Now our example looks much better - it is just a one-liner.

After the above refactoring, besides the conversion, the JSON is nowhere to be seen - this is expected. From now on, we will simply not talk about JSON, and focus on Expr. Conversion from JSON to Expr can be easily written, by recursing on the JSON object and emitting the corresponding Java object. And we do not even need to work with text, or JSON, to begin with - the above Expr construction is an easy way to test our programs. We can also imagine adding a parser that goes straight from text to Expr later on - we don't have to do it now, our time is better spent focusing on the compiler itself.
  
    static int eval(Expr expr) {  
      if (expr instanceof Lit) {  
        return ((Lit) expr).val;  
      } else if (expr instanceof Plus) {  
        return eval(((Plus) expr).left) + eval(((Plus) expr).right);  
      } else if (expr instanceof Mult) {  
        return eval(((Mult) expr).left) * eval(((Mult) expr).right);  
      } else {  
        throw new RuntimeException("Unexpected value: " + expr.getClass());  
      }  
    }

We have to now fix the definition for prettyPrint() and evaluate(). While we are at it, let's also move prettyPrint() into Java's toString(), and rename evaluate() into eval(): our code is, and will, remain short and concise. The code for pp(), likewise, is a mechanical transformation, which we will skip presenting.

One problem with the above code is that it relies heavily on instanceof, and downcasting, which is frowned upon in Java. This could be fixed by making pp() and eval() abstract functions in Expr, and having each subclass override it.

    // In Lit
    public String toString() {return String.valueOf(val);}  
  
    int eval() {return val;}

    // In Plus
    public String toString() {return "(" + left.toString() + "+" + right.toString() + ")";}  
  
    int eval() {return left.eval() + right.eval();}

The @override function for Lit and Plus. The case for Mult is almost the same, so I will not show it here.

Alas, we can't write a compiler yet, as our language is so trivial a compiler is pointless. Let's add some more features.

## Input

Right now our language is really, really boring. It reacts the same (output a constant number) no matter what, while most program behaves differently according to the context.

The easiest way to do this is to add variables to the language. For now, for the sake of complexity, all variables are defined externally and inputted by the user when the program run. We will add defining and assigning to variables some other times.

    static class Var extends Expr {  
      String name;  
      Var(String name) {this.name = name;}  
    
      public String toString() {return name;}  
    }

OK. On to eval(). Hmm...

    int eval() {  
      throw new RuntimeException("...");
    }

What do I put in here? Seems like we are stuck! For a good reason: we now have unknown variables in our language, but we don't know which value they are. Luckily, they are all user-defined variables, so we can require our user to pass in a Mapping, environment(env) from variable to int. 

    // In Expr
    abstract int eval(Map<String, Integer> env);

Now our definitions for Lit, Plus, and Mult are broken. We can fix Lit by accepting the env and doing nothing for it. For Plus and Mult, we pass env into the recursive call. 

    int eval(Map<String, Integer> env) {return env.get(name);}

Now for Var, we look up the value from the environment.

Pretty Good! I wrote another example, which multiplies 2 n*n matrices, where n = 2, and add up all the cell in the resulting matrix, into a single number. The code is hidden because it is not too related to what we are doing right now. Running the example on 2 matrices with all cells as 1 will give 8, which is expected. However, printing the matrix will give:

    ((((0+((0+(a_0_0*b_0_0))+(a_0_1*b_1_0)))+((0+(a_0_0*b_0_1))+(a_0_1*b_1 ...

Urgh. What a mess! Look like there is a bunch of 0 in our program. We can write a function, simp(), just like pp() and eval(), to simplify the input program and remove them.

## Simplification

    // In Expr
    Expr simp() {return this;}
    
The case for Lit and Var is trivial: there is nothing to simplify, so we just `return this;`. In fact, we had moved this code into the base class, Expr, because it is always correct to simplify, by doing nothing!

Let's get back to programming, and fill out the case for Plus.

    Expr left = this.left.simp();  
    Expr right = this.right.simp();
    return mkPlus(left, right);

We begin by traversing left and right, simplifying them. Note that the above code is universal - we can also do this for the simp() for Mult (of course, we have to change mkPlus into mkMult). This is because if a == and b == y, mkPlus(a, b) == mkPlus(x, y). Even when we have nothing to do for a Node, we can and should still optimize by recursing. This way its children can do its things. Now, onto some real optimization for Plus.

      // After left, right is defined, before mkPlus
      if (left.equals(Cal.mkLit(0))) {  
        return right;  
      } else if (right.equals(Cal.mkLit(0))) {  
        return left;  
      }

If left or right is 0, we just return the other Expr. Note that I had override equals for all the Expr. I won't show them because they are not interesting, but you can see them in the code. Another case:

    // After left, right is defined, before mkPlus
    if (left instanceof Lit && right instanceof Lit) {  
      return mkLit(((Lit) left).val + ((Lit) right).val);  
    }

If left and right are both Lit, we can do our simplification by adding them up.

The case for Mult is very similar:

 - 0: Recurse on the children
 - 1: If both expr is Lit, multiply the val and return a Lit
 - 2: If any expr is 0, return 0
 - 3: If any expr is 1, return the other expr
 - 4: otherwise return mkMult(left, right)

Now calling example.simp().pp() will give a string of length 126, as opposed to length 146, and we can see, with our eyes, that it is a bit prettier:

    (((((a_0_0*b_0_0)+(a_0_1*b_1_0))+((a_0_0*b_0_1)+(a_0_1*b_1_1)))+((a_1_ ...

## Deja Vu

If we look at our simp(), we will see that it is very much like eval()! If we give it an Expr with no Var in it, it will always return a Lit! And the code that will be executed, in that case, is exactly the code for a definitional Interpreter. This is not a coincidence: the definitional Interpreter is a recurring echo, which we will see multi, multiple times in the book.

## Staging

Ignore the scary title for now. Let's try to make our Interpreter faster. I had increased n from 2 to 4, so we are now multiplying 2 4*4 matrices, and run the eval() in a loop, and profiling the resulting code.

    public static void profileEval(int n, int length) {  
      Expr example = getExample(n);  
      Map<String, Integer> env = getExampleEnv(n);  
      for (int i = 0; i < length; ++i) {  
        example.eval(env);  
      }  
    }

The profiler tells me that most time is spent in `Map.get()`. Why? What is in a Map?

There are multiple ways to implement a Map from String to Int. The most notable examples are search trees, hash maps, and tries.
However, in order to look up a key, they all need to traverse the String to compare/compute the hash/traverse the trie and look at multiple buckets/nodes to find the value. Looking at multiple places is not good, as it requires multiple memory fetches, which may be a cache miss and stall the pipeline, and it also requires conditional jump, which may fail the branch predictor and require conditional jump. In short - we will like to look only once. If env is an Array, there is fewer cache miss as the values are compactly stored, and there are no failed branch prediction because there is no branch.

    // Inside Expr
    abstract int yolo(int[] env);

We will call our new function yolo because you only look once (into the array).
The change for Lit, Plus, Mult are all mechanical and not shown, but we are stuck on Input again: we don't have a way to go from String name to an index in env! We can fix this, by adding a mapping from String to index as an argument to yolo:

    // In Expr
    abstract int yolo(Map<String, Integer> loc, int[] env);
    // In Var
    int yolo(Map<String, Integer> loc, int[] env) {  
      int idx = loc.get(name);  
      return env[idx];  
    }

How do we turn the old Map<String, Integer> env, to a Map<String, Integer> loc and int[] env though? We can walk the whole Expr, and put every unique Var into a Map once. 

    // In Expr
    abstract void locate(Map<String, Integer> loc);
    // In Var
    void locate(Map<String, Integer> loc) {  
      if (!loc.containsKey(name)) {  
        loc.put(name, loc.size());  
      }  
    }

The other cases are uninteresting.

    static int[] envToLocEnv(Map<String, Integer> env, Map<String, Integer> loc) {  
      int[] arr = new int[loc.size()];  
      for (Map.Entry<String, Integer> x : loc.entrySet()) {  
        arr[x.getValue()] = env.get(x.getKey());  
      }  
      return arr;  
    }

Now, a function to turn the old Env into the new Env. Now we can profile our code again:

    public static void profileYolo(int n, int length) {  
      Expr example = getExample(n);  
      Map<String, Integer> env = getExampleEnv(n);  
      Map<String, Integer> loc = new HashMap<>();  
      example.locate(loc);  
      int[] locEnv = envToLocEnv(env, loc)  
      for (int i = 0; i < length; ++i) {  
        example.yolo(loc, locEnv);  
      }  
    }

This isn't any faster. Duh - we are still calling .get(name) inside yolo(), but the point is to not call it! Let's take a moment to look at yolo() for Var and think.

    int idx = loc.get(name);  
    return env[idx];  

One thing to note is that the code has two lines. One line uses only loc, and one line uses env, alongside the value produced by loc. Furthermore, loc is calculated using Expr only, and env is defined by the user. If we have an Expr, which will be run multiple times, we can execute `int idx = loc.get(name);` only once, store the result, and only execute `return env[idx];` every time we run the Expr. This way, we will spend some time when we get the Expr, to precompute indexes, but then we will be very fast!

    // In Expr
    abstract Function<int[], Integer> again(Map<String, Integer> loc);

We can represent it by a Function returning a Function. The idea is, for the same Expr, we will call again once, but we can call the result multiple times, each time representing a run of the program.

    // In Lit
    Function<int[], Integer> again(Map<String, Integer> loc) {return env -> val;}

The case for Lit is simple. We return a Function, which takes env and ignores it. It is just like the old yolo() function.

    // In Plus
    Function<int[], Integer> again(Map<String, Integer> loc) {  
      Function<int[], Integer> left = this.left.again(loc);  
      Function<int[], Integer> right = this.right.again(loc);  
      return env -> left.apply(env) + right.apply(env);  
    }

In Plus, we recurse, just as always. One important thing to notice in this code: the one-liner

    return env -> left.again(loc).apply(env) + right.again(loc).apply(env);
 
is not what we want: every time the inside function is executed, we are calling again() again, but we only want to call again() once.

    // In Var
    Function<int[], Integer> again(Map<String, Integer> loc) {  
      int idx = loc.get(name);  
      return env -> env[idx];  
    }

The case for Var. The lambda perfectly separates the two worlds - a world where we only have loc, but we can do heavy computation (because it is run once), and a world with env, but we want to execute ASAP (because it is run multiple times). The world is called a stage, and usually, there are two stages: the compile and the run time.

Wait, compile time? We separate our interpreter into two stages, run one stage once and run the next stage multiple times. A compiler also works in two stages, compiling the program once and executeing it many time.  But note that our compiler is very much like our definitional interpreter, eval(), the difference only being splitting lookup into two phases, and the stage separation. This is what "A compiler is just a staged definitional interpreter" mean! Hurray! Now we have a compiler with 20 lines of code!

Some profiling shows that our code is now about 4x as fast, by removing the hash table lookup at runtime. Some profiling will show that the bottleneck is no longer HashMap or any particular Java library call, but time is instead spent during all the recursive calls.

## Code Generation

What else is there to optimize? To understand this, we have to understand that calling a nonstatic method in Java is somewhat slow, as opposed to e.g. indexing into an array, or doing int addition. Objects, unlike int in an array, are not tightly packed together (on the heap). This means accessing objects takes possibly a few orders of magnitude slower than accessing local variables (on the stack), or sequential access to an array. (And no, putting Objects into an array will not help because they are boxed). Furthermore, calling a nonstatic method will do dynamic dispatch, which requires jumping to an unknown place in the code, which will induce pipeline stall which is also very expensive.

However, note that in our calculator language, there is no Object, and there is no dynamic dispatch. So, where are they from? We introduced them by calling the Function returned by again(). This is known as the interpretive overhead (A compiler can still have interpretive overhead because an interpreter (in this case, JVM Function) might be used to interpret the compiled result.).

To combat this, we can, instead of calling the returned value of again(), find a more efficient method to execute it. We start by using a class, LExpr (LocatedExpr), to store the return of again(), now renamed to located:

    abstract static class LExpr {  
      abstract int eval(int[] env);  
    }

located is changed to return anonymous class, instead of an anonymous function:

    LExpr located(Map<String, Integer> loc) {  
      return new LExpr() {  
        int eval(int[] env) {  
          return val;  
        }  
      };  
    }

OK, back onto the point. What is more efficient than the Java Runtime? The Java Compiler! Instead of turning LExpr into Function, we can turn LExpr into a String, which represents some Java code. We can then compile the java code and execute it.

    // In LExpr
    abstract String compile();
    // In Lit.located
    String compile() {return String.valueOf(val);}
    // In Lit.Plus
    String compile() {return "(" + left.compile() + "+" + right.compile() + ")";}

Huh. Look very familiar...

    String compile() {return "env[" + idx + "]";}

Isn't this just the code for eval(), our definitional interpreter, only you put it in quotation marks? Precisely. This is called quoting, where we, instead of executing a code, just store the representation of that code, so we can do stuff with it later. Again, note how we are using Java's feature to implement Calculator's feature, only this time, we use the Java compiler instead of the Java runtime. You might recall that this code is also exactly our pp(), which is not a coincidence. PrettyPrinting  output a representation of the code, which is also what compile() does as oppose to eval(). With compile() coded up we can execute the code 10x as fast then LExpr's eval() - which, keep in mind, is 4x as fast then Expr's eval. So, we achieve a whooping 40x speedup!

## More Refactoring

The code for Plus and Mult has way, way too much similarity. We can refactor them into an ApplyBinOp class, which has two children, left and right, alongside a BinOp, an abstract class, with the function `abstract int eval(int left, int right)`. The code for Plus and Mult is now merged, with the only difference being they use different BinOp.

## Conclusion

If you want a 1-day intro to compiler, this is it. Stop reading.
This chapter is a microcosm of the whole book: It contains programming language design, evaluation, performance debugging by profiling and reasoning about hardware, optimization, staging, and finally code generation, with all of them interacting with the definitional interpreter. The rest of the book is an extension of all we saw before, only in more depth.

After all, a compiler isn't a menacing dragon, to be conquered by a knight, but a dragon-friend, whom once you befriend, will be amazed by its beauty.

## Challenge

- 0: rewrite LExpr's compile to generate python code instead of java code. When you do that, of the three languages (Calculator, Java, Python), which are the metalanguage, and which are the objectlanguage?
- 1: Introduce Minus, and Divide, and think about what simp() rules there is. Is `(a + b) - b -> a` a good rule? What about `(a / b) * b -> a` and `(a * b) / b -> a`? How about `a * 2 -> a + a`? Mult is more expensive so we want to do + instead, right?
-  2: look at the code that generates the Expr that represents the sum of the resulting matrix multiplication. Try to understand it, and modify it so it returns the sum of the resulting matrix multiplication, but with each element squared. LExpr.eval() it. Is it about as fast as the code, unchanged, as the bottleneck is in the matrix multiplication, not the squaring/summing? 

<!--stackedit_data:
eyJoaXN0b3J5IjpbNTYwMDc1NjE4LC0xNDc0MzMwMTE0LC01Nj
g5MDU3NjksLTEyMjE2MDM3MTAsODI1ODMwNTk4LDkxMDEwMDcw
NSwtMTg3MDE3NDgxMywtMTM1OTQ2ODA2Miw1Nzg0NTY3NjMsLT
E1NDE1MzkyNTAsMzI1MTQ4NjksLTYxOTk0NTUyNSwyMTE4OTgw
MzY2LC03NjEyNDQ5MzEsMTIyNDg2MjAwNywtNDY2OTEwNDIsOD
A4MzMzMzU1LDEyMDQ5NjYxMTgsLTkzODczNjYsLTk5ODcxMDIw
OV19
-->