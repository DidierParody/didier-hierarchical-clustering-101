# Pseudocódigo: clustering jerárquico aglomerativo

Este documento describe el algoritmo en pseudocódigo, desde la versión más simple hasta cómo lo usamos en la práctica. Los diagramas equivalentes están en [`diagramas.md`](diagramas.md), y la implementación en [`../notebooks/clustering_jerarquico.ipynb`](../notebooks/clustering_jerarquico.ipynb).

## Contenido

1. [Notación](#1-notación)
2. [Algoritmo aglomerativo (versión básica)](#2-algoritmo-aglomerativo-versión-básica)
3. [Distancia entre clusters (criterios de enlace)](#3-distancia-entre-clusters-criterios-de-enlace)
4. [Actualización de Lance–Williams](#4-actualización-de-lancewilliams)
5. [Cortar el dendrograma](#5-cortar-el-dendrograma)
6. [Ejemplo a mano con 6 puntos](#6-ejemplo-a-mano-con-6-puntos)
7. [Flujo de la práctica](#7-flujo-de-la-práctica)
8. [Complejidad](#8-complejidad)
9. [Equivalencias con SciPy y scikit-learn](#9-equivalencias-con-scipy-y-scikit-learn)

---

## 1. Notación

| Símbolo | Significado |
|---|---|
| `X = {x₀, …, xₙ₋₁}` | Los `n` puntos de entrada |
| `d(x, y)` | Distancia entre dos **puntos** (por ejemplo, euclidiana) |
| `D(A, B)` | Distancia entre dos **clusters**, según el criterio de enlace |
| `C` | Conjunto de clusters activos en cada momento |
| `Z` | Matriz de enlace: una fila por fusión, `[a, b, altura, tamaño]` |
| `|A|` | Número de puntos del cluster `A` |

Los puntos originales tienen los ids `0 … n−1`. El cluster creado en la fusión `i` recibe el id `n + i`. Es la misma convención que usa SciPy.

---

## 2. Algoritmo aglomerativo (versión básica)

```text
ALGORITMO Aglomerativo(X, d, enlace)
    ENTRADA:  X       → n puntos
              d       → distancia entre puntos
              enlace  → "single" | "complete" | "average" | "ward"
    SALIDA:   Z       → matriz de enlace de (n − 1) filas

    // Paso 1: cada punto empieza siendo su propio cluster
    C ← { {x₀}, {x₁}, …, {xₙ₋₁} }

    // Paso 2: distancias iniciales entre todos los pares
    PARA cada par de clusters (A, B) en C HACER
        D[A, B] ← d(A, B)
    FIN PARA

    // Paso 3: fusionar n − 1 veces
    PARA i ← 0 HASTA n − 2 HACER
        (A, B) ← el par de C con la menor D[A, B]      // decisión voraz
        h      ← D[A, B]                               // altura de la fusión

        N ← A ∪ B                                      // nuevo cluster, id = n + i
        C ← (C − {A, B}) ∪ {N}

        agregar la fila [id(A), id(B), h, |N|] a Z

        // Paso 4: distancia del nuevo cluster a los demás
        PARA cada cluster K en C, con K ≠ N HACER
            D[N, K] ← ActualizarDistancia(A, B, K, enlace)   // sección 4
        FIN PARA
    FIN PARA

    DEVOLVER Z
FIN ALGORITMO
```

**Propiedades clave:**

- **Voraz:** en cada paso toma la mejor decisión local.
- **Irreversible:** una fusión nunca se deshace.
- **Determinista:** con los mismos datos y el mismo enlace, el resultado es siempre el mismo (salvo empates).

---

## 3. Distancia entre clusters (criterios de enlace)

```text
FUNCIÓN DistanciaEnlace(A, B, enlace)
    SEGÚN enlace HACER
        "single":    DEVOLVER mínimo  de d(a, b) para a ∈ A, b ∈ B
        "complete":  DEVOLVER máximo  de d(a, b) para a ∈ A, b ∈ B
        "average":   DEVOLVER promedio de d(a, b) para a ∈ A, b ∈ B
        "ward":      μA ← centroide(A);  μB ← centroide(B)
                     DEVOLVER raíz( 2 · |A|·|B| / (|A|+|B|) ) · ‖μA − μB‖
    FIN SEGÚN
FIN FUNCIÓN
```

| Enlace | Idea | Tiende a formar |
|---|---|---|
| single | par más cercano | cadenas, formas alargadas |
| complete | par más lejano | grupos compactos de diámetro parecido |
| average | promedio de todas las parejas | un punto intermedio |
| ward | menor aumento de la varianza intra-cluster | grupos esféricos de tamaño parecido |

---

## 4. Actualización de Lance–Williams

Recalcular `D[N, K]` desde los puntos sería costoso. Lance y Williams (1967) mostraron que basta con las distancias que ya teníamos:

```text
FUNCIÓN ActualizarDistancia(A, B, K, enlace)
    // Distancias conocidas antes de fusionar A y B
    dAK ← D[A, K];   dBK ← D[B, K];   dAB ← D[A, B]
    nA ← |A|;   nB ← |B|;   nK ← |K|

    SEGÚN enlace HACER
        "single":    DEVOLVER mín(dAK, dBK)
        "complete":  DEVOLVER máx(dAK, dBK)
        "average":   DEVOLVER (nA · dAK + nB · dBK) / (nA + nB)
        "ward":      T ← nA + nB + nK
                     DEVOLVER raíz( ((nA+nK)·dAK² + (nB+nK)·dBK² − nK·dAB²) / T )
    FIN SEGÚN
FIN FUNCIÓN
```

Fórmula general:

```text
D(A∪B, K) = αA·D(A,K) + αB·D(B,K) + β·D(A,B) + γ·|D(A,K) − D(B,K)|
```

| Enlace | αA | αB | β | γ |
|---|---|---|---|---|
| single | 1/2 | 1/2 | 0 | −1/2 |
| complete | 1/2 | 1/2 | 0 | +1/2 |
| average | nA/(nA+nB) | nB/(nA+nB) | 0 | 0 |
| ward (sobre d²) | (nA+nK)/T | (nB+nK)/T | −nK/T | 0 |

---

## 5. Cortar el dendrograma

El árbol completo se convierte en una partición cortándolo por altura o por número de clusters.

```text
FUNCIÓN Cortar(Z, n, criterio, valor)
    // criterio = "altura"   → valor es la altura t del corte
    // criterio = "clusters" → valor es el número k de clusters
    SI criterio = "clusters" ENTONCES
        fusiones_a_aplicar ← n − k
    SI NO
        fusiones_a_aplicar ← número de filas de Z con altura ≤ t
    FIN SI

    grupos ← { {0}, {1}, …, {n−1} }
    PARA i ← 0 HASTA fusiones_a_aplicar − 1 HACER
        unir en grupos los clusters Z[i].a y Z[i].b
    FIN PARA

    DEVOLVER una etiqueta por punto según su grupo
FIN FUNCIÓN
```

**¿Dónde cortar?**

1. Busca el **salto más grande** entre alturas de fusión consecutivas.
2. Confírmalo con el **coeficiente de silueta** para varios valores de `k`.
3. Contrasta con el conocimiento del dominio.

---

## 6. Ejemplo a mano con 6 puntos

Puntos (los mismos del laboratorio de la guía interactiva):

| Punto | x | y |
|---|---|---|
| p0 | 0.12 | 0.30 |
| p1 | 0.21 | 0.20 |
| p2 | 0.62 | 0.26 |
| p3 | 0.76 | 0.38 |
| p4 | 0.42 | 0.80 |
| p5 | 0.30 | 0.68 |

Matriz de enlace `Z` resultante:

| Paso | Single | Altura | Ward | Altura |
|---|---|---|---|---|
| 1 | p0 + p1 → C6 | 0.135 | p0 + p1 → C6 | 0.135 |
| 2 | p4 + p5 → C7 | 0.170 | p4 + p5 → C7 | 0.170 |
| 3 | p2 + p3 → C8 | 0.184 | p2 + p3 → C8 | 0.184 |
| 4 | **C6 + C8** → C9 | 0.414 | **C6 + C7** → C9 | 0.746 |
| 5 | C7 + C9 → C10 | 0.420 | C8 + C9 → C10 | 0.754 |

Los tres primeros pasos son idénticos: solo se unen pares de puntos. En el paso 4 los enlaces **discrepan**:

- *single* une `{p0, p1}` con `{p2, p3}`, porque p1 y p2 están cerca.
- *Ward* une `{p0, p1}` con `{p4, p5}`, porque es la fusión que menos aumenta la varianza.

---

## 7. Flujo de la práctica

Lo que hace el notebook, paso a paso:

```text
PROCEDIMIENTO PracticaClusteringJerarquico
    1. X, y_real ← make_blobs(n_samples = 150, centers = 3, random_state = 42)
       // y_real solo se usa al final para evaluar

    2. X_esc ← StandardScaler().fit_transform(X)
       // media 0 y desviación 1 en cada variable

    3. Z ← linkage(X_esc, method = "ward")
       dibujar dendrogram(Z) con una línea de corte

    4. PARA k ← 2 HASTA 10 HACER
           registrar la altura de la fusión que pasa de k a k − 1 clusters
           registrar silhouette_score para k clusters
       FIN PARA
       k ← el valor antes del mayor salto de altura, confirmado por la silueta

    5. modelo    ← AgglomerativeClustering(n_clusters = k, linkage = "ward")
       etiquetas ← modelo.fit_predict(X_esc)

    6. evaluar:
           silueta     ← silhouette_score(X_esc, etiquetas)
           cofenética  ← cophenet(Z, pdist(X_esc))
           ARI         ← adjusted_rand_score(y_real, etiquetas)

    7. PARA cada enlace en ["single", "complete", "average", "ward"] HACER
           repetir 5 y 6, y comparar (también con make_moons)
       FIN PARA
FIN PROCEDIMIENTO
```

---

## 8. Complejidad

| Versión | Tiempo | Memoria |
|---|---|---|
| Básica (sección 2) | O(n³) | O(n²) |
| Con cola de prioridad | O(n² log n) | O(n²) |
| SLINK (single), CLINK (complete) | O(n²) | O(n) |
| Cadena de vecinos más cercanos (Ward, complete, average) | O(n²) | O(n²) |

La memoria de la matriz de distancias (`n(n−1)/2` valores) es el límite práctico. Con `n = 50 000` son unos 10 GB.

---

## 9. Equivalencias con SciPy y scikit-learn

| Pseudocódigo | SciPy | scikit-learn |
|---|---|---|
| `Aglomerativo(X, d, enlace)` | `linkage(X, method=enlace, metric=d)` | `AgglomerativeClustering(linkage=enlace, metric=d)` |
| Matriz `Z` | valor devuelto por `linkage` | `children_` y `distances_` |
| `Cortar(Z, n, "clusters", k)` | `fcluster(Z, k, criterion="maxclust")` | `n_clusters=k` |
| `Cortar(Z, n, "altura", t)` | `fcluster(Z, t, criterion="distance")` | `distance_threshold=t`, `n_clusters=None` |
| Dibujar el árbol | `dendrogram(Z)` | no incluido; se usa SciPy |
| Correlación cofenética | `cophenet(Z, pdist(X))` | no incluido |
