---
layout: post
title: AutoCAD selection sets in IronRuby
---


Using selection set filters in the AutoCAD .Net api has always been less than enjoyable.
Take, for example, this bit of code from the [online .Net Developer's Guide](ttp://docs.autodesk.com/ACD/2010/ENU/AutoCAD%20.NET%20Developer%27s%20Guide/index.html)

```csharp
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

[CommandMethod(\"FilterSelectionSet\")]
public static void FilterSelectionSet()
{
  // Get the current document editor
  Editor acDocEd = Application.DocumentManager.MdiActiveDocument.Editor;

  // Create a TypedValue array to define the filter criteria
  TypedValue[] acTypValAr = new TypedValue[1];
  acTypValAr.SetValue(new TypedValue((int)DxfCode.Start, \"CIRCLE\"), 0);

  // Assign the filter criteria to a SelectionFilter object
  SelectionFilter acSelFtr = new SelectionFilter(acTypValAr);

  // Request for objects to be selected in the drawing area
  PromptSelectionResult acSSPrompt;
  acSSPrompt = acDocEd.GetSelection(acSelFtr);

  // If the prompt status is OK, objects were selected
  if (acSSPrompt.Status == PromptStatus.OK)\
  {
    SelectionSet acSSet = acSSPrompt.Value;
    Application.ShowAlertDialog(\"Number of objects selected: \" + acSSet.Count.ToString());
  }
  else
  {
    Application.ShowAlertDialog(\"Number of objects selected: 0\");
  }
}
```

The thing that bothers me the most is having to build that ugly TypedValue array

````csharp
TypedValue[] acTypValAr = new TypedValue[1];
acTypValAr.SetValue(new TypedValue((int)DxfCode.Start, \"CIRCLE\"), 0);
````
Being an old LISP hacker,  I know most of the DxfCodes (8 for layer, 0 for type, 62 for color, etc) or I can find them easily.  But why am I forced to remember those codes at all?  If I know that I want to filter on type or layer or color, why do I have to either know the appropriate code or cast the enum member to an int.  And why do I have to keep up with the index  (ie, that 0 just after \"CIRCLE\"),  )?
Why can't I just do something like
```ruby
filter = SsFilter.new
filter.Layer = "my_layer"
filter.Type = "Circle"
```
Well now I can.  I have extended my [acadhelper](http://github.com/davidbl/acadhelper) to include a new class called SsFilter
```ruby
class SsFilter
  attr_accessor :data
  def initialize
    @data = {}
    @elements = {:Type => 0, :Text => 1, :BlockName => 2, :LineType => 6, :TextStyle => 7, :Layer => 8,
                 :StartPoint => 10, :CenterPoint => 10, :EndPoint => 11, :Elevation => 38, :Thickness => 39,
                 :TextHeight => 40, :TextWidth => 41, :Rotation => 50, :Oblique => 51, :Color => 62}
    @keys = @elements.keys
  end
  def filter
    typed_value = Ads::TypedValue[]
    new_filter = System::Array.of(typed_value).new(@data.size)
    i = 0
    if @data.size > 0
      @data.each_pair do  |key,value|
        new_filter.set_value(Ads::TypedValue.new( @elements[key], value), i)
        i+=1
      end
    end
    Aei::SelectionFilter.new(new_filter)
  end

  def method_missing(method_name, *args)
    if method_name.to_s.match(/(\\w+)=/) && @keys.include?($1.to_sym)
      @data[$1.to_sym] = args[0]\
    elsif @keys.include?(method_name)
      return @data[method_name]
    else
      super(method_name, *args)
    end
  end
end
```

This new class uses a bit of method_missing magic to create a very friendly and readable way of creating a selection set filter.

The C# code above can now, using other methods from my acadhelper gem,  be written as 
```ruby
require 'rubygems'
require 'acadhelper'
include AcadHelper

def filter_selection_set
  filter = SsFilter.new
  filter.Type = "Circle"
  ss = select_on_screen filter
  result = "Number of objects selected: "
  result += ss ?  ss.count.to_s : "0"
  alert result
end
```

And I did have to make a minor change to the select_on_screen method to support the new filter class
and still allow for hand-built filter arrays
```ruby
def select_on_screen( filter_data=[])
  begin
    if filter_data.is_a?(SsFilter) #new test
      filter = filter_data.filter
    else
      filter = build_selection_filter filter_data
    end

    ssPrompt = ed.GetSelection( filter)
    if ssPrompt.Status == Aei::PromptStatus.OK
      return ssPrompt.Value
    else
      nil
    end
    rescue Exception => e
      puts_ex e
    end
end
```

The new class has been added to the gem on [github](http://github.com/davidbl/acadhelper) but
it  does not yet support logical grouping or relational tests.  Those are coming soon
