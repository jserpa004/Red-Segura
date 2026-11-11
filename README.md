# RedSegura

Aplicación de consola en **Java 17** (proyecto **Maven**, abre directo en Apache NetBeans) que modela la
movilidad entre universidades, centros de salud, nodos de transporte y puntos de encuentro de Sincelejo
con **árboles (ABB y AVL)** y **grafos mixtos** (BFS, DFS y Dijkstra), todo implementado desde cero.

> Los tiempos son **datos didácticos de simulación**. No son información oficial de tránsito ni de
> respuesta a emergencias.
| | |
|---|---|
| **Autor** | José Serpa |
| **Asignatura** | Estructura de Datos II |
| **Docente** | Jaider Reyes |
| **Universidad** | Corporación Universitaria Antonio José de Sucre |
## Cómo ejecutarlo

| Forma | Comando / pasos |
|---|---|
| NetBeans | *File → Open Project* → carpeta `RedSegura` → *Run Project* (F6) |
| Menú interactivo | `mvn -q compile exec:java` |
| Demostración Q1–Q7 | `mvn -q compile exec:java -Dexec.args="--demo"` |
| Reporte (demo + Big O) | `mvn -q compile exec:java -Dexec.args="--report"` → `reporte_redsegura.txt` |
| Pruebas | `mvn test` |
| JAR ejecutable | `mvn package` y luego `java -jar target/redsegura-1.0.0.jar --demo` |

Requisitos: JDK 17 o superior y Maven (NetBeans ya trae Maven). La salida usa UTF-8; en una consola de
Windows ejecute antes `chcp 65001` para ver bien las tildes. En NetBeans se ve bien sin cambios.
En `docs/reporte_demo.txt` hay un ejemplo de la salida completa.

## Arquitectura

```
co.edu.redsegura
├── model   Vertex, Edge, RedSeguraException        (datos inmutables + validaciones)
├── tree    BinarySearchTree, AvlTree, SearchResult, RotationType
├── graph   MixedGraph, Neighbor, GraphRepresentations,
│           BreadthFirstSearch, DepthFirstSearch, DijkstraSolver, PathResult
├── core    RedSeguraCore (fachada), DatasetFactory, SearchComparison, RotationDemo
└── app     RedSeguraApplication (CLI), QueryRunner (Q1..Q7), ComplexityAnalysis
```

```
RedSeguraApplication ──► RedSeguraCore ──► AvlTree  (índice de lugares por etiqueta)
                                      └──► MixedGraph ──► BFS · DFS · Dijkstra · Representaciones
```

Contrato de integración: la aplicación solo conoce `RedSeguraCore`; el núcleo usa `AvlTree` y `MixedGraph`
únicamente por métodos públicos. Ningún módulo lee campos privados de otro. Los errores de dominio son
`RedSeguraException` (no verificada) y la capa de aplicación los captura sin terminar el programa.

## Componentes del enunciado

| # | Componente | Clase | Qué ofrece |
|---|---|---|---|
| 1 | Modelo | `Vertex`, `Edge` | ID, etiqueta, tipo/peso, orientación; rechaza vacíos, pesos ≤ 0 y lazos |
| 2 | ABB | `BinarySearchTree` | `insert`, `search` (con pasos), `delete`, `inOrder`, `preOrder`, `postOrder`, `height` |
| 3 | AVL | `AvlTree` | `insert`/`delete` con alturas, factor de balance y rotaciones LL, RR, LR, RL (con bitácora) |
| 4 | Grafo mixto | `MixedGraph` | vértices, aristas y arcos, vecinos, IDs duplicados/inexistentes, bloqueo de conexiones |
| 5 | Representaciones | `GraphRepresentations` | lista, matriz de adyacencia ponderada y matriz de incidencia |
| 6 | BFS | `BreadthFirstSearch` | cola, visitados, niveles y árbol BFS |
| 7 | DFS | `DepthFirstSearch` | recursivo e iterativo, ciclos y componentes débiles |
| 8 | Ruta mínima | `DijkstraSolver` | `PriorityQueue`, relajación, distancias, predecesores y camino |
| 9 | Integración | `RedSeguraApplication` | CLI, errores controlados, pruebas JUnit, reporte, demo |

No se usa ninguna biblioteca que implemente AVL, BFS, DFS o Dijkstra; solo `java.util` (`ArrayList`,
`HashMap`, `ArrayDeque`, `PriorityQueue` como estructura de datos).

## Decisiones de diseño

- **Clave de los árboles = etiqueta del lugar**, valor = `Vertex`. Así la búsqueda por nombre (Q1) compara
  ABB contra AVL sobre los mismos datos. Claves duplicadas reemplazan el valor; claves nulas se rechazan.
- **Altura**: árbol vacío = 0, un nodo = 1. **Factor de balance** = h(izq) − h(der).
- **ABB iterativo** en `insert`/`search` (no desborda la pila con entrada ordenada); `delete` usa el sucesor in-order.
- **AVL**: tras cada inserción/eliminación se recalcula la altura y se rebalancea. Casos: bf > 1 con hijo izq. de
  bf ≥ 0 → **LL**; bf > 1 con hijo izq. de bf < 0 → **LR**; bf < −1 con hijo der. de bf ≤ 0 → **RR**; en otro caso **RL**.
  LR y RL son rotaciones dobles y se registran como una sola entrada del tipo correspondiente.
- **Pesos > 0**: Dijkstra requiere pesos no negativos y el 0 se confundiría con "sin conexión" en la matriz.
- **Grafo mixto**: una arista no dirigida se recorre en ambos sentidos; un arco dirigido solo de origen a destino.
- **Matriz de incidencia**: arco dirigido = +1 en el origen y −1 en el destino; arista no dirigida = +1 en ambos extremos.
- **Matriz de adyacencia**: `.` = sin conexión; con conexiones paralelas se conserva el menor peso.
- **Bloqueo de conexiones** (Q6): se marca la conexión como bloqueada sin borrarla; desaparece de vecinos,
  matrices y algoritmos, y se puede restaurar.
- **Desempates en Dijkstra**: relajación con `<` estricto, así que ante igual costo se queda la primera ruta hallada.

## Resultados de la demostración (Q1–Q7)

- **Q1** — "Hospital Universitario": con inserción en el orden del dataset el ABB usa 3 pasos (altura 7) y el AVL 1 paso
  (altura 4). Con inserción alfabética (peor caso del ABB) el ABB llega a altura 12 y usa 5 pasos; el AVL se mantiene en altura 4 y usa 4.
- **Q2** — 30-20-10 → LL; 10-20-30 → RR; 30-10-20 → LR; 10-30-20 → RL. En los cuatro la raíz queda en 20, altura 2.
- **Q3** — imprime lista de adyacencia, matriz 12×12 y matriz de incidencia 12×18.
- **Q4** — BFS: `S02, S01, S06, S07, S09, S05, S08, S10, S12, S03, S04, S11`. DFS: `S02, S01, S07, S09, S05, S10, S08, S12, S03, S04, S11, S06`.
  Difieren desde la posición 3 porque BFS agota el nivel (cola FIFO) y DFS se hunde por el primer vecino.
- **Q5** — S02 → S08 = **24 min** por S02, S01, S07, S08 (resultado de control: CUMPLE).
- **Q6** — al bloquear R16 el costo **sigue en 24 min**, pero la ruta cambia a S02, S01, S09, S10, S08. Hay un empate:
  ambas rutas cuestan 24, y Dijkstra se quedaba con la de R16 por encontrarla antes. Si además se bloquea R09 (la otra entrada a S08), el terminal queda inaccesible.
- **Q7** — S99 no existe: se informa el error y el programa continúa.

## Análisis de complejidad

(n = claves, V = vértices, E = conexiones, h = altura)

| Operación | Tiempo | Espacio |
|---|---|---|
| ABB insert/search/delete | O(h): O(log n) promedio, **O(n)** peor caso | O(1) iterativo |
| AVL insert/search/delete | **O(log n)** peor caso | O(log n) pila |
| Rotaciones | O(1) | O(1) |
| Recorridos de árbol | O(n) | O(h) + salida |
| `addVertex`/`addEdge` | O(1) | O(1) |
| `neighbors(v)` | O(grado) | O(grado) |
| Lista / matriz de adyacencia / incidencia | O(V+E) / O(V²+E) / O(V·E) | O(V+E) / O(V²) / O(V·E) |
| BFS, DFS, ciclos, componentes | O(V+E) | O(V) |
| Dijkstra (PriorityQueue) | O((V+E) log V) | O(V+E) |

## Pruebas (JUnit 5)

`mvn test` ejecuta 59 pruebas en 8 clases: `ModelTest`, `BinarySearchTreeTest`, `AvlTreeTest`, `MixedGraphTest`,
`GraphRepresentationsTest`, `TraversalTest`, `DijkstraSolverTest` y `RedSeguraCoreTest`. Cubren casos normales,
**vértice inexistente**, **grafo desconectado** y **pesos inválidos**, las cuatro rotaciones, eliminación con
rebalanceo, una prueba aleatoria del invariante AVL y el resultado de control de 24 minutos.


```
