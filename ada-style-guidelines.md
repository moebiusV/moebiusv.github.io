---
title: Ada functional style
permalink: /ada-style-guidelines.html
---

# Ada functional style

A contributor's guide. Ada here is written as a strongly typed functional
pipeline: minimize mutable state, drop the boilerplate loops, and favor a
readable expression over a procedural block. Not "code golf" and not point-free
cleverness. Ada 2012 and 2022 features do the work. Each point shows the
imperative form to avoid or minimize, then the functional form to write.
Rules 1–20 are expression-oriented; rules 21–30 add the next layer — immutable
updates, algebraic data, and generic abstraction.

## Core expressions and immutability

**1. Constants by default.** Declare a variable only when it must mutate after
initialization.

```ada
--  wrong
X : Integer := 0;
X := Compute (Y);
```

```ada
--  right
X : constant Integer := Compute (Y);
```

**2. Expression functions.** A function that just computes a value is an
expression function, not a `begin ... end` body.

```ada
--  wrong
function Double (X : Integer) return Integer is
begin
   return X * 2;
end Double;
```

```ada
--  right
function Double (X : Integer) return Integer is (X * 2);
```

**3. `if` expressions, not `if` statements.** Initialize in the declaration;
never assign inside a branch.

```ada
--  wrong
Result : Integer;
if X > 0 then
   Result := X;
else
   Result := 0;
end if;
```

```ada
--  right
Result : constant Integer := (if X > 0 then X else 0);
```

**4. `case` expressions, not `case` statements.** The compiler checks that the
alternatives are exhaustive.

```ada
--  wrong
Kind : Token_Kind;
case K is
   when Red    => Kind := T_Stop;
   when Green  => Kind := T_Go;
   when Yellow => Kind := T_Slow;
end case;
```

```ada
--  right
Kind : constant Token_Kind :=
   (case K is
       when Red    => T_Stop,
       when Green  => T_Go,
       when Yellow => T_Slow);
```

**5. `declare` expressions for a local temp.** When an expression needs a
temporary to avoid computing twice, a `declare` expression keeps it an
expression function instead of falling back to a body.

```ada
--  wrong
function Area (R : Float) return Float is
   S : constant Float := R * R;
begin
   return Pi * S;
end Area;
```

```ada
--  right
function Area (R : Float) return Float is
   (declare S : constant Float := R * R; begin Pi * S);
```

**6. Minimize `out` / `in out` parameters.** Prefer a function that returns new
state.  But Ada is not a purely functional language, and the programs built
here — parsers advancing a cursor, bindings filling a caller's buffer, daemons
holding live state — genuinely mutate.  Reach for `in out` when mutation is the
point; the rule is *minimize and localize*, not *never*.

```ada
--  prefer: return new state when the value is small and caller-owned
function With_Status (C : Context; S : Status) return Context is
   (C with delta Status => S);
```

```ada
--  acceptable: mutate in place when the value is large, or mutation is the point
procedure Advance (C : in out Context; S : Status);
```

**7. Delta aggregates for one-field updates.** Return a new copy with the field
changed; do not overwrite a field of an existing record.

```ada
--  wrong
New_State : State := Old_State;
New_State.Status := Active;
```

```ada
--  right
New_State : constant State := (Old_State with delta Status => Active);
```

**8. Container aggregates.** Initialize with `[...]`, not a run of `.Append` or
`.Insert` calls.

```ada
--  wrong
V : Vectors.Vector;
V.Append (1);
V.Append (2);
V.Append (3);
```

```ada
--  right
V : constant Vectors.Vector := [1, 2, 3];
```

## Loops, iteration and reductions

**9. No imperative `while` / `for` to accumulate.** A loop that builds a total,
a string, or a value is a fold or a comprehension in disguise.

```ada
--  wrong
Total : Integer := 0;
for I of Values loop
   Total := Total + I;
end loop;
```

```ada
--  right
Total : constant Integer := Values'Reduce ("+", 0);
```

**10. Quantified expressions for searches.** `for all` / `for some` replace a
loop that walks a structure looking for a condition.

```ada
--  wrong
All_Active : Boolean := True;
for Node of Tree loop
   if not Node.Is_Active then
      All_Active := False;
      exit;
   end if;
end loop;
```

```ada
--  right
All_Active : constant Boolean := (for all Node of Tree => Node.Is_Active);
```

**11. Filter at the iterator with `when`.** When a loop must run for side
effects, filter with `when` rather than an `if` inside the body.

```ada
--  wrong
for Item of List loop
   if Item.Is_Valid then
      Process (Item);
   end if;
end loop;
```

```ada
--  right
for Item of List when Item.Is_Valid loop
   Process (Item);
end loop;
```

**12. `'Reduce` for folds.** A stateful accumulator becomes a fold.

```ada
--  wrong
Combined : Unbounded_String := Null_Unbounded_String;
for I of Parts loop
   Combined := Combined & I;
end loop;
```

```ada
--  right: "&" is overloaded, so 'Reduce takes a named reducer
function Concat (A, B : Unbounded_String) return Unbounded_String is (A & B);

Combined : constant Unbounded_String :=
   Parts'Reduce (Concat, Null_Unbounded_String);
```

**13. Array comprehensions for maps.** `[for X of Source => F]` transforms a
structure declaratively.

```ada
--  wrong
Squares : Int_Array := (others => 0);
for I in Source'Range loop
   Squares (I) := Source (I) * Source (I);
end loop;
```

```ada
--  right
Squares : constant Int_Array := [for X of Source => X * X];
```

**14. Bounded recursion for tree walks.** Walk a tree by recursion that returns
a value and passes immutable context, not by a global cursor and flags.

```ada
--  wrong
Count : Natural := 0;
procedure Walk (N : Node) is
begin
   if N.Is_Match then
      Count := Count + 1;      --  global state
   end if;
   for C of N.Children loop
      Walk (C);
   end loop;
end Walk;
```

```ada
--  right
function Count_Matches (N : Node) return Natural is
   ((if N.Is_Match then 1 else 0)
    + [for C of N.Children => Count_Matches (C)]'Reduce ("+", 0));
```

## Types, contracts and architecture

**15. Subtypes over procedural validation.** Let the type system enforce
validity; do not write defensive `if ... then raise`.

```ada
--  wrong
procedure Check (X : Integer) is
begin
   if X < 0 then
      raise Constraint_Error;
   end if;
   --  use X
end Check;
```

```ada
--  right
subtype Index is Integer range 1 .. 100;
--  an out-of-range value cannot be built; no check needed
```

**16. Declarative predicates.** A type should refuse to hold invalid data.

```ada
--  wrong
procedure Set_Mode (M : Integer) is
begin
   if M not in 0 | 1 | 2 then
      raise Constraint_Error;
   end if;
   --  use M
end Set_Mode;
```

```ada
--  right
type Mode is new Integer with
   Static_Predicate => Mode in 0 | 1 | 2;
```

**17. `Pre` / `Post` conditions.** Push requirements to the subprogram boundary;
keep the body the transformation only.

```ada
--  wrong
function Pop (S : Stack) return Element is
begin
   if Is_Empty (S) then
      raise Constraint_Error;
   end if;
   --  ...
end Pop;
```

```ada
--  right
function Pop (S : Stack) return Element
   with Pre => not Is_Empty (S);
```

**18. Isolate side effects.** I/O, files and the network live at the edges; the
core is a pure `Data -> Transformation -> New_Data` pipeline.

```ada
--  wrong: the core does I/O
function Process (Data : Input_Data) return Output_Data is
begin
   Put_Line ("processing");        --  I/O in the core
   Write_Log (Data.Name);          --  side effect in the core
   return Transform (Data);
end Process;
```

```ada
--  right: the core is pure, the caller does I/O
function Process (Data : Input_Data) return Output_Data is
   (Transform (Data));

--  at the edge:
Put_Line (Image (Process (Data)));
```

**19. Extended return for a complex immutable object.** Build it in an extended
return; the caller then sees it as a constant.

```ada
--  wrong
function Make_Config return Config is
   C : Config := Null_Config;
begin
   C.Name := ...;
   C.Port := ...;
   return C;
end Make_Config;
```

```ada
--  right
function Make_Config return Config is
begin
   return C : Config do
      C.Name := ...;
      C.Port := ...;
   end return;
end Make_Config;
```

**20. Name intermediates over point-free density.** Readability and explicit
data flow trump brevity.

```ada
--  wrong: one dense pipeline
Total : constant Integer :=
   [for X of Values when X > 0 => X * 2]'Reduce ("+", 0);
```

```ada
--  right: named steps
Positive : constant Int_Array := [for X of Values when X > 0 => X];
Doubled  : constant Int_Array := [for X of Positive => X * 2];
Total    : constant Integer   := Doubled'Reduce ("+", 0);
```

## Immutable updates

Ada's functional style is not "make Ada look like OCaml." The language has
mechanisms that give the benefits associated with ML-style programming —
explicit data variants, total transformations, immutable values, parametric
abstraction, local reasoning, and explicit effects — while keeping nominal
typing, contracts, representation control, and a systems-programming model.
An `access function` is not the higher-order idiom; a generic formal subprogram
is.

**21. Array delta aggregates.** Record `with delta` has an array form. It is
the immutable indexed update. Array delta aggregates are one-dimensional only
(RM 4.3.4); a multi-dimensional update is still an assignment or a rebuild.

```ada
--  wrong
Next : Int_Array := Current;
Next (I) := Next (I) + 1;
```

```ada
--  right
Next : constant Int_Array := (Current with delta I => Current (I) + 1);
```

GNAT's `Current'Update (I => Current (I) + 1)` is an implementation-defined
attribute, not standard Ada. Prefer the delta aggregate. Both copy; neither
mutates `Current`.

**22. Target name `@`, only in an assignment.** `@` is legal only in the
expression of an assignment statement (RM 5.2.1). It is not legal in a
declaration, and a delta aggregate does not give it a special meaning of its
own. Use it when mutation is already the point. The immutable form names the
old value.

```ada
--  wrong: @ is not the base of a declaration
Next : constant State := (Old with delta Count => @.Count + 1);
```

```ada
--  right: immutable update names the source
Next : constant State :=
   (Old with delta
       Count => Old.Count + 1,
       Last  => Old.Last & Item);
```

```ada
--  right: @ only where an assignment is already justified
Node.Parent :=
   (@ with delta
       Count => @.Count + 1,
       Sum   => @.Sum + Value);
```

## Algebraic data

**23. Discriminated records are Ada's closest native equivalent to a sum
type.** The equivalence is not exact: Ada variants are nominal, the
discriminant is a real component, and recursive cases need an indirection.
What transfers is the discipline. One type, alternatives distinguished by a
discriminant, consumed by an exhaustive case expression.

```ada
type Node_Kind is (Leaf, Branch);

type Tree;
type Tree_Ref is access constant Tree;

type Tree (Kind : Node_Kind) is record
   case Kind is
      when Leaf   => Value : Integer;
      when Branch => Left, Right : Tree_Ref;
   end case;
end record;

function Sum (T : Tree) return Integer is
   (case T.Kind is
       when Leaf   => T.Value,
       when Branch => Sum (T.Left.all) + Sum (T.Right.all));
```

`Tree_Ref` is a shared pointer, not an owner. Do not treat it as a Rust box
or an OCaml implicit heap cell. A library-level tree allocated with `new` is
the simple case. A stack-allocated node cannot be designated by a
library-level access. SPARK code should use an arena index instead of
`access`.

**24. Option and Result, matched by case.** Absence and failure are values.
The core does not raise, and it does not use a sentinel. Bind is a case
expression, or a generic whose formal is the continuation. It is not an
`access function` parameter: an access-to-subprogram designates a subprogram
and does not carry an environment of captured locals.

```ada
type Option (Present : Boolean := False) is record
   case Present is
      when False => null;
      when True  => Value : Integer;
   end case;
end record;

function None return Option is ((Present => False));
function Just (V : Integer) return Option is ((Present => True, Value => V));  --  `Some` is reserved

function Map_Some (O : Option) return Option is
   (case O.Present is
       when False => None,
       when True  => Just (O.Value * 2));
```

```ada
type Result (Ok : Boolean := True) is record
   case Ok is
      when True  => Value : Integer;
      when False => Why  : Error_Kind;
   end case;
end record;

function Step (R : Result) return Result is
   (case R.Ok is
       when False => R,
       when True  => Compute (R.Value));
```

The generic continuation belongs with the functor pattern below, not with an
access parameter.

## Controlled construction

**25. Smart constructors enforce the invariant.** A private type whose
visible part contains an expression function cannot see the full view, and a
conversion written there does not establish the constraint. Declare the
constructor in the visible part. Complete it where the full type is visible.
The range is what rejects a bad value; a `Pre` only documents the same rule.

```ada
package Units is
   type Celsius is private;
   function C (Degrees : Float) return Celsius
      with Pre => Degrees >= -273.15;
   function To_Float (T : Celsius) return Float;
private
   type Celsius is new Float range -273.15 .. Float'Last;
end Units;
```

```ada
package body Units is
   function C (Degrees : Float) return Celsius is
      (Celsius (Degrees));          --  range check, Constraint_Error if not
   function To_Float (T : Celsius) return Float is
      (Float (T));
end Units;
```

An expression function may complete the visible declaration in the private
part, after the full type (AI12-0103). Prefer the body unless the package is
required to have no body. The point of the pattern is that an out-of-range
argument cannot become a `Celsius`, not that the type is spelled `private`.

## Higher-order abstraction

**26. Generics are the functor.** Ada has no first-class modules. A generic
package with a formal type and a formal subprogram is the architectural
equivalent that does exist: formal type plus formal operation, instantiation,
concrete abstraction. This is the higher-order idiom. Do not teach
`F : access function (...)` as the normal one.

```ada
generic
   type Element is private;
   with function Transform (X : Element) return Element;
package Mapped is
   type Vector is array (Positive range <>) of Element;
   function Apply (Source : Vector) return Vector;
end Mapped;
```

```ada
package body Mapped is
   function Apply (Source : Vector) return Vector is
      ([for X of Source => Transform (X)]);
end Mapped;
```

```ada
function Square (X : Integer) return Integer is (X * X);
package Int_Map is new Mapped (Integer, Square);
--  Int_Map.Apply (Values)
```

A generic function is the small form of the same idea. The formal array type
must be matched by an unconstrained actual with the same index subtype and
component type (RM 12.5.3). Named associations make that match readable.

```ada
generic
   type Element is private;
   type Index is (<>);
   type Vector is array (Index range <>) of Element;
   with function Transform (X : Element) return Element;
function Map (Source : Vector) return Vector;

function Map (Source : Vector) return Vector is
begin
   return [for X of Source => Transform (X)];
end Map;

type Int_Array is array (Positive range <>) of Integer;
function Square_All is new Map
   (Element   => Integer,
    Index     => Positive,
    Vector    => Int_Array,
    Transform => Square);
```

Filter is a comprehension at the call site, or a builder that returns a
container. A generic that returns a shorter unconstrained array is legal in
outline and sharp in the bounds rules; do not publish it without a GNAT run.

**27. Local expression functions are `let`.** A body is the right construct
when the transformation needs a named local function. Declare expressions
cannot hold subprograms (RM 4.5.9: constants and object renamings only). Keep
each local an expression function, and keep the body to one return.

```ada
function Normalize (X : Float) return Float is
   function Clamp (V : Float) return Float is
      (if V < 0.0 then 0.0 elsif V > 1.0 then 1.0 else V);
begin
   return Clamp (X / Scale);
end Normalize;
```

## Functional collections

**28. A functional list whose spine operations copy is not a functional
list.** `Tail` is only a structural operation if it is cheap. Do not introduce
an abstraction whose mathematical look hides a linear copy. Four
representations are different designs:

- a persistent linked list, `access constant`, shared tails, not SPARK
- a copying linked list, simple and usually the wrong default
- a vector plus offset, `Tail` advances the offset and shares storage
- a bounded or arena-indexed list, the SPARK representation

The API can still read as Nil and Cons. The representation has to be chosen
before the recursion is written.

```ada
function Is_Nil (L : List) return Boolean;
function Head (L : List) return Element with Pre => not Is_Nil (L);
function Tail (L : List) return List    with Pre => not Is_Nil (L);
function Cons (H : Element; T : List) return List;

--  only if Tail shares structure or advances an offset
function Length (L : List) return Natural is
   (if Is_Nil (L) then 0 else 1 + Length (Tail (L)));
```

If `Tail` copies, fold with `'Reduce` or walk an index. Recursive `Map` on a
copying spine is quadratic.

**29. Persistent-style builders, not standard maps.** `Ada.Containers`
maps and vectors are imperative. A constant initialized from an aggregate
does not give the container persistent update semantics, and a comprehension
that rebuilds an `Ordered_Maps.Map` is not the natural operation. Small pure
tables are association lists. Large tables are mutated in place, under the
same rule as a cursor or a buffer.

```ada
type Pair is record
   Name : Unbounded_String;
   N    : Integer;
end record;

type Table is array (Positive range <>) of Pair;

function With_Score (T : Table; Name : String; N : Integer) return Table is
   (declare
       Kept : constant Table :=
          [for P of T when To_String (P.Name) /= Name => P];
    begin
       Kept & Pair'(To_Unbounded_String (Name), N));
```

That builder copies. It is the right shape for a small pure table and the
wrong shape for a large one. For a large one, `Insert` on an `in out` map is
the honest operation, not a persistent metaphor laid over
`Ada.Containers`.

## Contracts and effects

**30. `Global => null` is a SPARK contract, not an effect system.** It says
the subprogram has no global variable inputs or outputs. Ada does not track
effects the way an ML or Haskell effect system does. `pragma Pure` is a
library-unit annotation: the unit declares no library-level state and depends
only on pure units. It is not a per-subprogram marker.

```ada
function Double (X : Integer) return Integer
   with Global => null;
```

```ada
pragma Pure (Transforms);
```

Membership choice lists are the pattern guard. They need no separate matching
language, and they compose with the quantified expressions already in the
guide. A discriminant-dependent component like `Value` is present only in some
variants, so read it inside a case expression, which narrows the discriminant
instead of leaving a runtime discriminant check.

```ada
function Is_Stop (K : Color) return Boolean is
   (K in Red | Yellow);

All_Safe : constant Boolean :=
   (for all N of Nodes =>
      (case N.Kind is
          when Leaf   => N.Value >= 0,
          when others => True));
```

`Contract_Cases` is the SPARK form of a function specified by clauses. Use it
when the postcondition splits on the input, not as a second body.

```ada
function Abs_Val (X : Integer) return Integer
   with Global => null,
        Contract_Cases =>
          (X >= 0 => Abs_Val'Result = X,
           X < 0  => Abs_Val'Result = -X);
```

## Beyond these rules

**Compiler switches are your linter.** Build strict, and treat warnings as
errors in development.

```ada
package Compiler is
   for Default_Switches ("Ada") use
     ("-gnat2022", "-gnatwa", "-gnatVa", "-gnato", "-gnatwe");
end Compiler;
```

`-gnatwe` is right for development but brittle in a package rebuilt against
each new GNAT, so a shipped package may leave it out.

**SPARK for core logic.** A package with no side effects and no hidden state can
be marked, and SPARK then refuses any accidental aliasing, side effect or
uninitialized state.

```ada
pragma SPARK_Mode (On);
```

**`Unbounded_String` composed functionally.** Return and compose with `&`, never
a mutable buffer with `Append`.

```ada
--  wrong
Buffer : Unbounded_String := Null_Unbounded_String;
Append (Buffer, "prefix:");
Append (Buffer, Value);
```

```ada
--  right
Result : constant Unbounded_String := To_Unbounded_String ("prefix:") & Value;
```

**Short-circuit `and then` / `or else`, always.** Never plain `and` / `or` for
boolean logic: the second operand may raise or cost needlessly.

```ada
--  wrong: 10 / X is evaluated even when X = 0
if X /= 0 and 10 / X > 2 then
   ...
end if;
```

```ada
--  right
if X /= 0 and then 10 / X > 2 then
   ...
end if;
```
