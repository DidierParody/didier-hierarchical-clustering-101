# Diagramas en Mermaid

Código Mermaid de los diagramas de la práctica. Cada bloque está pensado para **importarse en Excalidraw** y retocarse a mano. El pseudocódigo de referencia está en [`pseudocodigo.md`](pseudocodigo.md).

## Cómo importarlos en Excalidraw

1. Abre [excalidraw.com](https://excalidraw.com).
2. En la barra de herramientas, abre **Más herramientas** (el icono de la caja de herramientas) y elige **Mermaid a Excalidraw**.
3. Pega el código de un bloque (sin las líneas de apertura y cierre con tres acentos graves) y pulsa **Insertar**.
4. Ajusta la posición, los colores y los textos a tu gusto.
5. Exporta dos archivos con el nombre sugerido en cada diagrama y guárdalos en [`diagramas/`](diagramas/):
   - `.excalidraw`: el archivo editable;
   - `.png` o `.svg`: la imagen para el README y las diapositivas.

> Todos los diagramas son de tipo `flowchart`, porque es el único tipo que Excalidraw convierte en formas editables. Otros tipos, como `timeline`, se insertan como una imagen fija.

## Contenido

| # | Diagrama | Archivo sugerido |
|---|---|---|
| 1 | [Algoritmo aglomerativo](#1-algoritmo-aglomerativo) | `01-algoritmo` |
| 2 | [Aglomerativo vs. divisivo](#2-aglomerativo-vs-divisivo) | `02-aglomerativo-vs-divisivo` |
| 3 | [Criterios de enlace](#3-criterios-de-enlace) | `03-criterios-de-enlace` |
| 4 | [Dendrograma del ejemplo: single](#4-dendrograma-del-ejemplo-single) | `04-dendrograma-single` |
| 5 | [Dendrograma del ejemplo: Ward](#5-dendrograma-del-ejemplo-ward) | `05-dendrograma-ward` |
| 6 | [¿Qué enlace elegir?](#6-qué-enlace-elegir) | `06-elegir-enlace` |
| 7 | [Flujo de la práctica](#7-flujo-de-la-práctica) | `07-flujo-practica` |
| 8 | [Línea de tiempo](#8-línea-de-tiempo) | `08-linea-de-tiempo` |

---

## 1. Algoritmo aglomerativo

Corresponde a la sección 2 de [`pseudocodigo.md`](pseudocodigo.md#2-algoritmo-aglomerativo-versión-básica).

```mermaid
flowchart TD
    A(["Inicio: n puntos"]) --> B["Cada punto es su propio cluster"]
    B --> C["Calcular la matriz de distancias entre todos los pares"]
    C --> D{"¿Queda más de un cluster?"}
    D -- "Sí" --> E["Buscar el par A, B con la menor distancia de enlace"]
    E --> F["Fusionar A y B en un nuevo cluster"]
    F --> G["Guardar la fusión en Z: a, b, altura, tamaño"]
    G --> H["Actualizar distancias con Lance-Williams"]
    H --> D
    D -- "No" --> I["Dibujar el dendrograma con Z"]
    I --> J["Cortar el árbol a una altura o en k clusters"]
    J --> K(["Fin: etiqueta de cluster por punto"])
```

---

## 2. Aglomerativo vs. divisivo

```mermaid
flowchart LR
    subgraph AGL["Aglomerativo - AGNES - de abajo hacia arriba"]
        direction TB
        A1["n clusters de 1 punto"] --> A2["Fusionar los 2 más cercanos"]
        A2 --> A3["Repetir n - 1 veces"]
        A3 --> A4["1 cluster con todo"]
    end
    subgraph DIV["Divisivo - DIANA - de arriba hacia abajo"]
        direction TB
        D1["1 cluster con todo"] --> D2["Partir un cluster en 2"]
        D2 --> D3["Repetir hasta que cada punto quede solo"]
        D3 --> D4["n clusters de 1 punto"]
    end
```

---

## 3. Criterios de enlace

Corresponde a la sección 3 de [`pseudocodigo.md`](pseudocodigo.md#3-distancia-entre-clusters-criterios-de-enlace).

```mermaid
flowchart TD
    Q{"¿Cómo mido la distancia entre los clusters A y B?"}
    Q --> S["Single: el par de puntos más cercano"]
    Q --> C["Complete: el par de puntos más lejano"]
    Q --> AV["Average: el promedio de todas las parejas"]
    Q --> W["Ward: cuánto aumenta la varianza al fusionar"]
    S --> S2["Sigue formas alargadas. Riesgo: efecto cadena"]
    C --> C2["Grupos compactos. Riesgo: sensible a outliers"]
    AV --> AV2["Término medio, más robusto al ruido"]
    W --> W2["Grupos esféricos de tamaño parecido. Solo euclidiana"]
```

---

## 4. Dendrograma del ejemplo: single

Los 6 puntos de la sección 6 de [`pseudocodigo.md`](pseudocodigo.md#6-ejemplo-a-mano-con-6-puntos). Las flechas van de cada cluster hacia el cluster que lo contiene, y el número indica la altura de la fusión.

```mermaid
flowchart BT
    p0(("p0")) --> C6["C6 - altura 0.135"]
    p1(("p1")) --> C6
    p4(("p4")) --> C7["C7 - altura 0.170"]
    p5(("p5")) --> C7
    p2(("p2")) --> C8["C8 - altura 0.184"]
    p3(("p3")) --> C8
    C6 --> C9["C9 - altura 0.414"]
    C8 --> C9
    C7 --> C10["C10 - altura 0.420 - raíz"]
    C9 --> C10
```

---

## 5. Dendrograma del ejemplo: Ward

Mismos 6 puntos. Los tres primeros pasos coinciden con single; en el paso 4 Ward une `C6` con `C7`, no con `C8`.

```mermaid
flowchart BT
    p0(("p0")) --> C6["C6 - altura 0.135"]
    p1(("p1")) --> C6
    p4(("p4")) --> C7["C7 - altura 0.170"]
    p5(("p5")) --> C7
    p2(("p2")) --> C8["C8 - altura 0.184"]
    p3(("p3")) --> C8
    C6 --> C9["C9 - altura 0.746"]
    C7 --> C9
    C8 --> C10["C10 - altura 0.754 - raíz"]
    C9 --> C10
```

---

## 6. ¿Qué enlace elegir?

Guía rápida, no una regla absoluta. Siempre conviene comparar varios.

```mermaid
flowchart TD
    A{"¿La distancia es euclidiana?"}
    A -- "No: coseno, Manhattan..." --> B{"¿Hay mucho ruido u outliers?"}
    B -- "Sí" --> AV["Average"]
    B -- "No" --> CO["Complete o average"]
    A -- "Sí" --> C{"¿Esperas grupos redondos y de tamaño parecido?"}
    C -- "Sí" --> W["Ward: opción por defecto"]
    C -- "No" --> D{"¿Grupos alargados o no convexos, y datos limpios?"}
    D -- "Sí" --> S["Single"]
    D -- "No" --> AV2["Average"]
```

---

## 7. Flujo de la práctica

Corresponde a la sección 7 de [`pseudocodigo.md`](pseudocodigo.md#7-flujo-de-la-práctica) y al notebook [`clustering_jerarquico.ipynb`](../notebooks/clustering_jerarquico.ipynb).

```mermaid
flowchart LR
    A["1. Datos: make_blobs, 150 puntos"] --> B["2. Escalar: StandardScaler"]
    B --> C["3. Jerarquía: linkage con Ward"]
    C --> D["4. Dendrograma"]
    D --> E{"5. ¿Cuántos clusters?"}
    E --> E1["Salto en las alturas"]
    E --> E2["Coeficiente de silueta"]
    E1 --> F["6. AgglomerativeClustering con k = 3"]
    E2 --> F
    F --> G["7. Evaluar: silueta, cofenética, ARI"]
    G --> H["8. Comparar enlaces: blobs y lunas"]
```

---

## 8. Línea de tiempo

Autores y aportes principales.

```mermaid
flowchart LR
    T1["1948 - Sørensen: enlace complete"] --> T2["1951 - Florek y otros: enlace single"]
    T2 --> T3["1958 - Sokal y Michener: UPGMA, average"]
    T3 --> T4["1963 - Sokal y Sneath: Numerical Taxonomy"]
    T4 --> T5["1963 - Ward: mínima varianza"]
    T5 --> T6["1967 - Lance y Williams: fórmula unificada"]
    T6 --> T7["1967 - Johnson: dendrogramas y ultramétricas"]
    T7 --> T8["1973 - Sibson: SLINK en O de n cuadrado"]
    T8 --> T9["1990 - Kaufman y Rousseeuw: AGNES y DIANA"]
    T9 --> T10["2013 - Campello y otros: HDBSCAN"]
```
