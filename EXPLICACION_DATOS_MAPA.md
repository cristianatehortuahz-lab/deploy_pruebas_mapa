# 🔍 Explicación Técnica: Separación de Datos (Publicaciones vs. Proyectos)

Este documento detalla el mecanismo técnico utilizado en el **Mapa de Coautorías** para separar y gestionar la visualización de redes de publicaciones de las redes de proyectos de investigación.

---

## 🏗️ Estructura del Archivo de Datos (`baseData.json`)

Para evitar la sobrecarga y permitir una alternancia instantánea entre capas, el archivo JSON centraliza toda la información pero la divide en llaves específicas:

### 1. Conexiones (Aristas)
*   **`edgesInternalPubs` / `edgesAllPubs`**: Contienen las conexiones de coautoría basadas en publicaciones académicas. Usan la propiedad `jointPubs` para definir el peso de la relación.
*   **`edgesInternalGrants` / `edgesAllGrants`**: Contienen las conexiones basadas en proyectos de investigación. Usan la propiedad `jointGrants` para definir el peso.

### 2. Investigadores (Nodos)
Cada nodo dentro de `nodesInternal` y `nodesAll` posee contadores independientes:
*   `pubs`: Número total de publicaciones del investigador.
*   `grants`: Número total de proyectos en los que participa.

---

## ⚙️ Lógica de Gestión de Capas (`network_logic.js`)

El motor de visualización utiliza tres componentes principales para manejar esta división:

### 1. Estado de la Capa (`window.currentLayer`)
Una variable global que actúa como "interruptor". Puede tener los valores `'pubs'` o `'grants'`. Por defecto, el mapa arranca en `'pubs'`.

### 2. Función de Intercambio (`applyLayerToData`)
Cuando se cambia de pestaña en la interfaz, esta función se ejecuta para reasignar las referencias que D3.js está utilizando:
```javascript
if (window.currentLayer === 'pubs') {
    data.edgesInternal = data.edgesInternalPubs;
    data.edgesAll = data.edgesAllPubs;
} else {
    data.edgesInternal = data.edgesInternalGrants;
    data.edgesAll = data.edgesAllGrants;
}
```

### 3. Escalamiento Dinámico (`getMetric`)
El tamaño de los nodos en el grafo no es estático. La función `getMetric` decide qué valor numérico enviar al motor de escalado de D3 basándose en la capa activa:
```javascript
function getMetric(d) {
    return window.currentLayer === 'pubs' ? (d.pubs || 0) : (d.grants || 0);
}
```

---

## 🎨 Interfaz de Usuario (`index.html`)

En la cabecera del mapa, existen botones con el atributo `data-layer`. Estos botones disparan un evento que:
1. Actualiza `window.currentLayer`.
2. Ejecuta `applyLayerToData()` para cambiar la fuente de las aristas.
3. Llama a `updateGraph()` para redibujar el mapa con las nuevas posiciones y tamaños.

Este diseño permite que el usuario cambie entre ver **quién publica con quién** y **quién investiga con quién en proyectos** de forma fluida y sin recargar la página.
