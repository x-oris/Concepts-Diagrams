# Cpp Polymorphism Types
***

To enter the world of this module, we need first to know & understand the overall types of polymorphism:

***
### 1. Static Polymorphism (Compile-Time Polymorphism) | (Ad-Hoc Polymorphism)

* ***Function Overloading***, same function name, difffernet parameter list, the polymorphism happens in the compile time, when it arrives into the runtime, the compiler has already changed the name of the polymorphic function using **Name Mangling**, by adding suffixes to the name, creating two seperate function, therefore the polymorphism has already dissolved at this point.

* ***Operator Overloading***, Redifines how operators work for user-defined types, aslo resolved at compile time.

### 2. Dynamic Polymorphism (Runtime Polymorphism) | (SubType Polymorphism)

* Virtual functions, requires ```virtual``` keyword in the base class, usese base class pointer/refrence, implemented via **Vtable**, without the ```vitual``` keyword base version is called.

Example by ChatGPT
```
class Animal {
public:
    virtual void speak();
};

class Dog : public Animal {
public:
    void speak();
};

main:
Animal* a = new Dog();
a->speak();   // calls Dog::speak()
```
***
### Deep dive into the concepts:

### 1. Function Overloading + Operator Overloading


Let's say we have this code:
```
void foo(int x);
void foo(double x);
```
The compiler rewrites the names:
```
_Z3fooi (foo(int x));
_Z3food (foo(double));
```
No ambiguity at link time, each overload is a completely seperate function, this is called **Name Mangling**, this happens at the compile time.

Through what's called **Overload Resolution**, the compiler take the final decision on what function should be called and executed, and the function name is hard wired into the generated code.
The compiler follows this step to complete this behavior:

* #### First | Build the candidate set:

Collect all the functions named ``foo`` visible in the scope.
Candidate set = { foo(int), foo(double), foo(long) }

* #### Second | Build the candidate set:

Here the compiler looks for which function should be called based on the parameter argument given, in our case,
all the polymorphic function are viable & valide.

```main: foo(10);```

```foo(int)``` -> viable | ```foo(double)``` -> viable (int to double) | ```foo(long)``` -> viable (int to long)

If a function has wrong number of paramters passed or requires impossible conversions, it is immediately discarded.

* #### Third | Rank Conversions (Important):

We can think of this as a ladder and the compiler will never climb down if it can stay higher.

```
Exact Match
Promotion
Standard Conversion
User-Defined Conversion
Ellipsis (...)
```
***
#### Exact Match:
```
void foo(int);
foo(10); -> Matched
```

No conversion, no ladder movement happens. It works the same with const & refrence binding.

***Note***: Exact macth does not mean "identical in text", but no semantic converstion cost.
***

### Promotion:
```
char   → int
short  → int
float  → double
bool   → int
```

```
void foo(int);
foo('a'); Char -> Int (Promotion)
```

CPU usually operates on ``int`` anyway, no loss of infomation.
***
### Standard Conversion:

```
int    → double
int    → long
T*     → const T*
Derived* → Base*
```

```
void foo(double);
foo(10); Int to Double
```
***

#### Promotion vs Standard

```
void foo(int);
void foo(double);

foo('a');
```
(Table by ChatGPT)
| Overload    | Conversion |
| ----------- | ---------- |
| foo(int)    | Promotion  |
| foo(double) | Standard   |

```foo(int)``` gets choosed as the executed function.
***
#### User-Defined Converstion:

```
class A {
    public:
        A(int); (Constructor)
        void foo(A);
};

foo(10);
```

Since A has a constructor, the compiler convert ``int`` to ``A``, foo(A) is viable, and it is selected assuming no better overload exists.
Once the compiler has decided which function to use, it call the constructor creating a temporary **A**, by calling ```A::A(int)```.
The conversion can work with refrence binding, ```void foo(const A&);```, but not without the ``const`` keyword, since we cannot bind a temporary to non-const reference.

```
class A {
public:
    A(int);
    A(double);
};

foo(10);
```

In this case, foo(10) has an int, the compiler follows the exact match which is ```A(int)```, meanwhile ```A(double)``` is a standard conversion, therefore ```A(int)``` is chosen.

***Note***: Only one user-defined conversion is allowed, you can not chain them unless explicitly done.
***

#### Ellipsis:

```
void foo(...);
```
This is the last ladder match, no type checking, used only if nothing else matches.
***

### 2. Subtype Polymorphism (Runtime Polymorphism):

In the subtype polymorphism we treat the derived objects as base objects and still we are allowed to use the derived behavior at runtime.

Let's consider this example given by ChatGPT:

```
class Animal {
public:
    void speak() {
        std::cout << "Animal sound\n";
    }
};

class Dog : public Animal {
public:
    void speak() {
        std::cout << "Woof\n";
    }
};

Animal* a = new Dog();
a->speak();   // calls Animal::speak()
```

When we invoke ```a->speak();```, the compiler uses the behavior of the base class ```Animal```, because the compiler decides which function to call at compile time based on the static type of the pointer ``(Animal*)``, the compiler literally hard-codes it into : ``Animal::speak(a);``.

* ``Virtual`` keyword and its functionality:

The keyword tells the compiler to not decide at the compile time, but to leave it till the runtime, and decide based on the actual object, this switches from **Static Binding** which is at the compile time to **Dynamic Binding** which is at the runtime.

And now Ladies & Gentlemen, let's dive into the inner internals of this;
### The Hidden : ``Vtable`` & ``Vptr``:

Example as introduction to the Subtype Polymorphism:
```
Animal *x = new Dog();
a->speak();
```

In this case we have ``Animal``, the base class & ``Dog`` the derived class that inherits from ``Animal``.
Dynamic polymorphism needs at Runtime, to look at the actual object (``Dog``) not the pointer type, then call the correct overriden ``speak();`` function.

To make this phenomen happen, we have something called the **Virtual Table**.

#### What is the **Vtable** ?

* A table of function pointers.
* One per class
* Created by the compiler.
* Stored in read-only memory. (Unchangable once stored).

[Example by ChatGPT]

```Animal vtable:
[0] &Animal::speak
[1] &Animal::~Animal

Dog vtable:
[0] &Dog::speak
[1] &Dog::~Dog
```
#### What is the **Vptr** ?

* A hidden pointer.
* Stored inside each object.
* Points to that object's class vtable.
***

The constructor sets the vptr, construction order matters;
Let's say we have this order...
``Animal`` constructor:
* Sets ``Vptr`` that points to ``Animal`` Vtable.
``Dog`` constructor:
* Overwrites ```vptr``` to point to ```Dog``` Vtable.

**Note to ease confusion**: Dog is an object (holding Dog & Animal at same point), therefore there is only one Vptr.

So basically, when a derived object finishes construction, the object's ``vptr``, is updated to point to the derived class's vtable, and that vtable contains the address of the derived overide in the same slot as the base virtual function.

The proccess of calling the appropriate function through the **Vtable** is called **Dynamic Dispatch**, (Virtual dispatch = Dynamic dispatch)

* Read ``Vptr`` from object.
* Index into vtable.
* Fetch function pointer.
* Call it with ``this``.

The virtual dispatch happens only when the function is **Virtual** and the call is made through the base pointer or reference.
No virtual dispatch occurs if the object is directly used, function is not virtual or inside of constructors/deconstructors of the **Base Class**.

***To see the example for the non dispatch in constructor & deconstructor in the base class, Check the code in this folder (TODO)***.

