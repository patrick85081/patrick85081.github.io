---
title: "快速上手 System.Text.Json"
date: 2022-04-12T20:21:21+08:00
draft: false
tags: 
  - Json
description: "現在有些C#程式使用到Json時，慢慢地從 Newtown.Json 轉換成 System.Text.Json ，System.Text.Json 有些手法還是需要學習一下。"
---
現在有些C#程式使用到Json時，慢慢地從`Newtown.Json`轉換成`System.Text.Json`，`System.Text.Json`有些手法還是需要學習一下。

# 範例 Json 資料
這是本次範例使用的Json資料
``` json
{
  // Commit
  "Field": {
    "AA": 1234,
    "BB": "中文",
  }
}
```

## Json Option
System.Text.Json在 序列化 與 反序列化 有些 Option 需要注意
* 序列化
  預設不允許註解、不允許 尾端欄位 有逗號
``` cs
new JsonDocumentOptions
{
    // 允許 Json 註解
    CommentHandling = JsonCommentHandling.Skip,
    // 允許 尾端欄位 有逗號
    AllowTrailingCommas = true
}
```
* 反序列化
  預設 沒有開啟中文的編碼，所以有需要輸出中文需要特別設定。
``` cs
new JsonSerializerOptions() 
{ 
    Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
}
```

---
# Utf8JsonWriter / Utf8JsonReader
這是裡面較為低階的處理方式，屬於一次一個節點的處理，`JsonConverter`會需要此操作，所以還是必須學會。
``` cs
var stream = new MemoryStream();
var writer = new Utf8JsonWriter(stream, new JsonWriterOptions { Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping});
// {
writer.WriteStartObject();

//   "Field": {
writer.WritePropertyName("Field");
writer.WriteStartObject();

//      "AA": 1234,
writer.WritePropertyName("AA");
writer.WriteNumberValue(1234);

//      "BB": "中文",
writer.WritePropertyName("BB");
writer.WriteStringValue("中文");

//   }
writer.WriteEndObject();

// }
writer.WriteEndObject();

writer.Flush();
Encoding.UTF8.GetString(stream.ToArray()).Dump();
// {"Field":{"AA":1234,"BB":"中文"}}


var reader = new Utf8JsonReader(stream.ToArray());
while (reader.Read())
{
    switch (reader.TokenType)
    {
        case JsonTokenType.StartObject:
            break;
        case JsonTokenType.StartArray:
            break;
        case JsonTokenType.Comment:
            break;

        case JsonTokenType.PropertyName:
            Console.WriteLine($"PropertyName = {reader.GetString()}");
            break;

        case JsonTokenType.String:
            Console.WriteLine($"Value = {reader.GetString()}");
            break;
        case JsonTokenType.Number:
            Console.WriteLine($"Value = {reader.GetInt32()}");
            break;
        case JsonTokenType.Null:
            break;
        case JsonTokenType.True:
            break;
        case JsonTokenType.False:
            break;
    }
}
/*
PropertyName = Field
PropertyName = AA
Value = 1234
PropertyName = BB
Value = 中文
*/
```

# JsonDocument / JsonElement
可以做資料讀取，沒辦法做修改資料，如果需要還是要透過`Utf8JsonWriter`來寫。
``` cs
var doc = JsonDocument.Parse(json,
    new JsonDocumentOptions
    {
        // 允許 Json 註解
        CommentHandling = JsonCommentHandling.Skip,
        // 允許 尾端欄位 有逗號
        AllowTrailingCommas = true
    });

doc.RootElement
    .GetProperty("Field")
    .GetProperty("AA")
    .GetInt32()
    .Dump();
// 123

doc.RootElement
    .GetProperty("Field")
    .GetProperty("BB")
    .GetString()
    .Dump();
// 中文

doc.RootElement.GetRawText().Dump()
/*
{
  "Field": {
    "AA": 1234,
    "BB": "\u4E2D\u6587"
  }
}
*/
```

# JsonNode / JsonObject / JsonArray / JsonValue
這是 .Net6 才提供的API，這裡使用起來比較接近`Newtown.Json`裡的`JObject`、`JArray`一樣好用，而且支援 **修改** 功能。
``` cs
var node = JsonNode.Parse(json, 
    documentOptions: new JsonDocumentOptions
    { 
        // 允許 Json 註解
        CommentHandling = JsonCommentHandling.Skip, 
        // 允許 尾端欄位 有逗號
        AllowTrailingCommas = true
    });
node["Field"]["BB"].GetValue<string>().Dump();
// 中文

node["Field"]["BB"].ToJsonString().Dump();
// "\u4E2D\u6587"

node["Field"]["BB"].ToJsonString(
    new JsonSerializerOptions() 
    { 
        Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping
    }).Dump();
// "中文"

jo["Field"]["CC"] = "1234";
jo.Dump();
// {"Field":{"AA":1234,"BB":"中文","CC":"1234"}}
```