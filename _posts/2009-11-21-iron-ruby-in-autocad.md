---
layout: post
title: Ruby and AutoCAD via IronRuby
---

Inspired by a blog post by Kean Walmsley here, I have been working on some wrapper and helper functions for driving AutoCAD using Ruby.

IronRuby and Kean's loader function gives the AutoCAD developer full access to the managed API. But any that has coded against that API knows it can be verbose and lead to some not-so-readable code, like the C# code below.

```csharp
[CommandMethod(\"AddLine\")]
public static void AddLine()
{
  // Get the current document and database
  Document acDoc = Application.DocumentManager.MdiActiveDocument;
  Database acCurDb = acDoc.Database;

  // Start a transaction
  using (Transaction acTrans = acCurDb.TransactionManager.StartTransaction())
  {
    // Open the Block table for read
    BlockTable acBlkTbl;
    acBlkTbl = acTrans.GetObject(acCurDb.BlockTableId, OpenMode.ForRead) as BlockTable;

    // Open the Block table record Model space for write
    BlockTableRecord acBlkTblRec;
    acBlkTblRec = acTrans.GetObject(acBlkTbl[BlockTableRecord.ModelSpace],
      OpenMode.ForWrite) as BlockTableRecord;

    // Create a line that starts at 5,5 and ends at 12,3
    Line acLine = new Line(new Point3d(5, 5, 0), new Point3d(12, 3, 0));
    acLine.SetDatabaseDefaults();

    // Add the new object to the block table record and the transaction
    acBlkTblRec.AppendEntity(acLine);
    acTrans.AddNewlyCreatedDBObject(acLine, true);

    // Save the new object to the database
    acTrans.Commit();
  }
}

```

In Ruby, using my AcadHelper module, that code becomes

```ruby
def add_line
  line = create_line [1,1], [2,4,5]
  line.add_to_model_space
end
```

Add in a begin-rescue-end block and it still only 8 lines of, in my opinion, very readable code

```ruby
 def add_line
  begin
    line = create_line [1,1], [2,4,5]
    line.add_to_model_space
  rescue Exception => e
    puts_ex e
  end
end
```

I hope Ruby catches on as development option for AutoCAD.

As a long-time AutoLISP hacker, I love having an interpreted language to use for development.
Ruby offers that, along with full access to the API and the beauty and power of Ruby

I have posted an initial verision of my AcadHelper.rb on github.

And development continues. I will be updated everything as time allows. I have added auto-loading of functions, create_text and create_mtext functions,
Editor.GetEntity helper, an exception and backtrace print helper.

Next on the list is selection sets. If anyone wants to help out or has ideas, please let me know
