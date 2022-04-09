---
title: "EF6 資料庫欄位預設值"
date: 2022-01-24T22:19:36+08:00
draft: false
tags: 
  - EntityFramework
description: "有接觸過SQL的朋友應該都知道資料庫欄位有個功能可以設定預設值，當新增資料時此欄位沒給資料時，資料庫會自動賦予預設值，這個需求再簡單不過了，但是在EF6上解法稍微複雜一些，而且還不能跨資料庫，因為它有部分是調用原始SQL方式，不過這裡先記錄一下方法。"
---

有接觸過SQL的朋友應該都知道資料庫欄位有個功能可以設定預設值，當新增資料時此欄位沒給資料時，資料庫會自動賦予預設值，這個需求再簡單不過了，但是在EF6上解法稍微複雜一些，而且還不能跨資料庫，因為它有部分是調用原始SQL方式，不過這裡先記錄一下方法。

EntityFramework6是一個很不錯的ORM框架，但是它在某些資料庫較為細節的部分有些不夠完善，有些在EntityFrameworkCore都有簡單的解決方法，因為歷史包袱還是要繞些遠路。

# 方法一 C# 端解決方案
這種方式在SQL端還是沒有預設值，但是使用C#物件屬性預設值的方式，如果以完美主義的角度來說還是不夠。
``` cs
public class MyEntity
{
    public string Json { get; set; } = "{}";
}
```

# 方法二 Entity Framework 6 Default Value For Sql
.Net 已經內建 `DefaultValueAttribute` ，所以我們就不另外建立。

* `DbContext` 增加 `Conventions`，將`DefaultValueAttribute` 對應到字串資料上
``` cs
public class DataContext : DbContext
{
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        modelBuilder.Conventions.Add(new AttributeToColumnAnnotationConvention<DefaultValueAttribute, string>(
            "SqlDefaultValue",
            (p, attribute) => 
                attribute.SingleOrDefault().Value.ToString()));
    }
}
```

* `Entity` 中增加 `DefaultValue`，裡面可以增加預設值，除了填寫常數以外，也可以填寫`gettime()`之類的SQL方法，功能也就更多元。
``` cs
public class MyEntity
{
    [DefaultValue("'{}'")]
    public string Json { get; set; } = "{}";
}
```

* 增加 `MigrationSqlGenerator`，在新增資料表、修改資料表、新增欄位、修改欄位時去修改預設值，但是移除預設值時會去執行指定的SQL語法。
``` cs
internal class DefaultValueSqlServerMigrationSqlGenerator : SqlServerMigrationSqlGenerator
{
    private int dropConstraintCount;

    protected override void Generate(AddColumnOperation addColumnOperation)
    {
        SetAnnotatedColumn(addColumnOperation.Column, addColumnOperation.Table);
        base.Generate(addColumnOperation);
    }

    protected override void Generate(AlterColumnOperation alterColumnOperation)
    {
        SetAnnotatedColumn(alterColumnOperation.Column, alterColumnOperation.Table);
        base.Generate(alterColumnOperation);
    }

    protected override void Generate(CreateTableOperation createTableOperation)
    {
        SetAnnotatedColumns(createTableOperation.Columns, createTableOperation.Name);
        base.Generate(createTableOperation);
    }

    protected override void Generate(AlterTableOperation alterTableOperation)
    {
        SetAnnotatedColumns(alterTableOperation.Columns, alterTableOperation.Name);
        base.Generate(alterTableOperation);
    }

    private void SetAnnotatedColumn(ColumnModel column, string tableName)
    {
        if (column.Annotations.TryGetValue("SqlDefaultValue", out var values))
        {
            if (values.NewValue == null)
            {
                column.DefaultValueSql = null;
                using var writer = Writer();

                // Drop Constraint
                writer.WriteLine(GetSqlDropConstraintQuery(tableName, column.Name));
                Statement(writer);
            }
            else
            {
                column.DefaultValueSql = (string)values.NewValue;
            }
        }
    }

    private void SetAnnotatedColumns(IEnumerable<ColumnModel> columns, string tableName)
    {
        foreach (var column in columns)
        {
            SetAnnotatedColumn(column, tableName);
        }
    }

    private string GetSqlDropConstraintQuery(string tableName, string columnName)
    {
        var tableNameSplitByDot = tableName.Split('.');
        var tableSchema = tableNameSplitByDot[0];
        var tablePureName = tableNameSplitByDot[1];

        var str = $@"DECLARE @var{dropConstraintCount} nvarchar(128)
SELECT @var{dropConstraintCount} = name
FROM sys.default_constraints
WHERE parent_object_id = object_id(N'{tableSchema}.[{tablePureName}]')
AND col_name(parent_object_id, parent_column_id) = '{columnName}';
IF @var{dropConstraintCount} IS NOT NULL
EXECUTE('ALTER TABLE {tableSchema}.[{tablePureName}] DROP CONSTRAINT [' + @var{dropConstraintCount} + ']')";

        dropConstraintCount++;
        return str;
    }
}
```

## 結果
之後如果有Migration時候會發現，`DbMigration`中出現 增加移除 `Annotations`的程式碼了。
``` cs
public partial class Add_MyEntity_Table : DbMigration
{
    public override void Up()
    {
        CreateTable(
            "dbo.MyEntity",
            c => new
                {
                    Id = c.Long(nullable: false, identity: true),
                    Type = c.Int(nullable: false),
                    Json = c.String(nullable: false,
                        annotations: new Dictionary<string, AnnotationValues>
                        {
                            { 
                                "SqlDefaultValue",
                                new AnnotationValues(oldValue: null, newValue: "'{}'")
                            },
                        }),
                })
            .PrimaryKey(t => t.Id);
    }
    
    public override void Down()
    {
        DropTable("dbo.MyEntity",
            removedColumnAnnotations: new Dictionary<string, IDictionary<string, object>>
            {
                {
                    "Json",
                    new Dictionary<string, object>
                    {
                        { "SqlDefaultValue", "'{}'" },
                    }
                },
            });
    }
}
```
# 參考資料
[Entity Framework 6 Code first Default value](https://stackoverflow.com/questions/19554050/entity-framework-6-code-first-default-value)