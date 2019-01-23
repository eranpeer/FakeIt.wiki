One of my teammates came across an issue where they were trying to compare two vectors of objects:

```C++
#include <boost/fakeit.hpp>
#include <boost/test/unit_test.hpp>
#include <vector>

using namespace fakeit;

class AType
{
public:
   int a = 1;
   int b = 2;
};

class AnInterface
{
public:
   virtual void AFunction(std::vector<AType> input) = 0;
};

struct VectorFixture
{
   VectorFixture()
   {
      Fake(Method(mockInterface, AFunction));
   }

   Mock<AnInterface> mockInterface;
};

BOOST_AUTO_TEST_SUITE(Vector)

BOOST_FIXTURE_TEST_CASE(VecType, VectorFixture)
{
   std::vector<AType> inputs;

   AType a;
   a.a = 3;
   inputs.push_back(a);
   AType b;
   b.b = 4;
   inputs.push_back(b);

   AnInterface& ai = mockInterface.get();
   ai.AFunction(inputs);

   Verify(Method(mockInterface, AFunction).Using(inputs));
}

BOOST_AUTO_TEST_SUITE_END()
```

This gave the unfortunate error of:

```
1>------ Build started: Project: FakeItVector, Configuration: Debug Win32 ------
1>FailingCheck.cpp
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2919): error C2672: 'operator __surrogate_func': no matching overloaded function found
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2943): note: see reference to function template instantiation 'bool std::_Equal_unchecked1<_InIt1,_InIt2,_Pr>(_InIt1,_InIt1,_InIt2,_Pr &,std::false_type)' being compiled
1>        with
1>        [
1>            _InIt1=const AType *,
1>            _InIt2=const AType *,
1>            _Pr=std::equal_to<void>
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2962): note: see reference to function template instantiation 'bool std::_Equal_unchecked<_InIt1,const AType*,_Pr>(_InIt1,_InIt1,_InIt2,_Pr &)' being compiled
1>        with
1>        [
1>            _InIt1=const AType *,
1>            _Pr=std::equal_to<void>,
1>            _InIt2=const AType *
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2974): note: see reference to function template instantiation 'bool std::_Equal_no_deprecate1<const AType*,_InIt2,_Pr>(_InIt1,_InIt1,_InIt2,_Pr &,std::random_access_iterator_tag,std::random_access_iterator_tag)' being compiled
1>        with
1>        [
1>            _InIt2=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _Pr=std::equal_to<void>,
1>            _InIt1=const AType *
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2984): note: see reference to function template instantiation 'bool std::_Equal_no_deprecate<_InIt1,_InIt2,_Pr>(_InIt1,_InIt1,_InIt2,_Pr &)' being compiled
1>        with
1>        [
1>            _InIt1=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _InIt2=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _Pr=std::equal_to<void>
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(3008): note: see reference to function template instantiation 'bool std::equal<_InIt1,_InIt2,std::equal_to<void>>(_InIt1,_InIt1,_InIt2,_Pr)' being compiled
1>        with
1>        [
1>            _InIt1=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _InIt2=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _Pr=std::equal_to<void>
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\vector(1814): note: see reference to function template instantiation 'bool std::equal<std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>>(_InIt1,_InIt1,_InIt2)' being compiled
1>        with
1>        [
1>            _InIt1=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>,
1>            _InIt2=std::_Vector_const_iterator<std::_Vector_val<std::_Simple_types<AType>>>
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(6364): note: see reference to function template instantiation 'bool std::operator ==<AType,std::allocator<_Ty>>(const std::vector<_Ty,std::allocator<_Ty>> &,const std::vector<_Ty,std::allocator<_Ty>> &)' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(6363): note: while compiling class template member function 'bool fakeit::internal::EqMatcherCreator<std::vector<AType,std::allocator<_Ty>>>::Matcher::matches(const T &) const'
1>        with
1>        [
1>            _Ty=AType,
1>            T=std::vector<AType,std::allocator<AType>>
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(6369): note: see reference to class template instantiation 'fakeit::internal::EqMatcherCreator<std::vector<AType,std::allocator<_Ty>>>::Matcher' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(6368): note: while compiling class template member function 'fakeit::TypedMatcher<std::vector<AType,std::allocator<_Ty>>> *fakeit::internal::EqMatcherCreator<std::vector<_Ty,std::allocator<_Ty>>>::createMatcher(void) const'
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7407): note: see reference to function template instantiation 'fakeit::TypedMatcher<std::vector<AType,std::allocator<_Ty>>> *fakeit::internal::EqMatcherCreator<std::vector<_Ty,std::allocator<_Ty>>>::createMatcher(void) const' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7407): note: see reference to class template instantiation 'fakeit::internal::EqMatcherCreator<std::vector<AType,std::allocator<_Ty>>>' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7663): note: see reference to function template instantiation 'void fakeit::MatchersCollector<0,std::vector<AType,std::allocator<_Ty>>>::CollectMatchers<std::vector<_Ty,std::allocator<_Ty>>>(const Head &)' being compiled
1>        with
1>        [
1>            _Ty=AType,
1>            Head=std::vector<AType,std::allocator<AType>>
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7663): note: see reference to function template instantiation 'void fakeit::MatchersCollector<0,std::vector<AType,std::allocator<_Ty>>>::CollectMatchers<std::vector<_Ty,std::allocator<_Ty>>>(const Head &)' being compiled
1>        with
1>        [
1>            _Ty=AType,
1>            Head=std::vector<AType,std::allocator<AType>>
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7765): note: see reference to function template instantiation 'void fakeit::MethodMockingContext<void,std::vector<AType,std::allocator<_Ty>>>::setMatchingCriteria<std::vector<_Ty,std::allocator<_Ty>>,void>(const std::vector<_Ty,std::allocator<_Ty>> &)' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7765): note: see reference to function template instantiation 'void fakeit::MethodMockingContext<void,std::vector<AType,std::allocator<_Ty>>>::setMatchingCriteria<std::vector<_Ty,std::allocator<_Ty>>,void>(const std::vector<_Ty,std::allocator<_Ty>> &)' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\local\binget\fakeit_2.0.4\single_header\boost\fakeit.hpp(7764): note: while compiling class template member function 'fakeit::MockingContext<void,std::vector<AType,std::allocator<_Ty>>> &fakeit::MockingContext<void,std::vector<_Ty,std::allocator<_Ty>>>::Using(const std::vector<_Ty,std::allocator<_Ty>> &)'
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\tmp\fakeitvector\failingcheck.cpp(46): note: see reference to function template instantiation 'fakeit::MockingContext<void,std::vector<AType,std::allocator<_Ty>>> &fakeit::MockingContext<void,std::vector<_Ty,std::allocator<_Ty>>>::Using(const std::vector<_Ty,std::allocator<_Ty>> &)' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>d:\tmp\fakeitvector\failingcheck.cpp(24): note: see reference to class template instantiation 'fakeit::MockingContext<void,std::vector<AType,std::allocator<_Ty>>>' being compiled
1>        with
1>        [
1>            _Ty=AType
1>        ]
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2919): error C2893: Failed to specialize function template 'unknown-type std::equal_to<void>::operator ()(_Ty1 &&,_Ty2 &&) const'
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2919): note: With the following template arguments:
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2919): note: '_Ty1=const AType &'
1>c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility(2919): note: '_Ty2=const AType &'
1>Done building project "FakeItVector.vcxproj" -- FAILED.
```

After some parsing of MS's very verbose error output, I distilled it down to:

```
Error	C2893	Failed to specialize function template 'unknown-type std::equal_to<void>::operator ()(_Ty1 &&,_Ty2 &&) const'	FakeItVector	c:\program files (x86)\microsoft visual studio 14.0\vc\include\xutility	2919	
```
Trying a similar thing with a simple vector (`vector<int>` instead of `vector<AType>`) worked fine.
```C++

class OtherInterface
{
public:
   virtual void OtherFunction(std::vector<int> input) = 0;
};


struct VectorFixture
{
	VectorFixture()
	{
		Fake(Method(mockInterface, AFunction));
      Fake(Method(mockOther, OtherFunction));
	}

	Mock<AnInterface> mockInterface;
   Mock<OtherInterface> mockOther;
};

BOOST_FIXTURE_TEST_CASE(VecScalar , VectorFixture)
{
   std::vector<int> inputs = { 1, 2, 3 };

   OtherInterface& oi = mockOther.get();
   oi.OtherFunction(inputs);

   Verify(Method(mockOther, OtherFunction).Using(inputs));
}
```
Gives

```
Running 1 test cases...

*** No errors detected
```

So, I tried adding an operator to AType.
```
class AType
{
public:
	int a = 1;
	int b = 2;

   bool operator==(AType& other)
   {
      if (this->a == other.a && this->b == other.b)
      {
         return true;
      }
      return false;
   }
};
```

But that didn't solve it, apparently there isn't a way to compare vectors generically? Adding an equality operator for the vector worked though!

```C++
bool operator==(std::vector<AType> one, std::vector<AType> two)
{
   if (one.size() != two.size())
   {
      return false;
   }

   for (size_t i = 0; i < one.size(); i++)
   {
      if (!(one[i] == two[i]))
      {
         return false;
      }
   }
   return true;
}
```

We were then able to test and see successes and failures. Yay!