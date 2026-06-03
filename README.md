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
iex prime_sum_sequential.ex
iex prime_sum_parallel.ex
iex pi_sequential.ex
iex pi_parallel.ex
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
| 1 (sequential) | 0.22 | 1.00 |
| 2 | 0.16 | 1.38 |
| 4 | 0.11 | 2.00 |
| 8 | 0.09 | 2.44 |

### Sum of Primes — Elixir

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 1.02 | 1.00 |
| 2 | 0.69 | 1.48 |
| 4 | 0.39 | 2.62 |
| 8 | 0.29 | 3.52 |

### Numerical Integration pi — C

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 0.029 | 1.00 |
| 2 | 0.016 | 1.81 |
| 4 | 0.0077 | 3.77 |
| 8 | 0.0049 | 5.90 |

### Numerical Integration pi — Elixir

| Threads | Time (s) | Speedup |
|---|---|---|
| 1 (sequential) | 0.13 | 1.00 |
| 2 | 0.06 | 2.16 |
| 4 | 0.03 | 4.30 |
| 8 | 0.03 | 4.33 |

### Analysis

**The Sum of Primes problem** shows a clear improvement as the number of threads increases in both languages. Since each number can be tested independently, the workload is highly parallelizable. Elixir achieves a higher relative speedup (3.52× with 8 threads) than C (2.44×), although C remains much faster in absolute execution time.

**The Numerical Integration of pi** scales very efficiently in C, reaching a speedup of approximately 5.92× with 8 threads. This indicates that the workload is evenly distributed among threads and that synchronization costs are relatively low.

**In Elixir, the pi integration** scales well up to 4 threads (4.33× speedup), but no additional improvement is observed with 8 threads. This suggests that runtime overhead, scheduling costs, or hardware limitations become the dominant factors beyond 4 threads.

