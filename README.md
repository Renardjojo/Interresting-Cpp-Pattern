# Interresting-Cpp-Pattern
Description of pattern that I discover. Source, brief and use examples

## Summary :
- [SoA or AoS](##_SoA_or_AoS: "SoA or AoS")
- [Bool wrapping](##_Bool_wrapping: "Bool wrapping")
- [RAII (WIP)](##_RAII(Resource_acquisition_is_initialization): "RAII pattern")
- [RVO (WIP)](##_RVO(Return_Value_Optimization): "RVO pattern")
- [SFINAE](##_SFINAE(Substitution-Failure-is-not-an-Error): "SFINAE pattern")
- [CRTP](##_CRTP(Curiously-recurring-template-pattern): "CRTP pattern")
- [Strong_type](##_Strong_type: "Type_strong pattern")
- [Phantom](##_Phantom: "Phantom pattern")
- [Template expression (WIP)](##_Template_expression: "Template expression")
- [Familly generator](##_Familly_generator: "Familly generator")


## Bool wrapping:
### Brief :
Must we use Array of struct or struct of array ? 
Struct of array should be better option for system that must be upgrade like Mesh (position/Uv/normal or Position/Color/normal ?)
### Sources :
- https://www.bfilipek.com/2014/04/flexible-particle-system-container.html : Great article about particle system that use example of AoS confronting to SoA



## Bool wrapping:
### Brief :
When multiple boolean is store on structur, these boolean size 1 bytes instead of 1 bit one behind the other. In example :

```cpp
#include <string>
#include <iostream>
#include <vector>
 
class t1
{
    bool a, b, c, d, e, f, g, h, i;
};

class t2
{
    bool a : 1;
    bool b : 1;
    bool c : 1;
    bool d : 1;
    bool e : 1;
    bool f : 1;
    bool g : 1;
    bool h : 1;
};

int main()
{
    std::cout << sizeof(t1) << std::endl; //8
    std::cout << sizeof(t2) << std::endl; //1
}
```

### Sources :


## SFINAE(Substitution-Failure-is-not-an-Error):
### Brief :

### Note :
In c++ 2020 SFINAE can be replace by "concept"

### Sources : 
- http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1219r1.html : Homogeneous variadic function parameters (speack about variadic SFINAE) by James Touton


## CRTP(Curiously-recurring-template-pattern):
### Brief :
It's a template pattern that allow to avoid function copy on inherrance. CRTP is not static virtual and do not avoid virtual implemantion. Don't use CRTP, which is not dynamic polymorphism, to create dynamic polymorphism like :
```cpp
template <typename Derived>
struct Base{
  void interface(){
    static_cast<Derived*>(this)->implementation();
  }
  void implementation(){
    std::cout << "Implementation Base" << std::endl;
  }
};

struct Derived1: Base<Derived1>{
  void implementation(){
    std::cout << "Implementation Derived1" << std::endl;
  }
};

struct Derived2: Base<Derived2>{
  void implementation(){
    std::cout << "Implementation Derived2" << std::endl;
  }
};

int main()
{
    std::vector<std::unique_ptr<Base>> pBases; /*Don't work. Witch Base ? Base<Derived1> or Base<Derived2> ??*/
}
```
However, this pattern can be similare to static polimorphism and avoid usage of virtual keyword.
```cpp
// crtp.cpp

#include <iostream>

template <typename Derived>
struct Base{
  void interface(){
    static_cast<Derived*>(this)->implementation();
  }
  void implementation(){
    std::cout << "Implementation Base" << std::endl;
  }
};

struct Derived1: Base<Derived1>{
  void implementation(){
    std::cout << "Implementation Derived1" << std::endl;
  }
};

struct Derived2: Base<Derived2>{
  void implementation(){
    std::cout << "Implementation Derived2" << std::endl;
  }
};

struct Derived3: Base<Derived3>{};

template <typename T>
void execute(T& base){
    base.interface();
}

int main()
{
  Derived1 d1;
  execute(d1);
    
  Derived2 d2;
  execute(d2);
  
  Derived3 d3;
  execute(d3);
  
}
```

In additionnal, this pattern allow to avoid code diplication like :
```cpp
/*Without CRTP*/

struct IShape
{
    virtual ~IShape() = default;
    virtual std::unique_ptr<IShape> clone() const = 0; /*Use vtable*/
};

struct Circle : IShape
{
    std::unique_ptr<IShape> clone() const override { return std::make_unique<Circle>(*this); } /*Yea ok*/
};

struct Square : IShape
{
    std::unique_ptr<IShape> clone() const override { return std::make_unique<Square>(*this); } /*Umhhh duplication ? So bad*/
};

/*With CRTP*/

template <class Derived>
struct CRTPShape
{
    std::unique_ptr<CRTPShape> clone() const /*don't use vtable*/
    { 
        return std::make_unique<Derived>(static_cast<const Derived&>(*this));
    }
};

struct CRTPCircle : CRTPShape<Circle>
{
    /*Clone function is automatically define by the compilator*/
};

struct CRTPSquare : CRTPShape<Square>
{
    /*Same*/
};
```

### Sources : 
- https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern : Wikipedia
- https://www.fluentcpp.com/2017/05/16/what-the-crtp-brings-to-code/ : Tuto


## Strong_type : 

### Brief :
Patern to explicit unit like meter, kilometer, second, degres, radian... With compile error if unit is not respected. Allow explicit declaration and avoid error like : 
```cpp
using meter = int;
using kilogram = int;

int bmi(meter height, kilogram weight);

auto result = bmi(w, h); /*Error w and h is not the same but it compile*/
```

Thanks to strong_type you can do implicit conversion like Chrono library :

```cpp
void foo (milliseconds d)
{
    std::cout << d.count() << "ms\n";
}

foo(3s); /*Implicit conversion from second to millisecond*/
```

### Sources : 
- https://www.youtube.com/watch?v=P32hvk8b13M : Chrono cpp cone
- https://foonathan.net/2016/10/strong-typedefs/ : blog type_strong library



## Phantom : 
### Brief :
Phantom is a type use on template that is never use to define internal method type. It's only type to create a different class type :

```cpp

template<typename T, typename Phantom>
struct Duration
{
  explicit Duration(const T& in_duration) { [...] }
}

Duration<uint32_t, Seconds> /*Seconds is the phantom type*/
Duration<uint32_t, Hours>   /*Hours is the phantom type*/

/*Implementation of both duration is strictly identical*/
```
With this solution, Duration<uint32_t, Seconds> != Duration<uint32_t, Hours> 

### Sources : 
- https://www.youtube.com/watch?v=ojZbFIQSdl8&feature=youtu.be&t=24m9s : CppCon 2016: Ben Deane “Using Types Effectively"
- https://medium.com/@snowp/transparent-phantom-types-in-c-de6ac5bed1d1 : article "Transparent Phantom Types in C++" by Snow Pettersen


## Family generator
Familly generator allow you to create ID for a specific type

```cpp
#include <iostream>
 
class family {
    static std::size_t identifier() noexcept {
        static std::size_t value = 0;
        return value++;
    }

public:
    template<typename>
    static std::size_t type() noexcept {
        static const std::size_t value = identifier();
        return value;
    }
};

int main()
{
    std::cout << family::type<int>() << std::endl; //0
    std::cout << family::type<float>() << std::endl; //1
    std::cout << family::type<int>() << std::endl; //0
    std::cout << family::type<unsigned int>() << std::endl //2;
}
 ```
### Sources :
- https://skypjack.github.io/2019-02-14-ecs-baf-part-1/ : The first article thhat I see and discuss about it
