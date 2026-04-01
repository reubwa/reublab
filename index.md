Reublab is a programming language designed for readability and ease of use. It's a personal project, so it may not be the most well-formed product to ever exist, but I publish it in the hopes that someone else may find it useful! (And I'm always open to suggestions and critiques [here](https://github.com/reubwa/reublab/discussions))

With features from the C family of languages and other features inspired by less well-known languages, like Elixir, Reublab not only aims to bridge the gap between languages like Python and languages like C#, but blend this with aspects of Functional Programming.

At the moment, this documentation simply highlights some of the more unique features of Reublab, so it's important to note that the language makes heavy use of C-style syntax (i.e. curly braces) but for type definitions, uses syntax familiar to TypeScript developers.

Version 1 (April 2026) ([Read this on GitHub](https://github.com/reubwa/reublab/blob/main/index.md))

# Structure and Modules

Modules are file-scoped and entry points are entirely optional.

```
module App.Core.Networking;

// The optional execution entry point
@main
pub struct AppRunner {
    pub static fn main(args : list<str>) {
        putln("Programme started.");
    }
}
```

# Variables

Variables support strong typing with optional inference. The language uses nowt to represent the absence of a value (null).

```
let age = 30;                  // Inferred as int
let username : str = "Reuben"; // Explicitly typed
let missing : str = nowt;      // Safe null-equivalent

// Unary operators are fully supported
let counter = 0;
counter++;
--counter;
```

# Global I/O

> The $ symbol indicates interpolation (and typical of other programing languages, curly braces wrap the interpolated statement)

> Use of the -ln functions will result in a newline being added after I/O

## Output
- ```put(...)```
- ```put$(...)```
- ```putln(...)```
- ```putln$(...)```

## Input
- ```get(...)```
- ```get$(...)```
- ```getln(...)```
- ```getln$(...)```

# Object-Orientated Foundations

Data shapes and behavioural contracts are bridged using ```struct```. Classes support inheritance, strict access modifiers, and encapsulated state.

```
pub struct ISerializable {
    props { Id : str - req; }
    fn Serialize() : str;
}

pub sealed class User : ISerializable {
    // Properties assigned upon instantiation
    props {
        Id : str - req;
        Role : str = "Guest";
    }

    // Encapsulated internal state
    priv state loginCount = 0;

    // Operator overloading via the 'exts' block
    exts {
        op[==] (this, rhs : User) : bool => this.Id === rhs.Id;
    }
}
```

## Modifiers
- ```pub```
- ```priv```
- ```prot```
- ```internal```
- ```static```
- ```sealed ```

# Functions & Foreign Code (FFI)

Functions can utilise standard statement blocks or concise expression bodies. Use ```call``` blocks to natively fall back to traget languages (like TypeScript or C#).

```
// Standard block
pub fn calculateTax(amount : num) : num {
    return amount * 0.2;
}

// Expression body
internal static fn greet() => putln("Hello!");

// Foreign Function Interface (FFI) returning back to Reublab
pub fn parseData(raw : str) : dict<str, str> {
    let result = call [javascript] (raw) => {
        return JSON.parse(raw);
    };
    => result as dict<str, str>; 
}
```

# The Pipeline Ecosystem

Data flows strictly left-to-right. Pipelines replace deep nesting and temporary variables.

```
// 1. Standard Pipe
list -> filter(i => i > 0) -> count();

// 2. Inline Scope Pipe
// Creates a temporary variable 'data' that dies after the next step
fetch() -(data)-> parse(data) -> putln("Done");

// 3. Inline Unwrapping Pipe
// Unpacks the object's properties directly into the pipeline scope
User { Name = "Reuben" } -> with(this) -> putln(Name);

// 4. The Guard Pipe
// Halts execution if the condition inside the braces fails
get("Age? ") -> toInt() -{this >= 18}-> grantAccess();

// 5. The Ternary Pipe
// Branches data. Merges back with `->` or terminates cleanly with `|`
fetchData()
    -{this.Status === 200}-> processSuccess() !-> logError() |

// 6. The Error Pipe
// Safely catches exceptions or 'nowt'
fetchData() :(NetworkError): !> putln("Failed to connect");

// 7. Right-Hand Declaration
// Injects the final pipeline value into a new local variable
get("Query? ") -> query : str;
```

# Block Scoping (```with```)

For multi-line mutations or reads against a single object, the ```with``` block creates a temporary shadow scope, removing the need to repeatedly reference the object.

```
let player = Player { Name = "Hero", Health = 100 };

with (player) {
    Health -= 20;
    putln$("Player {Name} was hit!");
    SaveState();
}
```

# Declarative Native UI

UI components are standalone, reactive blocks. They use abstract library namespaces (like ```UI.Box```) to ensure they can be transpiled to Web, iOS, or Android without changing the syntax.

```
with "Reublab.Core.UI" as UI;

pub sealed def "CounterCard" {
    
    props { StartValue : int = 0; }
    priv state count = StartValue;

    UI.Box {
        direction = "column";
        padding = 16;
        
        UI.Cont { text = "Count is: {count}"; }
        
        UI.Button {
            text = "Increment";
            onclick = => { count++; };
        }
    }
}
```

This abstraction will require further work, however it is intended that the default abstraction should be as platform-agnostic as possible, with platform-specific features (such as stylesheets) seperated into extensions/additional libraries.

# Advanced Pattern Matching (```switch```)

```switch``` cases are strictly evaluated from top to bottom. Matching against literals, relational operators, nested properties, and closures are all also supported.

```
let statusMessage = response switch {
    // 1. Concrete literals (Must go first)
    200 => "Success",
    
    // 2. Relational patterns against the root value
    {>= 500} => "Server Fault",
    
    // 3. Destructuring with nested property conditions
    {User{Role == "Admin"}} => "Access Granted",
    
    // 4. Type matching with specific property capturing
    {NetworkException(RetryCount = r)} ={r > 3}=> "Max retries exceeded",
    
    // 5. Whole-object type capture into a closure
    {str} =(msg)=> putln$("String error received: {msg}"),
    
    // 6. The catch-all discard
    _ => "Unknown State"
};
```
