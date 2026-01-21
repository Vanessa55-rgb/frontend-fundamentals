# Virtual DOM

## ¿Qué es el Virtual DOM?

El Virtual DOM es una representación ligera en memoria del DOM real. Es un objeto JavaScript que replica la estructura del DOM, permitiendo realizar operaciones de manera más eficiente antes de actualizar el DOM real.

## El Problema: Manipulación del DOM Real

```javascript
// Manipulación directa del DOM (costosa)
const container = document.querySelector('#app');

// Cada operación causa reflow/repaint
container.innerHTML = '<h1>Título</h1>';           // Reflow 1
container.innerHTML += '<p>Párrafo 1</p>';         // Reflow 2
container.innerHTML += '<p>Párrafo 2</p>';         // Reflow 3
container.innerHTML += '<button>Click</button>';   // Reflow 4

// Total: 4 reflows (muy ineficiente)
```

### ¿Por qué es costoso manipular el DOM?

1. **Reflow**: Recalcular posiciones y dimensiones de elementos
2. **Repaint**: Redibujar elementos en pantalla
3. **Sincronización**: El navegador debe sincronizar cambios con el motor de renderizado
4. **Árbol grande**: El DOM puede ser enorme con miles de nodos

```javascript
// Ejemplo: Actualizar 1000 elementos
const lista = document.querySelector('#lista');

for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  lista.appendChild(li); // 1000 operaciones DOM
}
// Muy lento: cada appendChild toca el DOM real
```

---

## La Solución: Virtual DOM

### Concepto

```
Estado Cambia → Nuevo Virtual DOM → Algoritmo Diff → Actualización Mínima del DOM Real
```

El Virtual DOM actúa como una capa intermedia:

1. **Cambio de estado**: El estado de la aplicación cambia
2. **Nuevo Virtual DOM**: Se crea una nueva representación virtual
3. **Diffing**: Se compara el Virtual DOM anterior con el nuevo
4. **Reconciliación**: Solo se actualizan las diferencias en el DOM real

### Representación Simplificada

```javascript
// DOM Real
<div id="app">
  <h1 class="title">Hola</h1>
  <p>Texto</p>
</div>

// Virtual DOM (objeto JavaScript)
{
  type: 'div',
  props: { id: 'app' },
  children: [
    {
      type: 'h1',
      props: { className: 'title' },
      children: ['Hola']
    },
    {
      type: 'p',
      props: {},
      children: ['Texto']
    }
  ]
}
```

---

## Cómo Funciona el Virtual DOM

### Paso 1: Crear Virtual DOM

```javascript
// Función para crear nodos virtuales
function createElement(type, props = {}, ...children) {
  return {
    type,
    props,
    children: children.flat()
  };
}

// Uso
const vdom1 = createElement(
  'div',
  { id: 'app' },
  createElement('h1', { className: 'title' }, 'Hola Mundo'),
  createElement('p', {}, 'Este es un párrafo')
);

console.log(vdom1);
/*
{
  type: 'div',
  props: { id: 'app' },
  children: [
    {
      type: 'h1',
      props: { className: 'title' },
      children: ['Hola Mundo']
    },
    {
      type: 'p',
      props: {},
      children: ['Este es un párrafo']
    }
  ]
}
*/
```

### Paso 2: Convertir Virtual DOM a DOM Real

```javascript
function render(vnode) {
  // Si es texto
  if (typeof vnode === 'string' || typeof vnode === 'number') {
    return document.createTextNode(vnode);
  }

  // Crear elemento
  const element = document.createElement(vnode.type);

  // Aplicar propiedades
  Object.keys(vnode.props).forEach(key => {
    if (key === 'className') {
      element.className = vnode.props[key];
    } else if (key === 'onClick') {
      element.addEventListener('click', vnode.props[key]);
    } else {
      element.setAttribute(key, vnode.props[key]);
    }
  });

  // Renderizar hijos recursivamente
  vnode.children.forEach(child => {
    element.appendChild(render(child));
  });

  return element;
}

// Uso
const domElement = render(vdom1);
document.getElementById('root').appendChild(domElement);
```

### Paso 3: Algoritmo de Diffing

El algoritmo de diffing compara dos árboles virtuales y determina qué cambios son necesarios.

```javascript
function diff(oldVNode, newVNode) {
  // Caso 1: Nodo fue removido
  if (!newVNode) {
    return { type: 'REMOVE' };
  }

  // Caso 2: Nodo nuevo fue agregado
  if (!oldVNode) {
    return { type: 'CREATE', vnode: newVNode };
  }

  // Caso 3: Nodo cambió de tipo
  if (oldVNode.type !== newVNode.type) {
    return { type: 'REPLACE', vnode: newVNode };
  }

  // Caso 4: Es texto y cambió
  if (typeof oldVNode === 'string' || typeof oldVNode === 'number') {
    if (oldVNode !== newVNode) {
      return { type: 'UPDATE_TEXT', text: newVNode };
    }
    return null;
  }

  // Caso 5: Props cambiaron
  const propsPatches = diffProps(oldVNode.props, newVNode.props);

  // Caso 6: Hijos cambiaron (recursivo)
  const childrenPatches = diffChildren(oldVNode.children, newVNode.children);

  return {
    type: 'UPDATE',
    propsPatches,
    childrenPatches
  };
}

function diffProps(oldProps, newProps) {
  const patches = [];

  // Props que cambiaron o se agregaron
  Object.keys(newProps).forEach(key => {
    if (oldProps[key] !== newProps[key]) {
      patches.push({
        type: 'SET_PROP',
        key,
        value: newProps[key]
      });
    }
  });

  // Props que se removieron
  Object.keys(oldProps).forEach(key => {
    if (!(key in newProps)) {
      patches.push({
        type: 'REMOVE_PROP',
        key
      });
    }
  });

  return patches;
}

function diffChildren(oldChildren, newChildren) {
  const patches = [];
  const length = Math.max(oldChildren.length, newChildren.length);

  for (let i = 0; i < length; i++) {
    patches.push(diff(oldChildren[i], newChildren[i]));
  }

  return patches;
}
```

### Paso 4: Aplicar Cambios 

```javascript
function patch(parent, patches, index = 0) {
  if (!patches) return;

  const element = parent.childNodes[index];

  switch (patches.type) {
    case 'CREATE':
      parent.appendChild(render(patches.vnode));
      break;

    case 'REMOVE':
      parent.removeChild(element);
      break;

    case 'REPLACE':
      parent.replaceChild(render(patches.vnode), element);
      break;

    case 'UPDATE_TEXT':
      element.textContent = patches.text;
      break;

    case 'UPDATE':
      // Aplicar cambios de props
      patches.propsPatches.forEach(propPatch => {
        if (propPatch.type === 'SET_PROP') {
          if (propPatch.key === 'className') {
            element.className = propPatch.value;
          } else {
            element.setAttribute(propPatch.key, propPatch.value);
          }
        } else if (propPatch.type === 'REMOVE_PROP') {
          element.removeAttribute(propPatch.key);
        }
      });

      // Aplicar cambios de hijos recursivamente
      patches.childrenPatches.forEach((childPatch, i) => {
        patch(element, childPatch, i);
      });
      break;
  }
}
```

---

## Ejemplo Completo: Mini Virtual DOM

```javascript
class VirtualDOM {
  constructor() {
    this.currentVTree = null;
  }

  // Crear elemento virtual
  h(type, props = {}, ...children) {
    return {
      type,
      props,
      children: children.flat()
    };
  }

  // Renderizar virtual node a DOM real
  render(vnode) {
    if (typeof vnode === 'string' || typeof vnode === 'number') {
      return document.createTextNode(vnode);
    }

    const element = document.createElement(vnode.type);

    // Props
    Object.entries(vnode.props).forEach(([key, value]) => {
      if (key.startsWith('on')) {
        const event = key.slice(2).toLowerCase();
        element.addEventListener(event, value);
      } else if (key === 'className') {
        element.className = value;
      } else if (key === 'style' && typeof value === 'object') {
        Object.assign(element.style, value);
      } else {
        element.setAttribute(key, value);
      }
    });

    // Children
    vnode.children
      .map(child => this.render(child))
      .forEach(childElement => element.appendChild(childElement));

    return element;
  }

  // Diff
  diff(oldVNode, newVNode) {
    if (!oldVNode) return { type: 'CREATE', vnode: newVNode };
    if (!newVNode) return { type: 'REMOVE' };

    if (typeof oldVNode !== typeof newVNode || 
        (typeof oldVNode === 'object' && oldVNode.type !== newVNode.type)) {
      return { type: 'REPLACE', vnode: newVNode };
    }

    if (typeof oldVNode === 'string' || typeof oldVNode === 'number') {
      if (oldVNode !== newVNode) {
        return { type: 'UPDATE_TEXT', text: newVNode };
      }
      return null;
    }

    return {
      type: 'UPDATE',
      props: this.diffProps(oldVNode.props, newVNode.props),
      children: this.diffChildren(oldVNode.children, newVNode.children)
    };
  }

  diffProps(oldProps, newProps) {
    const patches = [];

    Object.keys({ ...oldProps, ...newProps }).forEach(key => {
      const oldValue = oldProps[key];
      const newValue = newProps[key];

      if (newValue === undefined) {
        patches.push({ type: 'REMOVE', key });
      } else if (oldValue === undefined || oldValue !== newValue) {
        patches.push({ type: 'SET', key, value: newValue });
      }
    });

    return patches;
  }

  diffChildren(oldChildren, newChildren) {
    const patches = [];
    const length = Math.max(oldChildren.length, newChildren.length);

    for (let i = 0; i < length; i++) {
      patches.push(this.diff(oldChildren[i], newChildren[i]));
    }

    return patches;
  }

  // Patch
  patch(parent, patches, index = 0) {
    if (!patches) return;

    const element = parent.childNodes[index];

    switch (patches.type) {
      case 'CREATE':
        parent.appendChild(this.render(patches.vnode));
        break;

      case 'REMOVE':
        parent.removeChild(element);
        break;

      case 'REPLACE':
        parent.replaceChild(this.render(patches.vnode), element);
        break;

      case 'UPDATE_TEXT':
        element.textContent = patches.text;
        break;

      case 'UPDATE':
        // Aplicar props
        patches.props.forEach(propPatch => {
          if (propPatch.type === 'SET') {
            if (propPatch.key === 'className') {
              element.className = propPatch.value;
            } else if (propPatch.key.startsWith('on')) {
              const event = propPatch.key.slice(2).toLowerCase();
              element.addEventListener(event, propPatch.value);
            } else {
              element.setAttribute(propPatch.key, propPatch.value);
            }
          } else if (propPatch.type === 'REMOVE') {
            element.removeAttribute(propPatch.key);
          }
        });

        // Aplicar children
        patches.children.forEach((childPatch, i) => {
          this.patch(element, childPatch, i);
        });
        break;
    }
  }

  // Actualizar vista
  update(container, newVTree) {
    const patches = this.diff(this.currentVTree, newVTree);

    if (!this.currentVTree) {
      container.appendChild(this.render(newVTree));
    } else {
      this.patch(container, patches, 0);
    }

    this.currentVTree = newVTree;
  }
}

// Uso
const vdom = new VirtualDOM();
const container = document.getElementById('app');

// Estado inicial
let count = 0;

function render() {
  return vdom.h(
    'div',
    { className: 'counter' },
    vdom.h('h1', {}, `Contador: ${count}`),
    vdom.h('button', {
      onClick: () => {
        count++;
        update();
      }
    }, 'Incrementar'),
    vdom.h('button', {
      onClick: () => {
        count--;
        update();
      }
    }, 'Decrementar')
  );
}

function update() {
  const newVTree = render();
  vdom.update(container, newVTree);
}

// Primera renderización
update();
```

---

## Ventajas del Virtual DOM

### 1. Rendimiento Optimizado

```javascript
// Sin Virtual DOM: múltiples reflows
function updateWithoutVDOM() {
  const list = document.querySelector('#list');
  list.innerHTML = ''; // Reflow 1
  
  for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    list.appendChild(li); // Reflow en cada iteración
  }
}

// Con Virtual DOM: un solo reflow
function updateWithVDOM() {
  const newVTree = createVirtualList(1000);
  vdom.update(container, newVTree); // Calcula diferencias
  // Solo aplica cambios necesarios en un batch
}
```

### 2. Abstracción y Facilidad

```javascript
// DOM directo: imperativo
const div = document.createElement('div');
div.className = 'card';
const h1 = document.createElement('h1');
h1.textContent = 'Título';
div.appendChild(h1);
document.body.appendChild(div);

// Virtual DOM: declarativo
const card = h('div', { className: 'card' },
  h('h1', {}, 'Título')
);
```

### 3. Batching de Actualizaciones

```javascript
// Virtual DOM agrupa cambios
function multipleUpdates() {
  setTitle('Nuevo título');      // Cambio 1
  setDescription('Nueva desc');  // Cambio 2
  setCount(count + 1);          // Cambio 3
  
  // Virtual DOM calcula todos los cambios
  // y actualiza el DOM real una sola vez
}
```

### 4. Cross-platform

El Virtual DOM es solo JavaScript, puede renderizar en diferentes plataformas:

```javascript
// Web: React DOM
ReactDOM.render(<App />, document.getElementById('root'));

// Mobile: React Native
AppRegistry.registerComponent('MyApp', () => App);

// VR: React 360
ReactVR.render(<VRApp />, document.getElementById('root'));
```

---

## Desventajas y Consideraciones

### 1. Memoria Adicional

```javascript
// Virtual DOM mantiene dos árboles en memoria
const oldVTree = { /* árbol anterior */ };
const newVTree = { /* árbol nuevo */ };

// Para aplicaciones muy grandes, esto puede ser costoso
```

### 2. Overhead del Algoritmo

```javascript
// El diffing tiene un costo computacional
// O(n³) en el peor caso, optimizado a O(n) en la práctica

// Para cambios muy simples, puede ser más lento que DOM directo
document.getElementById('counter').textContent = count; // Más rápido
// vs
vdom.update(container, newVTree); // Más lento (pero más escalable)
```

### 3. No Siempre es Más Rápido

```javascript
// Caso donde DOM directo es mejor:
// Actualización de un solo elemento conocido
element.textContent = newValue; // Instantáneo

// Caso donde Virtual DOM es mejor:
// Actualización de múltiples elementos con lógica compleja
renderComplexList(data); // Virtual DOM calcula diferencias óptimas
```

---

## Comparación: Virtual DOM vs DOM Real

| Aspecto | DOM Real | Virtual DOM |
|---------|----------|-------------|
| **Velocidad individual** | Más rápido | Overhead del diff |
| **Actualizaciones múltiples** | Lento (múltiples reflows) | Rápido (batch) |
| **Complejidad** | Alta (imperativo) | Baja (declarativo) |
| **Memoria** | Menos | Más (dos árboles) |
| **Debugging** | Directo | Abstracción extra |
| **Mantenibilidad** | Difícil | Fácil |
| **Escalabilidad** | Difícil | Excelente |

---

## Optimizaciones del Virtual DOM

### 1. Keys en Listas

```javascript
// ❌ Sin keys: React/Virtual DOM no sabe qué cambió
const list = items.map(item =>
  h('li', {}, item.name)
);

// ✅ Con keys: puede identificar elementos únicos
const list = items.map(item =>
  h('li', { key: item.id }, item.name)
);

// Beneficio:
// Sin key: eliminar item del medio requiere re-renderizar todos
// Con key: solo elimina el elemento específico
```

### 2. ShouldComponentUpdate (React)

```javascript
// React usa esto para evitar renders innecesarios
class MyComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Solo re-renderizar si cambió algo relevante
    return this.props.data !== nextProps.data;
  }
  
  render() {
    return <div>{this.props.data}</div>;
  }
}

// React.memo para componentes funcionales
const MyComponent = React.memo(({ data }) => {
  return <div>{data}</div>;
}, (prevProps, nextProps) => {
  // Retornar true si NO debe actualizar
  return prevProps.data === nextProps.data;
});
```

### 3. Fragmentos

```javascript
// ❌ Crea div innecesario
const element = h('div', {},
  h('p', {}, 'Párrafo 1'),
  h('p', {}, 'Párrafo 2')
);

// ✅ Usa fragmento (no crea nodo extra en DOM)
const element = h(Fragment, {},
  h('p', {}, 'Párrafo 1'),
  h('p', {}, 'Párrafo 2')
);
```

---

## Alternativas al Virtual DOM

### 1. Svelte (Sin Virtual DOM)

Svelte compila componentes a código imperativo eficiente en tiempo de build.

```javascript
// Svelte
<script>
  let count = 0;
</script>

<button on:click={() => count++}>
  Contador: {count}
</button>

// Compilado a:
function update() {
  button.textContent = `Contador: ${count}`;
}
```

**Ventajas**: Más rápido, menos bundle size
**Desventajas**: Menos flexible, requiere compilación

### 2. Solid.js (Fine-grained Reactivity)

```javascript
// Solid.js
const [count, setCount] = createSignal(0);

<button onClick={() => setCount(count() + 1)}>
  Contador: {count()}
</button>
```

**Ventajas**: Muy rápido, reactivo
**Desventajas**: Curva de aprendizaje

### 3. Lit (Template Literals)

```javascript
// Lit
class MyCounter extends LitElement {
  static properties = { count: {} };
  
  render() {
    return html`
      <button @click=${() => this.count++}>
        Contador: ${this.count}
      </button>
    `;
  }
}

```

## Recursos Adicionales

- [React Virtual DOM](https://reactjs.org/docs/faq-internals.html)
- [Virtual DOM Implementation](https://github.com/Matt-Esch/virtual-dom)
- [How Virtual DOM works](https://www.codecademy.com/article/react-virtual-dom)
- [Svelte: Virtual DOM is pure overhead](https://svelte.dev/blog/virtual-dom-is-pure-overhead)
