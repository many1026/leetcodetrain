# LeetCode 937 — Reorder Data in Log Files (mi proceso)

## Qué entendí del problema
Tenemos una lista de logs de dos tipos:
- **letter-logs**: el contenido (después del id) empieza con letra.
- **digit-logs**: el contenido empieza con dígito.

Reglas de orden:
1) Primero van todos los **letter-logs**, ordenados:
   - por **contenido** (lo que está después del primer espacio),
   - y si empatan, por **id** (lo que está antes del primer espacio).
2) Después van los **digit-logs** en el **mismo orden original** (no se ordenan).

---

## Pregunta: “¿El identificador es todo lo que está antes del primer espacio?”
Sí. El **id** es todo lo que va antes del primer espacio.

---

## Pregunta: “¿Qué significa ‘orden alfabético’ del contenido?”
Significa **orden lexicográfico** (como diccionario): comparar strings carácter por carácter.

---

## Pistas que me llevé (directo)
- Separar cada log en **id** y **contenido** usando el **primer espacio**.
- Determinar si es digit-log o letter-log viendo el **primer carácter del contenido**.
- Guardar letter-logs en una lista y digit-logs en otra.
- Ordenar **solo** letter-logs por (contenido, id).
- Devolver letter-logs ordenados + digit-logs sin cambiar.

---

## Pregunta: “¿Cómo pregunto si algo es número o letra en Python?”
- `ch.isalpha()` → True si es letra  
- `ch.isdigit()` → True si es dígito

---

## Pregunta: “Las funciones fallan si hay espacios”
No: solo hay que evaluar **un carácter** (por ejemplo, el primer carácter del contenido), no toda la string con espacios.

Ejemplo: si `idx` es donde está el primer espacio en `log`,
el primer carácter del contenido es `log[idx + 1]`.

---

## Error que me pasó (y corrección)
Mi idea inicial usaba posiciones fijas tipo `logs[5]` y/o `dict`.
Eso está mal porque:
- cada log puede tener longitud variable,
- no necesito dict, necesito **listas**,
- digit-logs no se ordenan.

---

## Cómo hice el “tie-break” (si contenido empata, ordenar por id)
Ordenar letter-logs con una clave de orden:
- primero contenido,
- luego id.

---

## Código final (para pegar en LeetCode)
```python
from typing import List

class Solution:
    def reorderLogFiles(self, logs: List[str]) -> List[str]:
        letter_logs = []
        digit_logs = []

        for log in logs:
            idx = log.find(" ")
            first_char = log[idx + 1]  # primer caracter del contenido
            if first_char.isalpha():
                letter_logs.append(log)
            else:
                digit_logs.append(log)

        # ordenar solo letter-logs: (contenido, id)
        letter_logs.sort(key=lambda log: (log[log.find(" ") + 1:], log[:log.find(" ")]))
        return letter_logs + digit_logs
