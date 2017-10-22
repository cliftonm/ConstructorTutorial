
Constructors in C# - A Tutorial
==

Introduction
-
This short tutorial covers different C# constructor syntax, their usage, and some of the pros and cons.

Classes
-
Classes fall broadly into three categories:

- Classes that do not manage any internal state.  These classes are typically static class, implementing static methods, and do not require being instantiated.  Even if they are instantiated, the methods that state-less classes implement expect all parameters to be passed into them and any resulting parameters to be returned by the method or with `out` or `ref` parameters.  Stateless objects are one of the cornerstones of "declarative", or "functional", programming.
- Classes that maintain or manage state.  These classes may have constructors that let you initialize the class state.  It is this kind of class that this tutorial will cover.  The state of an instance of a class may be exposed as properties (less commonly fields) for read or read/write operations (very rarely write-only operations.)  Sometimes a class ends up being little more than a container for a collection of values for the purpose of organizing the values into a useful structure.  Methods that a stateful class implements often use the fields/properties of the class instance in whatever "computation" the method performs.  And, contrary to stateless classes, these methods may update the state of the instance rather than explicitly returning the new state.  Stateful classes are one of the cornerstones of "imperative programming."
- Lastly, while most classes expect that more than one instance of a class will be created, a special kind of class, the "singleton", prevents the developer from creating more than one instance.  This kind of class is often useful for managing a physical resource or "one and only one" instance of something, like a routing table for a web server.

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

Constructors with Parameters
-
The Vehicle class above becomes more useful when we can initialize the fields using a constructor that allows us to pass in the values:

    public class Vehicle
    {
        protected string make;
        protected string model;
        protected int year;
    
        public Vehicle(string make, string model, int year)
        {
            this.make = make;
            this.model = model;
            this.year = year;
        }
    }
However, as soon as we create a constructor with parameters, the default constructor *is no longer created for us.*  Therefore, this instantiation:

    var vehicle = new Vehicle();

no longer works.  The compiler generates an error:

    There is no argument given that corresponds to the required formal parameter 'make' of 'Vehicle.Vehicle(string, string, int)'

We can of course fix that by passing in the appropriate values:

    var vehicle = new Vehicle("Suzuki", "SX4", 1999);

When is a Default Constructor Helpful?
-
One the foremost reasons for implementing a default constructor (one that does not take any parameters) is for serialization -- being able to save the instance to a database or other format such as XML or JSON.  Given the above class and this code:

        static void Main(string[] args)
        {
            var vehicle = new Vehicle("Suzuki", "SX4", 1999);
            var xml = Serialize(vehicle);
            Console.WriteLine(xml);
        }

        static string Serialize(object obj)
        {
            XmlWriterSettings xws = new XmlWriterSettings();
            xws.OmitXmlDeclaration = true;
            XmlSerializerNamespaces ns = new XmlSerializerNamespaces();
            ns.Add("", "");
            XmlSerializer xs = new XmlSerializer(obj.GetType());
            StringBuilder sb = new StringBuilder();
            TextWriter tw = new StringWriter(sb);
            XmlWriter xtw = XmlWriter.Create(tw, xws);
            xs.Serialize(xtw, obj, ns);
            string xml = sb.ToString();

            return xml;
        }

While the code compiles without errors, when we try to run the code, we get a runtime error:

    Unhandled Exception: System.InvalidOperationException: ConstructorTutorialCode.Vehicle cannot be serialized because it does not have a parameterless constructor

We can fix this by explicitly declaring the default constructor:
    public class Vehicle
    {
        protected string make;
        protected string model;
        protected int year;

        public Vehicle() { }

        public Vehicle(string make, string model, int year)
        {
            this.make = make;
            this.model = model;
            this.year = year;
        }
    }

Now, running the program results in the following XML:

	<Vehicle />

But notice that none of the values are emitted.
> **Note:**
> The default constructor can be marked protected or private and the serialization still works.  Designating the default constructor as protected or private prevents the class from being instantiated externally without supplying required values.  Designating a default constructor as private prevents any derived class from referencing the default constructor.

Properties
-
Properties are entwined with classes and their initialization and serialization.  Refactoring the `Vehicle` class to use properties instead of fields, and using the shorthand syntax (notice the `XmlAttribute` decoration, which helps make the XML more readable):

    public class Vehicle
    {
        [XmlAttribute]
        public string Make { get; set; }
        [XmlAttribute]
        public string Model { get; set; }
        [XmlAttribute]
        public int Year { get; set; }

        private Vehicle() { }

        public Vehicle(string make, string model, int year)
        {
            Make = make;
            Model = model;
            Year = year;
        }
    }

Results in the values being serialized:

    <Vehicle Make="Suzuki" Model="SX4" Year="1999" />

But can the XML be deserialized?

        static void Main(string[] args)
        {
            var vehicle = new Vehicle("Suzuki", "SX4", 1999);
            var xml = Serialize(vehicle);
            Console.WriteLine(xml);

            XmlSerializer xs = new XmlSerializer(typeof(Vehicle));
            StringReader sr = new StringReader(xml);
            var vehicle2 = (Vehicle)xs.Deserialize(sr);
        }

Why yes!  This works just fine with a protected or private default constructor!  As with serialization, if we don't provide the default constructor, the deserialization call fails at runtime.

Properties Creates a Can of Worms for Class Construction
-

Once we've created writeable properties, we can initialize the class (assuming it implements a default constructor) with a property initialization syntax.  First, let's simply the `Vehicle` class to a bare-bones implementation -- one with only properties and therefore having an implicit default constructor (we'll remove the `XmlAttribute` decorations for now):

    public class Vehicle
    {
        public string Make { get; set; }
        public string Model { get; set; }
        public int Year { get; set; }
    }

We can no longer instantiate the class like this:

    var vehicle = new Vehicle("Suzuki", "SX4", 1999);

Instead, we can instantiate it like this:

    var vehicle = new Vehicle() { Make = "Suzuki", Model = "SX4", Year = 1999 };

> **Note:** The parameters can be initialized in any order.

### The Pros of This Syntax
There are a few advantages to this syntax:
1. The property being initialized is explicitly stated, make the semantics of the construction clearer.
2. Not all the properties need to be initialized.  Some class implementations may not require all of its properties to be initialized, and rather than implementing multiple constructors:

        public Vehicle() { }
        public Vehicle(string make) { Make = make; }
        public Vehicle(string make, string model) { Make = make; Model = model; }

(you get the idea)

or default values (which can lead to a lot of questioning about whether the default value is the right default):

    public Vehicle(string make, string model="", int year=2001) { Make = make; Model = model; Year = year;}

A default constructor with property initialization can be simpler, particularly if it's a once-only construction used, most likely, internally by the application code.

### The Cons of This Syntax
1. It can be more verbose, particularly if there is a large set of properties that need to be initialized (which in itself is an indicator of bad code design.)
2. Because the initialization of properties is now optional, the methods in the class usually must be explicitly responsible for validating that expected properties hold expected values, and problems are typically discovered at run-time (usually not good) rather than compile-time.
3. The initialization of properties in this form makes them writable, allowing other parts of the program to alter their values.  This can lead to unintended side-effects.

Classes that are exposed as API's (such as in a library) for other applications to use should probably never permit this form of construction.

Named Arguments
-

A middle ground called "named arguments" lets us use explicit naming but allows us to create read-only properties.  Given (note the fully specified constructor is back):

    public class Vehicle
    {
        public string Make { get; protected set; }
        public string Model { get; protected set; }
        public int Year { get; protected set; }

        public Vehicle() { }

        public Vehicle(string make, string model, int year)
        {
            Make = make;
            Model = model;
            Year = year;
        }
    }

We can construct vehicles without expectations of the parameter order (as long we provide all the parameters that the constructor requires) like this:

    var vehicle1 = new Vehicle(make : "Suzuki", model: "SX4", year: 1999);
    var vehicle2 = new Vehicle(model: "SX4", year: 1999, make: "Suzuki");

Named arguments are optional if the constructor provides a default value.  The constructor call allows you to mix both unnamed and named required and named optional arguments.  Given:

        public Vehicle(string make, string model = "", int year = 2001)
        {
            Make = make;
            Model = model;
            Year = year;
        }

The class can be instantiated like this:

    var vehicle = new Vehicle("Suzuki", year: 1999);

This approach has a distinct advantage when dealing with optional parameters, in that only the options that you want to specify need to be specified.  Without named parameters, the class would either have to provide all variations of construction, or the optional parameters would have to be provided with their defaults (which you might now know) until you get to the parameter you want to explicitly set.  For example:

    var vehicle = new Vehicle("Suzuki", "", 1999);

In the above code, we have to provide the default for `make` so that we can get to the argument in the constructor for specifying `year`.

Cloning
-

A common use case for a constructor is to clone the object, for example:

        public Vehicle(Vehicle toClone)
        {
            Make = toClone.Make;
            Model = toClone.Model;
            Year = toClone.Year;
        }

Singletons
-

When only a single instance of a class should ever be instantiated, a common practice for such "singleton" classes is to create a `protected` or `private` constructor along with a `public` `static` "factory" method and a `static` field for the single instance of the class:

    public class MoonVehicle
    {
        protected static MoonVehicle instance;

        public static MoonVehicle Instance
        {
            get
            {
                if (instance == null) new MoonVehicle();

                return instance;
            }
        }

        protected MoonVehicle()
        {
            instance = this;
        }
    }

Here there is the possibility of one and only one moon vehicle, and we have to use a singleton factory to obtain that single instance:

    var moonVehicle = MoonVehicle.Instance;

Attempting to instantiate `MoonVehicle`:

    var moonVehicle = new MoonVehicle();

gives us a compile-time error:

    'MoonVehicle.MoonVehicle()' is inaccessible due to its protection level


Routing Multiple Constructors to a Single Constructor
-

Often a class performs some additional initialization during construction.  When the class implements multiple constructors, it can be easy to forget to call the initialization method.  For example: (note that this is a contrived example):

    public class Vehicle
    {
        public string Make { get; protected set; }
        public string Model { get; protected set; }
        public int Year { get; protected set; }

        protected int numberOfPassengers;

        public Vehicle() { }

        public Vehicle(Vehicle toClone)
        {
            Make = toClone.Make;
            Model = toClone.Model;
            Year = toClone.Year;
            numberOfPassengers = GetNumberOfPassengers();
        }

        public Vehicle(string make, string model, int year)
        {
            Make = make;
            Model = model;
            Year = year;
            numberOfPassengers = GetNumberOfPassengers();
        }

        protected int GetNumberOfPassengers()
        {
            return 4;
        }

Note that both constructors have to call `GetNumberOfPassengers`.  This can be avoided using the `this` syntax in the constructor that clones the class:

        public Vehicle(Vehicle toClone) :
            this(toClone.Make, toClone.Model, toClone.Year)
        {
        }

        public Vehicle(string make, string model, int year)
        {
            Make = make;
            Model = model;
            Year = year;
            numberOfPassengers = GetNumberOfPassengers();
        }

This form is particularly useful when there is a single constructor that "does all the initialization" and all other constructors (, either exposed as public or as internal constructors) are derivatives of this main constructor.  The opposite is also sometimes useful -- the "does all the initialization" constructor might be `protected` or `private`, and the exposed constructors call into it using the `this` syntax.

Named arguments can be used with this form of construction as well:

        public Vehicle(string make) :
            this(make, model: "", year: 2001)
        {
        }

as well as with default values in the final constructor:

        public Vehicle(string make) :
            this(make, year: 2001)
        {
        }

        public Vehicle(string make, string model = "", int year = 1991)
        {
            Make = make;
            Model = model;
            Year = year;
            numberOfPassengers = GetNumberOfPassengers();
        }

Finally, the syntax for named arguments is more "Javascript-like" for initialization of structures, so you'll see this form a lot in ASP.NET/Razor cshtml files.  For example:

    WebGrid grid = new WebGrid(Model, defaultSort: "FirstName", rowsPerPage : 5); 

Base Classes
-
C# is a single-inheritance language, meaning that a subclass cannot be derived from more than one super-class.  The explicit parameter argument and named argument call to a base class can be used.  A couple examples, assuming some default values in the constructor for `Vehicle`:

    public class Truck : Vehicle
    {
        public Truck() : base("Ford", "F-150", 2017)
        {
        }

        public Truck(int year) : base("Ford", year: year)
        {
        }
    }

It is useful to note that the this approach for a single constructor initialization can still be used:

    public class Truck : Vehicle
    {
        public Truck() : this(2017)
        {
        }

        public Truck(int year) : base("Ford", year: year)
        {
        }
    }

Conclusion
-
This tutorial was written using the awesome [StackEdit](https://stackedit.io/editor) editor.
