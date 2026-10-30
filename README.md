# Project 2: Build Your Own Vec

**Author:** Tamok Nguyen



### Getting started

You will build Rust's `Vec` yourself, twice. First a slow version that is safe and simple, then a fast one that manages its own memory with raw pointers. Then you will time the versions against each other.

Each vector has a playground you can run and edit freely. Run these from this top folder:

```bash
cargo run --bin slow_playground   # try out SlowVec and FixedSizeArray
cargo run --bin slow_memory       # watch what gets dropped, and when
cargo run --bin fast_playground   # try out FastVec, MALLOC, and raw pointers
cargo run --bin fast_memory       # watch what gets dropped and freed, and when
```

All four crash with `not yet implemented` until you write the functions they call.

### The folders

```
slow_vec/    Part 1. Your SlowVec goes in src/lib.rs
fast_vec/    Part 2. Your FastVec goes in src/lib.rs, plus src/growth.rs and src/timing.rs
             src/benchmark.rs runs the benchmark and draws the plots
fixed/       FixedSizeArray, which SlowVec is built on. Read it if you are curious
malloc/      the memory allocator FastVec uses. It records every allocation and free
tracker/     wraps values so you can see which ones have been dropped
```

**Change only what the handout asks for**: the functions marked `todo!`, the `Drop` for `FastVec`, your choices in `benchmark.rs`, and the writeup at the bottom of this file. Leave everything else alone.

### About the warnings

A fresh clone builds with a lot of warnings, all of them some version of "you declared this and never used it." That is expected, since you haven't written the code yet.

### Running your tests

There is one test file per checkpoint. Run them from this top folder:

```bash
cargo test --test cp1     # SlowVec
cargo test --test cp2     # FastVec's get and push, and growth
cargo test --test cp3     # remove, clear, Drop, and timing
cargo test                # all of them
```

Add part of a test's name to run just that one: `cargo test --test cp2 push_memory`.

**If a run stops partway with `SIGSEGV`, `SIGBUS`, `SIGABRT`, or some other signal**, one of your `unsafe` blocks has corrupted memory, and it took the whole test run down with it. The last test named before the crash is the place to start. Try running it on its own to diagnose the problem.

### The benchmark

```bash
cargo run --release --bin benchmark
```

Remember, the `--release` matters: without it the timings are slow and misleading. This writes `benchmark_slow.png` and `benchmark_growth.png` into the top folder, which you'll need for the writeup. Commit both: they have to be on `main` when you submit.

## Your writeup

Answer the questions from the project handout here, under the headings below. Leave the headings where they are, and leave everything above this line alone. Answers should be about 2-3 sentences per question unless the handout says otherwise.

### 1. SlowVec against FastVec
Looking at my graph, SlowVec gets slower pretty quickly as more elements are added, while FastVec stays almost flat. At 5,000 elements, SlowVec took around 700 ms, while FastVec was barely visible on the graph. This is because SlowVec has to move all the previous elements every time it pushes, so the total work is 1 + 2 + ... + n = n(n+1)/2, which is O(n²).

### 2. Why push is fast on average
For 16 pushes, Double takes 31 steps total: 16 for adding the elements and 15 for moving elements during resizing (1 + 2 + 4 + 8). Even though some pushes take longer because the vector has to resize, it doesn't happen that often since the capacity doubles each time. So overall, the average amount of work per push stays about the same, which is O(1).

### 3. Your growth experiments
I tried Multiply(1.5), Multiply(2.5), and Multiply(3.0) on growth-multiply, and Add(100), Add(10000), and Add(100000) on growth-add. From my graphs, Multiply(3.0) was the fastest in the first experiment at around 1.94 ms, while Add(100) was really slow at around 1,320 ms. I decided to merge growth-multiply because it showed the differences better, but I also noticed that bigger growth factors leave more unused space. For example, multiplying by 3 leaves about two-thirds of the space empty right after resizing, while doubling leaves about half.

### 4. Why `Add` eventually loses
Add(k) only adds the same number of slots every time, so when the vector gets bigger, it still has to resize pretty often and move more and more elements. Using a bigger k makes it faster because it doesn't resize as much, but it doesn't really fix the problem as n keeps growing. That's why Add(k) is still O(n²) overall, while Double is O(n).

### 5. Two kinds of leak
Tracker checks if the elements themselves are getting dropped, while MALLOC checks if the memory used by the vector is actually freed. For example, if Drop only freed the memory without dropping the elements, MALLOC would say everything is fine, but Tracker would catch the missing drops. It could also happen the other way around if the elements were dropped but the memory block wasn't freed.

### 6. The bug in `clear`
The problem with clear was that it freed the memory but forgot to drop the elements inside first. I fixed it by using ptr::drop_in_place on each element before freeing the memory. SlowVec didn't have this problem because Rust automatically handles dropping the elements in FixedSizeArray.

### 7. Something that failed at first, and how you adapted
One thing I messed up at first was cloning the professor's GitHub repository instead of making my own copy. When I tried pushing my code, I got a permission error and realized I was working with the wrong repository. I ended up creating my own private repo and starting again, which helped me understand the Git branch process better. I also had some syntax errors with missing brackets that I fixed by checking the errors in VS Code

### AI citation
I used ChatGPT to help me write and debug some of the Rust functions, especially the pointer and memory management parts. I also used it when I got stuck with Git commands, setting up branches, and understanding the benchmark graphs