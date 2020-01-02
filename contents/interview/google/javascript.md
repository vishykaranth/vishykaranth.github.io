---
layout: page
title: Javascript
permalink: /google-javascript/
---

## Javascript Basics to Advanced 

### Variables 

~~~javascript 
var a = 'word';
var b = false;
var c = true;
var d = 0
var e = 1
var f = 2
var g = null

console.log(a || b); // 'word'
console.log(c || a); // true
console.log(b || a); // 'word'
console.log(e || f); // 1
console.log(f || e); // 2
console.log(d || g); // null
console.log(g || d); // 0
console.log(a && c); // true
console.log(c && a); // 'word'
~~~

### Objects 

~~~javascript 
const bankAccount = (initialBalance) => {
  var balance = initialBalance;

  return {
    getBalance: function() {
      return balance;
    },
    deposit: function(amount) {
      balance = balance + amount;
      return balance;
    },
  };
};

const account = bankAccount(100);

console.log(account.getBalance()); // 100
console.log(account.deposit(10)); // 110
~~~