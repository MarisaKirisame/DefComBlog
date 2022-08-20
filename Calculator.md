# Calculator Language

> A compiler is just a staged definitional interpreter. What's the problem?

Dont worry about the above quote. Let's design a programming language, and write a compiler for it!

## Design

What should the language look like? 

JSON. To be precise, the source code of the program is a JSON file. By doing so, we do not need to parse the program from a textual format - it was already done. Even though parsing make your program look nice (because it is not in JSON), and parsing is deep and useful knowledge, we will completely skip parsing: it is very complex, and if we talk about it, we will spend most of our time talking about it, leaving little time for everything else. 

OK. What feature should it have?

Almost none. The more feature a language have, the harder it is to design and implement. To ease such a task, we will be doing, essentially, agile development. We will design a simple language, write the compiler, refactor the code, add more feature to the language and repeat. This way, the learning curve will be smoother, and you get to see the conflict between different programming language features, and the conflict between performance and expressivity, as the language evolve.

## Enough talking! Let's get to the action!

Alright, alright, chill. 

We shall begin with a language so simple, your (non-scientific) calculator will laugh at it. It has exactly three features: number (an integer), plus (add two number), and multiply (time two number).

Now we have to specify the JSON format for our program.

For a number x, the format is {type: "Literal", value: x}.
For a + b, the format is {type: "Plus", left: a, right: b}
For a * b, the format is {type: "Multiply", left: a, right: b}

As an example, (1 + 2) * (3 + 4) is written as

    {type: "Multiply", 
      left: {type: "Plus", 
        left: {type: "Literal", value: 1}, 
        right: {type: "Literal", value: 2}},
      right: {type: "Plus",
        left: {type: "Literal", value: 3}, 
        right: {type: "Literal", value: 4}}}

What a pain in the ass. Even though we don't have a parser, we can still write a pretty-printer: it output the code in a prettier, actually-human readable format.

## Pretty Print

We will write it in Java, cause everyone know Java. I am gonna code it with you, step by step. This is like pair programming.
    
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

Unlike Literal, Plus and Multiply contain more JSON as children, in left and right field, which we have to handle by calling prettyPrint again - recursion! 

        case "Plus" -> "(" + prettyPrint(j.getJSONObject("left")) +  
          "+" + prettyPrint(j.getJSONObject("right")) + ")";  
        case "Multiply" -> "(" + prettyPrint(j.getJSONObject("left")) +
          "*" + prettyPrint(j.getJSONObject("right")) + ")";  

Now calling prettyPrint() on our example return "((1+2)*(3+4))". Look pretty neat.

## Evaluate

Moving on to the evaluator, which take a JSON, and return an Int:

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

If you compare evaluate() to prettyPrint(), you will find that they are almost the same! The biggest difference is that one of them have + as an operator that add two Int, and one of them have + as "+" the string. 

This is not a coincidence. Think about how we can process program in our calculator language:
 - Given a program in the JSON format, we must retrieve it's type so we can work on it
 - For the base case, we can immediately return the correct value
 - For the recursive case, we have to handle the children by recursing

Without any of the three step, the program is incorrect, and cannot be fixed without essentially re-introducing it in. This pattern is thus universal: you will see it again, a lot.

## Meta-Language, Object-Language, and the Definitional Interpreter
Lets step back and reflect on what we had done: in less then 10 lines of code, we had implemented an Interpreter! What is the magic? Before I answer this question, some terminology: when we use a program X to manipulate another program Y, the language X is in, is called the meta-language, and Y's language is the object-language.
A definitional interpreter is when we use meta-language's feature to implement the corresponding object-language's feature. In this case, evaluate() is a definitional interpreter, because we use Java's Int to implement Calculator's Int. Definitional Interpreters are known for their simplicity, because they essentially does nothing.

However, despite doing nothing, they are very useful. The most straight-forward and popular use of a Definitional Interpreter is to give new syntax, or to provide eval(), for already-existing features, also known as inventing a novel programming language. We could take a Definitional Interpreter, and add a new language feature which we actually implement. We could also take a Definitional Interpreter, add some print to every case, and now we had a tracing debugger. Or, as the quote at the beginning of the chapter dictate, a compiler is also a Definitional Interpreter but slightly modified. Essentially, a Definitional Interpreter is a solid foundation, allowing you to make a bunch of modification to it, to get all sort of useful things.

## Refactoring
Manipulating JSON is hard! I have to remember the name and the type of everything, and that sure is tiresome! I am gonna refactor the JSON (JavaScript object) into Java Objects, since we are writing Java not JavaScript.

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
  
    static class Mult extends Expr {  
      Expr left, right;  
  
      Mult(Expr left, Expr right) {  
        this.left = left;  
        this.right = right;  
      }  
    }

The class definitions.

    static Expr mkLit(int val) {return new Lit(val);}  
  
    static Expr mkPlus(Expr left, Expr right) {return new Plus(left, right);}  
  
    static Expr mkMult(Expr left, Expr right) {return new Mult(left, right);}

Some shorthands.

    mkMult(mkPlus(mkLit(1), mkLit(2)), mkPlus(mkLit(3), mkLit(4)));

Now our example look much better - it is just a one-liner.  After the above refactoring, the JSON is nowhere to be seen - this is expected. From now on, we will simply not talk about JSON, and focus on Expr. A conversion from JSON to Expr can be easily written, and we do not even need to work with text, or JSON, to begin with - the above Expr construction is a easy way to test our programs. We can also imagine adding a parser that go straight from text to Expr lateron - we dont have to do it now, our time is better spent focusing on compiler itself.

    static String pp(Expr expr) {  
      if (expr instanceof Lit) {  
        return String.valueOf(((Lit) expr).val);  
      } else if (expr instanceof Plus) {  
        return "(" + pp(((Plus) expr).left) + "+" + pp(((Plus) expr).right) + ")";  
      } else if (expr instanceof Mult) {  
        return "(" + pp(((Mult) expr).left) + "*" + pp(((Mult) expr).right) + ")";  
      } else {  
        throw new RuntimeException("Unexpected value: " + expr.getClass());  
      }  
    }  
  
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

We have to now fix the definition for prettyPrint() and evaluate(). While we are at it, let's also move prettyPrint() into Java's toString(), and rename evaluate() into eval(): our code is, and will, remain short and concise.

One problem with the above code, is that it rely heavily on instanceof, and downcasting, which is frown upon in Java. This could be fixed by making pp() and eval() abstract function in Expr, and have each subclass override it.

    // In Lit
    public String toString() {return String.valueOf(val);}  
  
    int eval() {return val;}

    // In Plus
    public String toString() {return "(" + left.toString() + "+" + right.toString() + ")";}  
  
    int eval() {return left.eval() + right.eval();}

The @override function for Lit and Plus. The case for Mult is almost the same, so I will not show it here.

Alas, we cant write a compiler yet, as our language is so trivial a compiler is pointless. Let's add some more feature.

## Input

Right now our language is really, really boring. It react the same (output a constant number) no matter what, while most program behave differently according to the context.

The easiest way to do this is to add variable to the language. For now, for the sake of complexity, all variables are defined externally, inputted by the user when the program is ran. We will add defining and assigning to variable some other times.

    static class Var extends Expr {  
      String name;  
      Var(String name) {this.name = name;}  
    
      public String toString() {return name;}  
    }

OK. On to eval(). Hmm...

    int eval() {  
      throw new RuntimeException("...");
    }

What do I put in here? Seems like we are stuck! For a good reason: we now have unknown variables in our language, but we dont know which value they are. Luckily, they are all user-defined variable, so we can require our user pass in a Mapping, environment(env) from variable to int. 

    // In Expr
    abstract int eval(Map<String, Integer> env);

Now our definitions for Lit, Plus and Mult are broken. We can fix Lit by accepting the env and doing nothing for it. For Plus and Mult, we pass env into the recursive call. 

    int eval(Map<String, Integer> env) {return env.get(name);}

Now for Var, we lookup the value from the environment.

Pretty Good! I had wrote another example, which multiply 2 n*n matrix, where n = 2, and add up all the cell in the resulting matrix, into a single number. Running the example on 2 matrix with all cell as 1, will give 8, which is expected. However, printing the matrix will give:

    ((((0+((0+(a_0_0*b_0_0))+(a_0_1*b_1_0)))+((0+(a_0_0*b_0_1))+(a_0_1*b_1 ...

Urgh. What a mess! Look like there is a bunch of 0 in our program. We can write a function, simp(), just like pp() and eval(), to simplify the input program and remove them.

## Simplification

    // In Expr
    Expr simp() {return this;}
    
The case for Lit and Var is trivial: there is nothing to simplify, so we just {return this;}. In fact, we had moved this code into the base class, Expr, because it is always correct to simplify, by doing nothing!

By correctness, we mean that, given a program X, X.simp() is equal to X. By equality, X and Y are equal, if and only if forall env, X.eval(env) == Y.eval(env). When writing optimizations, this is critical to keep in mind - we dont want to change the meaning of user program!

Let's get back to programming, and fill out the case for Plus.

    Expr left = this.left.simp();  
    Expr right = this.right.simp();
    return mkPlus(left, right);

We begin by traversing left and right, simplifying them. Note that the above code is universal - we can also do this for the simpl() for Mult (of course, we have to change mkPlus into mkMult). This is because if a == and b == y, mkPlus(a, b) == mkPlus(x, y). Even when we have nothing to do for a Node, we can and should still optimize by recursing. This way its children can do it's things. Now, onto some real optimization for Plus.

      // After left, right is defined, before mkPlus
      if (left.equals(Cal.mkLit(0))) {  
        return right;  
      } else if (right.equals(Cal.mkLit(0))) {  
        return left;  
      }

If left or right is 0, we just return the other Expr. Note that I had override equals for all the Expr. I wont show them because they are not interesting, but you can see them in the code. Another case:

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

If we look at our simp(), we will see that it is very much like eval()! In fact, if we give it an Expr with no Var in it, it will always return a Lit! And the code that will be executed, in that case, is exactly the code for a Definitional Interpreter. This is not a coincidence: the Definitional Interpreter is a recurring echo, which we will see multi, multiple of time in the book.

## Staging

Ignore the scary title for now. Lets try to make our Interpreter faster. I had increase n from 2 to 4, so we are now multiplying 2 4*4 matrix, and run the eval() in a loop, and profiled the resulting code.

The profiler tell me that most time is spend in Map.get(). Why? What is in a Map?

There are multiple ways to implement a Map from String to Int. Most notable examples are search tree, hash map, and a trie.
However, inorder to lookup a key, they all need to traverse the String to compare/compute the hash/traverse the trie, and look at multiple buckets/nodes to find the value. Looking at multiple places is not good, as it require multiple memory fetch, which may be a cache miss and stall the pipeline, and it also require conditional jump, which may fail the branch predictor and require conditional jump. In short - we will like to look only once. If env is an Array, there are less cache miss as the values are compactly stored, and there are no failed branch prediction because there are no branch.

    // Inside Expr
    abstract int yolo(int[] env);

We will call our new function yolo, because you only look once (into the array).
The change for Lit, Plus, Mult are all mechanical and not be shown, but we are stuck on Input again: we dont have a way to go from String name, to an index in env! We can fix this, by adding a mapping from String to index as an argument to yolo:

    abstract int yolo(Map<String, Integer> loc, int[] env);


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyMzMzNjk0MDksMTgxMzA5MDkxNyw2OT
U4NTMxOTIsNTc5ODQ5ODQ4LC0xMzkxMzg0Nzg0LDE3ODU5Mjkw
MDcsODc0MzkzMzQ4LDExNTEzMzc1NTQsMTY3Njg4MzMxMSwtMT
MyNTg2MDM4NywtMjAxMjY2MTkxNCw3NzYyNTU0ODIsLTQ3Nzcw
MTIwNiwtMTA5Njk4MjI3NSw3MjYxMTA4MzYsMTc1NjYzNzIyOS
wtNjk4Mzg4NTIsMzIyMDIwNzMyLC0xMTM1Mzc1NDc5LDg0MDc0
OTMxN119
-->