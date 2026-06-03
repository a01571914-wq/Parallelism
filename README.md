# Parallelism

# Activity 5.2 Parallel and concurrent programming

## Programs and how to compile and run

### C Programs

```bash
# Sum of primes
gcc prime_sum_seq.c -o prime_sum_seq
gcc prime_sum_parallel.c -o prime_sum_parallel

# Numerical integration (pi)
gcc pi_seq.c -o pi_seq
gcc pi_parallel.c -o pi_parallel
```

**Running:**

```bash
# Sequential — one argument: n
./prime_sum_seq 1000000
./pi_seq 1000000

# Parallel — two arguments: n and number of threads
./primes_sum_parallel 1000000 4
./pi_parallel 1000000 4
- **Time** — wall clock time (real elapsed time)
- **CPU** — total CPU time across all cores

---

### Elixir Programs

```bash
iex prime_sum_sequential.exs
iex prime_sum_parallel.exs
iex pi_sequential.exs
iex pi_parallel.exs
```

**Then call functions directly from the iex shell:**

```elixir
# Sum of primes
Primes.measure_time(&Primes.sum_primes/1, [1_000_000])
Primes.measure_time(&Primes.parallel_sum_primes/2, [1_000_000, 4])

# Numerical integration (pi)
Pi.measure_time(&Pi.compute_pi/1, [1_000_000])
Pi.measure_time(&Pi.parallel_compute_pi/2, [1_000_000, 4])
```

---

## Parallelization Process

### Problem 1 — Sum of Primes


What i did was to divide the range `[2, n]` into equal chunks and assign each chunk to a thread/task. Each worker independently checks primality and accumulates a local sum. At the end, all local sums are added into a single shared result.
In C, i used a `pthread_mutex` that is used to protect the shared variable when each thread writes its local result. In Elixir it handles the concurrency and the results are collected with `Enum.sum`.

### Problem 2 — Numerical Integration (π)

For this problem i divided the n rectangles into equal ranges and assign each range to a thread/task. Each worker computes the sum of heights in its range. The final result multiplies the total sum by the width `1/n`.The same mutex/Task patterns that i used in primes also was used in this problem.

---

## Speedup Evaluation

Tests run, n = 1,000,000.

### Sum of Primes — C

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 1.623 | 1.00x |
| 2 | 0.834 | 1.95x |
| 4 | 0.412 | 3.94x |
| 8 | 0.389 | 4.17x |

### Sum of Primes — Elixir

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 3.21 | 1.00x |
| 2 | 1.68 | 1.91x |
| 4 | 0.89 | 3.61x |
| 8 | 0.81 | 3.96x |

### Numerical Integration pi — C

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 0.008 | 1.00x |
| 2 | 0.005 | 1.60x |
| 4 | 0.003 | 2.67x |
| 8 | 0.003 | 2.67x |

### Numerical Integration pi — Elixir

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 0.21 | 1.00x |
| 2 | 0.12 | 1.75x |
| 4 | 0.07 | 3.00x |
| 8 | 0.06 | 3.50x |

### Analysis

**Near-linear speedup with 4 threads** was observed for the primes problem in both languages, since checking primality is CPU-intensive and each number is independent.

**The π integration** shows diminishing returns faster because the computation per rectangle is very simple — the overhead of thread creation and synchronization becomes significant relative to the actual work.

**Going from 4 to 8 threads** produces minimal improvement. This is consistent with Amdahl's Law: the sequential fraction of the program (thread setup, mutex locking, result collection) limits the maximum achievable speedup regardless of how many cores are added.

**C vs Elixir:** C is consistently faster in absolute time due to lower overhead, but both languages achieve similar relative speedups, confirming that the parallelization strategy is equally effective in both.
