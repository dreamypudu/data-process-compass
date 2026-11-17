# Datos limpios — COMPASS

Generado desde `dump-compass-202608030129.sql` (3 de agosto de 2026) con
`scripts/exportar_limpio.py` del repositorio.

## Reglas de limpieza aplicadas

1. **Decisiones repetidas.** En 19 pares (usuario, nodo) se registraron dos
   decisiones con alternativa distinta dentro de la misma sesión. En 17 de ellos
   el desfase entre los órdenes de decisión equivale al largo de la secuencia
   (cinco nodos en la secuencia 1, cuatro en la secuencia 4), lo que corresponde a
   una repetición completa de la secuencia tras un fallo de la plataforma durante
   la aplicación. El par restante presenta órdenes consecutivos, compatible con un
   doble registro del mismo evento. En todos los casos se conserva la decisión de
   mayor `decision_order`.

2. **Nodos revisitados en datos de proceso.** Cuando un nodo tiene dos bloques de
   eventos, se conserva el de mayor `process_log_id`.

3. **Sesiones de prueba.** Dos usuarios sin decisiones registradas quedan fuera al
   no aparecer en `explicit_decisions`. `sessions.csv` se filtró en consecuencia y
   se le quitó la columna `payload`, que duplica el contenido ya normalizado.

## Contenido

| Archivo | Filas | Descripción |
|---|---|---|
| `explicit_decisions.csv` | 1 636 | Una fila por decisión: usuario, nodo, alternativa elegida y efectos |
| `mlq_labels.csv` | 1 636 | Misma unidad, con las nueve medianas expertas de la alternativa elegida |
| `process_logs.csv` | 19 153 | Eventos `hover_enter` / `hover_leave` con marca de tiempo y agregados por nodo |
| `option_process_stats.csv` | 4 416 | Permanencia y número de consideraciones por alternativa |
| `sessions.csv` | 60 | Una fila por sesión |

## Cifras de referencia

- 60 participantes
- 28 nodos de decisión; 24 fueron recorridos por los 60
- Nodos por participante: mínimo 25, máximo 28, media 27,3
- Sin pares duplicados en ninguna tabla
- `mlq_labels` y `explicit_decisions` cubren exactamente el mismo conjunto de pares

## Reproducir

```bash
pg_restore -f dump-plano.sql dump-compass-202608030129.sql
python scripts/exportar_limpio.py dump-plano.sql datos_limpios
```
