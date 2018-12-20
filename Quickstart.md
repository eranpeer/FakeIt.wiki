* [Includes](#includes)
* [Stubbing](#stubbing)
* [Faking](#faking)
* [Invocation Matching](#invocation-matching)
* [Argument Matching](#argument-matching)
* [Verification](#verification)
* [Spying](#spying)
* [Reset Mock to Initial State](#reset-mock-to-initial-state)
* [Inheritance & Dynamic Casting](#inheritance--dynamic-casting)
* [Mocking Overloaded Methods](#mocking-overloaded-methods)

FakeIt is designed for simplicity and expressiveness. This design goal is achieved by using many of C++11 language features, including Variadic templates, Lambda expressions & User-defined literals and more.
Some of these C++11 features (like User-defined literals) are not fully supported by MS Visual Studio 2013. If you need to compile your code with MSVC2013 please refer to the [MSC++ Quickstart](Quickstart-MSC). If your using VS2015, GCC or Clang this Quickstart is good for you.

## Includes
You should include the following headers in your test file:
```C++
#include <fakeit.hpp>
```
You may also want to add:
```C++
using namespace fakeit;
```
## Stubbing
Assuming we have the following interface:
```C++
struct SomeInterface {
   virtual int foo(int) = 0;
   virtual int bar(int,int) = 0;
};
```
```C++
Mock<SomeInterface> mock;
// Stub a method to return a value once
When(Method(mock,foo)).Return(1);

// Stub multiple return values (The next two lines do exactly the same)
When(Method(mock,foo)).Return(1,2,3);
When(Method(mock,foo)).Return(1).Return(2).Return(3);

// Return the same value many times (56 in this example)
When(Method(mock,foo)).Return(56_Times(1));

// Return many values many times (First 100 calls will return 1, next 200 calls will return 2)
When(Method(mock,foo)).Return(100_Times(1), 200_Times(2));

// Always return a value (The next two lines do exactly the same)
When(Method(mock,foo)).AlwaysReturn(1);
Method(mock,foo) = 1;
```
**What if I want to be more specific?**
```C++
// Stub foo(1) to return the value '100' once (The next two lines do the same)
When(Method(mock,foo).Using(1)).Return(100);
When(Method(mock,foo)(1)).Return(100);

// Stub 'foo(1)' to always return '100'. For all other calls always return 0.
When(Method(mock,foo)).AlwaysReturn(0); // Any invocation of foo will return 0
When(Method(mock,foo).Using(1)).AlwaysReturn(100); // override only for 'foo(1)'

// The next two lines do exactly the same
When(Method(mock,foo).Using(1)).AlwaysReturn(0);
Method(mock,foo).Using(1) = 0;
```
**Can I stub a method to throw an exception?**
```C++
// Throw once
When(Method(mock,foo)).Throw(exception());
// Throw several times
When(Method(mock,foo)).Throw(exception(),exception());
// Throw many times
When(Method(mock,foo)).Throw(23_Times(exception()));
// Always throw
When(Method(mock,foo)).AlwaysThrow(exception());
```
**Nice, but sometimes I need something more freestyle**
```C++
// Do whatever you want using lambda expressions
When(Method(mock,foo)).Do([](int a)->int{ ... });
When(Method(mock,foo)).AlwaysDo([](int a)->int{ ... });
// Or, with C++14:
When(Method(mock,foo)).AlwaysDo([](auto a){ ... }); 
```
**Can I stub the destructor?**

Sure, you can stub a virtual destructor:
```C++
struct SomeInterface {
   virtual ~SomeInterface() = 0;
};
Mock<SomeInterface> mock;
When(Dtor(mock)).Do([](){ ... });
```
## Faking
In many cases we stub methods to simply do nothing. You can do that by explicitly stubbing the methods to a do-nothing behavior or you can simply use Faking.
```C++
struct SomeInterface {
   virtual void foo(int) = 0;
   virtual int bar(int,int) = 0;
   virtual ~SomeInterface() = 0;
};
Mock<SomeInterface> mock;
```
```C++
// Following 3 lines do exactly the same:
Fake(Method(mock,foo)); 
When(Method(mock,foo)).AlwaysReturn(); 
When(Method(mock,foo)).AlwaysDo([](...){}); 

// And here is another example:
Fake(Method(mock,bar)); 
When(Method(mock,bar)).AlwaysReturn(0); 
When(Method(mock,bar)).AlwaysDo([](...){return 0;}); 
```
It is also possible to Fake multiple methods in one line with the syntax:

*Fake(method1, method2, ...)*
```C++
Fake(Method(mock,foo), Method(mock,bar));
```
**Cool, can I fake the destructor?**

Sure, simply write:
```C++
Fake(Dtor(mock)); // same as When(Dtor(mock)).AlwaysDo([](){});
```
## Argument Matching
```C++
// Stub foo to return 1 only when:  arg > 1.
When(Method(mock,foo).Using(Gt(1)).Return(1);

// Stub foo to return 1 only when:  arg >= 1.
When(Method(mock,foo).Using(Ge(1)).Return(1);

// Stub foo to return 1 only when:  arg < 1.
When(Method(mock,foo).Using(Lt(1)).Return(1);

// Stub foo to return 1 only when:  arg <= 1.
When(Method(mock,foo).Using(Le(1)).Return(1);

// Stub foo to return 1 only when:  arg != 1.
When(Method(mock,foo).Using(Ne(1)).Return(1);

// Stub foo to return 1 only when:  arg == 1.
// The following 2 lines do exactly the same
When(Method(mock,foo).Using(Eq(1)).Return(1);
When(Method(mock,foo).Using(1)).Return(1);

// Stub foo to return 1 for any value.
// The following 2 lines do exactly the same
When(Method(mock,foo).Using(Any<int>()).Return(1);
When(Method(mock,foo).Using(_).Return(1);

// Stub foo to return 1 when arg1 == 1 and arg2 is any int.
// The following 3 lines do exactly the same:
When(Method(mock,foo).Using(1, _)).Return(1);
When(Method(mock,foo).Using(1, Any<int>())).Return(1);
When(Method(mock,foo).Using(Eq(1), _)).Return(1);
```
## Invocation Matching
**How do I match only the invocations where argument 'a' is bigger than argument 'b'?**

In these cases argument matching is not powerful enough since every argument matcher is limited only to one argument. What we need here is Invocation Matching. Here is how it works:
```C++
// Stub foo to return 1 only when argument 'a' is even.
auto argument_a_is_even = [](int a){return a%2==0;};
When(Method(mock,foo).Matching(argument_a_is_even)).Return(1);

// Throw exception only when argument 'a' is negative.
auto argument_a_is_negative = [](int a){return a < 0;};
When(Method(mock,foo).Matching(argument_a_is_negative)).Throw(exception());

// Stub bar to throw exception only when argument 'a' is bigger than argument 'b'.
auto a_is_bigger_than_b = [](int a, int b){return a > b;};
When(Method(mock,bar).Matching(a_is_bigger_than_b)).Throw(exception());
// Or, with C++14:
When(Method(mock,bar).Matching([](auto a, auto b){return a > b;})).Throw(exception());

```

## Verification
```C++
Mock<SomeInterface> mock;
When(Method(mock,foo)).AlwaysReturn(1);

SomeInterface & i = mock.get();

// Production code:
i.foo(1);
i.foo(2);
i.foo(3);
i.bar(2,1);

// Verify foo was invoked at least once. (The four lines do exactly the same)
Verify(Method(mock,foo));
Verify(Method(mock,foo)).AtLeastOnce();
Verify(Method(mock,foo)).AtLeast(1);
Verify(Method(mock,foo)).AtLeast(1_Time);

// Verify foo was invoked exactly 3 times. (The next two lines do exactly the same)
Verify(Method(mock,foo)).Exactly(3);
Verify(Method(mock,foo)).Exactly(3_Times);

// Verify foo(1) was invoked exactly once
Verify(Method(mock,foo).Using(1)).Once();
Verify(Method(mock,foo).Using(1)).Exactly(Once);

// Verify bar(a>b) was invoked exactly once
Verify(Method(mock,bar).Matching([](int a, int b){return a > b;})).Exactly(Once);
// Or, with C++14:
Verify(Method(mock,bar).Matching([](auto a, auto b){return a > b;})).Exactly(Once);

```
**Wow, can I verify the order of invocations too?**
```C++
// Verify foo(1) was invoked anywhere before foo(3)
Verify(Method(mock,foo).Using(1), Method(mock,foo).Using(3));
```
**What about an exact sequence? Can I verify an exact sequence of invocations?**

Sure, you represent a sequence in the following way:

Two consecutive invocations of foo:
```C++
Method(mock,foo) * 2
```
An invocation of foo followed by an invocation of bar:
```C++
Method(mock,foo) + Method(mock,bar)
```
Two consecutive invocations of foo + bar, i.e. foo + bar + foo + bar
```C++
(Method(mock,foo) + Method(mock,bar)) * 2
```
This way you can represent any sequence of invocations.

To verify that a specific sequence exists in the actual invocation sequence simply write:
```C++
// verify the actual invocation sequence contains two consecutive invocations of foo at least once.
Verify(Method(mock,foo) * 2); 

// verify the actual invocation sequence contains two consecutive invocations of foo exactly once.
Verify(Method(mock,foo) * 2).Exactly(Once);

// verify the actual invocation sequence contains an invocation of foo(1) followed by bar(1,2) exactly twice.
Verify(Method(mock,foo).Using(1) + Method(mock,bar).Using(1,2)).Exactly(2_Times);
```
**Can a sequence involve more than one mock instance?**

Sure, a sequence can involve multiple mock instances.
```C++
Mock<SomeInterface> mock1;
Mock<SomeInterface> mock2;

When(Method(mock1,foo)).AlwaysReturn(0);
When(Method(mock2,foo)).AlwaysReturn(0);

SomeInterface & i1 = mock1.get();
SomeInterface & i2 = mock2.get();

// Production code:
i1.foo(1);
i2.foo(1);
i1.foo(2);
i2.foo(2);
i1.foo(3);
i2.foo(3);

// Verify exactly 3 occurrences of the sequence {mock1.foo(any int) + mock2.foo(any int)}.
Verify(Method(mock1,foo) + Method(mock2,foo)).Exactly(3_Times);

```
  
### Verify No Other Invocations
```C++
Mock<SomeInterface> mock;
When(Method(mock,foo)).AlwaysReturn(0);
When(Method(mock,bar)).AlwaysReturn(0);
SomeInterface& i  = mock.get();

// call foo twice and bar once.
i.foo(1);
i.foo(2);
i.bar("some string");

// verify foo(1) was called.
Verify(Method(mock,foo).Using(1));

// Verify no other invocations of any method of mock. 
// Will fail since foo(2) & bar("some string") are not verified yet.
VerifyNoOtherInvocations(mock);

// Verify no other invocations of method foo only.
// Will fail since foo(2) is not verified yet.
VerifyNoOtherInvocations(Method(mock,foo));
 
Verify(Method(mock,foo).Using(2));

// Verify no other invocations of any method of mock. 
// Will fail since bar("some string") is not verified yet.
VerifyNoOtherInvocations(mock);

// Verify no other invocations of method foo only.
// Will pass since both foo(1) & foo(2) are now verified.
VerifyNoOtherInvocations(Method(mock,foo));

Verify(Method(mock,bar)); // verify bar was invoked (with any arguments)

// Verify no other invocations of any method of mock. 
// Will pass since foo(1) & foo(2) & bar("some string") are now verified.
VerifyNoOtherInvocations(mock);.
```
### Verification Scoping
Verification Scoping is the method of explicitly specifying the set of actual invocations used to verify a sequence.

Assuming we have the following interfaces:
```C++
struct IA {
   virtual void a1(int) = 0;
   virtual void a2(int) = 0;
};
struct IB {
   virtual void b1(int) = 0;
   virtual void b2(int) = 0;
};
```
And the following 2 mock objects
```C++
Mock<IA> aMock;
Mock<IB> bMock;
```
And the production code creates the following actual invocation list
```C++
aMock.a1(1);
bMock.b1(1);
aMock.a2(1);
bMock.b2(1);
```
Then
```C++
// Will PASS since the scenario {aMock.a1 + bMock.b1} is part of the   
// actual list {aMock.a1 + bMock.b1 + aMock.a2 + bMock.b2} 
Using(aMock,bMock).Verify(Method(aMock, a1) + Method(bMock, b1)); 

// Will FAIL since the scenario {aMock.a1 + bMock.b1} is not part of the   
// actual list {aMock.a1 + aMock.a2} 
Using(aMock).Verify(Method(aMock, a1) + Method(bMock, b1)); 

// Will PASS since the scenario {aMock.a1 + aMock.a2} is part of the   
// actual list {aMock.a1 + aMock.a2}
Using(aMock).Verify(Method(aMock, a1) + Method(aMock, a2)); 
```

By default, FakeIt uses all the mock objects that are involved in the verified scenario to implicitly define the verification scope. I.e. the two following lines do exactly the same:
```C++
// Use all methods invocations of aMock & bMock explicitly
Using(aMock,bMock).Verify(Method(aMock, a1) + Method(bMock, b1)); 

// Use all methods invocations of aMock & bMock implicitly
Verify(Method(aMock, a1) + Method(bMock, b1)); 
```
## Spying
In some cases it is very useful to spy an existing object. FakeIt is the ONLY C++ open source mocking framework that supports spying.
```C++
class SomeClass {
public:
   virtual int func1(int arg) {
      return arg;
   }
   virtual int func2(int arg) {
      return arg;
   }
};

SomeClass obj;
Mock<SomeClass> spy(obj);

When(Method(spy, func1)).AlwaysReturn(10); // Override to return 10
Spy(Method(spy, func2)); // Spying func2 without changing any behavior

SomeClass& i = spy.get();
cout << i.func1(1); // will print 10. 
cout << i.func2(1); // func2 is not stubbed.
```
In general, all stubbing & verifying features work on spy objects the same way they work on mock objects.
## Reset Mock to Initial State
In most cases you will need to reset the mock objects to the initial state before/after each test method. To do that simply add the following line for each mock object to the setup/teardown code of your tests.
```C++
mock.Reset();
```
You can also keep the stubbing and clear only the collected invocation data by:
```C++
mock.ClearInvocationHistory();
```
## Inheritance & Dynamic Casting
```C++
struct A {
  virtual int foo() = 0;
};

struct B : public A {
  virtual int foo() override = 0;
};

struct C : public B
{
   virtual int foo() override = 0;
};
```
**upcast support**
```C++
Mock<C> cMock;		
When(Method(cMock, foo)).AlwaysReturn(0);

C& c = cMock.get();
B& b = c;
A& a = b;

cout << c.foo(); // prints 0
cout << b.foo(); // prints 0
cout << a.foo(); // prints 0
```
**dynamic_cast support**
```C++
Mock<C> cMock;
When(Method(cMock, foo)).AlwaysReturn(0);

A& a = cMock.get(); // get instance and upcast to A&

B& b = dynamic_cast<B&>(a); // downcast to B&
cout << b.foo(); // prints 0

C& c = dynamic_cast<C&>(a); // downcast to C&
cout << c.foo(); // prints 0
```
## Mocking Overloaded Methods
When mocking an overloaded method all you have to do is to specify the prototype of the method.
The following example code demonstrates mocking of overloaded methods:
```C++
struct SomeInterface {
  virtual int func() = 0;
  virtual int func(int) = 0;
  virtual int func(int, std::string) = 0;
};

Mock<SomeInterface> mock;

//stub the func with prototype: int()
When(OverloadedMethod(mock,func, int()) ).Return(1); 
//stub the func with prototype: int(int)
When(OverloadedMethod(mock,func, int(int)) ).Return(2); 
//stub the func with prototype: int(int,std::string)
When(OverloadedMethod(mock,func, int(int, std::string)) ).Return(3);

SomeInterface& i = mock.get();
cout << i.func();     // will print 1
cout << i.func(1);    // will print 2
cout << i.func(1,""); // will print 3
```
Mocking const overloaded methods is accomplished similarly:
```C++
struct SomeInterface {
  virtual int func(int) = 0;
  virtual int func(int) const = 0;
};

Mock<SomeInterface> mock;

//stub the func with prototype: int(int)
When(OverloadedMethod(mock,func, int(int)) ).Return(1);
//stub the const func with prototype: int(int)
When(ConstOverloadedMethod(mock,func, int(int)) ).Return(2);

      SomeInterface& v = mock.get();
const SomeInterface& c = mock.get();

cout << v.func(1);    // will print 1
cout << c.func(1);    // will print 2
```