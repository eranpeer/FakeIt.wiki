## Stubbing
```C++
// Stub a method to return a value once
When(mock[&SomeInterface::foo]).Return(1);

// Stub multiple return values
When(mock[&SomeInterface::foo]).Return(1,2,3);
// or
When(mock[&SomeInterface::foo]).Return(1).Return(2).Return(3);

// Return the same value many time (56 in this example)
When(mock[&SomeInterface::foo]).Return(56_Times(1));

// Return many values many times. (First 100 calls will return 1, next 200 calls will return 2)
When(mock[&SomeInterface::foo]).Return(100_Times(1),200_Times(2));

// Always return a value
When(mock[&SomeInterface::foo]).AlwaysReturn(1);

```