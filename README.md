# context_proposal
A context language feature proposal for next C++ editions

# What's a context?
Everyone knows problems and issues with global variables and their better representative - singleton pattern.

Does anyone wonders why there are so many warnings about it?
It is useful and convinient as you can access it from anywhere and it is easy to forward data from one side of the program to another.
Yet it becomes quickly a huge problem for many reasons as it harder to reason about large programs with global variables as a completely different part of the code might be messing with the global variable resulting in errors in a completely different place.

But I imagine everybody had following recurring problem:
One needs to add data or a simple method in one part of the code yet the variable is accessible only 10 classes down the call stack... thus to get access to it one needs to implement like 10 methods for each of the classes. So one needs to implement 10 forwarding functions.
Ups... sorry did I say 10? Some of the classes are interfaces and thus to make the code compile one needs to implement the functions in all other implementations of the interface.
Excellent it is fixed after adding 40 function implementations that do nothing... but now we have a problem, a client made that uses out codebase made a private implementation of one of such classes and now his code is not compiling. Furthermore, all engineers on clients side that worked on that code have already left the company...

How to avoid such problems without usage of global variables?
Use context pattern.
Usually context is a variable that is passed to all classes.
Wait so how is it different from global variables?
1) You can create several contextes and run classes completely independently from another.
2) One can separate the context variable into two parts (1) shared state accessible acrossing every one (2) private state that dictates how to use the shared variable so there won't be interferences between the classes.

I guess (2) it too abstract so let's me provide an example.
Consider following problem: you need to initialize in easy-to-configurate way arbitrary number of classes with arbitrary number of subclasses and each of these subclasses/classes requires varity of doubles, int, string parameters. And everything needs to be easy to understand. How the hell to manage that?

Use context pattern with `boost::ptree` as the shared variable and `boost::string_path` as the private state.

For those unfamilliar with `boost::ptree` (property tree) and `boost::string_path` see the [documentation](https://www.boost.org/doc/libs/1_72_0/doc/html/property_tree.html).
Basically, one can view `boost::ptree` as a map string to string yet for better comprehension to us humans, it has filesystem-like structure:
`"folder1/folder2/variable1" = "value"`.

How to use the `ptree` for class initialization?
Let each class accept shared pointer to the `ptree` and a copy of its owner's `string_path` and add to it its personal "subfolder".
E.g.,

```
classA
{
public:
classA(shared_ptr<ptree> Tree, string_path Path):
 ClassB(Tree, Path / "classA")
{
   Path /= "classA";
   varA1 = Tree->get<int>(Path / "varA1", 0);
   varA2 = Tree->get<double>(Path / "varA2", 0.);
   varA3 = Tree->get<string>(Path / "varA3", "");
}
...
private:
 int varA1;
 double varA2;
 string varA3
 ClassB varB;
}

ClassB
{
public:
ClassB(shared_ptr<ptree> Tree, string_path Path)
{
   Path /= "classB";
   varB1 = Tree->get<int>(Path / "varB1", 0);
}
...
private:
 int varB1;
}
```

So in this example `classA` will initialize its variables `varA1, varA2, varA3` from `"classA/varA1","classA/varA2","classA/varA3"` fields of `ptree` and `classA::classB::varB1` will be initialized from field `"classA/classB/varB1"`. Yet if ClassB is initialized alone, without `classA` it will initialize its field `classB::varB1` from `"classB/varB1"`.

See, not so complex isn't it?
It does require discipline and integration of the variables into the general framework.
Aside from that, it is a general solution to an annoying and complex problem.
One could argue about performance but as long as slow methods are used only during initialization there won't be any issues and it solves a lot of problems regarding development time of program which is extremely valuable.

