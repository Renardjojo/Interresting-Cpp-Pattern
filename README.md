# Interresting-Cpp-Pattern
Description of pattern that I discover. Source, brief and use examples

##Summary :
- [SFINAE](##SFINAE "SFINAE pattern")
- [CRTP](##CRTP "CRTP pattern")
- [Strong_type](##Strong_type "Type_strong pattern")

##SFINAE : 
###Brief :

### Sources : 

##CRTP :
###Brief :

### Sources : 

##Strong_type : 

###Brief :
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
