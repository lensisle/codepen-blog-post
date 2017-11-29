We all know that our beloved Javascript language is somewhat insecure, so errors at runtime (hello 'undefined' is not an object) occur very frequently when systems grow in complexity.

One way to improve security in that regard is by declaring a factory function that allows us to create and use our plain old javascript objects in a more secure way using ES6 proxies.

```
const proxyHandler = {
    get(target, property) {
      if (property in target) {
        return target[property];
      } else {
        throw new ReferenceError(`Property ${property} does not exist.`);
      }
    }
};

function createSafeObject(insecureObject) {
  const proxiedObject = new Proxy(insecureObject, proxyHandler);
  Object.preventExtensions(proxiedObject);
  return proxiedObject;
}

function Vehicle(model, year) {
  this.model = model;
  this.year = year;
  return createSafeObject(this);
}

const car = new Vehicle('Mustang', 1969); // super safe car!
```

**Explanation**

```
const proxyHandler = {
    get(target, property) {
      if (property in target) {
        return target[property];
      } else {
        throw new ReferenceError(`Property ${property} does not exist.`);
      }
    }
};
```

The variable **proxyHandler** is passed as an argument to the Proxy constructor to 'intercept' access to a target (like a trap). A handler will override the default behaviour provided by the language and define a custom one using a function. In this case we are overriding the 'get' operation to modify the access of a property of the target in some other way.

```
function createSafeObject(insecureObject) {
  const proxiedObject = new Proxy(insecureObject, proxyHandler);
  Object.preventExtensions(proxiedObject);
  return proxiedObject;
}
```

Here we wrap our original object into a Proxy using the handler that was previously defined. In addition to that, we call Object.preventExtensions to ensure that consumers do not add properties at runtime.

After creating a vehicle instance, we will create an object that prevents run-time extensions and warns us if we try to access to undefined properties.

```
const car = new Vehicle('Mustang', 1969);
console.log(car.model) // 'Mustang'
console.log(car.year) // 1969
console.log(car.color) // Property color not found
car.mileage = 10000; // Error
```

The best thing is that our object is still a Vehicle even when we are wrapping the original instance with a proxy.

```
const car = new Vehicle('Mustang', 1969);
console.log(car instanceof Vehicle); // true
```

ps: You can also use Object.freeze and get similar results, but you won't be able to register custom errors or check only certain properties.