import multiprocessing as mp
import time
import math

# Heavy cell update to ensure parallel processing has a clear advantage
def update_cell(args):
    grid, x, y = args
    t = grid[x][y]

    neighbours = []
    if x > 0: neighbours.append(grid[x-1][y])
    if x < len(grid)-1: neighbours.append(grid[x+1][y])
    if y > 0: neighbours.append(grid[x][y-1])
    if y < len(grid[0])-1: neighbours.append(grid[x][y+1])

    avg = sum(neighbours) / len(neighbours)

    # Very heavy maths loop to guarantee CPU load
    for _ in range(200):
        avg = avg + math.sin(avg) * 0.05 - math.cos(t) * 0.03 + math.tan(avg) * 0.001

    return avg

# Serial version (1 core)
def step_serial(grid):
    new = [[0]*len(grid[0]) for _ in range(len(grid))]
    for x in range(len(grid)):
        for y in range(len(grid[0])):
            new[x][y] = update_cell((grid, x, y))
    return new

# Parallel version (all cores)
def step_parallel(grid, cores):
    coords = [(grid, x, y) for x in range(len(grid)) for y in range(len(grid[0]))]

    # the Large chunksize reduces overhead dramatically
    with mp.Pool(processes=cores) as pool:
        results = pool.map(update_cell, coords, chunksize=2000)

    size = len(grid[0])
    return [results[i:i+size] for i in range(0, len(results), size)]

if __name__ == "__main__":
    size = 400          # Larger grid = more work
    timesteps = 4       # Enough steps to show difference
    cores = mp.cpu_count()

    print("grid:", size, "x", size)
    print("timesteps:", timesteps)
    print("cpu_cores_detected:", cores)
    print("----")

    base = [[15 + (x % 10) for y in range(size)] for x in range(size)]

    # Serial
    sgrid = base
    serial_total = 0
    for step in range(1, timesteps + 1):
        start = time.time()
        sgrid = step_serial(sgrid)
        t = time.time() - start
        serial_total += t
        print("serial_step", step, "time:", round(t, 3))

    print("----")

    # Parallel
    pgrid = base
    parallel_total = 0
    for step in range(1, timesteps + 1):
        start = time.time()
        pgrid = step_parallel(pgrid, cores)
        t = time.time() - start
        parallel_total += t
        print("parallel_step", step, "time:", round(t, 3))

    print("----")
    print("serial_total:", round(serial_total, 3))
    print("parallel_total:", round(parallel_total, 3))
    print("speedup:", round(serial_total / parallel_total, 2))
