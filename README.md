Introduction
============
This short tutorial covers the pros and cons of different constructor usage in C#.

Classes
-
Classes fall broadly into categories:

- Classes that do not manage any internal state.  These classes are typically static class, implementing static methods, and do not require being instantiated.  Even if they are instantiated, the methods that state-less classes implement expect all parameters to be passed into them and any resulting parameters to be returned by the method or with `out` or `ref` parameters.  Stateless objects are one of the cornerstones of "declarative", or "functional", programming.
- Classes that maintain or manage state.  These classes may have constructors that let you initialize the class state.  It is this kind of class that this tutorial will cover.  The state of an instance of a class may be exposed as properties (less commonly fields) for read or read/write operations (very rarely write-only operations.)  Sometimes a class ends up being little more than a container for a collection of values for the purpose of organizing the values into a useful structure.  Methods that a stateful class implements often use the fields/properties of the class instance in whatever "computation" the method performs.  And, contrary to stateless classes, these methods may update the state of the instance rather than explicitly returning the new state.  Stateful classes are one of the cornerstones of "imperative programming."

The Default Constructor
-

Given a container for a vehicle which includes its make, model, and year, exposed as properties:

    public class Vehicle
    {
      protected string make;
      protected string model;
      protected int year;
    }
C# implements a default constructor for this class, such that we can instantiate it with:

    var vehicle = new Vehicle();

This isn't very helpful because we have no way of initializing the fields `make`, `model`, and `year`.

