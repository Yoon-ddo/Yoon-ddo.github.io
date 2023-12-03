---
title: React03_JavaScript Immutability
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - Project
---

# 1. JavaScript Immutability
* Many Blog's writer who wrote about React recommand to study JavaScript.
* Because React is a library which is based on JavaScript.
* Especially JavaScript Immutability.
* **What is the Immutability?**
  * It means unchangeable
  * It can prevent the original data from being damaged.

<br><br><br>

# 2. Keep variable's name Unchangeable
* const
* If you use Variable type of <span style="color:red">const</span>, you can keep variable's name.
* You can't change const variable's value. If you try, It's going to return Error.

<br>

```Javascript
var a = 1; 
a = 2;
console.log(a); // 2

const c = 1;
c = 2; // error
```

<br><br><br>

# 2. Data Type in JavaScript
* Primitive : Number, String, Boolean, Null, Unfined, Symbol ...
  * <span style="color:red">It can't be changed</span> 
* Object : Object, Array, Function ...
  * <span style="color:red">It can be changed</span>  
* JavaScript treat ***Primitive Data Type*** and ***Obect Data Type*** differently

<br>

## 2-1. Value

```JavaScript
var prm1 = 1;
var prm2 = 1;
console.log(p1===p2); // true (Two of variable's values are in same place.)
 
var obj1 = {name:'kim'}
var obj2 = {name:'kim'}
console.log(o1===o2); // false (Two of variable's values are in different place.)
```

## 2-2. Change the Value

```JavaScript
var prm1 = 1;
var prm2 = 1;
var prm3 = prm2;
console.log(prm3); // 1

var prm3 = 2;
console.log(prm3); // 2


var obj1 = {name:'John'};
var obj2 = {name:'John'};
var obj3 = obj1; 
console.log(obj3); // {name:'John'}

obj3.name = 'Kevin';
console.log(obj1, obj3, obj1 === obj3); // {name:'Kevin'} {name:'Kevin'} true
```

## 2-3. Copy and assign

```JavaScript
var obj1 = {name:'John'}
var obj2 = Object.assign({}, obj1); // obj2 become {name:'John'}
console.log(obj2); // {name:'John'}

obj2.name = 'Kevin';
console.log(obj1, obj2, obj1 === obj2); // {name:'John'} {name:'Kevin'} false
```

## 2-4. Modify value of copy without changing original

```JavaScript
var obj1 = {name:'John', score:[1,2]}
var obj2 = Object.assign({}, obj1);
obj2.score = o2.score.concat(); // copy obj2.score
obj2.score.push(3);
console.log(o1, o2, o1 === o2, o1.score === o2.score);
// {name:'John', score:[1,2]} {name:'John', score:[1,2,3]} false false
```

* If you obj2.score.push(3) without obj2.score = o2.score.concat() (= copy),  obj1 is also gonna change like as below
  * {name:'John', score: [1,2,3]}
  * And They are also different,  so `obj1 === obj2` return false.

## 2-5. Object freeze
* You can't change value, when you use `freeze`

```JavaScript
var obj1 = {age : 5};
obj1 = Object.freeze(obj1);
obj1.age = 20;
console.log(obj1.age); // 5
Object.isFrozen(obj1) // true

obj1.name = 'John';
console.log(obj1, obj1.age); // {age : 5}, 5
```

* But you can change child's value. If you use only `Object.freeze()`
```JavaScript
// But you can change child's value.
var emp = { name : "Mark", address : {  street: "Rohini", city: "Delhi",  }, };
Object.freeze(emp);
emp.name = 'Jenny';
emp.address.city = 'Seoul';
console.log(emp.name, emp.address.city) // Mark , Seoul
```

* If you wanna keep all of Object values, command like as below
```JavaScript
function deepFreeze(object) {
  var propNames = Object.getOwnPropertyNames(object);
  for (let name of propNames) {
    let value = object[name];

    object[name] =
      value && typeof value === "object" ? deepFreeze(value) : value;
  }

  return Object.freeze(object);
}

var emp2 = {
       name : 'Amy',
       address :  {
                    street: "Rohini",
                    city: "Delhi",
                  },
       favorite : {
                    food : {
                             fruit : 'apple',
                             vegetable : 'onion',
                           },
                  },
          };


deepFreeze(emp2);

emp2.favorite.food.fruit = "Mango";
console.log(emp2.favorite.food.fruit) // 'apple'
```

<br><br><br>


# 999. Studying JavaScript Immutability from
* [Learn JavaScript Blog1](https://geniee.tistory.com/6)
* [Learn JavaScript Blog2](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)
* This Page is Only for study myself. My English is not perfect.

