# Efficient Productivity

Performance has always been important to some degree. Whether it be on the order
of years to seconds, there is some form of an acceptable performance goal.
In the last 15 years, efficient productivity has become the more nuanced
goal as programs are now running in datacenters, mobile devices, and embedded
devices,

An example of inefficient and unproductive code is endlessly polling in an
infinite loop when there is no action to take. For instance:

```rust,norun
use std::time::{Duration, Instant};

let deadline = Instant::now() + Duration::from_secs(60 * 60);
loop {
  let now = Instant::now();
  if deadline <= now {
    break;
  }
}
```

The above task could be more efficiently accomplished by registering a callback
to a runtime or even, at the worst case, causing the thread to sleep for a
period of time and then only occasionally checking. As written, the above code
will cause the CPU to spin and do a significant amount of useless work.

Efficient productivity does not mean having the best performance. Mobile devices
are relatively powerful from a processing capability point of view, but battery
usage is a concern for the user. In some cases, battery life is conserved by
programs which are able to quickly perform computations. However, in other
cases, programs can opt for less performant behavior but improve the overall
experience for the end user.
