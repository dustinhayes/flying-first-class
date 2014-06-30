# Flying First Class
#### An exploration of JavaScript's function first classness

## From Wikipedia
- In computer science, a programming language is said to have first-class functions if it treats functions as first-class citizens. Specifically, this means the language supports passing functions as arguments to other functions, returning them as the values from other functions, and assigning them to variables or storing them in data structures.

#### Well use this function as an example:
```javascript
function sum(a, b, c) {
    return a + b + c;
}
```

## Assignment and Storage
We can assign references to functions using the var keyword, or store them in objects.

### Assignment
```javascript
var sum = function sum(a, b, c) {
    return a + b + c;
};

sum(1, 2, 3);
// 6
```

### Storage
```javascript
var math = {
    sum: function sum(a, b, c) {
        return a + b + c;
    }
};

math.sum(1, 2, 3);
// 6
```

## Function Can Fly Anywhere
More importantly, we can pass functions around.

### Parameter Values
```javascript
sum(1, 2, function () {
    return 3;
}());
// 6
```

### Argument Values
```javascript
var sum =  function sum(a, b, func) {
    return a + b + func();
};

sum(1, 2, function () {
    return 3;
});
// 6
```

The ability to call functions from within other functions is, in my opinion, the most powerful feature of the JavaScript language.

## A Real Life Example: `Array.prototype.forEach`

### Example
```javascript
var array = [1, 2, 3];

array.forEach(function (elm, ind, arr) {
    console.log(elm, 'is at index', ind, 'in', arr);
});
// 1 "is at index" 0 "in" [1, 2, 3]
// 2 "is at index" 1 "in" [1, 2, 3]
// 3 "is at index" 2 "in" [1, 2, 3]
```

### Implementation
Remember, `this` is equal to the array `forEach()` was called on.
```javascript
Array.prototype.forEach = function forEach(func) {
    
    var arrlen = this.length,
        curitr;

    for (curitr = 0; curitr < arrlen; curitr = curitr + 1) {

        func(this[curitr], curitr, this);
    
    }

};
```

The example above clarifies the common question, "Where did these arguments come from?". When you supply `forEach` a function to execute, it sets up the parameters that will be supplied to the function passed in.

## More Power with Call and Apply
Each function you create comes with two methods (functions as object properties), `call` and `apply`. These methods almost do the same thing. They both allow you to execute a function, provide what `this` will equal within that function, and supply any parameters the function expects. The difference is in the parameter end of things. For `call`, the parameters are supplied as a comma separated list, as you are used to doing. `apply`, on the other hand is supplied a single array as a parameter list, which gets expanded into individual parameters.

### Signature
```javascript
// Call
func.call(this_val, param1, param2);

// Apply
func.apply(this_val, [param1, param2]);
```

### Example
We use `null` for the `this` value when it's irrelevant.

#### Call
```javascript
sum.call(null, 1, 2, 3);
// 6
```

#### Apply
```javascript
sum.apply(null, [1, 2, 3]);
// 6
```

## Real Life Example: Context in Array Iterators and Partial Application

### Context in Array Iterators
In the `forEach` example above, I had to pass the array as the third parameter to gain access to it. I would be nice to have that be available via the `this` value. We can accomplish that with `call`.

```javascript
Array.prototype.forEach = function forEach(func, this_val) {
    
    var arrlen = this.length,
        curitr;

    for (curitr = 0; curitr < arrlen; curitr = curitr + 1) {

        func.call(this_val, this[curitr], curitr, this);
    
    }

};
```

```javascript
var array = [1, 2, 3];

array.forEach(function (elm, ind) {
    console.log(elm, 'is at index', ind, 'in', this);
}, array);
// 1 "is at index" 0 "in" [1, 2, 3]
// 2 "is at index" 1 "in" [1, 2, 3]
// 3 "is at index" 2 "in" [1, 2, 3]
```

### Partial Application
To implement partial application, we rely heavily on the fact that the arguments object can by coerced into a true array, and that `apply()` expects it's parameter list as an array.

#### Implementation
```javascript
var makearr = function makearr( o ) {
        return Array.prototype.slice.call( o );
    },

    partial = function partial() {
        var args = makearr( arguments ),
            func = args.shift();
            
        return function () {
            var all = args.concat(
                makearr( arguments )
            );
            
            return func.apply( null, all );
        }
    };
```

##### Example of using `partial()`
```javascript
var sum_1_2 = partial( sum, 1, 2);

sum_1_2(3);
// 6

console.dir( sum_1_2 );
```
