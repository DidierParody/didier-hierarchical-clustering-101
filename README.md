# didier-hierarchical-clustering-101

Práctica de **clustering jerárquico aglomerativo** con scikit-learn y SciPy.

## Contenido

| Archivo | Descripción |
|---|---|
| [`notebooks/clustering_jerarquico.ipynb`](notebooks/clustering_jerarquico.ipynb) | Implementación guiada paso a paso: datos, escalado, dendrograma, elección de `k`, modelo, evaluación y comparación de criterios de enlace. |
| [`docs/pseudocodigo.md`](docs/pseudocodigo.md) | Pseudocódigo del algoritmo, Lance–Williams, corte del dendrograma, ejemplo a mano con 6 puntos y equivalencias con SciPy y scikit-learn. |
| [`docs/diagramas.md`](docs/diagramas.md) | Código Mermaid de 8 diagramas, listo para importar en Excalidraw. |
| [`docs/diagramas/`](docs/diagramas/) | Diagramas exportados de Excalidraw (`.excalidraw` y `.png`). |
| [`presentacion/clustering-jerarquico.pdf`](presentacion/clustering-jerarquico.pdf) | Presentación de 17 diapositivas: idea, funcionamiento, autores, práctica y resultados. |
| [`presentacion/clustering-jerarquico.pptx`](presentacion/clustering-jerarquico.pptx) | La misma presentación en PowerPoint, editable y con notas del orador. Usa las fuentes gratuitas [Kalam](https://fonts.google.com/specimen/Kalam), [Nunito](https://fonts.google.com/specimen/Nunito) y [Fira Code](https://fonts.google.com/specimen/Fira+Code); instálalas para verla con la estética original. |
| [`presentacion/clustering-jerarquico.html`](presentacion/clustering-jerarquico.html) | La misma presentación en HTML, con enlaces a los documentos. Descárgala y ábrela en el navegador (necesita internet para cargar las fuentes). |

## Qué cubre el notebook

1. Datos de ejemplo con `make_blobs` (150 puntos, 3 grupos).
2. Escalado con `StandardScaler`.
3. Matriz de enlace y dendrograma con `scipy.cluster.hierarchy`.
4. Elección del número de clusters: salto en las alturas de fusión y coeficiente de silueta.
5. Modelo con `sklearn.cluster.AgglomerativeClustering`.
6. Evaluación con silueta, correlación cofenética y ARI.
7. Comparación de los enlaces *single*, *complete*, *average* y *Ward*, incluido un caso no convexo (`make_moons`).

## Cómo ejecutarlo

```bash
python -m venv .venv
source .venv/bin/activate        # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook notebooks/clustering_jerarquico.ipynb
```
