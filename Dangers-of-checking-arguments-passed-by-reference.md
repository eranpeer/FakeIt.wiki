# _<ins>**THIS PAGE HAS BEEN REMOVED BECAUSE THERE IS AN ONGOING DISCUSSION ELSEWHERE**</ins>_

For up to date information about this subject, check out this issue: https://github.com/eranpeer/FakeIt/issues/274

The link above contains workarounds for this bug in the main post.

***

***

***

## Original content

It is common for arguments that are simply going to be read to be passed to methods by const reference, such as:

```C++
void MyMethod(const std::string& printMe);
```

This is a very common pattern in C++ to avoid copying the parameter into the method when it isn't needed.
There is a [known issue](https://github.com/eranpeer/FakeIt/issues/31) (that might not be possible to fix) with FakeIt where these passed in values will fail verification. This is because FakeIt will store the reference (i.e. a pointer to the original spot in memory that was passed in) and not the value it contained at the time of the call. Thus if that location in memory changes (set to something else, pushed off the stack, deleted, etc) the verification will fail (or worse, crash!). 
For an example of this in practice, observe these two interfaces to be used:

```C++
class SimpleVal
{
public:
   ~SimpleVal() {}
   virtual void Set(std::string value) = 0;
};
class SimpleRef
{
public:
   ~SimpleRef() {}
   virtual void Set(const std::string& value) = 0;
};
```

The `SimpleVal` class has a set method that forces the implementation to make a copy of value whereas the `SimpleRef` class passes the value by reference allowing an implementation to not perform the copy. 

Running the following test code works as expected:

```C++
BOOST_AUTO_TEST_CASE(TrivialExampleVal)
{
   Mock<SimpleVal> mock;
   SimpleVal& obj = mock.get();
   Fake(Method(mock, Set));

   std::string testValue = "Test Value";
   obj.Set(testValue);
   testValue = "Changes to something else";

   Verify(Method(mock, Set).Using("Test Value")).Once(); // Works fine
}
```

Fakeit, implementing SimpleVal::Set, makes a copy of the argument and stories it in the collection of invocations.
However, when using the reference example:

```C++
BOOST_AUTO_TEST_CASE(TrivialExampleRef)
{
   Mock<SimpleRef> mock;
   SimpleRef& obj = mock.get();
   Fake(Method(mock, Set));
   std::string testValue = "Test Value";
   obj.Set(testValue);
   testValue = "Changes to something else";
   
   Verify(Method(mock, Set).Using("Test Value")).Once(); // Fails
}
```

The tests fails because it looks to FakeIt like the invocation is `mock.Set("Changes to something else")`. This is because the invocation that fakeit stored just kept the reference to the memory where the string was. By the time it got around to doing the verification, this had changed.

This is a really trivial example, which may have the reader thinking "just don't change it after your call". In practice this would probably be an object that does some logic then returns to the test code. A better example is:

```C++
class LogicToTestVal
{
public:
   LogicToTestVal(SimpleVal* s, std::string value) : s(s), value(value) {}

   void Logic()
   {
      s->Set(value);

      // Do more logic with value
      value = "change to something else";
      // More logic with value
   }

private:
   SimpleVal* s;
   std::string value;
};

class LogicToTestRef
{
public:
   LogicToTestRef(SimpleRef* s, std::string value) : s(s), value(value) {}

   void Logic()
   {
      s->Set(value);

      // Do more logic with value
      value = "change to something else";
      // More logic with value
   }

   void WorseLogic()
   {
      std::string otherValue = "Test Value 2";
      s->Set(otherValue);
      // otherValue goes out of scope here, something else will probably be put on the stack here
   }

private:
   SimpleRef* s;
   std::string value;
};
```

Again with this, the value test passes:

```
BOOST_AUTO_TEST_CASE(ByVal)
{
   std::string testValue = "Test Value";
   Mock<SimpleVal> mockSV;
   SimpleVal& sv = mockSV.get();
   LogicToTestVal logic(&sv, testValue);

   Fake(Method(mockSV, Set));

   logic.Logic();

   Verify(Method(mockSV, Set).Using(testValue)).Once();
}
```

While the corresponding reference test fails:

```C++
BOOST_AUTO_TEST_CASE(ByRef)
{
   std::string testValue = "Test Value";
   Mock<SimpleRef> mockSR;
   SimpleRef& sr = mockSR.get();
   LogicToTestRef logic(&sr, testValue);
   Fake(Method(mockSR, Set));
   logic.Logic();
   Verify(Method(mockSR, Set).Using(testValue)).Once();
}
```

I've also added a more dangerous example to the reference logic. Where the referenced item goes out of scope, this test doesn't just fail, it crashes.

```C++
BOOST_AUTO_TEST_CASE(ByRefWorse)
{
   Mock<SimpleRef> mockSR;
   SimpleRef& sr = mockSR.get();
   LogicToTestRef logic(&sr, "");

   Fake(Method(mockSR, Set));

   logic.WorseLogic();

   // Crashes, referenced value went out of scope
   Verify(Method(mockSR, Set).Using("Test Value 2")).Once();
}
```

The full code for these examples can be found at the [gist for main.cpp](https://gist.github.com/teeks99/072ba5a093e6449a0f2dc2400948d713).