<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/9/93/MongoDB_Logo.svg" alt="MongoDB Logo" width="280">
</p>

# Tienda de Electrónica de Videojuegos — Documentación MongoDB

> Base de datos sencilla para una tienda online de productos de videojuegos.  
> Hecha para ejecutarse en **`mongosh`** (MongoDB Shell) y visualizarse en **MongoDB Compass**.


---

## Estructura de la Base de Datos

- **Base de datos**: `tienda_videojuegos`  
- **Colección principal**: `productos`

### Interfaz de MongoDB Compass
<p align="center">
  <img src="Mongo.jpg" alt="Interfaz de MongoDB Compass" width="700">
</p>

## Esquema del documento (`productos`)

| Campo      | Tipo      | Descripción                                 |
|------------|-----------|----------------------------------------------|
| `_id`      | ObjectId  | ID único generado por MongoDB                |
| `nombre`   | string    | Nombre del producto (ej. `"Xbox Series X"`) |
| `precio`   | number    | Precio en USD (ej. `499.99`)                |
| `stock`    | number    | Cantidad disponible en inventario           |
| `categoria`| string    | Categoría: `"Consola"`, `"Juego"`, `"Accesorio"`, `"PC Gaming"`, `"Hardware"` |

---

## 1. Creación e Inserción de Datos

### Crear / Usar la base de datos
Este comando crea la base de datos `tienda_videojuegos` si no existe, y cambia el contexto de `mongosh` para trabajar sobre ella.
```js
use tienda_videojuegos
```

###  Inserción Masiva de Datos (`insertMany`)
Este comando inserta un arreglo de documentos (en este caso, 100 registros) en la colección `productos`. Es la forma más eficiente de añadir datos iniciales al sistema.
```js
db.productos.insertMany([
  // ... (100 registros de productos omitidos por brevedad) ...
  // A continuación se muestra una muestra de la inserción original.
  //  CONSOLAS (10)
  { nombre: "Xbox Series X", precio: 499.99, stock: 15, categoria: "Consola" },
  { nombre: "PlayStation 5", precio: 499.99, stock: 10, categoria: "Consola" },
  //  JUEGOS (40)
  { nombre: "Juego: Call of Duty: Black Ops 6", precio: 69.99, stock: 100, categoria: "Juego" },
  { nombre: "Juego: Grand Theft Auto VI (Reserva)", precio: 79.99, stock: 200, categoria: "Juego" },
  //  ACCESORIOS (25)
  { nombre: "Control Xbox Wireless", precio: 69.99, stock: 50, categoria: "Accesorio" },
  { nombre: "Control DualSense PS5", precio: 69.99, stock: 45, categoria: "Accesorio" },
  //  PC GAMING (15)
  { nombre: "Teclado mecánico Razer BlackWidow V4", precio: 149.99, stock: 12, categoria: "PC Gaming" },
  { nombre: "Ratón Logitech G502 X", precio: 79.99, stock: 20, categoria: "PC Gaming" },
  //  HARDWARE (10)
  { nombre: "Router gaming ASUS ROG Rapture GT-AX11000", precio: 449.99, stock: 3, categoria: "Hardware" }
])
```

---

## 2. Consultas Básicas (CRUD)

### Actualizar (`updateOne`)
Modifica el primer documento que coincida con el filtro. En este caso, busca el producto con el nombre "Xbox Series X" y actualiza su `stock` a 20 unidades usando el operador `$set`.
```js
db.productos.updateOne(
  { nombre: "Xbox Series X" },
  { $set: { stock: 20 } }
)
```
El resultado `modifiedCount: 1` confirma que un documento fue modificado.
```json
{
  "acknowledged": true,
  "matchedCount": 1,
  "modifiedCount": 1,
  "upsertedCount": 0
}
```

### Eliminar (`deleteMany`)
Elimina todos los documentos que coincidan con el filtro. se eliminan **todos** los productos cuya `categoria` sea "Juego".
```js
db.productos.deleteMany(
  { categoria: "Juego" }
)
```
El resultado `deletedCount` muestra cuántos documentos fueron eliminados. Si se ejecutara sobre los datos iniciales, eliminaría los 40 juegos.
```json
{
  "acknowledged": true,
  "deletedCount": 40
}
```

### Selección (`find`)
Recupera documentos de una colección. Devuelve todos los documentos.
```js
db.productos.find()
```
Devuelve un cursor con todos los productos de la colección.
```json
[
  { "_id": "...", "nombre": "Xbox Series X", "categoria": "Consola" },
  { "_id": "...", "nombre": "PlayStation 5", "categoria": "Consola" }
]
```

---

## 3. Consultas con Filtros y Operadores

### Filtrar por categoría
Busca todos los productos que pertenecen exactamente a la categoría "Consola".
```js
db.productos.find(
  { categoria: "Consola" }
)
```

### Filtrar por precio menor a 100
Encuentra productos cuyo precio sea inferior a 100. El operador `$lt` significa "less than" (menor que).
```js
db.productos.find(
  { precio: { $lt: 100 } }
)
```

### Filtrar por precio entre 100 y 300
Usa los operadores `$gte` (mayor o igual que) y `$lte` (menor o igual que) para encontrar productos dentro de un rango de precios.
```js
db.productos.find(
  {
    precio: {
      $gte: 100,
      $lte: 300
    }
  }
)
```

### Filtrar por stock mayor a 50
Usa el operador `$gt` (greater than) para buscar productos con un inventario superior a 50 unidades.
```js
db.productos.find(
  { stock: { $gt: 50 } }
)
```

---

## 4. Consultas de Agregación (Estadísticas)

El framework de agregación permite procesar datos y devolver resultados calculados. Es ideal para obtener inteligencia de negocio.

### Contar el total de productos
La etapa `$count` cuenta el número de documentos que pasan por ella y devuelve el total en un campo con el nombre especificado.
```js
db.productos.aggregate([
  { 
    $count: "total_productos" 
  }
])
```
**Resultado:** `[ { total_productos: 100 } ]`
> **Análisis del resultado:**
> Este número es la métrica más básica para entender la escala del inventario. Confirma que los 100 productos iniciales están en la base de datos, sirviendo como una verificación rápida de la integridad de los datos.

### Sumar el valor total del inventario
Esta agregación agrupa todos los documentos (`_id: null`) y calcula una nueva métrica. Para cada producto, multiplica su `precio` por su `stock` y luego suma (`$sum`) todos estos valores.
```js
db.productos.aggregate([
  {
    $group: {
      _id: null,
      valor_total_inventario: {
        $sum: { $multiply: ["$precio", "$stock"] }
      }
    }
  }
])
```
**Resultado:** `[ { _id: null, valor_total_inventario: 80860.85 } ]` (El valor exacto depende de los datos actuales)
> **Análisis del resultado:**
> Esta cifra es crucial para la gestión financiera y contable. Representa el valor monetario de todo el inventario disponible.

### Obtener el precio promedio de los productos
Calcula el precio promedio de todos los productos en la colección usando el operador `$avg`.
```js
db.productos.aggregate([
  {
    $group: {
      _id: null,
      precio_promedio: { $avg: "$precio" }
    }
  }
])
```
**Resultado:** `[ { _id: null, precio_promedio: 151.08 } ]` (El valor exacto depende de los datos actuales)
> **Análisis del resultado:**
> El precio promedio ayuda a definir el posicionamiento de la tienda. ¿Es una tienda de gama alta, económica o intermedia? Comparar este promedio entre diferentes categorías (ej. `PC Gaming` vs. `Juego`) puede revelar qué segmentos de productos son más costosos.

### Precio mínimo y máximo
En una sola operación, obtiene el producto más barato (`$min`) y el más caro (`$max`) de todo el catálogo.
```js
db.productos.aggregate([
  {
    $group: {
      _id: null,
      precio_minimo: { $min: "$precio" },
      precio_maximo: { $max: "$precio" }
    }
  }
])
```
**Resultado:** `[ { _id: null, precio_minimo: 4.99, precio_maximo: 999.99 } ]`
> **Análisis del resultado:**
> Este rango define completo el mercado que abarca la tienda. El precio mínimo es el punto de entrada para clientes con bajo presupuesto, mientras que el máximo resalta los productos premium. 