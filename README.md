## CodeGenerator
This is a small .NET Core library that enables easy C# code generation based on Classes and its elements.  
It has ability to create `ClassModels` and write it to .cs files.  
Can specify their Members (`Constructor`, `Field`, `Property`, `Method`) including `Attributes` and `Parameters`.  
Defining `namespace` and `using` Directives is supported as well.  
Library can also generate `Enums` and `Interfaces`, and create `NestedClasses` inside parent class.  

**BaseElement** has Config for: `IndentSize, Comment, AccessModifier, BuiltInDataType, CustomDataType, Name`  
**CsGenerator** Settings are: `DefaultTabSize`: *4* | `OutputDirectory`: "*Output*" |  
-- List of components --  
**AccessModifier**: `public, private, protected, pnternal, protected_internal`  
**BuiltInDataType**: `void, bool, byte, int, long, decimal, float, double, char, string, object`  
**CommonDataType**: `DateTime, Guid`  
**KeyWord**: `this, abstract, partial, static, new, virtual, override, sealed, const, async, readOnly`  
IndentType: `None, Single, Double, Triple, Quadruple`  

For more complex code with indented IF segments specific bigger Indent should be set (in a loop) for all internal elements.

[![NuGet](https://img.shields.io/npm/l/express.svg)](https://github.com/borisdj/CsCodeGenerator/blob/master/LICENSE)

Also take a look into others packages:</br>
-Open source (MIT or cFOSS) authored [.Net libraries](https://infopedia.io/dot-net-libraries/) (@**Infopedia.io** personal blog post)
| №  | .Net library             | Description                                              |
| -  | ------------------------ | -------------------------------------------------------- |
| 1  | [EFCore.BulkExtensions](https://github.com/borisdj/EFCore.BulkExtensions) | EF Core Bulk CRUD Ops (Flagship Lib) |
| 2  | [EFCore.UtilExtensions](https://github.com/borisdj/EFCore.UtilExtensions) | EF Core Custom Annotations and AuditInfo |
| 3  | [EFCore.FluentApiToAnnotation](https://github.com/borisdj/EFCore.FluentApiToAnnotation) | Converting FluentApi configuration to Annotations |
| 4  | [FixedWidthParserWriter](https://github.com/borisdj/FixedWidthParserWriter) | Reading & Writing fixed-width/flat data files |
| 5  | [CsCodeGenerator](https://github.com/borisdj/CsCodeGenerator) | C# code generation based on Classes and elements |
| 6  | [CsCodeExample](https://github.com/borisdj/CsCodeExample) | Examples of C# code in form of a simple tutorial |

## How to use it
1 Install [![NuGet](https://img.shields.io/nuget/v/CsCodeGenerator.svg)](https://www.nuget.org/packages/CsCodeGenerator/) package: *'Install-Package CsCodeGenerator'*  
2 Following is first example of ComplexNumber class and then creating its ClassModel for writing to Complex.cs<br>

Class we want to generate:
````csharp
using System;
using System.ComponentModel;

namespace CsCodeGenerator.Tests
{
    [Description("Some class info")]
    public partial class ComplexNumber : SomeBaseClass, NumbericInterface
    {
        protected const double PI = 3.14;
        private string remark;

        public ComplexNumber() { }

        public ComplexNumber(double real, double imaginary = 0)
        {
            Real = real;
            Imaginary = imaginary;
        }

        public static string DefaultFormat { get; } = "a + b * i";

        public double Real { get; set; }

        public double Imaginary { get; set; }

        public virtual string Remark
        {
            get { return remark; }
            set { remark = value; }
        }

        public double Modul()
        {
            return Math.Sqrt(Real * Real + Imaginary * Imaginary);
        }

        public ComplexNumber Add(ComplexNumber input)
        {
            ComplexNumber result = new ComplexNumber();
            result.Real = Real + input.Real;
            result.Imaginary = Imaginary + input.Imaginary;
            return result;
        }

        // example of 2 KeyWords(new and virtual), usually here would be just virtual
        public new virtual string ToString()
        {
            return $"({Real:0.00}, {Imaginary:0.00})";
        }
    }
}
````

Code to do it:
````csharp
var usingDirectives = new List<string>
{
    "using System;",
    "using System.ComponentModel;"
};
string fileNameSpace = $"{Util.Namespace} CsCodeGenerator.Tests";
string complexNumberText = "ComplexNumber";

ClassModel complexNumberClass = new ClassModel(complexNumberText);
complexNumberClass.SingleKeyWord = KeyWord.Partial; //or: complexNumberClass.KeyWords.Add(KeyWord.Partial);
complexNumberClass.BaseClass =  "SomeBaseClass";
complexNumberClass.Interfaces.Add("NumbericInterface)";

var descriptionAttribute = new AttributeModel("Description")
{
    SingleParameter = new Parameter(@"""Some class info""")
};
complexNumberClass.AddAttribute(descriptionAttribute);

complexNumberClass.DefaultConstructor.IsVisible = true;

Constructor secondConstructor = new Constructor(complexNumberClass.Name);
secondConstructor.Parameters.Add(new Parameter(BuiltInDataType.Double, "real"));
secondConstructor.Parameters.Add(new Parameter(BuiltInDataType.Double, "imaginary") { Value = "0" });
secondConstructor.BodyLines.Add("Real = real;");
secondConstructor.BodyLines.Add("Imaginary = imaginary;");
complexNumberClass.Constructors.Add(secondConstructor);

var fields = new Field[]
{
    new Field(BuiltInDataType.Double, "PI") { SingleKeyWord = KeyWord.Const, DefaultValue = "3.14" },
    new Field(BuiltInDataType.String, "remark") { AccessModifier = AccessModifier.Private },
}.ToDictionary(a => a.Name, a => a);

var properties = new Property[]
{
    new Property(BuiltInDataType.String, "DefaultFormat")
    {
        SingleKeyWord = KeyWord.Static,
        IsGetOnly = true,
        DefaultValue = @"""a + b * i"""
    },
    new Property(BuiltInDataType.Double, "Real"),
    new Property(BuiltInDataType.Double, "Imaginary"),
    new Property(BuiltInDataType.String, "Remark")
    {
        SingleKeyWord = KeyWord.Virtual,
        IsAutoImplemented = false,
        GetterBody = "remark",
        SetterBody = "remark = value"

    },
}.ToDictionary(a => a.Name, a => a);

var methods = new Method[]
{
    new Method(BuiltInDataType.Double, "Modul")
    {
        BodyLines = new List<string> { "return Math.Sqrt(Real * Real + Imaginary * Imaginary);" }
    },
    new Method(complexNumberText, "Add")
    {
        Parameters = new List<Parameter> { new Parameter("ComplexNumber", "input") },
        BodyLines = new List<string>
        {
            "ComplexNumber result = new ComplexNumber();",
            "result.Real = Real + input.Real;",
            "result.Imaginary = Imaginary + input.Imaginary;",
            "return result;"
        }
    },
    new Method(BuiltInDataType.String, "ToString")
    {
        Comment = "example of 2 KeyWords(new and virtual), usually here would be just virtual",
        KeyWords = new List<KeyWord> { KeyWord.New, KeyWord.Virtual },
        BodyLines = new List<string> { "return $\"({Real:0.00}, {Imaginary:0.00})\";" }
    }
}.ToDictionary(a => a.Name, a => a);

complexNumberClass.Fields = fields;
complexNumberClass.Properties = properties;
complexNumberClass.Methods = methods;

FileModel complexNumberFile = new FileModel(complexNumberText);
complexNumberFile.LoadUsingDirectives(usingDirectives);
complexNumberFile.Namespace = fileNameSpace;
complexNumberFile.Classes.Add(complexNumberClass.Name, complexNumberClass);

CsGenerator csGenerator = new CsGenerator();
csGenerator.Files.Add(complexNumberFile.Name, complexNumberFile);
csGenerator.CreateFiles(); //Console.Write(complexNumberFile); 
````

## GeneratorModel Composition Structure:
````csharp
CsGenerator
|
|---Files
	|
	|---Enums
	|
	|---Classes
		|
		|---Fields
		|
		|---Constructors (DefaultConstructor first)
		|	|---Attributes
		|	|	|--- Parameters
		|	|
		|	|---Parameters
		|
		|---Properties
		|	|---Attributes
		|	|	|--- Parameters
		|	|
		|	|---Parameters
		|
		|---Methods
		|	|---Attributes
		|	|	|--- Parameters
		|	|
		|	|---Parameters
		|
		|---NestedClasses (recursively)
			|--- ...
````
