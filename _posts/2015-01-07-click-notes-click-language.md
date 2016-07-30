---
layout: post
title: Click Notes II - Click Script Language
categories: 
- Notes
- Network
tags:
- Notes
- Click
- Network
status: publish
type: post
published: true
author: Bo Yang
---

The [Click programming language](http://www.read.cs.ucla.edu/click/docs/language) was developed to configure Click routers, but nowadays you also can use it to write test cases for Click elements.

1. [Basic Syntax](#syntax)
2. [Element Group](#element_group)
3. [Compound Element](#comp_element)
4. [Script](#script)
5. [Testie](#testie)

### <a name="syntax">Basic Syntax</a>

The Click Script Language defines a configuration graph, which consists of connected elements. Each element has an element class specified by class name. Elements are connected through their input and output ports. Input and output ports are distinguished by number, while elements are distinguished by name. 

Click configuration strings are comma-separated lists of arguments delimited by parentheses. The fundamental syntax of Click Script Language is:

```cpp
name :: class(config-string);    // declare element object
name1, name2, ..., nameN :: class(config); // declaration shorhand
name1[port1] -> [port2]name2;    // connect two elements
name1[port1] -> [port2a]name2[port2b] -> [port3]name3;    // piggyback connections
name1 -> name2 :: class(config-string) -> name3;  // declaring elements inside connections is allowed
name1 -> class(config-string) -> name3;  // anonymous element
require(requirement[, requirement …]);   // list config requirements

n1, n2 :: class -> n3;  // many-to-one connections

// many-to-many connections:
// A many-to-many connection matches output ports to input ports. 
// There must be as many ports on the left as on the right.
// '=>' is the many-to-many connector.
c[0], c[1], c[2] => Paint(0), Paint(1), Paint(2) -> next;
c => Paint(0), Paint(1), Paint(2) -> next;
```

### <a name="element_group">Element Group</a>

An element group is one or more Click statements enclosed in parentheses. Within the parentheses, the special pseudoelements `input` and `output` refer to connections from outside the group. Click expands the group at parse time, so connections through `input` and `output` have no run-time overhead. The following five lines are equivalent:

```cpp
x -> y;
x -> ( input -> output ) -> y;
x -> ( [0] -> [0] ) -> y;
x -> (->) -> y;
x -> ( [0]->[0]; [1]->[1] ) => ( [0]->[0]; [1]->[1] ) -> y;
```

Line five uses the fact that connections may be repeated without error (the line expands to `x -> y; x -> y`), where the explicit semicolons are used to avoid ambiguity.

Element groups have implicit, overridable port specifications that list all their ports in sequential order. For example, these three lines are equivalent:

```cpp
x => ( [0]->[0]; [1]->[1] ) -> y;
x => [0,1] ( [0]->[0]; [1]->[1] ) [0,1] -> y;
x -> y; x [1] -> y;
```

An element group does **not** define a new scope. Its contents may refer to elements declared outside of the group, and declarations inside the group are visible after the group closes. This differs from compound elements, described next, which have a related syntax but additionally introduce a new scope.

### <a name="comp_element">Compound Element</a>

A **compound element** is a scoped collection of elements that behaves like a single element. A compound element can be used anywhere an element class is expected (that is, in a declaration or connection). Syntactically, a compound element is a set of Click statements enclosed in braces `{ }`. Inside the braces, the special names `input` and `output` represent connections from or to the outside.

Compound element classes are router configuration fragments consisted by Click elements and statements that are treated like element classes. For compound elements, only components remain in the final configuration graph, and all compound element structure is compiled away. This process is called **flattening**, during which compound element components are given names that reflect their origin. For example, a component named `e` of a compound element `compound` is named `compound/e` in the flattened configuration. 

```cpp
elementclass Name {    // compound elements
    ... Click Statements …  // defined by click statements but not C++ class
}
e :: {    // anonymous compound element class
    ... Click Statements … 
}
elementclass MyQueue Queue;   // define an alias for an existing element class
```

Like any element, compound elements may have input and output ports. Each connection to or from a compound element port is transformed by flattening into a connection to or from one of its components’ ports. Inside a compound element class, the special pseudoelements `input` and `output` specify how this transformation proceeds. Given a compound element e. If `e/input` connects to component `e/c` through port `i`, then every connection to `e`’s input port `i` will flatten into a connection to its component `e/c`.  For example, 

```cpp
elementclass Example {
    s1 :: InfiniteSource; s2 :: RatedSource;
    s1 -> [0]output; s2 -> [0]output;
}
e :: Example -> d :: Discard;

above code will be flattened to

e/s1 :: InfiniteSource; e/s2 :: RatedSource; d :: Discard;
e/s1 -> d; e/s2 -> d;
```

Compound element classes also can take varying number configuration arguments, input ports, and output ports. **Formal parameters** define the arguments that a compound element class should take. A **formal parameter** is a sequence of alphanumeric characters preceded by a dollar sign, such as `$var`. A compound element class may begin with a list of parameters. 

While formal parameters let a compound element class support a fixed number of positional arguments, **overloading** lets several compound element definitions with different numbers of arguments or ports share a single name. As a side-effect, overloading also support different behaviors in a compound element based on numbers of input/output ports. Every compound element class can correspond to a number of definitions, which are textually separated by `||`, which are distinguished by the numbers of formal parameters, input ports or output ports. Given a declaration of compound element, the interpreter checks the number of arguments and search for a matching definition. If found one, then it is expanded; otherwise, an error will occur. 

```cpp
// Example 1: argument overloading
elementclass ShapedQueue {				
    input -> Queue -> Shaper(10000) -> output;
    ||
    $cap | input -> Queue($cap) -> Shaper(10000) -> output; 
}				
q1 :: ShapedQueue;    // OK; uses first definition
q2 :: ShapedQueue(1024);    // OK; uses second definition
q3 :: ShapedQueue(1024, 10000);   // error—no matching definition 

// Example 1: port overloading - additional output port				
elementclass VerboseCheckIPHeader {
    input -> c :: CheckIPHeader -> output; c[1] -> Print(CheckIPHeader) -> Discard; 
    ||
    input -> c :: CheckIPHeader -> output; c[1] -> Print(CheckIPHeader) -> [1]output;	
} 
```

Overloading also allows user to add new definitions to an existing element class. For example, following definition will add a two-argument version of `Queue`:

```cpp
elementclass Queue {
  ... ||  // 'dot dot dot' is part of the syntax 
$capacity, $rate | input -> Queue($capacity)
                         -> Shaper($rate) -> output;
}
q1 :: Queue;        // built-in Queue
q2 :: Queue(1024);  // built-in Queue
q3 :: Queue(1024, 10000); // overloaded Queue definition above
```

When choosing the definition that corresponds to a given compound element declaration, Click only considers the definitions that were lexically visible at the point of declaration. 

Compound element configuration arguments are only meaningful inside the configuration strings of components. So that you cannot change the element class of a given component, or cause components to be added to or subtracted from the compound. However, it  is possible to build a compound element that sends packets through different sets of components based on the value of its configuration string. Following example uses StaticSwitch to implement a selective checksum check.(The `CheckIPHeader` element checks an IP header’s length and checksum for sanity; `CheckIPHeader2` does the work of `CheckIPHeader` except for the checksum check.)

```cpp
elementclass MaybeChecksum { $checksum_p |
    input -> sw :: StaticSwitch($checksum_p);
    sw[0] -> CheckIPHeader2 -> output;
    sw[1] -> CheckIPHeader -> output;
};
c1 :: MaybeChecksum(0);    // uses CheckIPHeader2, skips checksum
c2 :: MaybeChecksum(1);    // uses CheckIPHeader, checks checksum
```

### <a name="script">Script</a>

The [Script element](http://www.read.cs.ucla.edu/click/elements/script) implements a simple scripting language interpreter useful for controlling Click configurations. Scripts can set variables, call handlers, wait for prodding from other elements, and stop the router. Script element is defined in `click/elements/standard/Script.hh`.  For details of instructions and handlers, please refer to [http://www.read.cs.ucla.edu/click/elements/script](http://www.read.cs.ucla.edu/click/elements/script). 

In the Script element, each keyword is handled by a corresponding handler:

```cpp
void
Script::add_handlers()
{
    set_handler("step", Handler::OP_WRITE | Handler::h_nonconst, step_handler, 0, ST_STEP);
    set_handler("goto", Handler::OP_WRITE | Handler::h_nonconst, step_handler, 0, ST_GOTO);
    set_handler("run", Handler::OP_READ | Handler::READ_PARAM | Handler::OP_WRITE | Handler::h_nonconst, step_handler, 0, ST_RUN);
    set_handler("add", Handler::OP_READ | Handler::READ_PARAM, arithmetic_handler, ar_add, 0);
    set_handler("sub", Handler::OP_READ | Handler::READ_PARAM, arithmetic_handler, ar_sub, 0);
    set_handler("min", Handler::OP_READ | Handler::READ_PARAM, arithmetic_handler, ar_min, 0);
    set_handler("max", Handler::OP_READ | Handler::READ_PARAM, arithmetic_handler, ar_max, 0);
        set_handler("length", Handler::OP_READ | Handler::READ_PARAM, basic_handler, ar_length, 0);
    set_handler("unquote", Handler::OP_READ | Handler::READ_PARAM, basic_handler, ar_unquote, 0);
…….
}
```

All of the handlers defined in the Script element need to parse the script language, which is implemented in click/lib/confparse.[hh|cc]. In this way, the Script element could serve as an interpreter to the script instructions and handlers. 

### <a name="testie">Testie</a>

[Testie](http://www.read.cs.ucla.edu/click/docs/testie) is a simple test tool for Click elements, which enables the Test-Driven-Development of Click. Each testie file is written mainly in Click script language and incorporates a shell script to be run. Tool `click/test/testie` runs the Click script, and checks the expected error or output. A testie file may have following layout:

```shell
%info 
// a short description of the test.

%require [-q]
// prerequisites that must be satisfied before the test can run.

%include FILENAME
// interpolate the contents of another testie file.

%script
// Shell scripts that controls the test. Testie will run each command in sequence.
// Command “click” need to be specified to interpret Click scripts.

%file [-d] [+LENGTH] FILENAME...
// Create an input file for the script. FILENAME can be `stdin`, which sets the script's standard input.

%expectv [-ad] [+LENGTH] FILENAME...
%expect [-adiw] [+LENGTH] FILENAME…
%expectx [-adiw] [+LENGTH] FILENAME...
// An expected output file for the script. FILENAME can be 'stdout' or 'stderr'. 
// Testie will run the script, then compare the file generated by script with the provided data. The files are compared line-by-line. 
// The -a flag marks this expected output as an alternate. Testie will compare the script's output file with each provided alternate; the test succeeds if any of the alternates match. 
// The -d flag behaves as in %file. 
// The -i flag makes any regular expressions case-insensitive (text outside of regular expressions must match case), and the -w flag ignores any differences in amount of whitespace within a line.

%ignorex [-di] [+LENGTH] [FILENAME]
%ignore, %ignorev
// lines to be ignored.
```

In addition to above sections, you also can define one or multiple Script elements in the testie file, which group a set of Click Script instructions.

`click/test/testie` is the Perl tool to run testie files. Given a testie file, the testie interpreter will first read and parse each section defined in the testie file: `%require` files will be expanded, files defined in section `%file` will be created in a temporary directory, commands listed under `%script` will be recorded in a hash, and so on. After preprocessing, the commands will be executed by shell or interpreted by click. `click/test/testie` is a very good template of simple interpreter. 

### <a name="refer">References</a>
- [http://www.read.cs.ucla.edu/click/docs/language](http://www.read.cs.ucla.edu/click/docs/language)
- [Eddie Kohler, _The Click modular router_, Ph.D. thesis, MIT, November 2000. This has more detail and examples than the TOCS and SOSP papers of the same name.](http://pdos.csail.mit.edu/papers/click:kohler-phd/thesis.pdf)
- [http://www.read.cs.ucla.edu/click/elements/script](http://www.read.cs.ucla.edu/click/elements/script)
- [http://www.read.cs.ucla.edu/click/docs/testie](http://www.read.cs.ucla.edu/click/docs/testie)

