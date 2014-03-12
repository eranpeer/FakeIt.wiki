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

// Stub 'foo(1)' to always return '100'. For all other calls return 0.
When(mock[&SomeInterface::foo]).AlwaysReturn(0); // Any invocation of foo will return 0
When(mock[&SomeInterface::foo].Using(1)).AlwaysReturn(100); // override only for 'foo(1)'
```

