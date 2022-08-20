# Calculator Language

> A compiler is just a staged definitional interpreter. What's the problem?

Dont worry about the above quote. Let's design a programming language, and write a compiler for it!

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

We will write it in Java, cause everyone know Java. I am gonna code it with you, step by step. This is like pair programming.

    
    static String prettyPrint(JSONObject json) {  
      throw new RuntimeException("...");  
    };

We start by getting the 'type' field from the JSON, and see which case it is:
    
    static String prettyPrint(JSONObject json) {  
      String type = json.getString("type");
      return switch (type) { 
        case "Literal" -> throw new RuntimeException("...");  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected: " + type);  
    };  

Literal is easy - we just print the value: 

    static String prettyPrint(JSONObject json) {  
      String type = json.getString("type");
      return switch (type) { 
        case "Literal" -> String.valueOf(json.getInt("value"));  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected: " + type);  
    };  

Unlike Literal, Plus and Multiply contain more JSON as children, in left and right field, which we have to handle by calling prettyPrint again - recursion! 

    static String prettyPrint(JSONObject json) {  
      String type = json.getString("type");
      return switch (type) { 
        case "Literal" -> String.valueOf(json.getInt("value"));  
        case "Plus" -> "(" + prettyPrint(json.getJSONObject("left")) +  
          "+" + prettyPrint(json.getJSONObject("right")) + ")";  
        case "Multiply" -> "(" + prettyPrint(json.getJSONObject("left")) +
          "*" + prettyPrint(json.getJSONObject("right")) + ")";  
        default -> throw new RuntimeException("Unexpected: " + type);  
    };  

Now calling prettyPrint() on our example return "((1+2)*(3+4))". Look pretty neat.

Moving on to the evaluator, which take a JSON, and return an Int:

    static int evaluate(JSONObject json) {  
      throw new RuntimeException("...");    
    }

Get the type and match on it...

    static int evaluate(JSONObject json) {  
      String type = json.getString("type");  
      return switch (type) {  
        case "Literal" -> throw new RuntimeException("...");  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected value: " + type);  
      };  
    }

Handle the base case (no recursion, simple)...

    static int evaluate(JSONObject json) {  
      String type = json.getString("type");  
      return switch (type) {  
        case "Literal" -> json.getInt("value");  
        case "Plus" -> throw new RuntimeException("...");  
        case "Multiply" -> throw new RuntimeException("...");  
        default -> throw new RuntimeException("Unexpected value: " + type);  
      };  
    }

Handle the recursive case (recurse into the children, and combine the result)...

    static int evaluate(JSONObject json) {  
      String type = json.getString("type");  
      return switch (type) {  
        case "Literal" -> json.getInt("value");  
        case "Plus" -> evaluate(json.getJSONObject("left")) + evaluate(json.getJSONObject("right"));  
        case "Multiply" -> evaluate(json.getJSONObject("left")) * evaluate(json.getJSONObject("right"));  
        default -> throw new RuntimeException("Unexpected value: " + type);  
      };  
    }

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

    class Expr {}  
  
    class Lit extends Expr {  
      int val;  
  
      Lit(int val) {  
        this.val = val;  
      }  
    }  
  
    class Plus extends Expr {  
      Expr left, right;  
  
      Plus(Expr left, Expr right) {  
        this.left = left;  
        this.right = right;  
      }  
    }  
  
    class Mult extends Expr {  
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

We have to now fix the definition for prettyPrint() and evaluate(). While we are at it, let's also rename prettyPrint() into pp(), and evaluate() into eval(): our code is, and will, remain short and concise.

One problem with the above code, is that it rely heavily on instanceof, and downcasting, which is frown upon in Java. This could be fixed by making pp() and eval() abstract function in Expr, and have each subclass override it.

    String pp() {  
      return String.valueOf(val);  
    }  
  
    int eval() {  
      return val;  
    }

The @override function  for Lit.

    String pp() {  
      return "(" + left.pp() + "+" + right.pp() + ")";  
    }  
  
    int eval() {  
      return left.eval() + right.eval();  
    }

The @override function for Plus. The case for Mult is almost the same, so I will not show it here.

Alas, we cant write a compiler yet, as our language is so trivial a compiler is pointless. Let's add some more feature.

## Input

Right now our language is really, really boring. It react the same (output a constant number) no matter what, while most program behave differently according to the context.

The easiest way to do this is to add variable to the language. For now, for the sake of complexity, all variables are defined externally, inputted by the user when the program is ran. We will add defining and assigning to variable some other times.

    class Var extends Expr {  
      String name;  
      Var(String name) {  
        this.name = name;  
      }  
    
      String pp() {  
        return name;  
      }  
    }

OK. On to eval(). Hmm...

    int eval() {  
      throw new RuntimeException("...");  
    }

What do I put in here? Seems like we are stuck! For a good reason: we now have unknown variables in our language, but we dont know which value they are. Luckily, they are all user-defined variable, so we can require our user pass in a Mapping, from variable to int. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg2Mjc0NzczOCwzMzQ3MzY1OTUsLTIwNT
MwOTMxNjIsLTExMDQ1MzQ2OTMsLTE4ODQ5OTAxMzMsLTE1MjY5
NTI0NDgsNTU1OTg4NjcxLC02NjE0NzIyMzldfQ==
-->