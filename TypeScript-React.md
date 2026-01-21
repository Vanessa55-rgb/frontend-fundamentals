# TypeScript con React

## ¿Qué es TypeScript?

TypeScript es un superset de JavaScript que añade tipos estáticos opcionales. Se compila a JavaScript puro y ayuda a detectar errores en tiempo de desarrollo.

```typescript
// JavaScript
function sumar(a, b) {
  return a + b;
}
sumar(5, "10"); // "510" - Error silencioso

// TypeScript
function sumar(a: number, b: number): number {
  return a + b;
}
sumar(5, "10"); // Error en tiempo de compilación ✅
```

---

## Configuración de TypeScript en React

### Crear proyecto con TypeScript

```bash
# Create React App
npx create-react-app mi-app --template typescript

# Vite
npm create vite@latest mi-app -- --template react-ts

# Next.js
npx create-next-app@latest mi-app --typescript
```

### Migrar proyecto existente

```bash
# Instalar TypeScript
npm install --save-dev typescript @types/react @types/react-dom

# Crear tsconfig.json
npx tsc --init
```

### tsconfig.json básico

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

## Tipos Básicos en TypeScript

### Tipos Primitivos

```typescript
// String
let nombre: string = "Juan";
let apellido: string = 'Pérez';
let template: string = `Hola ${nombre}`;

// Number
let edad: number = 25;
let precio: number = 19.99;
let hex: number = 0xf00d;

// Boolean
let activo: boolean = true;
let completado: boolean = false;

// Null y Undefined
let nulo: null = null;
let indefinido: undefined = undefined;

// Any (evitar cuando sea posible)
let cualquierCosa: any = "texto";
cualquierCosa = 123;
cualquierCosa = true;

// Unknown (más seguro que any)
let desconocido: unknown = "texto";
if (typeof desconocido === "string") {
  console.log(desconocido.toUpperCase()); // OK
}

// Never (funciones que nunca retornan)
function error(mensaje: string): never {
  throw new Error(mensaje);
}

// Void (funciones sin retorno)
function log(mensaje: string): void {
  console.log(mensaje);
}
```

### Arrays y Tuplas

```typescript
// Arrays
let numeros: number[] = [1, 2, 3];
let nombres: Array<string> = ["Ana", "Luis"];
let mixto: (string | number)[] = [1, "dos", 3];

// Tuplas
let persona: [string, number] = ["Juan", 30];
let rgb: [number, number, number] = [255, 0, 0];

// Array de objetos
let usuarios: { nombre: string; edad: number }[] = [
  { nombre: "Ana", edad: 25 },
  { nombre: "Luis", edad: 30 }
];
```

### Objetos

```typescript
// Tipo objeto básico
let usuario: {
  nombre: string;
  edad: number;
  activo: boolean;
} = {
  nombre: "Juan",
  edad: 30,
  activo: true
};

// Propiedades opcionales
let config: {
  url: string;
  puerto?: number; // Opcional
  timeout?: number;
} = {
  url: "localhost"
};

// Propiedades de solo lectura
let punto: {
  readonly x: number;
  readonly y: number;
} = {
  x: 10,
  y: 20
};
// punto.x = 30; // Error ✅

// Index signature (propiedades dinámicas)
let diccionario: {
  [key: string]: string;
} = {
  nombre: "Juan",
  apellido: "Pérez"
};
```

---

## Interfaces y Types

### Interfaces

```typescript
// Interface básica
interface Usuario {
  id: number;
  nombre: string;
  email: string;
  edad?: number; // Opcional
  readonly createdAt: Date; // Solo lectura
}

const usuario: Usuario = {
  id: 1,
  nombre: "Juan",
  email: "juan@example.com",
  createdAt: new Date()
};

// Extender interfaces
interface Admin extends Usuario {
  rol: "admin" | "superadmin";
  permisos: string[];
}

const admin: Admin = {
  id: 1,
  nombre: "María",
  email: "maria@example.com",
  createdAt: new Date(),
  rol: "admin",
  permisos: ["leer", "escribir"]
};

// Interfaces para funciones
interface Calculadora {
  (a: number, b: number): number;
}

const sumar: Calculadora = (a, b) => a + b;
const restar: Calculadora = (a, b) => a - b;

// Interfaces híbridas
interface Contador {
  (inicio: number): string;
  intervalo: number;
  reset(): void;
}

// Interfaces con métodos
interface Producto {
  id: number;
  nombre: string;
  precio: number;
  calcularDescuento(porcentaje: number): number;
  mostrarInfo(): string;
}

const producto: Producto = {
  id: 1,
  nombre: "Laptop",
  precio: 1000,
  calcularDescuento(porcentaje) {
    return this.precio * (porcentaje / 100);
  },
  mostrarInfo() {
    return `${this.nombre}: $${this.precio}`;
  }
};
```

### Type Aliases

```typescript
// Type básico
type ID = number | string;
type Nombre = string;

// Type para objetos
type Usuario = {
  id: ID;
  nombre: Nombre;
  email: string;
};

// Union types
type Estado = "pendiente" | "aprobado" | "rechazado";
type Resultado = Usuario | Error;

// Intersection types
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type UsuarioConFechas = Usuario & Timestamped;

const usuario: UsuarioConFechas = {
  id: 1,
  nombre: "Juan",
  email: "juan@example.com",
  createdAt: new Date(),
  updatedAt: new Date()
};

// Type para funciones
type OperacionMatematica = (a: number, b: number) => number;

const multiplicar: OperacionMatematica = (a, b) => a * b;

// Tipos genéricos
type Respuesta<T> = {
  data: T;
  error: string | null;
  loading: boolean;
};

const respuestaUsuario: Respuesta<Usuario> = {
  data: { id: 1, nombre: "Juan", email: "juan@example.com" },
  error: null,
  loading: false
};
```

### Interface vs Type

```typescript
// Interfaces pueden ser extendidas y fusionadas
interface Animal {
  nombre: string;
}

interface Animal {
  edad: number; // Fusión de declaraciones
}

const perro: Animal = {
  nombre: "Rex",
  edad: 5
};

// Types NO pueden fusionarse (más restrictivos)
type Vehiculo = {
  marca: string;
};

// type Vehiculo = {  // Error: Duplicate identifier
//   modelo: string;
// };

// Types son mejores para unions y tuples
type EstadoPeticion = "idle" | "loading" | "success" | "error";
type Coordenadas = [number, number];

// Interfaces son mejores para objetos que serán extendidos
interface ConfiguracionBase {
  debug: boolean;
}

interface ConfiguracionDesarrollo extends ConfiguracionBase {
  hotReload: boolean;
}
```

---

## TypeScript con Componentes React

### Componentes Funcionales

```typescript
import { FC, ReactNode } from 'react';

// Forma 1: Con React.FC (incluye children automáticamente)
const Saludo: FC = () => {
  return <h1>Hola Mundo</h1>;
};

// Forma 2: Con tipado explícito de props (RECOMENDADO)
interface SaludoProps {
  nombre: string;
  edad?: number;
}

const SaludoPersonalizado = ({ nombre, edad }: SaludoProps) => {
  return (
    <div>
      <h1>Hola {nombre}</h1>
      {edad && <p>Tienes {edad} años</p>}
    </div>
  );
};

// Con children
interface ContenedorProps {
  children: ReactNode;
  className?: string;
}

const Contenedor = ({ children, className = '' }: ContenedorProps) => {
  return <div className={`contenedor ${className}`}>{children}</div>;
};

// Con eventos
interface BotonProps {
  texto: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  disabled?: boolean;
  tipo?: 'button' | 'submit' | 'reset';
}

const Boton = ({ 
  texto, 
  onClick, 
  disabled = false,
  tipo = 'button'
}: BotonProps) => {
  return (
    <button type={tipo} onClick={onClick} disabled={disabled}>
      {texto}
    </button>
  );
};

// Componente con genéricos
interface ListaProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
}

function Lista<T>({ items, renderItem }: ListaProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Uso
interface Usuario {
  id: number;
  nombre: string;
}

<Lista<Usuario>
  items={usuarios}
  renderItem={(usuario) => <span>{usuario.nombre}</span>}
/>
```

### Props con Valores por Defecto

```typescript
interface CardProps {
  titulo: string;
  descripcion?: string;
  imagen?: string;
  onClick?: () => void;
}

// Opción 1: Desestructuración con valores por defecto
const Card = ({ 
  titulo, 
  descripcion = 'Sin descripción',
  imagen = '/placeholder.png',
  onClick
}: CardProps) => {
  return (
    <div onClick={onClick}>
      <img src={imagen} alt={titulo} />
      <h3>{titulo}</h3>
      <p>{descripcion}</p>
    </div>
  );
};

// Opción 2: defaultProps (estilo antiguo)
Card.defaultProps = {
  descripcion: 'Sin descripción',
  imagen: '/placeholder.png'
};
```

### Eventos en React

```typescript
import { ChangeEvent, FormEvent, MouseEvent, KeyboardEvent } from 'react';

const Formulario = () => {
  // Input change
  const handleInputChange = (e: ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  // Textarea change
  const handleTextareaChange = (e: ChangeEvent<HTMLTextAreaElement>) => {
    console.log(e.target.value);
  };

  // Select change
  const handleSelectChange = (e: ChangeEvent<HTMLSelectElement>) => {
    console.log(e.target.value);
  };

  // Form submit
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log('Formulario enviado');
  };

  // Button click
  const handleClick = (e: MouseEvent<HTMLButtonElement>) => {
    console.log('Botón clickeado');
  };

  // Keyboard events
  const handleKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('Enter presionado');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleInputChange} onKeyDown={handleKeyDown} />
      <textarea onChange={handleTextareaChange} />
      <select onChange={handleSelectChange}>
        <option value="1">Opción 1</option>
      </select>
      <button onClick={handleClick}>Enviar</button>
    </form>
  );
};
```

---

## Hooks con TypeScript

### useState

```typescript
import { useState } from 'react';

// Tipo inferido
const [count, setCount] = useState(0); // number
const [nombre, setNombre] = useState(''); // string

// Tipo explícito
const [edad, setEdad] = useState<number>(0);
const [activo, setActivo] = useState<boolean>(false);

// Con interfaces
interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

const [usuario, setUsuario] = useState<Usuario | null>(null);

// Con tipo union
type Estado = 'idle' | 'loading' | 'success' | 'error';
const [estado, setEstado] = useState<Estado>('idle');

// Arrays
const [numeros, setNumeros] = useState<number[]>([]);
const [usuarios, setUsuarios] = useState<Usuario[]>([]);

// Objetos complejos
interface FormData {
  nombre: string;
  email: string;
  edad: number;
  intereses: string[];
}

const [formData, setFormData] = useState<FormData>({
  nombre: '',
  email: '',
  edad: 0,
  intereses: []
});

// Actualizar estado
const actualizarUsuario = () => {
  setUsuario({
    id: 1,
    nombre: 'Juan',
    email: 'juan@example.com'
  });
};

const actualizarForm = () => {
  setFormData(prev => ({
    ...prev,
    nombre: 'Nuevo nombre'
  }));
};
```

### useEffect

```typescript
import { useEffect } from 'react';

const Componente = () => {
  // Efecto básico
  useEffect(() => {
    console.log('Montado');

    return () => {
      console.log('Desmontado');
    };
  }, []);

  // Con dependencias tipadas
  const [userId, setUserId] = useState<number>(1);

  useEffect(() => {
    async function fetchUser() {
      const response = await fetch(`/api/users/${userId}`);
      const data: Usuario = await response.json();
      // ...
    }

    fetchUser();
  }, [userId]);

  // Con async/await y manejo de errores
  useEffect(() => {
    let cancelado = false;

    const cargarDatos = async () => {
      try {
        const res = await fetch('/api/datos');
        if (!res.ok) throw new Error('Error');
        const datos = await res.json();
        
        if (!cancelado) {
          // Actualizar estado
        }
      } catch (error) {
        if (error instanceof Error) {
          console.error(error.message);
        }
      }
    };

    cargarDatos();

    return () => {
      cancelado = true;
    };
  }, []);
};
```

### useRef

```typescript
import { useRef, useEffect } from 'react';

const Componente = () => {
  // Ref a elemento DOM
  const inputRef = useRef<HTMLInputElement>(null);
  const divRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    // Verificar que existe antes de usar
    if (inputRef.current) {
      inputRef.current.focus();
    }

    if (divRef.current) {
      console.log(divRef.current.offsetHeight);
    }
  }, []);

  // Ref para valores mutables
  const countRef = useRef<number>(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);

  const iniciarIntervalo = () => {
    intervalRef.current = setInterval(() => {
      countRef.current += 1;
      console.log(countRef.current);
    }, 1000);
  };

  const detenerIntervalo = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  return (
    <div ref={divRef}>
      <input ref={inputRef} />
      <button onClick={iniciarIntervalo}>Iniciar</button>
      <button onClick={detenerIntervalo}>Detener</button>
    </div>
  );
};
```

### useContext

```typescript
import { createContext, useContext, ReactNode } from 'react';

// Definir tipos del contexto
interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

interface AuthContextType {
  usuario: Usuario | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
}

// Crear contexto con tipo
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Provider con tipos
interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider = ({ children }: AuthProviderProps) => {
  const [usuario, setUsuario] = useState<Usuario | null>(null);
  const [loading, setLoading] = useState(false);

  const login = async (email: string, password: string) => {
    setLoading(true);
    try {
      const res = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      });
      const data = await res.json();
      setUsuario(data.usuario);
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    setUsuario(null);
  };

  return (
    <AuthContext.Provider value={{ usuario, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};

// Hook personalizado para usar el contexto
export const useAuth = () => {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth debe ser usado dentro de AuthProvider');
  }
  
  return context;
};

// Uso en componente
const Perfil = () => {
  const { usuario, logout } = useAuth();

  if (!usuario) return <p>No autenticado</p>;

  return (
    <div>
      <h1>{usuario.nombre}</h1>
      <button onClick={logout}>Cerrar sesión</button>
    </div>
  );
};
```

### useReducer

```typescript
import { useReducer, Reducer } from 'react';

// Definir tipos
interface Estado {
  count: number;
  error: string | null;
}

type Accion =
  | { type: 'incrementar' }
  | { type: 'decrementar' }
  | { type: 'reset' }
  | { type: 'setError'; payload: string }
  | { type: 'clearError' };

// Reducer tipado
const reducer: Reducer<Estado, Accion> = (estado, accion) => {
  switch (accion.type) {
    case 'incrementar':
      return { ...estado, count: estado.count + 1 };
    case 'decrementar':
      return { ...estado, count: estado.count - 1 };
    case 'reset':
      return { ...estado, count: 0 };
    case 'setError':
      return { ...estado, error: accion.payload };
    case 'clearError':
      return { ...estado, error: null };
    default:
      return estado;
  }
};

// Componente
const Contador = () => {
  const [estado, dispatch] = useReducer(reducer, {
    count: 0,
    error: null
  });

  return (
    <div>
      <p>Count: {estado.count}</p>
      {estado.error && <p>Error: {estado.error}</p>}
      <button onClick={() => dispatch({ type: 'incrementar' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrementar' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
};

// Ejemplo complejo: Formulario
interface FormularioEstado {
  values: {
    nombre: string;
    email: string;
    edad: number;
  };
  errors: {
    nombre?: string;
    email?: string;
    edad?: string;
  };
  touched: {
    nombre: boolean;
    email: boolean;
    edad: boolean;
  };
  isSubmitting: boolean;
}

type FormularioAccion =
  | { type: 'SET_FIELD'; field: string; value: string | number }
  | { type: 'SET_ERROR'; field: string; error: string }
  | { type: 'TOUCH_FIELD'; field: string }
  | { type: 'SUBMIT_START' }
  | { type: 'SUBMIT_SUCCESS' }
  | { type: 'SUBMIT_ERROR'; errors: Record<string, string> };

const formReducer: Reducer<FormularioEstado, FormularioAccion> = (estado, accion) => {
  switch (accion.type) {
    case 'SET_FIELD':
      return {
        ...estado,
        values: {
          ...estado.values,
          [accion.field]: accion.value
        }
      };
    case 'SET_ERROR':
      return {
        ...estado,
        errors: {
          ...estado.errors,
          [accion.field]: accion.error
        }
      };
    case 'TOUCH_FIELD':
      return {
        ...estado,
        touched: {
          ...estado.touched,
          [accion.field]: true
        }
      };
    case 'SUBMIT_START':
      return { ...estado, isSubmitting: true };
    case 'SUBMIT_SUCCESS':
      return { ...estado, isSubmitting: false };
    case 'SUBMIT_ERROR':
      return {
        ...estado,
        isSubmitting: false,
        errors: accion.errors
      };
    default:
      return estado;
  }
};
```

### Custom Hooks

```typescript
import { useState, useEffect } from 'react';

// Hook simple
function useContador(inicial: number = 0) {
  const [count, setCount] = useState(inicial);

  const incrementar = () => setCount(c => c + 1);
  const decrementar = () => setCount(c => c - 1);
  const reset = () => setCount(inicial);

  return { count, incrementar, decrementar, reset };
}

// Hook con genéricos
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Error en la petición');
        const json: T = await response.json();
        setData(json);
      } catch (err) {
        setError(err instanceof Error ? err : new Error('Error desconocido'));
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Uso
interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

const Componente = () => {
  const { data, loading, error } = useFetch<Usuario[]>('/api/usuarios');

  if (loading) return <p>Cargando...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data?.map(usuario => (
        <li key={usuario.id}>{usuario.nombre}</li>
      ))}
    </ul>
  );
};

// Hook con localStorage
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// Uso
const App = () => {
  const [nombre, setNombre] = useLocalStorage<string>('nombre', '');
  
  return (
    <input 
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
    />
  );
};
```

---

## Tipos Avanzados

### Utility Types

```typescript
// Partial: Hace todas las propiedades opcionales
interface Usuario {
  id: number;
  nombre: string;
  email: string;
}

type UsuarioParcial = Partial<Usuario>;
// { id?: number; nombre?: string; email?: string; }

function actualizarUsuario(id: number, cambios: Partial<Usuario>) {
  // Actualizar solo los campos enviados
}

// Required: Hace todas las propiedades obligatorias
interface Config {
  timeout?: number;
  retries?: number;
}

type ConfigCompleta = Required<Config>;
// { timeout: number; retries: number; }

// Readonly: Hace todas las propiedades de solo lectura
type UsuarioReadonly = Readonly<Usuario>;
const usuario: UsuarioReadonly = {
  id: 1,
  nombre: 'Juan',
  email: 'juan@example.com'
};
// usuario.nombre = 'Pedro'; // Error ✅

// Record: Crea objeto con claves y tipo de valor
type Roles = 'admin' | 'user' | 'guest';
type Permisos = Record<Roles, string[]>;

const permisos: Permisos = {
  admin: ['leer', 'escribir', 'eliminar'],
  user: ['leer', 'escribir'],
  guest: ['leer']
};

// Pick: Selecciona propiedades específicas
type UsuarioBasico = Pick<Usuario, 'id' | 'nombre'>;
// { id: number; nombre: string; }

// Omit: Excluye propiedades específicas
type UsuarioSinEmail = Omit<Usuario, 'email'>;
// { id: number; nombre: string; }

// Exclude: Excluye tipos de una unión
type T1 = string | number | boolean;
type T2 = Exclude<T1, boolean>; // string | number

// Extract: Extrae tipos de una unión
type T3 = Extract<T1, string | boolean>; // string | boolean

// NonNullable: Remueve null y undefined
type T4 = string | number | null | undefined;
type T5 = NonNullable<T4>; // string | number

// ReturnType: Obtiene el tipo de retorno de una función
function crearUsuario() {
  return {
    id: 1,
    nombre: 'Juan',
    email: 'juan@example.com'
  };
}

type Usuario = ReturnType<typeof crearUsuario>;
// { id: number; nombre: string; email: string; }

// Parameters: Obtiene tipos de parámetros de función
function saludar(nombre: string, edad: number) {
  return `Hola ${nombre}, tienes ${edad} años`;
}

type SaludarParams = Parameters<typeof saludar>;
// [nombre: string, edad: number]
```

### Tipos Condicionales

```typescript
// Tipo condicional básico
type EsString<T> = T extends string ? true : false;

type A = EsString<string>; // true
type B = EsString<number>; // false

// Extraer tipo de array
type ElementoArray<T> = T extends Array<infer E> ? E : T;

type Nums = ElementoArray<number[]>; // number
type Str = ElementoArray<string>; // string

// Tipo condicional con Props de React
type PropType<T, K extends keyof T> = T[K];

interface ButtonProps {
  onClick: () => void;
  disabled: boolean;
  children: string;
}

type OnClickType = PropType<ButtonProps, 'onClick'>; // () => void
```

### Mapped Types

```typescript
// Hacer propiedades opcionales
type Optional<T> = {
  [K in keyof T]?: T[K];
};

// Hacer propiedades readonly
type ReadonlyType<T> = {
  readonly [K in keyof T]: T[K];
};

// Agregar prefijo a propiedades
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

interface Usuario {
  nombre: string;
  edad: number;
}

type UsuarioPrefijado = Prefixed<Usuario, 'get'>;
// { getNombre: string; getEdad: number; }

// Getters
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UsuarioGetters = Getters<Usuario>;
// {
//   getNombre: () => string;
//   getEdad: () => number;
// }
```

---

## Migración de JavaScript a TypeScript

### Paso 1: Agregar tipos gradualmente

```typescript
// Antes (JS)
function calcularTotal(items) {
  return items.reduce((sum, item) => sum + item.precio * item.cantidad, 0);
}

// Después (TS)
interface Item {
  precio: number;
  cantidad: number;
}

function calcularTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.precio * item.cantidad, 0);
}
```

### Paso 2: Componente completo migrado

```typescript
// Antes (JavaScript)
import { useState, useEffect } from 'react';

export default function ProductList({ category }) {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(`/api/products?category=${category}`)
      .then(res => res.json())
      .then(data => {
        setProducts(data);
        setLoading(false);
      });
  }, [category]);
  
  const handleDelete = (id) => {
    setProducts(products.filter(p => p.id !== id));
  };
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <div>
      {products.map(product => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button onClick={() => handleDelete(product.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}

// Después (TypeScript)
import { useState, useEffect, FC } from 'react';

interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

interface ProductListProps {
  category: string;
}

const ProductList: FC<ProductListProps> = ({ category }) => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  
  useEffect(() => {
    const fetchProducts = async () => {
      try {
        const res = await fetch(`/api/products?category=${category}`);
        const data: Product[] = await res.json();
        setProducts(data);
      } catch (error) {
        console.error(error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchProducts();
  }, [category]);
  
  const handleDelete = (id: number): void => {
    setProducts(products.filter(p => p.id !== id));
  };
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <div>
      {products.map((product) => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button onClick={() => handleDelete(product.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
};

export default ProductList;
```

---

## Mejores Prácticas

### 1. Evitar `any`

```typescript
// ❌ Malo
const data: any = fetchData();

// ✅ Bueno
interface ApiResponse {
  data: User[];
  total: number;
}
const data: ApiResponse = await fetchData();

// ✅ Si no conoces el tipo, usa unknown
const data: unknown = fetchData();
if (typeof data === 'object' && data !== null) {
  // Ahora puedes usar data
}
```

### 2. Usar Union Types en lugar de Enums

```typescript
// ❌ Enum (genera código extra)
enum Status {
  Pending = 'pending',
  Active = 'active',
  Completed = 'completed'
}

// ✅ Union Type (sin código extra)
type Status = 'pending' | 'active' | 'completed';
```

### 3. Interfaces para Props, Types para otros casos

```typescript
// ✅ Interfaces para componentes
interface ButtonProps {
  text: string;
  onClick: () => void;
}

// ✅ Types para unions/intersections
type ButtonVariant = 'primary' | 'secondary' | 'danger';
type APIResponse<T> = { data: T; error: null } | { data: null; error: string };
```

### 4. Usar `as const` para constantes

```typescript
// ❌ Sin as const
const ROLES = ['admin', 'user', 'guest']; // string[]

// ✅ Con as const
const ROLES = ['admin', 'user', 'guest'] as const;
// readonly ['admin', 'user', 'guest']

type Role = typeof ROLES[number]; // 'admin' | 'user' | 'guest'
```

### 5. Tipado estricto en tsconfig.json

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

---

## Errores Comunes y Soluciones

### Error: Object is possibly 'null'

```typescript
// ❌ Error
const input = document.getElementById('input');
input.focus(); // Error: input puede ser null

// ✅ Solución 1: Non-null assertion
const input = document.getElementById('input')!;
input.focus();

// ✅ Solución 2: Optional chaining
input?.focus();

// ✅ Solución 3: Verificación
if (input) {
  input.focus();
}
```

### Error: Type 'string' is not assignable to type

```typescript
// ❌ Error
type Status = 'active' | 'inactive';
const status = 'active'; // string
const userStatus: Status = status; // Error

// ✅ Solución: as const
const status = 'active' as const;
const userStatus: Status = status; // OK
```

---

## Recursos

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [Total TypeScript](https://www.totaltypescript.com/)
