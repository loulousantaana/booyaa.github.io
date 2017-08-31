extends: post.liquid
title: C Sharp and .NET tips
date: 31 Aug 2017 18:18:18 +0100
path: 2017/csharp-dotnet
tags: c#,csharp,dotnet,.net
---

## NUnit TestCases with instances of a type

```csharp
public IEnumerable<TestCaseData> CanParseAsThingyTestCases
{
    get
    {
        Setup();
        yield return new TestCaseData(@"foo", new Thingy { name = "foo" });
        yield return new TestCaseData(@"bar", new Thingy { name = "bar" });        
    }
}
[TestCaseSource(" CanParseAsThingyTestCases")]
public void CanParseAsThing(string input, Thingy expected)
{
...
```

source: https://stackoverflow.com/a/4230328/105282

## RegEx

A somewhat convoluted example to demonstrate to avoid magic strings (ish)/

```csharp
string pattern = @"^(?<WANT>Foo)bar$";
string input = @"Foobar";
Regex r = Regex(patern, RegexOptions.SingleLine);
Match m = r.Match(input);
foreach (string groupName in r.GetGroupNames()) {
  if (groupName != "0") {
    Console.WriteLine($"Found {m.Groups[groupName].Value} in {groupName}");
  }
}
```

Note to self:
- Turn this into a useful example
  - https://codereview.stackexchange.com/a/6962/2500
  - https://stackoverflow.com/a/10417114/105282