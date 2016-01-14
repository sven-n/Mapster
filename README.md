![Mapster Icon](http://www.fancyicons.com/free-icons/103/pretty-office-5/png/128/order_128.png)

##Mapster - The Mapper of Your Domain

[![Build status](https://ci.appveyor.com/api/projects/status/krpp0nhspmklom1d?svg=true)](https://ci.appveyor.com/project/eswann/mapster)

### Basic usage
```
var result = TypeAdapter.Adapt<NewType>(original);
```
### Get it
```
PM> Install-Package GeekMapper
```
###Mapster 2.0 Release!
Mapster 2.0 is now become blistering fast! We upgraded the whole compilation unit while still maintain its functionalities. Here is benchmark.

| Engine          | Structs | Simple objects | Parent-Child | Parent-Children | Complex objects | Advance mapping |
|-----------------|--------:|---------------:|-------------:|----------------:|----------------:|----------------:|
| AutoMapper      |   11076 |          26121 |        21019 |           15841 |           18619 |           21987 |
| ExpressMapper   |     758 |           1701 |         1208 |            1323 |            3296 |            5252 |
| OoMapper        |       - |           1595 |         1182 |            1188 |            3221 |               - |
| ValueInjector   |    9071 |          21030 |        15698 |           12263 |           17334 |           31261 |
| TinyMapper      |       - |           1238 |            - |               - |               - |               - |
| Mapster         |       - |           2382 |         1892 |            1626 |            4287 |            6756 |
| **Mapster 2.0** | **610** |       **1323** |      **874** |         **787** |        **2292** |        **3654** |
| Native          |     593 |            796 |          496 |             659 |            2287 |            4266 |

(NOTE: Benchmark runner is forked from [ExpressMapper](https://github.com/Expressmapper/ExpressMapper). Benchmark was run against largest set of data, times are in milliseconds, lower is better. Blank values mean library did not supported.)

###Examples
####Mapping to a new object
Mapster makes the object and maps values to it.

    var destObject = TypeAdapter.Adapt<TSource, TDestination>(sourceObject);
    
or just
    
    var destObject = TypeAdapter.Adapt<TDestination>(sourceObject);

####Mapping to an existing object
You make the object, Mapster maps to the object.

    TDestination destObject = new TDestination();
    destObject = TypeAdapter.Adapt(sourceObject, destObject);

####Customized Mapping
When the default convention mappings aren't enough to do the job, you can specify complex source mappings.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .Ignore(dest => dest.Property)
        .Map(dest => dest.FullName, 
             src => string.Format("{0} {1}", src.FirstName, src.LastName));

#####Ignore Members & Attributes
By default, Mapster will automatically map properties with the same names. You can ignore members by using `Ignore` method.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .Ignore(dest => dest.Id);

Or you can ignore globally by using global setting.

    TypeAdapterConfig.GlobalSettings.IgnoreMembers.Add('Id');

You can ignore members annotated with specific attribute by using `IgnoreAttribute` method.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .IgnoreAttribute(typeof(JsonIgnoreAttribute));

Or

    TypeAdapterConfig.GlobalSettings.IgnoreAttributes.Add(typeof(JsonIgnoreAttribute));

#####Property mapping
By default, Mapster map property by name, but you can customize property mapping.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .Map(dest => dest.FullName, 
             src => string.Format("{0} {1}", src.FirstName, src.LastName));

The Map configuration can accept a third parameter that provides a condition based on the source.
If the condition is not met, the mapping is skipped altogether.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .Map(dest => dest.FullName, src => src.FullName, srcCond => srcCond.City == "Victoria");

You can also map even type of source and destination properties are different.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .Map(dest => dest.Gender,      //Genders.Male or Genders.Female
             src => src.GenderString); //"Male" or "Female"

#####Merge object
By default, Mapster will copy null values to destination properties (or convert destination properties to default values). You can merge objects by ignore null values in source properties by using `IgnoreNullValues` method. When you use this setting, null value will not copy to destination property.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .IgnoreNullValues(true);

Or you can ignore null values globally by using global setting.

    TypeAdapterConfig.GlobalSettings.IgnoreNullValues = true;

NOTE: after you set `IgnoreNullValues` in global settings to `true`, you can opt-out by setting `IgnoreNullValues` in `TypeAdapterConfig<TSource, TDestination>` to `false`.

#####Shallow copy
By default, Mapster will recursively copy nested objects. You can do shallow copying by setting `ShallowCopyForSameType` to `true`. When you use this setting, when types of source properties and destination properties are the same, it will not create new objects, but it just copy reference from source properties to destination properties.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .ShallowCopyForSameType(true);

Or you can set globally by using global setting.

    TypeAdapterConfig.GlobalSettings.ShallowCopyForSameType = true;

NOTE: after you set `ShallowCopyForSameType` in global settings to `true`, you can opt-out by setting `ShallowCopyForSameType` in `TypeAdapterConfig<TSource, TDestination>` to `false`.

#####Preserve reference (preventing circular reference stackoverflow)
By default, Mapster will not track reference (for performance reason). Therefore, when you copying circular reference objects, there will be stackoverflow exception. If you would like to copy circular reference objects, or you would like to preserve references (such as 2 properties point to the same object), you can preserve reference by setting `PreserveReference` to `true`

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .PreserveReference(true);

Or you can set globally by using global setting.

    TypeAdapterConfig.GlobalSettings.PreserveReference = true;

NOTE: after you set `PreserveReference` in global settings to `true`, you can opt-out by setting `PreserveReference` in `TypeAdapterConfig<TSource, TDestination>` to `false`.

####Mapping Lists Included
This includes lists, arrays, collections, enumerables etc...

    var destObjectList = TypeAdapter.Adapt<List<TSource>, List<TDestination>>(sourceList);

####Mapping Enums Included
Mapster maps enums to numerics automatically, but it also maps strings to and from enums automatically in a fast manner.  
The default Enum.ToString() in .Net is quite slow.  The implementation in Mapster is double the speed.  
Likewise, a fast conversion from strings to enums is also included.  If the string is null or empty, 
the enum will initialize to the first enum value.

In addition, fast Enum mapper extension methods are included for convenience.

    //Convert enum to string
    var myEnum = new SomeEnum.FirstValue;
    myEnum.ToFastString();

    //Convert string to enum
    var myEnumString = "FirstValue";
    myEnumString.ToFastEnum<SomeEnum>();
    


####Implicit TSource Mapping Inheritance
If a mapping configuration doesn't exist for a source ==> destination type, but a mapper does exist for a base source type 
to the destination, that mapping will be used.  This allows mappings for less derived source types to be used to 
satisfy multiple derived mappings.  FPR will search downward the source class hierarchy until it finds a matching configuration.   
If no match exists, it will create a default configuration (the same behavior if no mapping was present).
It **doesn't** combine derived configs, it will stop at the first match.  
For example, if you have:

    public class SimplePoco
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

    public class DerivedPoco : SimplePoco
    {...}

    public class DerivedPoco2 : SimplePoco
    {...}

    public class SimpleDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

The following mapping will be used when mapping the SimplePoco or either of the derived POCOs to the Simple DTO.

    TypeAdapterConfig<SimplePoco, SimpleDto>.NewConfig()
        .Map(dest => dest.Name, src => src.Name + "_Suffix");

If you don't wish a derived type to use the base mapping, just define a new configuration for that type.

    TypeAdapterConfig<DerivedPoco2, SimpleDto>.NewConfig();


####Implicit TSource and TDestination Mapping Inheritance
In some cases you may wish to derive a mapping based on both the source and destination types.  In such a case,
setting the global configuration AllowImplicitDestinationInheritance setting to true will allow inheritance of a mapping
based on both the source and destination types.  The source class hierarchy is traversed in an inside loop and the 
destination class hierarchy is traversed in an outside loop until a match is found. 

    public class SimplePoco
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

    public class DerivedPoco : SimplePoco
    {...}

    public class SimpleDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

    public class DerivedDto : DerivedDto
    {...}

The following mapping will be used for any permutation of Simple/Derived Poco to Simple/Derived Dto
unless overridden by a specific configuration. 

    TypeAdapterConfig.GlobalSettings.AllowImplicitDestinationInheritance = true;
    TypeAdapterConfig<SimplePoco, SimpleDto>.NewConfig()
        .Map(dest => dest.Name, src => src.Name + "_Suffix");

If you don't wish a specific permutation to use the base mapping, just define a new configuration for that permutation.

    TypeAdapterConfig<DerivedPoco, SimpleDto>.NewConfig();


#### Explicit Mapping Inheritance
In some cases, you may wish to have one mapping inherit from another mapping.  In this case, anything set on the derived mapping
will override settings on the base mapping.  So unlike implicit mapping inheritance, where the goal is to find an applicable mapping
configuration and configurations are not combined, with Explicit Inheritance, each derived mapping overrides the settings of the 
inherited mapping.  Use the Inherits<TBaseSource, TBaseDestination> configuration method to accomplish this where TBaseSource and 
TBaseDestination must be assignable from the derived configuration's TSource and TDestination respectively. 
For example, if you have:

    public class SimplePoco
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

    public class DerivedPoco : SimplePoco
    {...}

    public class SimpleDto
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
    }

    public class DerivedDto : SimpleDto
    {...}

And you have the following base mapping:

    TypeAdapterConfig<SimplePoco, SimpleDto>.NewConfig()
        .Map(dest => dest.Name, src => src.Name + "_Suffix");

You can base the mapping of the derived classes on this base mapping:
    
    TypeAdapterConfig<DerivedPoco, DerivedDto>.NewConfig()
        .Inherits<SimplePoco, SimpleDto>();


####Custom Destination Object Creation
You can provide a function call to create your destination objects instead of using the default object creation 
(which expects an empty constructor).  To do so, use the "ConstructUsing()" method when configuring.  This method expects
a function that will provide the destination instance. You can call your own constructor, a factory method, 
or anything else that provides an object of the expected type.

    //Example using a non-default constructor
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .ConstructUsing(src => new TDestination("constructorValue"));

    //Example using an object initializer
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .ConstructUsing(src => new TDestination{Unmapped = "unmapped"});

####Custom Type Resolvers
In some cases, you may want to have complete control over how an object is mapped.  In this case, you can
register a custom type resolver.  It's important to note that when using a custom type resolver, that 
all other mapping associated with the type is ignored.  So all mapping must take place within the resolver.
The custom type resolver must implement the ITypeResolver interface and register it using MapWith().

    //Example using MapWith resolver generic call.
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .MapWith<TCustomTypeResolver>();

    //Example using MapWith resolver factory function
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .MapWith(() => new TCustomTypeResolver());

    //Example using MapWith resolver instance
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .MapWith(customResolverInstance);

####Custom Value Resolvers
In some cases, you may want to encapsulate a value conversion into a separate class.  In this case, you can 
use a custom value resolver by registering it using Resolve().  The value resolver must implement the 
IValueResolver interface.

    //Example using MapWith resolver generic call.
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .Resolve<TCustomValueResolver, string>(dest => dest.Name);

    //Example using value resolver factory function
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .Resolve(dest => dest.Name, () => new TCustomValueResolver());

    //Example using value resolver instance
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .Resolve(dest => dest.Name, customValueResolver);

####Type-Specific Destination Transforms
This allows transforms for all items of a type, such as trimming all strings.  But really any operation 
can be performed on the destination value before assignment.  This can be set up at either a global
or a mapping level.

    //Global
    TypeAdapterConfig.GlobalSettings.DestinationTransforms.Upsert<string>(x => x.Trim());

    //Per mapping configuration
    TypeAdapterConfig<TSource, TDestination>.NewConfig()
        .DestinationTransforms.Upsert<string>(x => x.Trim());
    

####Conditional Mapping
The Map configuration can accept a third parameter that provides a condition based on the source.
If the condition is not met, the mapping is skipped altogether.

    TypeAdapterConfig<TSource, TDestination>()
        .NewConfig()
        .IgnoreMember(dest => dest.Property)
        .Map(dest => dest.FullName, src => src.FullName, srcCond => srcCond.City == "Victoria");

####Max Depth
When mapping nested or tree-type structures, it's often necessary to specify a max nesting depth to prevent overflows.

    TypeAdapterConfig<TSource, TDestination>
                .NewConfig()
                .MaxDepth(3);

####Queryable Extensions

    using(MyDbContext context = new MyDbContext())
    {
        // Build a Select Expression from DTO
        var destinations = context.Sources.Project().To<Destination>().ToList();

        // Versus creating by hand:
        var destinations = context.Sources.Select(c => new Destination(){
            Id = p.Id,
            Name = p.Name,
            Surname = p.Surname,
            ....
        })
        .ToList();
    }

####Forcing Explicit Mapping
In order to help with "Fail Fast" situations, the following strict mapping modes have been added.
An ArgumentOutOfRange exception will currently be thrown in the situations below if an appropriate mapping and/or source cannot be located.

Forcing all destination properties to have a corresponding source member or explicit mapping/ignore:

    //Default is "false"
    TypeAdapterConfig.GlobalSettings.RequireDestinationMemberSource = true;

Forcing all classes to be explicitly mapped:

    //Default is "false"
    TypeAdapterConfig.GlobalSettings.RequireExplicitMapping = true;
    //This means you have to have an explicit configuration for each class, even if it's just:
    TypeAdapterConfig<Source, Destination>.NewConfig();

####Validating Mappings
Both a specific TypeAdapterConfig<Source, Destination> or all current configurations can be validated.  This will throw
and ArgumentOutOfRangeException that contains all of the existing missing destination mappings.  In addition, if Explicit Mappings (above)
are enabled, it will also include errors for classes that are not registered at all with the mapper.

    //Validate a specific config
    var config = TypeAdapterConfig<Source, Destination>.NewConfig();
    config.Validate();

    //Validate globally
    TypeAdapterConfig<Source, Destination>.NewConfig();
    TypeAdapterConfig<Source2, Destination2>.NewConfig();
    TypeAdapterConfig.Validate();

####Mapper Instance Creation
In some cases, you need an instance of a mapper (or a factory function) to pass into a DI container.  Mapster has
the IAdapter and Adapter to fill this need:

    IAdapter instance = TypeAdapter.GetInstance();


###Assembly Scanning for Custom Mappings
To make it easier to register custom mappings, we've implemented an assembly scanning approach.
To allow this, either inherit from IRegistry or Registry in the Mapster.Registration namespace.

Override the Apply() method and perform your registrations there.  When your app starts up, use the Registrar class to perform registration.

    //Implement a registry class
    public class MyRegistry : Registry
    {
        public override void Apply()
        {
            TypeAdapterConfig<TSource, TDestination>.NewConfig()
                .IgnoreMember(dest => dest.CurrencyCode)
                .IgnoreMember(dest => dest.ExtraElements);
        }
    }

    //In my boostrap/startup code, call the registry
    //Method 1: Call registry directly with extension method
    new MyRegistry().Register();

    //Method 2: Scan the assembly (by assembly or class type)
    Registrar.RegisterFromAssemblyContaining<MyRegistry>();
    //or
    Assembly.GetExecutingAssembly().RegisterFromAssembly();


###Performance Comparisons
When aggregating times across complex and simple objects, we're seeing a ~30x speed improvement in comparison to AutoMapper, 
but it's obviously very dependent upon your situation.

####Benchmark "Complex" Object

The following test converts a Customer object with 2 nested address collections and two nested address sub-objects to a DTO.

    Customer c = new Customer()
    {
        Address = new Address() { City = "istanbul", Country = "turkey", Id = 1, Street = "istiklal cad." },
        HomeAddress = new Address() { City = "istanbul", Country = "turkey", Id = 2, Street = "istiklal cad." },
        Id = 1,
        Name = "Eduardo Najera",
        Credit = 234.7m,
        WorkAddresses = new List<Address>() { 
            new Address() { City = "istanbul", Country = "turkey", Id = 5, Street = "istiklal cad." },
            new Address() { City = "izmir", Country = "turkey", Id = 6, Street = "konak" }
        },
        Addresses = new List<Address>() { 
            new Address() { City = "istanbul", Country = "turkey", Id = 3, Street = "istiklal cad." },
            new Address() { City = "izmir", Country = "turkey", Id = 4, Street = "konak" }
        }.ToArray()
    };


Competitors : Handwriting Mapper, Mapster, FastMapper, AutoMapper

    Iterations : 100
    Handwritten Mapper:     1
    Mapster:                0
    AutoMapper:             10

    Iterations : 10000
    Handwritten Mapper:     4
    Mapster:                17
    AutoMapper:             507

    Iterations : 100000
    Handwritten Mapper:     29
    Mapster:                177
    AutoMapper:             5058

