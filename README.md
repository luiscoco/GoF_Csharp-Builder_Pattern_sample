# GoF_Csharp-Builder_Pattern_sample

This code sample demonstrates the Builder creational design pattern in C#. The Builder pattern is used to construct complex objects step by step while hiding the underlying creation process. This helps improve the readability and maintainability of the code, especially when dealing with complex object creation with multiple parameters and configurations.

Let's break down the code to understand how the Builder pattern is implemented:

## 1. HtmlElement (Product):
Represents an HTML element with a name and text content.
Contains a list of child elements.
Implements a recursive ToStringImpl method to convert the element and its children to a formatted string.
Provides a static factory method Create to create a new HtmlElement.

## 2. HtmlBuilder (Builder):
Responsible for constructing the HtmlElement object.
Supports both fluent and non-fluent methods for adding child elements to the root element.
Implements a Build method to return the final constructed HtmlElement.
Implements an implicit operator to convert a HtmlBuilder to an HtmlElement.

## 3. Demo:
Demonstrates the usage of the Builder pattern in various scenarios.
Creates HTML elements using both non-fluent and fluent approaches.
Shows how the Builder pattern is used with a factory method and implicit operator.
Key points to note:

The HtmlBuilder class is responsible for constructing the HtmlElement object. It provides methods like AddChild, AddChildFluent, and Build to create and add child elements to the root element.

The HtmlElement class defines a recursive ToStringImpl method that formats the HTML elements and their children as a string.

The HtmlBuilder class implements a non-fluent approach using regular methods like AddChild and a fluent approach using methods like AddChildFluent.

The HtmlBuilder class also has an implicit operator that allows converting a HtmlBuilder instance to an HtmlElement, making it more convenient to obtain the final result.

The Demo class showcases different ways of using the Builder pattern, including using factory methods and the implicit operator.

In summary, the Builder pattern helps separate the construction of complex objects from their representation, allowing for more flexible and readable code when creating objects with multiple configuration options.

```csharp
﻿using System.Collections.Generic;
using System.Text;
using static System.Console;

namespace DotNetDesignPatternDemos.Creational.Builder;

public record HtmlElement(string Name, string Text)
{
  public string Name = Name;
  public string Text = Text;
  private readonly Lazy<List<HtmlElement>> elements = new();
  public List<HtmlElement> Elements => elements.Value;
  private const int indentSize = 2;

  public HtmlElement() : this("", "") { }
  
  private string ToStringImpl(int indent)
  {
    var sb = new StringBuilder();
    var i = new string(' ', indentSize * indent);
    sb.Append($"{i}<{Name}>\n");
    if (!string.IsNullOrWhiteSpace(Text))
    {
      sb.Append(new string(' ', indentSize * (indent + 1)));
      sb.Append(Text);
      sb.Append('\n');
    }

    foreach (var e in Elements)
      sb.Append(e.ToStringImpl(indent + 1));

    sb.Append($"{i}</{Name}>\n");
    return sb.ToString();
  }

  public override string ToString()
  {
    return ToStringImpl(0);
  }
    
  public static HtmlBuilder Create(string name) => new(name);
}

public class HtmlBuilder
{
  private readonly string rootName;
  protected HtmlElement root = new();

  public HtmlBuilder(string rootName)
  {
    this.rootName = rootName;
    root.Name = rootName;
  }

  // not fluent
  public void AddChild(string childName, string childText)
  {
    var e = new HtmlElement(childName, childText);
    root.Elements.Add(e);
  }

  public HtmlBuilder AddChildFluent(string childName, string childText)
  {
    var e = new HtmlElement(childName, childText);
    root.Elements.Add(e);
    return this;
  }

  public static HtmlBuilder Init(string rootName)
  {
    return new HtmlBuilder(rootName);
  }

  public override string ToString() => root.ToString();

  public void Clear()
  {
    root = new HtmlElement{Name = rootName};
  }

  public HtmlElement Build() => root;


  public static implicit operator HtmlElement(HtmlBuilder builder)
  {
    return builder.root;
  }
}

public class Demo
{
  static void Main(string[] args)
  {
    {
      // if we want to build a simple HTML paragraph...
      var hello = "hello";
      var text = "";
      text += "<p>";  // <p> → text
      text += hello;  // <p>hello → text
      text += "</p>"; // <p>hello</p> → text
      WriteLine(text);

      // now we want an HTML list with 2 words in it
      var sb = new StringBuilder();
      var words = new[] {"hello", "world"};
      sb.Clear();
      sb.Append("<ul>");
      foreach (var word in words)
      {
        sb.Append($"<li>{word}</li>");
      }

      sb.Append("</ul>");
      WriteLine(sb);

      // ordinary non-fluent builder
      var builder = new HtmlBuilder("ul");
      builder.AddChild("li", "hello");
      builder.AddChild("li", "world");
      WriteLine(builder.ToString());

      // fluent builder
      sb.Clear();
      builder.Clear(); // disengage builder from the object it's building, then...
      builder.AddChildFluent("li", "hello").AddChildFluent("li", "world");
      WriteLine(builder);
    }

    // with factory method
    {
      var builder = HtmlElement.Create("ul");
      builder.AddChildFluent("li", "hello")
        .AddChildFluent("li", "world");
      WriteLine(builder);
    }
      
    // with implicit operator
    {
      var root = HtmlElement
        .Create("ul")
        .AddChildFluent("li", "hello")
        .AddChildFluent("li", "world");
      WriteLine(root);
    }
  }
}
```












