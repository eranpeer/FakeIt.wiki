## Stubbing
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

// Always return a value
When(mock[&SomeInterface::foo]).AlwaysReturn(1);
```
What if I want to be more specific?
```C++
// Return 100 only when calling foo with argument 1. (The next two lines do the same)
When(mock[&SomeInterface::foo].Using(1)).Return(100);
When(mock[&SomeInterface::foo](1)).Return(100);

// Return 100 only when calling foo with argument 1. For all other calls return 0.
When(mock[&SomeInterface::foo]).AlwaysReturn(0);
When(mock[&SomeInterface::foo].Using(1)).Return(100); // override the 'AlwaysReturn' only for 'foo(1)'
```

