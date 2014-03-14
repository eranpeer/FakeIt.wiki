Assuming we have the following interface:
```C++
struct SomeInterface {
   virtual int foo(int) = 0;
   virtual int bar(int,int) = 0;
};
```
### Stubbing
```C++
// Stub a method to return a value once
When(mock[&SomeInterface::foo]).Return(1);

// Stub multiple return values (The next two lines do exactly the same)
When(mock[&SomeInterface::foo]).Return(1,2,3);
When(mock[&SomeInterface::foo]).Return(1).Return(2).Return(3);

// Return the same value many times (56 in this example)
When(mock[&SomeInterface::foo]).Return(56_Times(1));

// Return many values many times (First 100 calls will return 1, next 200 calls will return 2)
When(mock[&SomeInterface::foo]).Return(100_Times(1), 200_Times(2));

// Always return a value (The next two lines do exactly the same)
When(mock[&SomeInterface::foo]).AlwaysReturn(1);
mock[&SomeInterface::foo] = 1;
```
What if I want to be more specific?
```C++
// Stub foo(1) to return the value '100' once (The next two lines do the same)
When(mock[&SomeInterface::foo].Using(1)).Return(100);
When(mock[&SomeInterface::foo](1)).Return(100);

// Stub 'foo(1)' to always return '100'. For all other calls always return 0.
When(mock[&SomeInterface::foo]).AlwaysReturn(0); // Any invocation of foo will return 0
When(mock[&SomeInterface::foo].Using(1)).AlwaysReturn(100); // override only for 'foo(1)'

// The next two lines do exactly the same
When(mock[&SomeInterface::foo].Using(1)).AlwaysReturn(0);
mock[&SomeInterface::foo].Using(1) = 0;
```
Can I stub a method to throw an exception?
```C++
// Throw once
When(mock[&SomeInterface::foo]).Throw(exception());
// Throw several times
When(mock[&SomeInterface::foo]).Throw(exception(),exception());
// Throw many times
When(mock[&SomeInterface::foo]).Throw(23_Times(exception()));
// Always throw
When(mock[&SomeInterface::foo]).AlwaysThrow(exception());
```
Nice, but sometimes I need something more freestyle
```C++
// Do whatever you want using lambda expressions
When(mock[&SomeInterface::foo]).Do([](int a)->int{ ... });
When(mock[&SomeInterface::foo]).AlwaysDo([](int a)->int{ ... });
```
### Invocation Matching
```C++
// Stub foo to return 1 only when argument 'a' is even.
auto agrument_a_is_even = [](int a){return a%2==0;};
When(mock[&SomeInterface::foo].Matching(agrument_a_is_even)).Return(1);

// Throw exception only when argument 'a' is negative.
auto agrument_a_is_negative = [](int a){return a < 0;};
When(mock[&SomeInterface::foo].Matching(agrument_a_is_negative)).Throw(exception());

// Stub bar to throw exception only when argument 'a' is bigger than argument 'b'.
auto a_is_bigger_than_b = [](int a, int b){return a > b;};
When(mock[&SomeInterface::bar].Matching(a_is_bigger_than_b)).Throw(exception());
```

### Verfication
```C++
Mock<SomeInterface> mock;
SomeInterface & i = mock.get();

// Production code:
i.foo(1);
i.foo(2);
i.foo(3);

// Verify foo was invoked at least once. (The four lines do exactly the same)
Verify(mock[&SomeInterface::foo]);
Verify(mock[&SomeInterface::foo]).AtLeastOnce();
Verify(mock[&SomeInterface::foo]).AtLeast(1);
Verify(mock[&SomeInterface::foo]).AtLeast(1_Time);

// Verify foo was invoked at exactly 3 times. (The next two lines do exactly the same)
Verify(mock[&SomeInterface::foo]).Exactly(3);
Verify(mock[&SomeInterface::foo]).Exactly(3_Times);

// Verify foo(1) was invoked exactly once
Verify(mock[&SomeInterface::foo].Using(1)).Once();
Verify(mock[&SomeInterface::foo].Using(1)).Exactly(Once);
```
Wow, can I verify the order of invocations too?
```C++
// Verify foo(1) was invoked before foo(3)
Verify(mock[&SomeInterface::foo].Using(1), mock[&SomeInterface::foo].Using(3));
```