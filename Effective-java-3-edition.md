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
    public class NutritionFacts {
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

            public NutritionFacts build() {
                return new NutritionFacts(this);
            }

            private NutritionFacts(Builder builder) {
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





