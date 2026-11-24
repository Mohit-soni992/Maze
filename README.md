MAZE = [
    "###############################",
    "#S    #       #     #         #",
    "# ### # ##### # ### # ## #### #",
    "# #   #     # #   #       #   #",
    "# # ####### # ### ####### # ###",
    "# #       # #   #       # #   #",
    "# ####### # ### ####### # ### #",
    "#       # #     #     # #     #",
    "####### # ## #### ### # ## ## #",
    "#     # #           # #     # #",
    "# ### # ####### ### # ##### # #",
    "# #   #       #     #     # # #",
    "# # ########### ########### # #",
    "# #                         #E#",
    "###############################",
]
ROWS, COLS = len(MAZE), len(MAZE[0])


def find(ch):
    for r in range(ROWS):
        for c in range(COLS):
            if MAZE[r][c] == ch:
                return (r, c)


START, END = find("S"), find("E")


def neighbors(node):
    r, c = node
    for dr, dc in ((-1, 0), (0, 1), (1, 0), (0, -1)):
        nr, nc = r + dr, c + dc
        if 0 <= nr < ROWS and 0 <= nc < COLS and MAZE[nr][nc] != "#":
            yield (nr, nc)


def h(node):  # Manhattan distance heuristic
    return abs(node[0] - END[0]) + abs(node[1] - END[1])


def build_path(parent):
    path, cur = [], END
    while cur is not None:
        path.append(cur)
        cur = parent[cur]
    return path[::-1]


def pop_min(pq):
    best = min(pq)
    pq.remove(best)
    return best


def dfs():
    stack, parent, order, seen = [START], {START: None}, [], set()
    while stack:
        cur = stack.pop()
        if cur in seen:
            continue
        seen.add(cur)
        order.append(cur)
        if cur == END:
            return build_path(parent), order
        for n in neighbors(cur):
            if n not in seen:
                parent[n] = cur
                stack.append(n)
    return None, order


def bfs():
    queue, i, parent, order = [START], 0, {START: None}, []
    while i < len(queue):
        cur = queue[i]
        i += 1
        order.append(cur)
        if cur == END:
            return build_path(parent), order
        for n in neighbors(cur):
            if n not in parent:
                parent[n] = cur
                queue.append(n)
    return None, order


def greedy():
    pq, parent, order = [(h(START), START)], {START: None}, []
    while pq:
        _, cur = pop_min(pq)
        order.append(cur)
        if cur == END:
            return build_path(parent), order
        for n in neighbors(cur):
            if n not in parent:
                parent[n] = cur
                pq.append((h(n), n))
    return None, order


def a_star():
    pq, parent, g, order = [(h(START), 0, START)], {START: None}, {START: 0}, []
    while pq:
        _, cost, cur = pop_min(pq)
        if cost > g[cur]:
            continue
        order.append(cur)
        if cur == END:
            return build_path(parent), order
        for n in neighbors(cur):
            nc = cost + 1
            if n not in g or nc < g[n]:
                g[n] = nc
                parent[n] = cur
                pq.append((nc + h(n), nc, n))
    return None, order


ALGOS = {"1": ("DFS", dfs), "2": ("BFS", bfs), "3": ("Greedy BFS", greedy), "4": ("A*", a_star)}


def render(order, path=None):
    explored, path_set = set(order), set(path or [])
    rows = []
    for r in range(ROWS):
        row = ""
        for c in range(COLS):
            p = (r, c)
            if p == START:
                row += "S"
            elif p == END:
                row += "E"
            elif MAZE[r][c] == "#":
                row += "#"
            elif p in path_set:
                row += "*"
            elif p in explored:
                row += "."
            else:
                row += " "
        rows.append(row)
    return "\n".join(rows)


def solve(name, func):
    path, order = func()
    print("\n" + name + " DONE   ( # wall | . explored | * final path )\n")
    print(render(order, path))
    if path:
        print("\nNodes explored:", len(order), "| Path length:", len(path) - 1)
    else:
        print("\nRasta nahi mila!")


def compare():
    results = []
    for name, func in ALGOS.values():
        path, order = func()
        results.append((name, len(order), len(path) - 1))
    print("\n{:<12}{:>10}{:>14}".format("Algorithm", "Explored", "Path length"))
    print("-" * 36)
    for name, ex, pl in results:
        print("{:<12}{:>10}{:>14}".format(name, ex, pl))
    mx = max(r[1] for r in results)
    print("\nNodes explored (kam = behtar):")
    for name, ex, _ in results:
        print("{:<12}{} {}".format(name, "#" * max(1, 30 * ex // mx), ex))
    best = min(r[2] for r in results)
    print("\nShortest path =", best, "->", ", ".join(r[0] for r in results if r[2] == best))


def main():
    while True:
        print("\n=== PATHMIND: MAZE SOLVER ===")
        print("1. DFS\n2. BFS\n3. Greedy BFS\n4. A*\n5. Compare all\n0. Exit")
        ch = input("Choose: ").strip()
        if ch == "0":
            break
        elif ch in ALGOS:
            solve(*ALGOS[ch])
        elif ch == "5":
            compare()
        else:
            print("Galat choice!")


main()
