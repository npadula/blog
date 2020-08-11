+++
date = 2020-08-11
menu =  "main"
title = "Visiting C#: Different ways of implementing the Visitor Pattern"
tags = [
    "programming",
    "design",
    "patterns",
    "visitor",
    "csharp"
]
+++

The purpose of the Visitor pattern is to add new behavior to classes without modifying them (too much). By using Visitor, we can create new behavior that acts upon certain entities, without placing that behavior on the entities themselves. 

For example, consider serialization: what is better? ```entity.serialize()``` or ```serializer.serialize(entity)``` ? I'd say that the second option is preferable because we may want to implement a complex serialization process with several dependencies that don't really relate to the entity itself. Let's see how can we achieve this in C#.

## Extension Methods

Probably the most idiomatic way we have to add behavior to 3rd party classes in C# is using extension methods. Extension methods are static methods defined in a static class, which the compiler recognizes (because of the ```this``` in the method signature) to be used as if it were a method of the 3rd party class we're extending.

In code this looks like this:

```
void Main()
{
	var x = new MyClass();
	x.DoWork();
}

public class MyClass {}

public static class Extensions {
    public static void DoWork(this MyClass obj){
        Console.WriteLine("Hello from extension");
    }
}
```

Thanks to C#'s syntactic sugar, we can use the DoWork method as if it were defined in MyClass itself. This proves a really convenient way of extending classes without modifying a single line of code of them.

While convenient, this approach has a few drawbacks: 
- It relies on static classes and methods, which can be anti-ergonomic in several scenarios (dependencyy instantiation, testing).
- **it's not a real visitor implementation**

It's not?

That's right, it's not, we can see why if we extend the previous example a bit more:

```
void Main()
{
	IBlah x = new MyClass();
	x.DoWork();
}

public class MyBaseClass {}

public class MyClass : MyBaseClass, IBlah {}

public interface IBlah {}


public static class Extensions {
	public static void DoWork(this MyBaseClass baseObj){
		Console.WriteLine("DoWork for BASE");
	}
	public static void DoWork(this IBlah blah){
		Console.WriteLine("DoWork for IBlah");
	}
	
	public static void DoWork(this MyClass blah){
		Console.WriteLine("DoWork for MyClass");
	}
}
```

This outputs: ```DoWork for IBlah```. 

The problem with this is that we do not benefit of polymorphism. The code in the Main method would be similar to how we would typically invoke a Visitor implementation, we call it with a given interface or class, and expect both the concrete type specific behavior and algorithm-specific behavior to be applied. 

If we were to stick with this approach, either our main code or our extension methods would end up full of ```if(x is IBlah){...}``` which is more than undesirable.

To solve this, we can use the traditional Visitor Pattern, as defined by the GoF.


## Traditional Visitor Pattern

The classic Visitor pattern is based in two things: class hierarchy traversal and **double dispatch**. Double dispatch is the technique that will allow us to achieve the two axes variation that we want. 

To achieve double dispatch, the pattern relies on two basic structures, the **Visitor** which is the class that contains the logic we want to add and the **Visitable** which represents our entity. Visitable is sometimes called component, or element. In UML, this is represented with the following diagrams:

![Traditional GoF Visitor Pattern](https://upload.wikimedia.org/wikipedia/commons/0/00/W3sDesign_Visitor_Design_Pattern_UML.jpg)

In code, this translates to:

```
void Main()
{
	List<IVisitable> entities = new List<IVisitable>{
		new A(),
		new B()
	};
	
	IVisitor v1 = new Visitor1();
	IVisitor v2 = new Visitor2();
	
	entities.ForEach(e => e.Accept(v1));
	entities.ForEach(e => e.Accept(v2));
}


public class A : IVisitable 
{
	public void Accept(IVisitor visitor){
		visitor.VisitA(this);
	}
}

public class B : A, IVisitable {
	public new void Accept(IVisitor visitor){
		visitor.VisitB(this);
	}
}



public interface IVisitable {
	 void Accept(IVisitor visitor);
}

public interface IVisitor {
	void VisitA(A a);
	void VisitB(B b);
}

public class Visitor1 : IVisitor {
	public void VisitA(A a){
		Console.WriteLine("Visited A from Visitor1");
	}
	
		public void VisitB(B b){
		Console.WriteLine("Visited B from Visitor1");
	}
}

public class Visitor2 : IVisitor {
	public void VisitA(A a){
		Console.WriteLine("Visited A from Visitor2");
	}
	
		public void VisitB(B b){
		Console.WriteLine("Visited B from Visitor2");
	}
}
```

The above outputs:

```
Visited A from Visitor1
Visited B from Visitor1
Visited A from Visitor2
Visited B from Visitor2
```

The double dispatch is achieved thanks to both the ```Accept``` and the ```Visit``` methods. ```Accept``` Exists in ```IVisitable``` and takes care of variation in the algorithm axis, this would be the _first_ dispatch. On the other hand, ```Visit``` exists in ```IVisitor```, and has specific names for each concrete ```IVisitable```, so that way there is no ambiguity, and we can achieve variation in the structural (class hierarchy) axis. This is the _second_ dispatch and it is done inside the ```Accept``` method, where ```Visit``` is invoked.

We can tweak this code a little bit to leverage operator overloading. Since ```Visit``` is invoked inside a concrete implementation of ```Accept```, inside this scope, the compiler is able to recognize the concrete type of ```this``` without ambiguity. This helps us avoid having different names for each concrete Visit method, variation is done with the parameters type themselves.

Resulting code:

```
void Main()
{
	List<IVisitable> entities = new List<IVisitable>{
		new A(),
		new B()
	};
	
	IVisitor v1 = new Visitor1();
	IVisitor v2 = new Visitor2();
	
	entities.ForEach(e => e.Accept(v1));
	entities.ForEach(e => e.Accept(v2));
}


public class A : IVisitable 
{
	public void Accept(IVisitor visitor){
		visitor.Visit(this);
	}
}

public class B : A, IVisitable {
	public new void Accept(IVisitor visitor){
		visitor.Visit(this);
	}
}



public interface IVisitable {
	 void Accept(IVisitor visitor);
}

public interface IVisitor {
	void Visit(A a);
	void Visit(B b);
}

public class Visitor1 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor1");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor1");
	}
}

public class Visitor2 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor2");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor2");
	}
}
```

So, what are the advantages of using the traditional Visitor Pattern?
- Language-agnostic, we don't even need oeprator overloading for it to work
- Variation in the two dimensions we wanted
- Easy to extend by adding new visitors

But it also comes with a few disadvantages:
- We ended up having to modify our entities after all (not too much though).
- Adding a new entity subclass may require to modify all of our existing visitors

Now, this is a pretty good solution, but can we do better by using C#-specific features?

## Dynamic Visitors

C# supports late-binding with the use of the ```dynamic``` keyword. [With this, we can make our visitor implementation less verbose, and even better, we can go back to our original premise of **adding behavior without touching the visitable classes**](https://blogs.u2u.be/kris/post/Farewell-Visitor).

```
void Main()
{
	List<dynamic> entities = new List<dynamic>{
		new A(),
		new B()
	};

	
	IVisitor v1 = new Visitor1();
	IVisitor v2 = new Visitor2();
	
	entities.ForEach(e => v1.Visit(e));
	entities.ForEach(e => v2.Visit(e));
}


public class A  {}

public class B : A {}


public interface IVisitor {
	void Visit(A a);
	void Visit(B b);
}

public class Visitor1 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor1");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor1");
	}
}

public class Visitor2 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor2");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor2");
	}
}
```

Output remains the same:
```
Visited A from Visitor1
Visited B from Visitor1
Visited A from Visitor2
Visited B from Visitor2
```

As we can see, our entities now don't need to implement any particular interface. Double dispatch and disambiguation are achieved not by an ```Accept``` in a concrete scope, but by leveraging C#'s **dynamic dispatch**. When ```Visit``` is invoked in the above example, the parameter passed is dynamic, [so the method overload resolution is done **using the runtime type and not the static type**](https://docs.microsoft.com/en-us/archive/blogs/samng/dynamic-in-c-iii-a-slight-twist), this way, the correct concrete type is selected, and we can achieve the same variation without altering classess or implementing interfaces.

### Dynamic Extension Method Visitor

Now we have a pretty decent solution. We can come closer to the original extension-method based solution if we make the dynamic dispatch happen in another place:

```
void Main()
{
	List<IBlah> entities = new List<IBlah>{
		new A(),
		new B()
	};
	
	entities.ForEach(e => e.DoWork());
}


public interface IBlah {}
public class A : IBlah  {}

public class B : A {}


public static class Extensions {
	public static void DoWork(this IBlah obj){
		dynamic x = obj;
		Visit(x);
	}
	
	
	private static void Visit(A a){
		Console.WriteLine("Visiting A from extension");
	}
	
	private static void Visit(B b){
		Console.WriteLine("Visiting B from extension");
	}

}
```

Output:
```
Visiting A from extension
Visiting B from extension
```

We have achieved extension method with dynamic dispatch! This is also pretty good as the syntactic sugar now makes our ```DoWork``` API easier to use. We can invoke it just like if it were defined on IBlah.

### Dynamic Runtime Visitor

Finally, if we don't want to go the static class route, we can go towards a more traditional Visitor implementation (with the ```Accept``` API in our entities) but without requiring them to implement any interface:

```
void Main()
{
	List<IBlah> entities = new List<IBlah>{
		new A(),
		new B()
	};

	
	IVisitor v1 = new Visitor1();
	IVisitor v2 = new Visitor2();
	
	entities.ForEach(e => e.Accept(v1));
	entities.ForEach(e => e.Accept(v2));
}

public interface IBlah {}

public class A : IBlah  {}

public class B : A {}


public interface IVisitor {
	void Visit(A a);
	void Visit(B b);
}

public class Visitor1 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor1");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor1");
	}
}

public class Visitor2 : IVisitor {
	public void Visit(A a){
		Console.WriteLine("Visited A from Visitor2");
	}
	
		public void Visit(B b){
		Console.WriteLine("Visited B from Visitor2");
	}
}


public static class VisitableExtensions {
	public static void Accept(this IBlah blah, IVisitor visitor){
		dynamic x = blah;
		visitor.Visit(x);
	}
}
```

With only one extension method we enable our entities to have an ```Accept``` method, which takes care of the double dispatch, and by accepting an ```IVisitor``` we have a more ergonomic and extensible Visitor implementation, that can be more easily handled in runtime.