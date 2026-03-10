# Primero.ai Assessment Practice (Equip)

This repo contains reference solutions (Python) with **in-code comments** that reflect an analyst-style reasoning process: edge cases, invariants, and implementation choices.

## Problems
- LeetCode 529 — Minesweeper (Medium)
- LeetCode 224 — Basic Calculator (Hard)
- LeetCode 68 — Text Justification (Hard)

---

## 529. Minesweeper (BFS)

```python
# 529. Minesweeper (BFS)
from typing import List
from collections import deque

class Solution:
    def updateBoard(self, board: List[List[str]], click: List[int]) -> List[List[str]]:
        # Observación: el click siempre cae en 'M' o 'E' (no revelado).
        # Caso base: si clickeo una mina, se acaba el juego.
        m, n = len(board), len(board[0])
        r0, c0 = click
        if board[r0][c0] == "M":
            board[r0][c0] = "X"
            return board

        # 8 direcciones (incluye diagonales). Ojo con bordes: siempre checar límites.
        dirs = [(-1,-1), (-1,0), (-1,1),
                (0,-1),          (0,1),
                (1,-1),  (1,0),  (1,1)]

        def count_adjacent_mines(r: int, c: int) -> int:
            # Observación: solo cuenta minas 'M' (no 'X'), porque 'X' solo existe si clickeas mina.
            cnt = 0
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < m and 0 <= nc < n and board[nr][nc] == "M":
                    cnt += 1
            return cnt

        # Elección: BFS para evitar recursion depth (DFS recursivo podría ser riesgoso en otros lenguajes).
        q = deque([(r0, c0)])
        seen = {(r0, c0)}  # Observación: evita re-enqueue infinito.

        while q:
            r, c = q.popleft()

            # Observación: solo procesamos celdas 'E' (no reveladas). Si ya cambió a 'B' o número, se ignora.
            if board[r][c] != "E":
                continue

            mines = count_adjacent_mines(r, c)
            if mines > 0:
                # Si hay minas alrededor: escribir el número y NO expandir.
                board[r][c] = str(mines)
                continue

            # Si no hay minas alrededor: es blanco 'B' y expandimos a vecinos 'E'.
            board[r][c] = "B"
            for dr, dc in dirs:
                nr, nc = r + dr, c + dc
                if 0 <= nr < m and 0 <= nc < n and board[nr][nc] == "E" and (nr, nc) not in seen:
                    seen.add((nr, nc))
                    q.append((nr, nc))

        return board
```

```python

# 224. Basic Calculator (stack + parsing)
from typing import List

class Solution:
    def calculate(self, s: str) -> int:
        # Observación: la expresión es válida, pero hay espacios y paréntesis.
        # '+' NO es unario; '-' SÍ puede ser unario (ej: "-(2+3)").
        res = 0          # resultado acumulado del "nivel" actual
        sign = 1         # signo actual (+1 o -1) aplicado al próximo número
        num = 0          # número en construcción (multi-dígito)
        stack = []       # guardamos (res_prev, sign_prev) al entrar a '('

        for ch in s:
            if ch.isdigit():
                # Ojo: números multi-dígito -> num = num*10 + dígito
                num = num * 10 + int(ch)

            elif ch in "+-":
                # Observación: antes de cambiar de operador, "cerramos" el número que veníamos leyendo.
                res += sign * num
                num = 0
                sign = 1 if ch == "+" else -1

            elif ch == "(":
                # Observación: al entrar a paréntesis, guardamos el estado actual y reiniciamos el interior.
                stack.append(res)
                stack.append(sign)
                res = 0
                sign = 1
                num = 0

            elif ch == ")":
                # Observación clave: al cerrar ')', primero incorporo num pendiente al res interno.
                res += sign * num
                num = 0

                # Luego combino con el estado externo: res_externo + sign_externo * res_interno
                prev_sign = stack.pop()
                prev_res = stack.pop()
                res = prev_res + prev_sign * res

            # espacios se ignoran

        # Observación: al final puede quedar un número pendiente que aún no se sumó.
        return res + sign * num

```

```python

# 68. Text Justification (greedy + reparto de espacios)
from typing import List

class Solution:
    def fullJustify(self, words: List[str], maxWidth: int) -> List[str]:
        # Observación: greedy por líneas (meto tantas palabras como quepan).
        res = []
        i = 0
        n = len(words)

        while i < n:
            j = i
            line_len = 0  # suma de longitudes de palabras (sin espacios)

            # Regla de "cabe": line_len + len(next_word) + espacios_minimos <= maxWidth
            # espacios_minimos entre palabras = (j - i) (cantidad de gaps si meto j palabras)
            while j < n and line_len + len(words[j]) + (j - i) <= maxWidth:
                line_len += len(words[j])
                j += 1

            num_words = j - i
            spaces = maxWidth - line_len  # total de espacios a repartir

            # Caso 1: última línea o línea con 1 palabra -> left-justified
            if j == n or num_words == 1:
                # Observación: última línea: 1 espacio entre palabras y el resto al final.
                line = " ".join(words[i:j])
                line += " " * (maxWidth - len(line))
                res.append(line)
                i = j
                continue

            # Caso 2: fully-justified (no es última y hay >=2 palabras)
            gaps = num_words - 1
            base = spaces // gaps      # espacios mínimos por gap
            extra = spaces % gaps      # sobran: van a la izquierda

            # Observación: los "extra" se asignan a los gaps más a la izquierda.
            parts = []
            for k in range(i, j - 1):
                parts.append(words[k])
                pad = base + (1 if (k - i) < extra else 0)
                parts.append(" " * pad)
            parts.append(words[j - 1])

            res.append("".join(parts))
            i = j

        return res

```
