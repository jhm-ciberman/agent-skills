---
name: csharp-docblock-writing
description: "Review and fix the quality of C# XMLDoc (`///`) and `//` comments on changed code. Use after coding a C# feature to strip overexplaining, keep <summary> minimal, move implementation detail out of the contract, fully document <param>/<returns>/<exception>, and stop docblock churn. Quality only - it does not hunt for correctness bugs."
---

# C# Docblock Writing

You just changed C# code. Do a focused pass over the `///` and `//` in the diff
and fix the docs. Quality only, not a bug hunt. Don't touch behavior.

Your default failure mode: you overexplain, leak the *how* into `<summary>`,
skip `<param>`/`<returns>`, and reword stable docs. Fix that here.

**The test for every sentence:** would deleting it change what a *caller* needs
to know? No -> it's implementation, cut it.

## Phase 0 - Gather the diff

`git diff @{upstream}...HEAD` (fall back to `git diff main...HEAD`,
`git diff HEAD~1`, or `git diff HEAD` for uncommitted work). Scope: every `///`
and `//` the diff added or changed, plus the docs on any public/protected member
it touched. If an arg named a file, scope to that.

## Phase 1 - Review

Walk every docblock and `//` comment in scope. Check each against the rules below
and note what's wrong: implementation leaked into `<summary>`, missing
`<param>`/`<returns>`/`<exception>`, overpromised guarantees, a stable doc reworded
for no reason, or an undocumented public/protected member.

## Phase 2 - Apply

Fix each issue in the file. Skip anything that would change behavior, and say so.

## Phase 3 - Re-read your own work

Re-read every docblock you just wrote or changed and interrogate it:

- Am I leaking implementation the caller doesn't need (the how, the internals, the
  collaborators it calls, the algorithm)?
- Is this really a **contract** a caller relies on, or did I just describe the body?
- Did I overpromise a guarantee the code doesn't actually make?
- Did I reword a stable docblock whose contract didn't change? Revert it.
- Did I document every `<param>`, `<returns>`, `<exception>`, `<typeparam>`?
- Could the `<summary>` lose a sentence and still hold?

Fix anything that fails. This pass exists because the act of "helpfully"
documenting is exactly when you reintroduce the leaks Phase 1 just removed.

Then summarize by file: what you cut, which `<param>`/`<returns>`/`<exception>` you
added, which missing docs you added. Call out any docblock you **left untouched on
purpose** because its contract was unchanged.

> Important: If after applying Phase 0 to 3, the diff is zero, that's not an error. That means
> that the code was already well-documented. Don't add or modify things if you don't have to.

---

The rest is the rule reference Phase 1 checks against.

## Coverage

- Public/protected members **must** be documented. Touched an undocumented one?
  Add the doc now.
- Around 90% of private methods get none (the `[RelayCommand]` case below is the
  exception).

## Don't invent a contract you can't verify

Coverage says document the public member you touched. It does not say guess. If the
code doesn't tell you what the member guarantees, a confident, plausible doc is
worse than none: it now lies to every caller. Leave it undocumented and raise it in
your Phase 3 summary, or ask. Never fill the gap with the likely answer.

Remember you can read more files until you "find" or understand the contract.
Guessing is not an option.

## Format: `<summary>` on its own lines

```csharp
// BAD - single line. Only OK for enum cases or other dense spots.
/// <summary>Restocks an item.</summary>

// GOOD - members get the block form, even for one short sentence
/// <summary>
/// Restocks an item.
/// </summary>
```

General rule: `<param>`, `<returns>`, `<typeparam>`, `<exception>`, `<inheritdoc/>` stay on
one line. Only `<summary>` (and a short `<remarks>`) takes the block form.
Of course, if you need to describe something genuinely complex, you can break this rule and use multiple lines for your `<param>`,
`<returns>`, `<throws>`, etc. Just don't do it by default.

## Stems (use the wording exactly, in the block form above)

```text
Property (get/set):   Gets or sets the maximum stock capacity.
Getter-only:          Gets the kind of market stall.
Setter-only:          Sets the baseline mood.
Bool property:        Gets a value indicating whether the stall is sold out.
Constructor (class):  Initializes a new instance of the <see cref="MarketStall"/> class.
Constructor (struct): Initializes a new instance of the <see cref="GridCoord"/> struct.
Constructor (multiple overloads): Initializes a new instance of the <see cref="MarketStall"/> class with the specified kind and initial stock.
Event:                Occurs when the stock changes.
```

Type and method summaries: any plain verb that fits. The codebase uses
`Represents`, `Provides`, `A <noun> for ...` for types, and `Gets`, `Returns`,
`Tries to`, `Attempts to`, `Checks if`, `Determines whether`, `Calculates`,
`Performs`, `Sets`, `Restocks`, ... for methods. Don't force a fixed set.

When a summary refers to a parameter, **the given X** is the phrasing you'll see
most: "Checks if the given villager can interact", "Gets the material for the
given texture", "the given `<see cref="Character"/>`". Alternatives like "the
specified" or "the passed-in" are fine too, this is just the common one.

## Keep `<summary>` minimal, no implementation

```csharp
// BAD - how it works, the caller never cares it's A*
/// <summary>
/// Runs A* over the nav grid and caches the result until terrain changes.
/// </summary>
public bool TryPathfind(Tile start, Tile end, out Path<Tile> path)

// GOOD - the effect
/// <summary>
/// Tries to find a path from the given start node to the goal node.
/// </summary>
public bool TryPathfind(Tile start, Tile end, out Path<Tile> path)
```

```csharp
// BAD - voice is the internals
/// <summary>
/// Sets the _claimedAt field to the current tick.
/// </summary>

// GOOD - voice is the effect
/// <summary>
/// Marks the reward as claimed.
/// </summary>
```

## Don't overpromise, but state real invariants

```csharp
// BAD - promises a clamp that isn't guaranteed, and leaks the death cascade
/// <summary>
/// Gets or sets health, a value from 0 to 100. At 0 the character dies, OnDeath
/// fires, and the body is removed next tick.
/// </summary>
public int Health { get; set; }

// OK - minimal, promises nothing the setter doesn't guarantee
/// <summary>
/// Gets or sets the character's current health.
/// </summary>
public int Health { get; set; }

// BEST - the range is a genuine guarantee here, so state it
/// <summary>
/// Gets the current water level of the well, from 0 to 100.
/// </summary>
public int WaterLevel { get; private set; } = 100;
```

## Document `<param>`, `<returns>`, `<typeparam>`, `<exception>`

```csharp
// GOOD - every param, a returns that names the real outcome, the exception it throws
/// <summary>
/// Sets the cell value at the specified coordinate.
/// </summary>
/// <param name="coord">The coordinate of the cell to set.</param>
/// <param name="value">The value to set the cell to.</param>
/// <exception cref="ArgumentOutOfRangeException">Thrown if the coordinate is out of bounds.</exception>
public void Set(GridCoord coord, T value)
```

```csharp
// GOOD - generic type documents its parameter
/// <summary>
/// Represents a grid of cells for spatial queries and pathfinding.
/// </summary>
/// <typeparam name="T">The type of data stored in each cell.</typeparam>
public class SpatialGrid<T>
```

## Linking: `<see cref>`, `<paramref>`, `<see langword>`

```csharp
/// <summary>
/// Gets the <see cref="CharacterView"/> for the given <paramref name="character"/>,
/// or <see langword="null"/> if none is registered.
/// </summary>
public CharacterView? GetView(Character character)
```

- `<see cref="X"/>` links a type or member.
- `<paramref name="x"/>` refers to a parameter in prose, not backticks or a bare name.
- `<see langword="null"/>` (also `true`, `false`) for language keywords. However, using null, True or False is also perfectly fine, and often more readable. Use langword when the keyword might be ambiguous in the prose, or when you want to emphasize the language aspect.

## Contract the signature doesn't show

The cases where the real contract is invisible from the signature, so the doc has
to carry it.

Async: the standard `<returns>` shape, with the task result spelled out.

```csharp
/// <summary>
/// Starts patrolling along the route.
/// </summary>
/// <param name="cancellationToken">Token to cancel the patrol.</param>
/// <returns>A task that represents the asynchronous patrol. The result is true if the patrol completed, false if it was canceled or a move failed.</returns>
public async Task<bool> PatrolAsync(CancellationToken cancellationToken)
```

A method that appends to a caller's list without clearing it: invisible, so state
it in `<remarks>`.

```csharp
/// <summary>
/// Collects the remaining cells into the provided buffer, in order from the current target.
/// </summary>
/// <remarks>The buffer is NOT cleared first.</remarks>
/// <param name="buffer">The list the remaining cells are added to.</param>
public void CollectRemainingCells(List<Vector2Int> buffer)
```

An ordinary `Dispose` frees internal resources. Don't enumerate them, that leaks
the implementation. The caller only needs to know it must be called, so
`<inheritdoc/>` is best and a generic "Releases all resources." is fine.

```csharp
// BAD - leaks the implementation (which listeners, which textures)
/// <summary>
/// Releases the event listeners and frees the render texture.
/// </summary>
public void Dispose()

// GOOD - the caller only needs to know it must be called
/// <summary>
/// Releases all resources.
/// </summary>
public void Dispose()

// BETTER - it's just the standard IDisposable contract
/// <inheritdoc/>
public void Dispose()
```

When `Dispose` is really a lifetime handoff (a token, a pool lease, a scope), name
that specific effect instead.

```csharp
/// <summary>
/// Returns this object to the pool.
/// </summary>
public void Dispose()
```

Nullable-annotation attributes (`[NotNullWhen(true)]`, etc.) are compiler plumbing.
Describe the null contract in prose, never name the attribute.

```csharp
/// <param name="target">The active target, or null if the agent isn't patrolling.</param>
/// <returns>True if an active target was returned, false otherwise.</returns>
public bool TryGetActiveTarget([NotNullWhen(true)] out PatrolTarget? target)
```

Protected virtual extension points: documented for the overrider, usually "Called when X".

```csharp
/// <summary>
/// Called when the agent reaches a patrol target.
/// </summary>
/// <param name="target">The target that was reached.</param>
protected virtual void OnTargetReached(PatrolTarget target)
{
    // Override in subclasses
}
```

## Boolean `<returns>`: comma form, name the condition

```csharp
// BAD - generated boilerplate, semicolon template, just echoes the method name
/// <returns>True if valid; otherwise, false.</returns>

// GOOD - comma form, names what true actually means
/// <returns>True if the item can be afforded with the given coins, false otherwise.</returns>
/// <returns>True if a path was found, false otherwise.</returns>
```

## Enums: type names the role, every case documented

```csharp
// GOOD - block form on the type, single-line summaries on the cases
/// <summary>
/// Specifies the kind of market stall.
/// </summary>
public enum StallKind
{
    /// <summary>A bakery stall.</summary>
    Bakery,

    /// <summary>A butcher stall.</summary>
    Butcher,
}

// BAD - the type summary lists the cases, so it rots the moment the enum grows
/// <summary>
/// The kind of stall. Can be Bakery, Butcher, Blacksmith, or Apothecary.
/// </summary>
public enum StallKind
```

```csharp
// GOOD - [Flags]: the summary carries the combination rule, None and All name their role
/// <summary>
/// Specifies which wall edges a cell must be attached to. All listed edges are required.
/// </summary>
[Flags]
public enum Walls
{
    /// <summary>No wall attachment required.</summary>
    None = 0,

    /// <summary>A wall borders the north edge.</summary>
    North = 1 << 0,

    /// <summary>A wall borders the east edge.</summary>
    East = 1 << 1,

    /// <summary>All walls border the cell.</summary>
    All = North | East | South | West
}
```

## Events: `Occurs when X`

```csharp
// GOOD
/// <summary>
/// Occurs when the stock of an item changes.
/// </summary>
public event EventHandler<StockChangedEventArgs>? StockChanged;
```

## `<inheritdoc/>` for interface/override implementations

```csharp
/// <summary>
/// A simple well that provides water.
/// </summary>
public class Well : IInteractable
{
    // GOOD - the contract is the interface's, even though the body has logic
    /// <inheritdoc/>
    public bool CanInteract(Villager villager)
    {
        return villager.HasFreeHands && this.WaterLevel > 0;
    }
}

// BAD - re-documents the same contract, free to drift from the interface
/// <summary>
/// Determines whether the villager can interact with this object.
/// </summary>
public bool CanInteract(Villager villager)
```

## Terse by default, a second sentence only when it earns it

```csharp
// BAD - padded restatement of the body
/// <summary>
/// This method restocks the specified item by adding the given quantity to the
/// stall's internal stock dictionary, capped at the maximum stock capacity.
/// </summary>

// GOOD - simple method, simple doc
/// <summary>
/// Restocks an item.
/// </summary>
```

```csharp
// GOOD - second sentence teaches a non-obvious concept
/// <summary>
/// Calculates the Manhattan distance from this coordinate to another.
/// This is the distance when you can only move orthogonally, like cars in a city grid.
/// </summary>

// GOOD - second sentence states a real caveat
/// <summary>
/// Gets the orthogonal neighbors of the coordinate. Only neighbors inside the grid are returned.
/// </summary>
```

```csharp
// GOOD - a property whose contract genuinely includes null + mutation semantics earns the words
/// <summary>
/// Gets the active house lot. Always reflects <see cref="World.CurrentHouseLot"/>, which can
/// change while the mode runs, and is <see langword="null"/> when the player is between lots.
/// </summary>
public HouseLot? HouseLot => GameViewModel.CurrentWorld.CurrentHouseLot;
```

## Private methods: usually none. `[RelayCommand]` is the exception

```csharp
// GOOD - ordinary private helper, no doc
private int IndexOf(GridCoord coord)
{
    return coord.Y * this.Width + coord.X;
}

// GOOD - a private [RelayCommand] generates a public command property, so document it
/// <summary>
/// Sells the selected item to the active stall.
/// </summary>
/// <param name="item">The item to sell.</param>
[RelayCommand]
private void SellItem(Item item)
```

## `//` comments: WHY, never WHAT

```csharp
// BAD - restates the line (junior-dev WHAT)
this.InitializeConsole(); // initializes the console

// BAD - references the edit / a past state, already rotting
// now also handles diagonal tiles, added in the pathfinding refactor

// GOOD - non-obvious WHY, timeless
weight *= 2.8f; // heavily penalize moving against the slope
```

Keep a lone `//` in an empty body (deliberate "left blank"). Keep the author's
playful comments verbatim. Never fake a docblock with a `//` above a signature.

## Complex and low-level code

Dense geometry/mesh code is the one place terse WHY comments earn their keep:
lowercase, about intent rather than what the line does, with trailing comments to
explain magic numbers.

```csharp
int connectionVerts = 2 * polygon.NumHoles; // 2 verts are added when joining a hole to the hull
weight *= 2.8f;                              // heavily penalize moving against the slope

// at least one point must be right of the bridge point for the ray to intersect
if (p0.X > bridge.X || p1.X > bridge.X)
```

Unsafe and hot-path code documents the contract, never the marshalling or the
fixed/pointer work.

```csharp
/// <summary>
/// Writes a value to the buffer.
/// </summary>
/// <typeparam name="T">The type of the value to write.</typeparam>
/// <param name="offset">The offset to write to.</param>
/// <param name="data">The value to write.</param>
[MethodImpl(MethodImplOptions.AggressiveInlining)]
public void Write<T>(int offset, ref T data) where T : unmanaged
```

A type whose whole identity is an algorithm may name it, with the reference in
`<remarks>` rather than `<summary>`. An algorithm chosen inside a method stays out
of its `<summary>` (the A* / `TryPathfind` case above).

```csharp
/// <summary>
/// Triangulates a polygon using the ear-clipping algorithm.
/// </summary>
/// <remarks>Based on https://www.geometrictools.com/Documentation/TriangulationByEarClipping.pdf</remarks>
public class Triangulator
```

## Don't churn stable docblocks

The rule you break most. When you refactor a member:

1. Read the original docblock as a **contract** (what a caller relies on).
2. Did the contract change?
   - **Yes** -> update the doc.
   - **No, only the implementation changed** -> keep the doc byte for byte.
   - **No, but it has a typo/prose error** -> fix only that, keep the contract.
3. Tempted to edit the doc to describe your code change? Don't. The doc
   describes the contract, not your edit.

A refactor turns `TotalWeight` from a stored field into a computed property. The
contract is unchanged, so the doc stays byte for byte (the `cref` link included).

```csharp
// BEFORE
/// <summary>
/// Gets the total weight the <see cref="Inventory"/> currently holds.
/// </summary>
public float TotalWeight { get; private set; }

// AFTER
/// <summary>
/// Gets the total weight the <see cref="Inventory"/> currently holds.
/// </summary>
public float TotalWeight => this.Items.Sum(i => i.Weight);
```
