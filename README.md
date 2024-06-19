## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

# Curso oficial de Next

```sh
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

## Estilos

En layout.tsx se importan los estilos que heredarán las páginas
En este caso estilos globales

```js
import '@/app/ui/global.css';
```

En este caso estilos css como módulos
Importando sobre el archivo de la página o del componente

```js
import styles from '@/app/ui/home.module.css';
```

### [clsx](https://github.com/lukeed/clsx)

clsx permite usar una clase u otra según se cupla una condición

```js
<span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
```

## Custom Fonts

Next optimiza las fuentes descargándolas y alojándolas con los archivos estáticos en el building

_/app/ui/fonts.ts_

```js
import { Inter } from 'next/font/google';
export const inter = Inter({ subsets: ['latin'] });
```

_/app/layout.tsx_

```js
import { inter } from '@/app/ui/fonts';
<body className={`${inter.className} antialiased`}>{children}</body>;
```

Con `Inter` como className en el `body` de la app, se agrega la fuente a todo lo que esté dentro del body, en este caso toda la app

## Imágenes

También son optimizadas mediante el componente `next/Image`

```js
<Image
  src="/hero-mobile.png"
  width={560}
  height={620}
  className="block md:hidden"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

_className="block md:hidden"_ significa que se va a mostrar a menos que el tamaño de la pantalla coincida con `md` de tailwind.

## Pages & Layouts

### Pages

_pages.tsx_ es un archivo especial que exporta un componente _react_. Éstas van a ser las únicas páginas visibles

### Layouts

```js
import SideNav from '@/app/ui/dashboard/sidenav';

export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
  );
}
```

Reciben por prop un objeto children que puede ser tanto las páginas que va a 'cubrir' como otros layouts

## Link component

El componente `<Link>` se utiliza como el elemento `<a>`, pero no re-renderiza la totalidad de la página si no es necesario, y hace la navegación del lado del cliente, no como petición al servidor.

Además, por defecto Next.js divide el código en sus distintas rutas para así dejarlo aislado, si se rompe una página no afecta a otras rutas.

Si hay navegación con `<Link>`, se raliza un prefetch de las páginas a donde apuntan los mismos.

## Funcionamiento de la [navegación](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works)

- Code Splitting: En el servidor se divide y se aisla el código por rutas, permitiendo crear un bundle pequeño y enviar al cliente sólo lo necesario.

- Prefetching: Pre carga una página a donde apunte un elemento `<Link>`, cuando este sea visible en el viewport. Para precargar rutas a mano se usa el hook `router.prefetch()`

- Caching: Por defecto las páginas visitadas guardan toda la información posible para que al volver a visitarlas más adelante, pueda reutilizar los distintos elementos y hacer una menor cantidad de peticiones.

- Partial Rendering: Cuando se comparten elementos entre rutas, al pasar de una a otra no se re-renderizan estos, sino que sólo los que cambian además de los nuevos.

- Soft Navigation: La navegación normal realiza una "hard navigation", pero con el Partial Rendering pasa a ser `Soft Navigation`.

- Back and Forward Navigation: Se mantiene la posición del scroll al volver.

- Routing between _pages/_ and _app/_:

## Links activos

Para saber en qué página se está, se marca el link de la página, esto se puede hacer tomando el peth de la URL y en base a ese path poner un estilo distinto al link activo.

## [Postgres connection](https://vercel.com/docs/storage/vercel-postgres/sdk)

Para conectar a la db postgres creada en la instancia de Vercel, se crean las variables de entorno necesarias y se usa la función `sql`, que pasándole una query sql usa la conexión a la db para enviar la petición.

```js
const { db } = require('@vercel/postgres');
//...
const client = await db.connect();
await client.sql`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`;
    // Create the "users" table if it doesn't exist
    const createTable = await client.sql`
      CREATE TABLE IF NOT EXISTS users (
        await seedUsers(client);
//...
```

## [Fetching Data](https://nextjs.org/learn/dashboard-app/fetching-data)

### API layer

Se pueden crear endpoints de API con los [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

### Server components

- Se puede usar `async/await` sin necesidad de `useEffect` ni `useState`.
- Se ejecuta en servidor, al resolver se envía el componente.
- Se puede hacer una petición a la BD de forma segura sin una API de por medio

### Server rendering strategies

Hay 3 estrategias distintas de SSR en Nextjs:

- Static rendering:
  - Se genera el render en tiempo de construcción de la página (build) si tiene que hacer fetching de datos, los hace una sola vez y queda cacheado, o cuando se hace una revalidadción del cache.
- Dynamic Rendering:
  - También realiza el render en el servidor, pero en tiempo de petición (request), por lo tanto cada vez que un usuario pide una página el servidor realiza un nuevo render, permitiendo ver datos actualizados, sin guardar en un cache, aunque puede tardar un poco más la respuesta, dependiendo de los procesos de fetching que lleve a cabo.
- Streaming:
  - Al igual que antes, renderiza en el servidor, pero va enviando los datos de la respuesta, separada en partes (chunks) con sentido propio, por ejemplo, se va enviando de a componentes, primero los más rápidos en renderizarse. Así el usuario puede interactuar con la pagina sin que esté del todo cargada

Por defecto next realiza Static Rendering, si se requiere que una página realice Synamic Rendering se puede utilizar la herramienta `unstable_noStore`:

```js
import { unstable_noStore as noStore } from 'next/cache';

export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();

```

También se puede usar `{cache: 'no-store'}` en la función fetch para obtener una página dinámica (renderizado en tiempo de petición).

## Streaming

Hay dos formas de realizar un streaming:

- A nivel de página, con el archivo loading.js.
- Para componentes específicos, con `<Suspense>`.

### loading.js...

El archivo loading es un archivo especial, y permite crear un componente que será renderizado como 'fallback' mientras carga una parte de la página. De esta manera, los componentes estáticos con mostrados inicialmente, mientras que los dinámicos esperan a a que estén todos listos para mostrarse, produciendose así el `Streaming`.

### Suspense

Con el componente `<Suspense>` se puede envolver otro componente que tenga algún proceso que demande tiempo, de esta manera se aísla el componente envuelto y se separa del resto de componentes dinámicos, haciendo que los demás puedan enviarse en una primera parte del stream y cuando se completen los procesos internos de este componente, también se envíe.

En ambos casos se puede usar un componente de placeholder o fallback, que se renderiza de forma estática, para mostrar algo en la pantalla hasta que se cargan los componentes.
