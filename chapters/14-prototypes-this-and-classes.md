# 14. Prototypes, this, and classes

[Back to notes index](../README.md)

| [Previous: Modules, npm, and project structure](13-modules-npm-and-project-structure.md) | [Notes index](../README.md) | [Next: Collections, iterators, and generators](15-collections-iterators-and-generators.md) |
|:--|:--:|--:|

## Objects can delegate through prototypes

JavaScript objects can inherit properties from another object through a prototype link. When a property is not found directly on an object, JavaScript looks up its prototype, then continues through the prototype chain.

~~~js
const baseNote = {
  describe() {
    return 'A study note';
  },
};

const note = Object.create(baseNote);
note.title = 'Prototypes';

console.log(note.title);
console.log(note.describe());
console.log(Object.hasOwn(note, 'describe')); // false
console.log(Object.getPrototypeOf(note) === baseNote); // true
~~~

The describe method is found on the prototype, while title is an own property. Object.hasOwn checks only direct properties. The in operator checks both the object and its prototype chain.

Most object literals inherit from Object.prototype. Objects created with Object.create(null) have no prototype, which can be useful for a dictionary when inherited keys should not be present.

~~~js
const dictionary = Object.create(null);
dictionary['safe-key'] = 'value';

console.log(Object.getPrototypeOf(dictionary)); // null
~~~

## Understand this from the call site

For a regular function or method, this depends on how the function is called. In a method call, the object before the dot is the receiver.

~~~js
const learner = {
  name: 'Ashish',
  introduce() {
    return 'I am ' + this.name;
  },
};

console.log(learner.introduce());
~~~

If a method is detached from its object, the call no longer has that receiver. In strict mode, this is undefined for a plain function call.

~~~js
const introduce = learner.introduce;

// Calling introduce() has no learner receiver.
~~~

To preserve the receiver, bind the method or call it through the object.

~~~js
const boundIntroduce = learner.introduce.bind(learner);
console.log(boundIntroduce());
~~~

call and apply invoke a function immediately with a chosen this value. bind returns a new function with a chosen this value for later calls.

~~~js
function describeRole(role) {
  return this.name + ' is a ' + role;
}

const person = { name: 'Ashish' };

console.log(describeRole.call(person, 'developer'));
console.log(describeRole.apply(person, ['developer']));
~~~

## Arrow functions capture this

An arrow function does not create its own this. It reads this from the surrounding lexical scope. This is useful for callbacks inside a method when the callback needs the surrounding object's this.

~~~js
const counter = {
  value: 0,
  start() {
    setTimeout(() => {
      this.value += 1;
      console.log(this.value);
    }, 100);
  },
};

counter.start();
~~~

Use a regular method when the function should receive its receiver from the call site. Use an arrow for a callback that should keep the surrounding this. Arrow functions cannot be used as constructors and do not have their own arguments object.

## Create instances with class

A class defines a constructor and methods for its instances. Calling a class with new creates an object and runs the constructor.

~~~js
class StudyNote {
  constructor(title, body) {
    this.title = title;
    this.body = body;
    this.reviewed = false;
  }

  markReviewed() {
    this.reviewed = true;
  }

  get summary() {
    return this.title + ': ' + this.body;
  }
}

const note = new StudyNote('Classes', 'Class methods use prototypes.');
note.markReviewed();

console.log(note.summary);
~~~

Methods declared in the class body are stored on StudyNote.prototype, so instances share the same method function. Instance fields such as title belong to each instance.

~~~js
console.log(Object.hasOwn(note, 'title')); // true
console.log(Object.hasOwn(note, 'markReviewed')); // false
console.log(Object.hasOwn(StudyNote.prototype, 'markReviewed')); // true
~~~

A getter can provide a computed property-like value. Do not make a getter perform surprising work such as a network request.

## Add private fields and static methods

A private field begins with # and can only be accessed inside the declaring class body. It helps protect an instance's internal state from direct external changes.

~~~js
class StudyCounter {
  #count = 0;

  increment() {
    this.#count += 1;
    return this.#count;
  }

  get value() {
    return this.#count;
  }

  static create() {
    return new StudyCounter();
  }
}

const counter = StudyCounter.create();
counter.increment();

console.log(counter.value); // 1
~~~

A static method belongs to the class itself rather than an instance. It is useful for a factory or behavior that does not depend on one instance.

## Extend a class carefully

A subclass can extend a base class and call super to run the base constructor. It can add fields or override methods.

~~~js
class ReferenceNote extends StudyNote {
  constructor(title, body, sourceUrl) {
    super(title, body);
    this.sourceUrl = sourceUrl;
  }

  openSource() {
    return this.sourceUrl;
  }
}

const reference = new ReferenceNote(
  'Prototypes',
  'Objects delegate property lookup.',
  'https://developer.mozilla.org/',
);

console.log(reference instanceof StudyNote); // true
console.log(reference.openSource());
~~~

Inheritance is useful when the subtype can safely stand in for the base type. Deep inheritance trees couple classes and make changes harder. Prefer composition when an object can use another object to perform a task without being a specialized version of it.

## Choose an object, factory, or class

A plain object is enough for a small record. A factory function is useful when object creation needs private closure state or a customized shape. A class is useful when many instances share methods and a clear lifecycle or behavior.

~~~js
function createProgress(initialValue = 0) {
  let value = initialValue;

  return {
    increment() {
      value += 1;
      return value;
    },
    getValue() {
      return value;
    },
  };
}

const progress = createProgress();
progress.increment();

console.log(progress.getValue());
~~~

Choose the simplest form that expresses the responsibilities. Do not create a class merely because a group of functions is related.

## Avoid common prototype mistakes

- Do not assign user-controlled keys into a normal object without considering special property names and prototype behavior.
- Use Object.hasOwn when checking a direct data property.
- Avoid modifying built-in prototypes such as Array.prototype.
- Remember that methods can lose their receiver when passed as callbacks.
- Use private fields when internal state should not be directly writable.
- Treat inheritance as a real relationship, not as a shortcut for code reuse.

Prototype-related security concerns are covered in Chapter 16.

## Hands-on: create a progress object

Build a class that stores a private current value, exposes a getter, and refuses to increment above a maximum. Add a reset method and test the boundary.

~~~js
class Progress {
  #value = 0;

  constructor(maximum) {
    this.maximum = maximum;
  }

  get value() {
    return this.#value;
  }

  increment() {
    if (this.#value < this.maximum) {
      this.#value += 1;
    }

    return this.#value;
  }

  reset() {
    this.#value = 0;
  }
}

const progress = new Progress(2);

console.log(progress.increment()); // 1
console.log(progress.increment()); // 2
console.log(progress.increment()); // Still 2
~~~

Add a test for a zero maximum and one for reset. Decide whether invalid maximum values should be rejected in the constructor.

## Notes to remember

- Objects look up missing properties through their prototype chain.
- this for a regular function depends on the call site.
- Arrow functions use this from their surrounding scope.
- Class methods are shared through the class prototype.
- Private fields restrict access to class internals.
- Use inheritance for a real subtype relationship and composition for collaborators.
- Plain objects and factory functions can be simpler than classes.

## References

- [MDN inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
- [MDN this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [MDN private elements](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements)
- [MDN Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [MDN function bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

---

| [Previous: Modules, npm, and project structure](13-modules-npm-and-project-structure.md) | [Notes index](../README.md) | [Next: Collections, iterators, and generators](15-collections-iterators-and-generators.md) |
|:--|:--:|--:|
