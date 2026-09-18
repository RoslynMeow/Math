# RoslynMeow.Math.Graph — Overview and Usage

[中文](ReadMe.md) | [English](ReadMe.en.md)

[![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-RoslynMeow.Math.Graph-2ea44f?logo=nuget)](https://github.com/RoslynMeow/Math/pkgs/nuget/RoslynMeow.Math.Graph)
[![Release](https://img.shields.io/github/v/release/RoslynMeow/Math?include_prereleases&sort=semver&label=release)](https://github.com/RoslynMeow/Math/releases)
[![License](https://img.shields.io/github/license/RoslynMeow/Math)](https://github.com/RoslynMeow/Math/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-netstandard2.0-512BD4)](https://dotnet.microsoft.com)

## Contents

- [Repository organization](#repository-organization)
- [Parser design](#parser-design)
- [Quick examples: parsing JSON / XML / CSV](#quick-examples-parsing-json--xml--csv)
- [Usage examples](#usage-examples)
- [Tests](#tests)

## Repository organization

- `GraphX/` – library source (target: .NET Standard 2.0)
  - `Core/IGraph.cs`: graph interface (nodes/edges/weights/basic operations).
  - `Core/Graph.cs`: the core graph implementation and helpers.
  - `Graph/`: concrete graph implementations (e.g. weighted graphs, if present).
  - `Algorithms/GraphMethods.cs`: algorithm collection (shortest path, minimum spanning tree, max flow, topological sort, etc., as implementations and extension methods).
  - `Algorithms/GraphMethods.Parser.cs`: text parser (parses simple textual descriptions into a graph).
  - `Parser/GraphTextParserBase.cs`: parser base abstraction (implement custom input format parsing).
  - `Parser/DefaultTextParser.cs`: the default text parser implementation (supports the a--b / a->b / a->b:W syntax).
  - `Error/`: custom exception types (e.g. `GraphMethodNotApplicableException`, `BFANWCDetectedException`, `InvalidArgumentException`, etc.).
  - `Helpers/LongWeightOperator.cs`: weight-operator abstraction and a `long` implementation example.
  - `Util/`: utility data structures (`BinaryHeap`, `UnionFind`, etc.).
  - `Structs/PathResult.cs`: path result container (node sequence, cost, reachability).

- Other projects
  - `GraphX.Tests/` – unit/integration tests (target: .NET 6).
  - `Bit/`, `RoslynMeow.Math.Fraction/` and similar are other math utility library projects in the workspace.

> Note: the paths above are based on the current workspace layout; actual file names or subdirectories may change with the implementation.

## Parser design

- An extensible parser abstraction `Parser/GraphTextParserBase.cs` is provided to map arbitrary input (text, CSV, line-by-line JSON, XML fragments, etc.) into graph connection descriptions.

- The default implementation `Parser/DefaultTextParser.cs` supports a concise syntax:
  - `a--b` (undirected, unweighted)
  - `a->b` (directed, unweighted)
  - `a->b:W` (directed, weighted)
  - `a--b:W` (undirected, weighted)

- Usage example:

```csharp
// Use the default parser
var parser = new RoslynMeow.Math.Graph.Parser.DefaultTextParser<string,long>();
var g = parser.Parse("s->a:1\na->b:2\n", new LongWeightOperator(), out var inferred);

// Custom parser: implement GraphTextParserBase to parse JSON or other formats
var myParser = new MyCustomParser<MyIdType,double>();
var g2 = myParser.Parse(text, myOp, out var type);
```

## Quick examples: parsing JSON / XML / CSV

The following sketches minimal approaches for several common input formats (Chinese first, English after):

1) JSON (the whole text is a JSON array)<br/>
- Idea: use `System.Text.Json.JsonSerializer` to deserialize to DTOs (fields `u, v, w, d`) and build the graph.

```csharp
public class EdgeDto { public string u { get; set; } public string v { get; set; } public long w { get; set; } public bool d { get; set; } }
var list = System.Text.Json.JsonSerializer.Deserialize<List<EdgeDto>>(jsonText);
var g = new Graph<string,long>(new LongWeightOperator());
foreach (var e in list) { if (!g.Exist(e.u)) g.AddNode(e.u); if (!g.Exist(e.v)) g.AddNode(e.v); g.AddEdge(e.u, e.v, e.w, e.d); }
```

2) Line-by-line JSON (each line is a small object)<br/>
- Idea: read line by line and call `JsonSerializer.Deserialize<EdgeDto>(line)`, or subclass `GraphTextParserBase` to extract tokens per line.

```csharp
var parser = new JsonLineParser(); // example: subclass GraphTextParserBase and implement TryParseTokens
var g = parser.Parse(text, new LongWeightOperator(), out var type);
```

3) XML (the whole document, or one entry per element)<br/>
- Idea: use `System.Xml.Linq.XDocument` or `XmlDocument`, find each `<edge u="A" v="B" w="3" d="true" />` node, read the attributes and add to the graph.

```csharp
var doc = System.Xml.Linq.XDocument.Parse(xmlText);
var g = new Graph<string,long>(new LongWeightOperator());
foreach (var xe in doc.Descendants("edge")) {
  var u = (string)xe.Attribute("u"); var v = (string)xe.Attribute("v");
  var w = (long?)xe.Attribute("w") ?? 1; var d = (bool?)xe.Attribute("d") ?? false;
  if (!g.Exist(u)) g.AddNode(u); if (!g.Exist(v)) g.AddNode(v);
  g.AddEdge(u, v, w, d);
}
```

4) CSV (simple comma-separated, columns u,v,w,d)<br/>
- Idea: split lines, then `line.Split(',')`, index the columns and build nodes/edges. For quoted/escaped CSV use a dedicated library (e.g. `CsvHelper`).

```csharp
var g = new Graph<string,long>(new LongWeightOperator());
foreach (var line in csvText.Split(new[]{'\r','\n'}, StringSplitOptions.RemoveEmptyEntries)) {
  var cols = line.Split(','); var u = cols[0].Trim('"'); var v = cols[1].Trim('"');
  var w = cols.Length>2 ? long.Parse(cols[2]) : 1; var d = cols.Length>3 ? bool.Parse(cols[3]) : false;
  if (!g.Exist(u)) g.AddNode(u); if (!g.Exist(v)) g.AddNode(v);
  g.AddEdge(u, v, w, d);
}
```

5) Extend a custom format via `GraphTextParserBase` (recommended)<br/>
- Idea: subclass `GraphTextParserBase<NodeType,TWeight>` and implement `TryParseTokens`, which must extract `leftToken, rightToken, weightToken, directed` from a line or text; the base class then handles conversion and graph construction.

```csharp
// Pseudocode: implement TryParseTokens to map one JSON / XML / CSV record to tokens
class MyParser : GraphTextParserBase<string,long> {
  protected override bool TryParseTokens(string line, out string left, out string right, out string? w, out bool d) { ... }
}
var my = new MyParser(); var g = my.Parse(text, new LongWeightOperator(), out var type);
```

## Usage examples

1. Construct or parse a graph

```csharp
// Legacy call (returns Graph<string,long>)
var baseG = RoslynMeow.Math.Graph.Core.Graph.Parse("s->a:1\na->b:2\n", new LongWeightOperator(), out _);
IGraph<string,long> ig = baseG;

// Or use the Parser abstraction
var parser = new RoslynMeow.Math.Graph.Parser.DefaultTextParser<string,long>();
var g = parser.Parse("s->a:1\na->b:2\n", new LongWeightOperator(), out _);
IGraph<string,long> ig2 = g;
```

2. Call algorithms (as extension methods on `IGraph`)

```csharp
// Shortest path - Dijkstra
var result = ig.Dijkstra<string,long>("s", "b", new LongWeightOperator(), includeNodeWeight:false);
if (result.Reachable) Console.WriteLine($"cost={result.Cost}");

// Bellman-Ford (negative-cycle detection)
var bf = ig.BellmanFord<string,long>("s", new LongWeightOperator());
var distMap = bf.Item1;

// Minimum spanning tree - Kruskal / Prim (for undirected graphs)
var mst = ig.Kruskal<string,long>();
var prim = ig.Prim<string,long>(ig.Nodes.First());

// Topological sort (directed acyclic graphs only)
var order = ig.TopologicalSort<string,long>();

// Max flow (Edmonds-Karp / Dinic, for directed graphs)
long mf = ig.EdmondsKarpMaxFlow<string>("s","t");
long mf2 = ig.DinicMaxFlow<string>("s","t");
```

3. Exceptions and preconditions

- If the graph does not meet an algorithm's preconditions (e.g. Dijkstra requires non-negative weights, Kruskal/Prim require undirected, topological sort requires acyclic, etc.), the method
- throws `RoslynMeow.Math.Graph.Error.GraphMethodNotApplicableException`, carrying `MethodName`, `Reason`, and `GraphSummary` for diagnostics.
- On detecting a negative-weight cycle, the relevant shortest-path algorithms (Bellman-Ford / the detection phase of Johnson) throw `BFANWCDetectedException`.

## Tests

- Parser tests were added: `GraphX.Tests/ParserTests/CustomParserTests.cs`, covering basic parsing checks for `DefaultTextParser` and a sample custom line-by-line JSON parser test.

For more details and examples, see the `GraphX` project source and the test cases in `GraphX.Tests`.
