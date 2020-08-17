# Creating and Destroying Objects

## Item 1: Consider static factory methods instead of constructors
- **public static factory method** that returns an instance of the class
- eg.
    ```java
    public static Boolean valueOf(boolean b) {
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
    ```
### Advantages: 

- **they have names**
- class can have multiple factory methods with same signature
- they are not required to create new object each time called (instance control)
- **they can return an object of any subtype of their return type** (can hide classes in API making tem private)
- class of returned object can vary from call to call as a function of the input (any subtype of returned type is permissible)
- class of the returned object need not exist when the method is written 

### Disadvantages:

- class with no public or protected constructors cannot be subclassed
- **static factory methods are hard to find for programmers**

### Common names:

| Name        | usage           |
| ------------- |:-------------:|
| from      | type-conversion |
| of      | aggregation |
| valueOf | alternative for *from* and *of* |
| instance / getInstance | return instance depending on arguments |
| create / newInstance | like *instance / getInstance* but always creates new resource|
| getType | like *getInstance* but *Type* specifies the return type eg. Files.getFileStore() |
| newType | like *newInstance* but -\|\|- |
| type | alternative for *getType* and *newType* |

---

## Item 2: Consider a builder when faced with many constructor parameters
- constructors and factory methods do not work well with large number of optional parameters
- solution: **builders** 
- eg.
    ```java
    public class ClassTobeBuilt {
        private final int required1;
        private final int required2;
        //optional parameters
        private final int ...

        public static class Builder {
            //Required parameters
            private final int required1;
            private final int required2;

            //Optional parameters - initialized by default values
            private final int optional1;
            ...
            public Builder(int required1, int required2) {
                this.required1 = required1;
                this.required2 = required2;
            }

            public Builder optional1(int val) {
                optional1 = val;
                return this;
            }
            ...

            public ClassTobeBuilt build() {
                return new ClassTobeBuilt(this);
            }

            private ClassTobeBuilt(Builder builder) {
                required1 = builder.required1;
                required2 = builder.required2;
                optional1 = builder.required1;
                ...
            }
        }
    }
    ```
### Advantages:
- **Allows to build *fluent API***
- **Simulates named optional parameters**
- Well suited to class hierarchies
- Can have multiple varargs parameters
- **Client code is easier to read and write**

### Disadvatages:
- In order to create object we have to create its builder (**problem in performace-critical applications**)
- **more verbose than static factory methods or constructors**

---

## Item 3: Enforce the singleton property with a private constructor or an enum type
- three ways to create singleton
  - public final field
    - API makes it clear it is a singleton
    - Is simpler than factory method
    ```java
    public class OnlyOne {
        public static final OnlyOne INSTANCE = new OnlyOne();
        private OnlyOne() {
            throw new AssertionError("The constructor should not be used");
    }
    ```
  - static factory method
    - flexibility to not be singleton
    - can be used as a supplier
    ```java
    public class OnlyOne {
    private static final OnlyOne INSTANCE = new OnlyOne();
    private OnlyOne() {
        throw new AssertionError("The constructor should not be used");
    }
    public static OnlyOne getInstance() {
        return INSTANCE;
    }
    ```
  - **single-element enum**
    - the simplest
    - free serialization
    - 100% guarantee no unwanted instatiations
  ```java
  public enum OnlyOne {
      INSTANCE;
  }
  ```
### Disadvatages:
- **it is difficult to test singleton class clients**
- **sometimes consider as antipattern**

---

## Item 4: Enforce nonistantiability with a private constructor
- for utility classes, singletons, classes using factory methods etc.
- for suprassing the default no args constructor
- could throw exception in private constructor to ensure it is not used
- prevent class from being subclassed (subclasses need to be able to invoke superclass constructor)
- eg.
  ```java
  public class WithNoPublicConstructor {
      //created to hide implicit public constructor
      private WithNoPublicConstructor() {
          throw new AssertionError("The constructor should not be used");
      }
  }
  ```
### Advantages:
- **prevents the nonsense of creating the instance of utility class**

---

## Item 5: Prefer dependency injection to hardwiring resources
- problem: many classes depend on many underlying resources
- static utility classes and singletons are not a valid  choice for class parameterized by underlying resource
- solution: pass the resource/factory (inject) into the constructor when creating a new instance
- applied by frameworks like Spring, Dagger or Guice
- eg.
  ```java
  public class SpellChecker {
      private final Lexicon dictionary; //a dependency

      public SpellChecker(Lexicon dictionary) {
          this.dictionary = Objects.requireNonNull(dictionary);
      }
  }
  ```

### Advantages:
- **improves flexibility, reusability and testability**

---

## Item 6: Avoid creating unnecessary objects
- **reuse a single object instead of creating a new instance**
- can be obtained by using static factory methods on immutable classes
  (eg. use `Boolean.valueOf(String)` instead of `new Boolean(String)`) - constructor <u>must</u> create new object
  - in other cases try to cache results
  - eg.
    ```java
    public class WithPatternMatching {
        //avoids compiling regex each time it is used and improving performance
        /*lazy initialization is tempting, but it is not reccomended as it greatly
        complicates the implementation with no mesurable performance improvement*/
        private static final Pattern PATTERN = Pattern.compile("regex");
        static boolean isMatchingThePattern(String s) {
            return PATTERN.matcher(s).matches();
        }
    }
    ```
- other case is adapter classes (views). They also do not have state on they own - eq. `keySet()` from `Map` interface
- **avoid unnecessary autoboxing**
- in most cases **avoid maitaining your own object pool** (the special case is database connection. It is extremely heavyweight obcjet and it is wise to have a obcjet pool in that case)

### Advantages:
- could lead to cleaner code
- **could improve performance**

### Disadvatages:
- **should be use wisely, as when it is misused then it could lead to hideous errors**
- could make making *defensive copy* a harder task

---

## Item 7: Eliminate obsolete obcjet references
