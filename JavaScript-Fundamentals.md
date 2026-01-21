# # JavaScript Fundamentals

## ¿Qué es JavaScript?

JavaScript es un lenguaje de programación interpretado, de alto nivel, orientado a objetos y basado en prototipos. Es uno de los lenguajes fundamentales de la web, utilizado principalmente para crear interactividad en páginas web.

## Características Principales

- **Interpretado**: No necesita compilación previa
- **Tipado Dinámico**: Las variables pueden cambiar de tipo
- **Basado en Prototipos**: Herencia mediante prototipos
- **First-class Functions**: Las funciones son ciudadanos de primera clase
- **Event-Driven**: Programación basada en eventos
- **Multi-paradigma**: Soporta programación funcional, orientada a objetos e imperativa

---

## 1. Tipos de Datos

### Tipos Primitivos

JavaScript tiene 7 tipos primitivos:

```javascript
// 1. String (cadena de texto)
let nombre = "Juan";
let apellido = 'Pérez';
let mensaje = `Hola, ${nombre}`; // Template literals

// 2. Number (número)
let entero = 42;
let decimal = 3.14;
let negativo = -100;
let infinito = Infinity;
let noEsNumero = NaN; // Not a Number

// 3. Boolean (booleano)
let verdadero = true;
let falso = false;

// 4. Undefined (indefinido)
let sinDefinir;
console.log(sinDefinir); // undefined

// 5. Null (nulo - ausencia intencional de valor)
let vacio = null;

// 6. Symbol (único e inmutable)
let simbolo = Symbol('descripcion');
let otroSimbolo = Symbol('descripcion');
console.log(simbolo === otroSimbolo); // false

// 7. BigInt (números enteros grandes)
let numeroGrande = 9007199254740991n;
let otraForma = BigInt(9007199254740991);
```

### Tipos de Referencia (Objetos)

```javascript
// Object (objeto)
let persona = {
  nombre: "Ana",
  edad: 25,
  ciudad: "Medellín"
};

// Array (arreglo)
let numeros = [1, 2, 3, 4, 5];
let mixto = [1, "dos", true, null, { nombre: "objeto" }];

// Function (función)
function saludar() {
  return "Hola";
}

// Date (fecha)
let ahora = new Date();

// RegExp (expresión regular)
let patron = /[a-z]+/;
```

### Verificación de Tipos

```javascript
console.log(typeof "hola");        // "string"
console.log(typeof 42);            // "number"
console.log(typeof true);          // "boolean"
console.log(typeof undefined);     // "undefined"
console.log(typeof null);          // "object" (bug histórico de JS)
console.log(typeof {});            // "object"
console.log(typeof []);            // "object"
console.log(typeof function(){}); // "function"

// Para arrays, usar:
console.log(Array.isArray([]));    // true
```

---

## 2. Variables

JavaScript tiene tres formas de declarar variables:

### var (Evitar en código moderno)

```javascript
var x = 10;
var x = 20; // Se puede redeclarar
x = 30;     // Se puede reasignar

// Problemas con var:
// 1. Scope de función (no de bloque)
function ejemploVar() {
  if (true) {
    var mensaje = "Hola";
  }
  console.log(mensaje); // "Hola" - accesible fuera del bloque
}

// 2. Hoisting (elevación)
console.log(edad); // undefined (no error)
var edad = 25;
```

### let (Recomendado para variables)

```javascript
let y = 10;
// let y = 20; // Error: no se puede redeclarar
y = 20;        // Se puede reasignar

// Scope de bloque
function ejemploLet() {
  if (true) {
    let mensaje = "Hola";
    console.log(mensaje); // "Hola"
  }
  // console.log(mensaje); // Error: mensaje no está definido
}

// Temporal Dead Zone
// console.log(nombre); // Error: Cannot access before initialization
let nombre = "Juan";
```

### const (Recomendado para constantes)

```javascript
const PI = 3.14159;
// PI = 3.14; // Error: no se puede reasignar

// const requiere inicialización
// const z; // Error: falta inicialización

// IMPORTANTE: const no hace inmutables los objetos
const persona = { nombre: "Ana" };
persona.nombre = "María"; // Permitido
persona.edad = 25;        // Permitido
// persona = {};          // Error: no se puede reasignar

// Para hacer inmutables los objetos:
const personaInmutable = Object.freeze({ nombre: "Pedro" });
personaInmutable.nombre = "Juan"; // No tiene efecto (en strict mode da error)
```

### Mejores Prácticas

```javascript
// 1. Usar const por defecto
const API_URL = "https://api.ejemplo.com";
const MAX_INTENTOS = 3;

// 2. Usar let cuando necesites reasignar
let contador = 0;
contador++;

// 3. No usar var
// var x = 10; // ❌ Evitar

// 4. Nombres descriptivos
const precioProducto = 100;           // ✅ Bueno
const p = 100;                        // ❌ Malo

// 5. camelCase para variables y funciones
const nombreCompleto = "Juan Pérez";
const calcularTotal = () => {};

// 6. UPPER_CASE para constantes globales
const DIAS_SEMANA = 7;
const COLOR_PRIMARIO = "#FF0000";
```

---

## 3. Operadores

### Operadores Aritméticos

```javascript
let a = 10;
let b = 3;

console.log(a + b);  // 13 - Suma
console.log(a - b);  // 7  - Resta
console.log(a * b);  // 30 - Multiplicación
console.log(a / b);  // 3.333... - División
console.log(a % b);  // 1  - Módulo (resto)
console.log(a ** b); // 1000 - Exponenciación

// Incremento y decremento
let x = 5;
console.log(x++);    // 5 (post-incremento)
console.log(x);      // 6
console.log(++x);    // 7 (pre-incremento)
console.log(x--);    // 7 (post-decremento)
console.log(--x);    // 5 (pre-decremento)
```

### Operadores de Comparación

```javascript
// Igualdad (compara valor, con conversión de tipo)
console.log(5 == "5");     // true
console.log(0 == false);   // true

// Igualdad estricta (compara valor y tipo)
console.log(5 === "5");    // false
console.log(5 === 5);      // true

// Desigualdad
console.log(5 != "5");     // false
console.log(5 !== "5");    // true

// Comparaciones
console.log(10 > 5);       // true
console.log(10 < 5);       // false
console.log(10 >= 10);     // true
console.log(10 <= 5);      // false
```

### Operadores Lógicos

```javascript
// AND (&&) - Todas las condiciones deben ser verdaderas
console.log(true && true);   // true
console.log(true && false);  // false

// OR (||) - Al menos una condición debe ser verdadera
console.log(true || false);  // true
console.log(false || false); // false

// NOT (!) - Invierte el valor booleano
console.log(!true);          // false
console.log(!false);         // true

// Ejemplos prácticos
let edad = 20;
let tieneLicencia = true;

if (edad >= 18 && tieneLicencia) {
  console.log("Puede conducir");
}

// Short-circuit evaluation
let nombre = null;
let nombreDefault = nombre || "Anónimo";
console.log(nombreDefault); // "Anónimo"
```

### Operador Ternario

```javascript
// condición ? valorSiVerdadero : valorSiFalso
let edad = 18;
let mensaje = edad >= 18 ? "Mayor de edad" : "Menor de edad";
console.log(mensaje); // "Mayor de edad"

// Anidados (usar con moderación)
let nota = 85;
let calificacion = nota >= 90 ? "A" :
                   nota >= 80 ? "B" :
                   nota >= 70 ? "C" : "F";
```

### Operadores Modernos

```javascript
// Nullish Coalescing (??) - Solo null o undefined
let valor = null;
console.log(valor ?? "default");  // "default"
console.log(0 ?? "default");      // 0
console.log("" ?? "default");     // ""

// Optional Chaining (?.)
let usuario = {
  nombre: "Ana",
  direccion: {
    ciudad: "Medellín"
  }
};

console.log(usuario?.direccion?.ciudad);    // "Medellín"
console.log(usuario?.telefono?.numero);     // undefined (no error)

// Spread Operator (...)
let arr1 = [1, 2, 3];
let arr2 = [...arr1, 4, 5];  // [1, 2, 3, 4, 5]

let obj1 = { a: 1, b: 2 };
let obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }
```

---

## 4. Funciones

### Declaración de Funciones

```javascript
// 1. Function Declaration (se eleva/hoisting)
function sumar(a, b) {
  return a + b;
}

console.log(sumar(5, 3)); // 8

// 2. Function Expression (no se eleva)
const restar = function(a, b) {
  return a - b;
};

console.log(restar(10, 4)); // 6

// 3. Arrow Function (funciones flecha)
const multiplicar = (a, b) => a * b;

console.log(multiplicar(4, 5)); // 20

// Arrow function con múltiples líneas
const dividir = (a, b) => {
  if (b === 0) {
    return "No se puede dividir por cero";
  }
  return a / b;
};
```

### Parámetros de Funciones

```javascript
// Parámetros por defecto
function saludar(nombre = "Usuario", saludo = "Hola") {
  return `${saludo}, ${nombre}!`;
}

console.log(saludar());                    // "Hola, Usuario!"
console.log(saludar("Ana"));               // "Hola, Ana!"
console.log(saludar("Ana", "Buenos días")); // "Buenos días, Ana!"

// Rest Parameters (parámetros rest)
function sumarTodos(...numeros) {
  return numeros.reduce((total, num) => total + num, 0);
}

console.log(sumarTodos(1, 2, 3, 4, 5)); // 15

// Destructuring en parámetros
function crearUsuario({ nombre, edad, ciudad = "Desconocida" }) {
  return `${nombre}, ${edad} años, de ${ciudad}`;
}

const usuario = { nombre: "Juan", edad: 30 };
console.log(crearUsuario(usuario)); // "Juan, 30 años, de Desconocida"
```

### Funciones de Orden Superior

```javascript
// Funciones que reciben o retornan otras funciones

// 1. Función que recibe otra función
function ejecutarOperacion(a, b, operacion) {
  return operacion(a, b);
}

const suma = (x, y) => x + y;
const resta = (x, y) => x - y;

console.log(ejecutarOperacion(10, 5, suma));  // 15
console.log(ejecutarOperacion(10, 5, resta)); // 5

// 2. Función que retorna otra función (Closure)
function multiplicadorPor(factor) {
  return function(numero) {
    return numero * factor;
  };
}

const duplicar = multiplicadorPor(2);
const triplicar = multiplicadorPor(3);

console.log(duplicar(5));   // 10
console.log(triplicar(5));  // 15
```

### Métodos de Arrays (Funciones de Orden Superior)

```javascript
const numeros = [1, 2, 3, 4, 5];

// map - Transforma cada elemento
const dobles = numeros.map(num => num * 2);
console.log(dobles); // [2, 4, 6, 8, 10]

// filter - Filtra elementos que cumplen una condición
const pares = numeros.filter(num => num % 2 === 0);
console.log(pares); // [2, 4]

// reduce - Reduce a un solo valor
const suma = numeros.reduce((total, num) => total + num, 0);
console.log(suma); // 15

// forEach - Ejecuta una función por cada elemento
numeros.forEach(num => console.log(num * 2));

// find - Encuentra el primer elemento que cumple la condición
const primerPar = numeros.find(num => num % 2 === 0);
console.log(primerPar); // 2

// some - Verifica si al menos uno cumple la condición
const hayPares = numeros.some(num => num % 2 === 0);
console.log(hayPares); // true

// every - Verifica si todos cumplen la condición
const todosPares = numeros.every(num => num % 2 === 0);
console.log(todosPares); // false
```

### Closures (Clausuras)

```javascript
// Un closure es cuando una función interna tiene acceso
// a variables de su función externa

function crearContador() {
  let cuenta = 0; // Variable privada
  
  return {
    incrementar: function() {
      cuenta++;
      return cuenta;
    },
    decrementar: function() {
      cuenta--;
      return cuenta;
    },
    obtenerValor: function() {
      return cuenta;
    }
  };
}

const contador = crearContador();
console.log(contador.incrementar()); // 1
console.log(contador.incrementar()); // 2
console.log(contador.decrementar()); // 1
console.log(contador.obtenerValor()); // 1
// console.log(cuenta); // Error: cuenta no está definida
```

### IIFE (Immediately Invoked Function Expression)

```javascript
// Función que se ejecuta inmediatamente
(function() {
  const mensaje = "Esta función se ejecuta inmediatamente";
  console.log(mensaje);
})();

// Con arrow function
(() => {
  console.log("IIFE con arrow function");
})();

// Útil para crear scope privado
const modulo = (function() {
  let privado = "No accesible desde fuera";
  
  return {
    publico: "Sí accesible",
    obtenerPrivado: function() {
      return privado;
    }
  };
})();

console.log(modulo.publico);           // "Sí accesible"
console.log(modulo.obtenerPrivado());  // "No accesible desde fuera"
// console.log(modulo.privado);        // undefined
```

---

## 5. Objetos y Arrays

### Objetos

```javascript
// Creación de objetos
const persona = {
  nombre: "Ana",
  edad: 25,
  ciudad: "Medellín",
  saludar: function() {
    return `Hola, soy ${this.nombre}`;
  }
};

// Acceso a propiedades
console.log(persona.nombre);      // "Ana"
console.log(persona["edad"]);     // 25

// Modificar propiedades
persona.edad = 26;
persona["ciudad"] = "Bogotá";

// Agregar propiedades
persona.email = "ana@ejemplo.com";

// Eliminar propiedades
delete persona.ciudad;

// Métodos modernos de objetos
const claves = Object.keys(persona);      // ["nombre", "edad", "email", "saludar"]
const valores = Object.values(persona);   // ["Ana", 26, "ana@ejemplo.com", function]
const entradas = Object.entries(persona); // [["nombre", "Ana"], ["edad", 26], ...]

// Destructuring
const { nombre, edad } = persona;
console.log(nombre); // "Ana"
console.log(edad);   // 26

// Con alias
const { nombre: nombrePersona, edad: edadPersona } = persona;
```

### Arrays

```javascript
// Creación
const frutas = ["manzana", "banana", "naranja"];
const numeros = new Array(1, 2, 3, 4, 5);

// Acceso
console.log(frutas[0]);     // "manzana"
console.log(frutas.length); // 3

// Métodos principales
frutas.push("uva");         // Agrega al final
frutas.pop();               // Elimina del final
frutas.unshift("pera");     // Agrega al inicio
frutas.shift();             // Elimina del inicio

// slice - Copia una porción (no modifica el original)
const algunasFrutas = frutas.slice(1, 3);

// splice - Modifica el array original
frutas.splice(1, 1, "kiwi"); // Elimina 1 elemento en posición 1 y agrega "kiwi"

// concat - Combina arrays
const verduras = ["lechuga", "tomate"];
const alimentos = frutas.concat(verduras);

// join - Convierte a string
const texto = frutas.join(", ");

// includes - Verifica si existe
console.log(frutas.includes("banana")); // true o false

// indexOf - Encuentra el índice
const indice = frutas.indexOf("naranja");

// Destructuring de arrays
const [primera, segunda, ...resto] = frutas;
console.log(primera);  // Primera fruta
console.log(resto);    // Array con el resto
```

---

## 6. Control de Flujo

### Condicionales

```javascript
// if...else
const edad = 18;

if (edad >= 18) {
  console.log("Mayor de edad");
} else if (edad >= 13) {
  console.log("Adolescente");
} else {
  console.log("Menor de edad");
}

// switch
const dia = "lunes";

switch (dia) {
  case "lunes":
  case "martes":
  case "miércoles":
  case "jueves":
  case "viernes":
    console.log("Día laboral");
    break;
  case "sábado":
  case "domingo":
    console.log("Fin de semana");
    break;
  default:
    console.log("Día no válido");
}
```

### Bucles

```javascript
// for
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// while
let contador = 0;
while (contador < 5) {
  console.log(contador);
  contador++;
}

// do...while (ejecuta al menos una vez)
let j = 0;
do {
  console.log(j);
  j++;
} while (j < 5);

// for...of (para arrays)
const frutas = ["manzana", "banana", "naranja"];
for (const fruta of frutas) {
  console.log(fruta);
}

// for...in (para objetos - itera sobre claves)
const persona = { nombre: "Ana", edad: 25 };
for (const clave in persona) {
  console.log(`${clave}: ${persona[clave]}`);
}

// break y continue
for (let i = 0; i < 10; i++) {
  if (i === 5) break;     // Sale del bucle
  if (i % 2 === 0) continue; // Salta a la siguiente iteración
  console.log(i);
}
```

---

## 7. Template Literals

```javascript
const nombre = "Juan";
const edad = 30;

// String concatenación tradicional
const mensaje1 = "Hola, mi nombre es " + nombre + " y tengo " + edad + " años.";

// Template literals (preferido)
const mensaje2 = `Hola, mi nombre es ${nombre} y tengo ${edad} años.`;

// Multilínea
const html = `
  <div>
    <h1>${nombre}</h1>
    <p>Edad: ${edad}</p>
  </div>
`;

// Expresiones
const precio = 100;
const impuesto = 0.19;
console.log(`Total: $${precio * (1 + impuesto)}`);

// Tagged templates
function etiqueta(strings, ...valores) {
  console.log(strings); // Partes literales
  console.log(valores); // Valores interpolados
}

etiqueta`Hola ${nombre}, tienes ${edad} años`;
```

---

## 8. Manejo de Errores

```javascript
// try...catch...finally
function dividir(a, b) {
  try {
    if (b === 0) {
      throw new Error("No se puede dividir por cero");
    }
    return a / b;
  } catch (error) {
    console.error("Error:", error.message);
    return null;
  } finally {
    console.log("Operación finalizada");
  }
}

console.log(dividir(10, 2));  // 5
console.log(dividir(10, 0));  // null

// Errores personalizados
class ErrorPersonalizado extends Error {
  constructor(mensaje) {
    super(mensaje);
    this.name = "ErrorPersonalizado";
  }
}

function validarEdad(edad) {
  if (edad < 0) {
    throw new ErrorPersonalizado("La edad no puede ser negativa");
  }
  return edad;
}

try {
  validarEdad(-5);
} catch (error) {
  if (error instanceof ErrorPersonalizado) {
    console.log("Error de validación:", error.message);
  }
}
```

---

## 9. Mejores Prácticas

```javascript
// 1. Usar const por defecto, let cuando sea necesario
const PI = 3.14159;
let contador = 0;

// 2. Nombres descriptivos
const calcularPrecioTotal = (precio, cantidad) => precio * cantidad;

// 3. Funciones pequeñas y específicas
// ❌ Malo
function procesarUsuario(usuario) {
  // 100 líneas de código
}

// ✅ Bueno
function validarUsuario(usuario) { /* ... */ }
function guardarUsuario(usuario) { /* ... */ }
function enviarEmailBienvenida(usuario) { /* ... */ }

// 4. Evitar anidar demasiado
// ❌ Malo
if (condicion1) {
  if (condicion2) {
    if (condicion3) {
      // Código
    }
  }
}

// ✅ Bueno (early return)
if (!condicion1) return;
if (!condicion2) return;
if (!condicion3) return;
// Código

// 5. Usar métodos de arrays en lugar de bucles
// ❌ Malo
const dobles = [];
for (let i = 0; i < numeros.length; i++) {
  dobles.push(numeros[i] * 2);
}

// ✅ Bueno
const dobles = numeros.map(n => n * 2);

// 6. Usar === en lugar de ==
if (valor === 5) { /* ... */ }

// 7. Comentar código complejo, no obvio
// Calcula el precio con descuento escalonado
const precioFinal = precio * (1 - obtenerDescuento(cantidad));
```

---

## Recursos Adicionales

- [MDN Web Docs - JavaScript](https://developer.mozilla.org/es/docs/Web/JavaScript)
- [JavaScript.info](https://javascript.info/)
- [Eloquent JavaScript (libro gratuito)](https://eloquentjavascript.net/)
