# Memory Layout in ECS

An [Entity Component System][wiki_ecs] is an architectural pattern commonly used
by games and intensive data processing programs. At its core, ECS is about
organizing and operating on data in a way which is optimized for today's
hardware. The architecture is the prime example for "data oriented design".

The common refrain is to think "array of structs" vs. "struct of arrays". At a
high level, most programs use "array of structs"; ECS uses "struct of arrays".

## Memory Layout

Reading from and writing to memory is perhaps the greatest bottleneck in
performance for today's machines. Adding 2 numbers together can happen in a
single CPU cycle. Fetching the two number values from RAM may take hundreds of
CPU cycles.

How data is laid out in memory has an impact on how often CPUs fetch from RAM.

### Array of Structs

Imagine there is a `Monster` type. The `Monster` has a `position`, a `velocity`,
and `health`. The type and fields could be defined like:

```rust,noplayground
struct Position {
  x: i32,
  y: i32,
}

struct Velocity {
  x: i32,
  y: i32,
}

struct Monster {
  pos: Position,
  vel: Velocity,
  health: u32,
}

struct Game {
  monsters: Vec<Monster>,
}
```

Assuming a `Monster` is an `Entity`, then the `Position`, `Velocity`, and
`Health` would each be a `Component`.

If there are multiple `Monster`s, then it is natural to store the data in a
collection like an array where instances are contiguously stored:

```text
┌────────────────────┬────────────────────┬────────────────────┐
│                    │                    │                    │
│      Instance      │      Instance      │      Instance      │
│                    │                    │                    │
└────────────────────┴────────────────────┴────────────────────┘
```

More specifically, a single instance's fields' values are laid out in memory,
then followed by all of the next instance's fields' values, and so forth:

```text
┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
│   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │
│ P │ P │ V │ V │ H │ P │ P │ V │ V │ H │ P │ P │ V │ V │ H │
│ o │ o │ e │ e │ e │ o │ o │ e │ e │ e │ o │ o │ e │ e │ e │
│ s │ s │ l │ l │ a │ s │ s │ l │ l │ a │ s │ s │ l │ l │ a │
│   │   │   │   │ l │   │   │   │   │ l │   │   │   │   │ l │
│ X │ Y │ X │ Y │ t │ X │ Y │ X │ Y │ t │ X │ Y │ X │ Y │ t │
│   │   │   │   │ h │   │   │   │   │ h │   │   │   │   │ h │
└───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

### Struct of Arrays

ECS changes the data layout to be defined like:

```rust,noplayground
struct Position {
  x: i32,
  y: i32,
}

struct Velocity {
  x: i32,
  y: i32,
}

struct Monsters {
  positions: Vec<Position>,
  velocities: Vec<Velocity>,
  health: Vec<u32>,
}

struct Game {
  monsters: Monsters,
}
```

While the `Position`, `Velocity`, and `Health` components still exist, the
`Monster` entity is more conceptual. In ECS systems, an `Entity` is treated
like an index into the various component arrays. In order to read all of the
first `Monster`'s properties, each array would need to be read.

The data could be laid out in memory like:

```text
┌───┬───┬───┬───┬───┬───┬───────┬───┬───┬───┬───┬───┬───┬─────┬───┬───┬───┐
│   │   │   │   │   │   │       │   │   │   │   │   │   │     │   │   │   │
│ P │ P │ P │ P │ P │ P │       │ V │ V │ V │ V │ V │ V │     │ H │ H │ H │
│ o │ o │ o │ o │ o │ o │       │ e │ e │ e │ e │ e │ e │     │ e │ e │ e │
│ s │ s │ s │ s │ s │ s │ . . . │ l │ l │ l │ l │ l │ l │ ... │ a │ a │ a │
│   │   │   │   │   │   │       │   │   │   │   │   │   │     │ l │ l │ l │
│ X │ Y │ X │ Y │ X │ Y │       │ X │ Y │ X │ Y │ X │ Y │     │ t │ t │ t │
│   │   │   │   │   │   │       │   │   │   │   │   │   │     │ h │ h │ h │
└───┴───┴───┴───┴───┴───┴───────┴───┴───┴───┴───┴───┴───┴─────┴───┴───┴───┘
```

The `...` represent memory which may be occuppied by other unrelated data.

### Why It Matters

The differences in memory layouts are important when processing data.
Specifically, a program is able to take better advantage of [cache
locality][wiki_locality_of_ref] when operating with an ECS architecture.

CPUs have various levels of on-die caches (L1, L2, L3, etc.) to effectively
speed up accessing memory. When a memory location is read and the value is not
found in any of the CPU caches, the value is fetched from RAM and is then
inserted into the CPU caches before continuing with the program. Fetching a
value from RAM takes a relatively significant amount of time compared to reading
directly from a CPU cache. While the CPU is waiting for the value, it can be
effectively stalled and eventually cannot proceed with execution if the memory
value is not available.

While assembly instructions may only read a single memory address at a time, the
CPU actually fetches "a cache line" of memory. Basically, the CPU fetches many
values around the requested memory address and puts all of the values in the CPU
caches. If memory is read at an address, there is a high probability that
adjacent memory addresses will be read soon (a.k.a. spatial locality). The CPU
is trying to predictively fill the CPU caches, avoid the latency with
additional fetches from RAM, and avoid stalling execution.

Imagine there is a function which needs to read all the positions of every
`Monster`. Under the common "array of structs", the CPU is told to fetch a
`Monster`'s position's X coordinate. The Y coordinate will likely also be
fetched since the CPU is filling the cache with adjacent memory values.
Prefetching the Y coordinate into the CPU cache is great since it is highly
likely the value will be needed.

The CPU will probably grab even more data in the same fetch, so the `Monster`'s
velocity and health may be put in the cache as well. Maybe even the next
`Monster`'s values or even more data depending on how big a "cache line" is.

Unfortunately, caches are not infinite in size. In reality, they are tiny
compared to the amount of physical RAM in most machines. So while the CPU may
put all of the `Monster`s data into the CPU cache, the function may never need
the velocity or the health values. The cache is partially filled with data which
is not currently needed. Eventually if enough memory is read, the cache may
evict the `Monster`s data and may have never read the velocity or health from
the cache.

With "struct of arrays", all of the `Monster`s position data for each `Monster`
is located together continuously. When a position is read in, the CPU is only
prefetching position data into the cache. For a single fetch, the total amount
of data prefetched remains the same, but now, the amount of useful position data
may have increased dramatically.

For example, suppose 128 bytes can be fetched at a time. In the "array of
structs" case, a single fetch would retrieve a Monster's `Position` and
`Velocity`. To read another `Monster`'s position, another fetch would have to be
made.

In the "struct of arrays" case, a single fetch would load 2 Monster's
`Position` values. The program could read the next `Monster`'s position from the
cache and would not have to make a fetch.

In this very simplified example, the "struct of arrays" case could require only
half the fetches from memory compared to the "array of structs" case.

The program is still relying on spatial locality, but it is more effectively
taking advantage of the memory prefetching by only loading relevant data.

## Conclusion

ECS takes advantage of modern CPU behavior with memory prefetching, specifically
with `Component` data being organized in a "struct of arrays". Furthermore, by
separating out the `Component` data, there are easier parallel processing
opportunities.

Of course, a disadvantage is if multiple properties of a single entity were
needed, then the data would have to be read from multiple arrays. It may be
difficult to debug and keep track of all of a single entity's properties.

In regards to Rust, many of the Rust game engines, such as [Bevy][bevy], use
ECS. The closing keynote for [RustConf
2018][youtube_rustconf_2018_closing_keynote] talked about ECS. My takeaway is to
keep an eye out for different architectures, designs, and patterns because their
ideas might lead to surprising results.

ECS has existed before Rust and is perhaps relevant to only some programs, but
it is interesting because it completely inverts a program's layout and takes
advantage of the hardware's assumptions to achieve a high level of performance.

[wiki_ecs]: https://en.wikipedia.org/wiki/Entity_component_system
[wiki_locality_of_ref]: https://en.wikipedia.org/wiki/Locality_of_reference
[bevy]: https://bevyengine.org/
[youtube_rustconf_2018_closing_keynote]: https://www.youtube.com/watch?v=aKLntZcp27M
