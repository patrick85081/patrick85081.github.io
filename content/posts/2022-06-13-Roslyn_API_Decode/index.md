---
title: "Roslyn API 練習"
date: 2022-06-13T14:11:46+08:00
draft: false
tags: 
  - Roslyn
description: "假設我有個 CSharp 程式碼檔案，我想要用寫程式來解析裡面的東西，這件事情就是C# Compile做的事情，微軟提供Roslyn API供我們來解析程式碼，今天來做個小練習，沒有什麼功用，利用 Roslyn API解析程式碼，並重新輸出程式碼。"
---

# 前言
假設我有個 CSharp 程式碼檔案，我想要用寫程式來解析裡面的東西，這件事情就是C# Compile做的事情，微軟提供Roslyn API供我們來解析程式碼，今天來做個沒有什麼功用的小練習，利用 Roslyn API解析程式碼，並重新輸出程式碼。

``` cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Threading.Tasks;

namespace SyntaxAnalysisTest
{
    /// <summary>
    /// 為賦新詞強說愁
    /// </summary>
    public interface IMyService
    {
        /// <summary>
        /// 隨機產生 GUID
        /// </summary>
        /// <returns>GUID</returns>
        Guid GetGuid();

        /// <summary>
        /// 產生指定數量的隨機數
        /// </summary>
        /// <param name="count">數量</param>
        /// <returns>隨機數陣列</returns>
        int[] GenRandNumbers(int count);

        /// <summary>
        /// 複雜參數
        /// </summary>
        /// <param name="ints">整數陣列</param>
        /// <param name="mapping">對照表</param>
        /// <param name="args">不定數量參數</param>
        [Description("TEST")]
        void Complex(List<int> ints, Dictionary<string, string> mapping = null, params string[] args);
    }

    public class MyService : IMyService
    {
        /// <summary>
        /// 隨機產生 GUID
        /// </summary>
        /// <returns>GUID</returns>
        public Guid GetGuid() => Guid.NewGuid();
        /// <summary>
        /// 產生指定數量的隨機數
        /// </summary>
        /// <param name="count">數量</param>
        /// <returns>隨機數陣列</returns>
        public int[] GenRandNumbers(int count) => new int[]{1, 2, 3};
        /// <summary>
        /// 複雜參數
        /// </summary>
        /// <param name="ints">整數陣列</param>
        /// <param name="mapping">對照表</param>
        /// <param name="args">不定數量參數</param>
        [Description("TEST")]
        public void Complex(List<int> ints, Dictionary<string, string> mapping = null, params string[] args) {}
    }
}
```

# 寫個小練習
這裡寫個遞回，把Roslyn API 解析出來的節點輸出回程式碼

Roslyn提供Syntax Tree表達樹，裡面的種類
* Syntax Node
  宣告（類別、介面）、子句、表達式等等
* Syntax Token
  子節點，包含 keyword name 符號等等程式碼元素 
* Syntax Trivia
  空白、換行、註解 等等
``` cs
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;

var code = File.ReadAllText("IMyService.txt");

var tree = CSharpSyntaxTree.ParseText(code);
var root = tree.GetCompilationUnitRoot();

OutputCode(root.ChildNodesAndTokens());


void OutputCode(IEnumerable<SyntaxNodeOrToken> children)
{
    var n = false;
    OutputCodeImpl(children, ref n, 0);
}

void OutputCodeImpl(IEnumerable<SyntaxNodeOrToken> children, ref bool isNewLine, int tabCount)
{
    foreach (var child in children)
    {
        if (!child.IsToken)
        {
            OutputCodeImpl(child.ChildNodesAndTokens(), ref isNewLine, tabCount);
            continue;
        }

        if (child.Kind() == SyntaxKind.EndOfFileToken)
            return;

        var txt = child.Kind() switch
        {
            SyntaxKind.UsingKeyword => "using",
            SyntaxKind.SemicolonToken => ";",
            SyntaxKind.IdentifierToken => child.ToString(),
            SyntaxKind.DotToken => ".",
            SyntaxKind.NamespaceKeyword => "namespace",
            SyntaxKind.OpenBraceToken => "{",
            SyntaxKind.CloseBraceToken => "}",
            SyntaxKind.PublicKeyword => "public",
            SyntaxKind.InterfaceKeyword => "interface",
            SyntaxKind.ClassKeyword => "class",
            SyntaxKind.ColonToken => ":",
            SyntaxKind.OpenParenToken => "(",
            SyntaxKind.CloseParenToken => ")",
            SyntaxKind.VoidKeyword => "void",
            SyntaxKind.IntKeyword => "int",
            SyntaxKind.OpenBracketToken => "[",
            SyntaxKind.CloseBracketToken => "]",
            SyntaxKind.ParamsKeyword => "params",
            SyntaxKind.StringKeyword => "string",
            SyntaxKind.NullKeyword => "null",
            SyntaxKind.CommaToken => ",",
            SyntaxKind.NewKeyword => "new",
            SyntaxKind.StringLiteralToken => child.ToString(),
            SyntaxKind.OmittedArraySizeExpressionToken => child.ToString(),
            SyntaxKind.EqualsGreaterThanToken => "=>",
            SyntaxKind.List => "List",
            SyntaxKind.LessThanToken => "<",
            SyntaxKind.GreaterThanToken => ">",
            SyntaxKind.NumericLiteralToken => child.ToString(),
            SyntaxKind.EqualsToken => "=",
            _ => $"*{child.Kind()}*",
        };

        // {
        if (child.Kind() == SyntaxKind.CloseBraceToken)
            tabCount -= 1;

        // \t
        if (isNewLine)
        {
            isNewLine = false;
            for (int i = 1; i <= tabCount; i++)
                Console.Write("\t");
        }

        Console.Write(txt);
        
        // }
        if (child.Kind() == SyntaxKind.OpenBraceToken)
            tabCount += 1;


        SyntaxTriviaList syntaxTriviaList = child.GetTrailingTrivia();
        foreach (var syntaxTrivia in syntaxTriviaList)
        {
            var kind = syntaxTrivia.Kind();
            // " "
            if (syntaxTrivia.Kind() == SyntaxKind.WhitespaceTrivia)
            {
                Console.Write(" ");
            }
            // \r\n
            else if (syntaxTrivia.Kind() == SyntaxKind.EndOfLineTrivia)
            {
                Console.WriteLine();
                isNewLine = true;
            }
            else
            {
                // 尚未實作的種類
                Console.WriteLine($"*{syntaxTrivia.Kind()}*");
            }
        }
    }
}
```

得到輸出結果
``` cs
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Threading.Tasks;

namespace SyntaxAnalysisTest
{

    public interface IMyService
    {
        Guid GetGuid();

        int[] GenRandNumbers(int count);

        [Description("TEST")]
        void Complex(List<int> ints, Dictionary<string, string> mapping = null, params string[] args);

    }

    public class MyService : IMyService
    {
        public Guid GetGuid() => Guid.NewGuid();

        public int[] GenRandNumbers(int count) => new int[]{1, 2, 3};

        [Description("TEST")]
        public void Complex(List<int> ints, Dictionary<string, string> mapping = null, params string[] args) {}

    }
}
```
