# Dagger

## Basics

* **Dagger** - is a fully static, compile-time dependency injection framework for Java, Kotlin, and Android. 

---

* **@Module** - annotates a class that will be able to provide the required objects. Modules are classes that contain code for creating objects. Each module includes objects that are close in meaning.

* **@Provides** - annotates methods of a module to create a provider method binding. The method's return type is bound to its returned value. The **component** implementation will pass dependencies to the method as parameters.

---

* **@Component** - annotates an interface or abstract class for which a fully-formed, dependency-injected implementation is to be generated from a set of **modules**. The generated class will have the name of the type annotated with @Component prepended with Dagger. For example, @Component interface MyComponent {...} will produce an implementation named DaggerMyComponent. 

* Every type annotated with **@Component** must contain at least one abstract component method. Component methods may have any name, but must have signatures that conform to either **provision** or **members-injection contracts**.

* **@Component(modules = [])** - a list of classes annotated with **Module** whose bindings are used to generate component implementation.

---

* **@Inject** - identifies injectable constructors, methods, and fields, which the **component** must field. May apply to static as well as instance members. An injectable member may have any access modifier (private, package-private, protected, public). Constructors are injected first, followed by fields, and then methods. Fields and methods in superclasses are injected before those in subclasses. Ordering of injection among fields and among methods in the same class is not specified. 

* Injectable constructors are annotated with **@Inject** and accept zero or more dependencies as arguments. **@Inject** can apply to at most one constructor per class.

* For example, when the **inject(activity: SomeActivity)** method is called, the component will extract the necessary objects from the modules and place them in the SomeActivity fields.

---

## Additional features

* **Lazy<T>** - a lazy object creation. Each Lazy computes its value on the first call to **get()** and remembers that same value for all subsequent calls to **get()**.

* **Provider<T>** - provides instances of T. Similar to Lazy, but each time **get()** is called, it creates a new object. Typically implemented by an injector. For any type T that can be injected, you can also inject **Provider<T>**. Compared to injecting T directly, injecting Provider<T> enables:
    * retrieving multiple instances.
    * lazy or optional retrieval of an instance.
    * breaking circular dependencies.
    * abstracting scope so you can look up an instance in a smaller scope from an instance in a containing scope.

---

* **@Named** - a string-based qualifier.

* If the **module** creates two objects of the same type, then when compiling such code, the dagger will generate an error, because it will not be able to determine which of the two methods it should use to create the object. The **Named** annotation solves this problem. In the annotation, we specify any text corresponding to the created object. Now for the component, these are two different objects, although of the same type. And when you need this object, you will need to tell the component which one you need. The same annotations are used for this.

* **@Qualifier** - identifies qualifier annotations. Anyone can define a new qualifier and use them instead of **@Named**. A qualifier annotation:
    * is annotated with **@Qualifier**, **@Retention(RUNTIME)**, and typically **@Documented**.
    * can have attributes.
    * may be part of the public API, much like the dependency type, but unlike implementation types which needn't be part of the public API.
    * may have restricted usage if annotated with **@Target**. While this specification covers applying qualifiers to fields and parameters only, some injector configurations might use qualifier annotations in other places (on methods or classes for example).

---

* **@Intoset** - the method's return type forms the generic type argument of a **Set<T>**, and the returned value is contributed to the set. The object graph will pass dependencies to the method as parameters. The **Set<T>** produced from the accumulation of values will be immutable.

* If we need to get several objects of the same type from the component, we can request them immediately as a Set.

* **@ElementsIntoSet** - the method's return type is **Set<T>** and all values are contributed to the set. The **Set<T>** produced from the accumulation of values will be immutable. An example use is to provide a default empty set binding, which is otherwise not possible using IntoSet.

* There may be a case when a module returns not a single object, but a set of objects, i.e. **Set**. And we need all the objects from this **Set** to get into our **Set**. In this case, the **ElementsIntoSet** annotation must be used.

---

* **@IntoMap** The method's return type forms the type argument for the value of a **Map<K, Provider<V>>**, and the combination of the annotated key and the returned value is contributed to the map as a key/value pair. The **Map<K, Provider<V>>** produced from the accumulation of values will be immutable.

* Similar to **IntoSet**. The component will be able to collect objects for us in **Map**. The difference is that we will need to specify for each object the key (using **@StringKey**) with which this object will be placed in the Map.

* **@StringKey** -  a MapKey annotation for maps with String keys.

* **@MapKey** - identifies annotation types that are used to associate keys with values returned by provider methods in order to compose a map.
Every provider method annotated with **@Provides** and **@IntoMap** must also have an annotation that identifies the key for that map entry. That annotation's type must be annotated with **@MapKey**.
Typically, the key annotation has a single member, whose value is used as the map key.

---

* **@Binds** - annotates abstract methods of a Module that delegate bindings. For example, to bind java.util.Random to java.security.SecureRandom a module could declare the following: **@Binds abstract Random bindRandom(SecureRandom secureRandom);**

* **@Binds** methods are a drop-in replacement for Provides methods that simply return an injected parameter. Prefer @Binds because the generated implementation is likely to be more efficient.

* A **@Binds** method:
    * Must be abstract.
    * May be scoped.
    * May be qualified.
    * Must have a single parameter whose type is assignable to the return type. The return type declares the bound type (just as it would for a @Provides method) and the parameter is the type to which it is bound.

---

## SubComponent and Scope

* **@Subcomponent** - a subcomponent that inherits the bindings from a parent Component or Subcomponent. That is, in addition to the objects in their modules, **SubComponent** sees all the objects from the modules of the parent component.

---

* **@Singleton** - identifies a type that the injector only instantiates once. Not inherited.

* **@Scope** - identifies scope annotations. A scope annotation applies to a class containing an injectable constructor and governs how the injector reuses instances of the type. By default, if no scope annotation is present, the injector creates an instance (by injecting the type's constructor), uses the instance for one injection, and then forgets it. If a scope annotation is present, the injector may retain the instance for possible reuse in a later injection. If multiple threads can access a scoped instance, its implementation should be thread safe. The implementation of the scope itself is left up to the injector.

* **@Reusable** - a scope that indicates that the object returned by a binding may be (but might not be) reused. **@Reusable** is useful when you want to limit the number of provisions of a type, but there is no specific lifetime over which there must be only one instance. It minimizes memory consumption, unlike **@Scope**, but there is no guarantee that it will not return the same object.

---

## Producers

* Similar to the **@Module**, **@Provides**, and **@Component** annotations, the dagger provides the **@ProducerModule**, **@Produces**, and **@ProductionComponent** annotations. Components and modules with such annotations will work asynchronously, i.e. create an object on another thread. To ensure asynchrony, the component does not return the object itself, but a **ListenableFuture**, on which you can hang a callback.

* **@ProducerModule** - annotates a class that contributes Produces bindings to the production component.

* **@Produces** - annotates methods of a producer module to create a production binding. If the method returns a **com.google.common.util.concurrent.ListenableFuture** or **com.google.common.util.concurrent.FluentFuture**, then the parameter type of the future is bound to the value that the future produces; otherwise, the return type is bound to the returned value. The production component will pass dependencies to the method as parameters.

* **@Production** - qualifies a type that will be provided to the framework for use internally. The only type that may be so qualified is **java.util.concurrent.Executor**. In this case, the resulting executor is used to schedule producer methods in a **ProductionComponent** or **ProductionSubcomponent**. This will make it clear to the dagger that it can use this Executor for its asynchronous purposes.

* **@ProductionComponent** - annotates an interface or abstract class for which a fully-formed, dependency-injected implementation is to be generated from a set of modules. The generated class will have the name of the type annotated with **@ProductionComponent** prepended with Dagger. For example, **@ProductionComponent** interface **MyComponent {...}** will produce an implementation named **DaggerMyComponent**.
Each Produces method that contributes to the component will be called at most once per component instance, no matter how many times that binding is used as a dependency. TODO(beder): Decide on how scope works for producers.

* **Producer<T>** - an interface that represents the production of a type T. You can also inject Producer<T> instead of T, which will delay the execution of any code that produces the T until get is called. Similar to **Lazy**. If the component returns an object wrapped in a **Producer**, then the creation of the object will only start when the **get** method is called.

---

## Builder

* **@Component.Builder** - A builder for a component. Dagger gives us the opportunity to describe the interface of the component builder ourselves. A builder is a type with setter methods for the modules, dependencies and bound instances required by the component and a single no-argument build method that creates a new component instance.

* Components may have a single nested static abstract class or interface annotated with @Component.Builder. If they do, then Dagger will generate a builder class that implements that type. Note that a component with a @Component.Builder may not also have a @Component.Factory.

* Builder types must follow some rules:
    * There must be exactly one abstract no-argument method that returns the component type or one of its supertypes, called the "build method".
    * There may be other other abstract methods, called "setter methods".
    * Setter methods must take a single argument and return void, the builder type or a supertype of the builder type.
    * There must be a setter method for each component dependency.
    * There must be a setter method for each non-abstract module that has non-static binding methods, unless Dagger can instantiate that module with a visible no-argument constructor.
    * There may be setter methods for modules that Dagger can instantiate or does not need to instantiate.
    * There may be setter methods annotated with **@BindsInstance**. These methods bind the instance passed to them within the component. 
    * There may be non-abstract methods, but they are ignored as far as validation and builder generation are concerned.

* **@BindsInstance** - marks a method on a component builder or a parameter on a component factory as binding an instance to some key within the component. When using our own builder, we can avoid using the module, and directly pass the object to the component using the builder and marks the method with **@BindsInstance**.

* For subcomponents, there is a similar annotation: **@Subcomponent.Builder**, which allows you to describe your builder. In addition, its builder allows you to slightly change the scheme for creating a subcomponent.