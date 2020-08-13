# Interresting-Cpp-Pattern
Description of pattern that I discover. Source, brief and use examples

## Summary :
- [SFINAE](##SFINAE "SFINAE pattern")
- [CRTP](##CRTP "CRTP pattern")
- [Strong_type](##Strong_type "Type_strong pattern")

## SFINAE : 
### Brief :

### Sources : 


## CRTP :
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

auto result = bmi(w, h);
```

### Sources : 
- https://www.youtube.com/watch?v=P32hvk8b13M : Chrono cpp cone
- https://foonathan.net/2016/10/strong-typedefs/ : blog type_strong library
