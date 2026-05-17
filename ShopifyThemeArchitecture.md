# Shopify Theme Architecture: Manual de Referencia Definitivo (Liquid + OS 2.0)

> **Estado del Manual: 💎 ACTIVO Y LISTO PARA CREACIÓN**  
> Diseñado como el estándar técnico definitivo para transformar referencias visuales (imágenes y mockups) en Shopify Themes de alta gama.  
> *Compatible con Online Store 2.0 (OS 2.0), Shopify CLI v3 y los principios de Ingeniería de Diseño Premium.*

---

## Visión General

Este manual técnico establece las bases estructuradas de desarrollo de **Shopify Themes nativos**. Proporciona un marco para traducir layouts visuales estáticos en código Liquid, HTML5 semántico, y CSS/JS hardware-acelerado altamente modular. 

A diferencia del enfoque headless de **Nucleofi** (React + API), aquí se trabaja directamente sobre el **servidor de renderizado de Shopify**, lo que requiere un dominio absoluto de la arquitectura de plantillas, la sintaxis de Liquid, los esquemas de configuración de los bloques y las optimizaciones de rendimiento en el lado del servidor.

---

## 1. Características del Sistema (Online Store 2.0)

| Componente | Especificación Técnica | Propósito en el Desarrollo |
|---|---|---|
| **Estructura Base** | JSON Templates + Liquid | Permite añadir/reordenar secciones dinámicamente desde el Editor |
| **Motor de Plantillas** | Liquid (Shopify Engine) | Renderizado ultra-rápido en servidor con tipado dinámico seguro |
| **Estilos (CSS)** | CSS Vanilla Custom + Variables HSL | Control visual completo sin dependencias masivas o Tailwind innecesario |
| **Animaciones** | CSS Hardware-Accelerated + Custom Springs | Micro-interacciones fluidas, transiciones de GPU a 60+ FPS |
| **Imágenes** | Filtros Liquid (`image_url` + `srcset`) | Entrega adaptativa de assets optimizados según pantalla |
| **Locales** | Esquema de traducción `/locales` con filtro `t` | Internacionalización nativa y personalizable por el comerciante |
| **Tooling** | Shopify CLI v3 | Servidor local con hot-reload y sincronización del Theme Editor |

---

## 2. Estructura de Directorios Obligatoria

Shopify requiere una estructura de directorios estricta y rígida. No se admiten carpetas de segundo nivel personalizadas.

```
mi-tema-premium/
├── assets/             # Hojas de estilo (.css), scripts (.js), imágenes y SVGs
├── config/             # Configuración general y esquemas de Theme Settings
├── layout/             # Envoltorios globales del tema (theme.liquid)
├── locales/            # Archivos JSON de traducción para multi-idioma
├── sections/           # Módulos Liquid reutilizables y configurables por el comerciante
├── snippets/           # Fragmentos pequeños de código Liquid reutilizable
└── templates/          # Estructuras de páginas (preferiblemente archivos .json)
    └── customers/      # Plantillas de cuentas de clientes (login, register, etc.)
```

### 2.1 Especificaciones por Directorio

### 📂 `layout/`
Contiene los archivos de plantilla que envuelven las páginas. El archivo principal es `theme.liquid`. Es el encargado de inicializar el HTML, los scripts globales, el SEO base y las fuentes tipográficas.
*   **Obligatorio:** Debe incluir `{{ content_for_header }}` en el `<head>` y `{{ content_for_layout }}` dentro del `<body>`.
*   **Diseño Premium:** Aquí inicializamos el sistema de CSS Variables globales (colores HSL, tipografía fluida, radios de curva).

### 📂 `templates/` (Online Store 2.0)
Define qué secciones aparecen en cada página y en qué orden. **En OS 2.0, los templates deben ser archivos `.json`** (por ejemplo, `index.json` para la Home, `product.json` para el detalle del producto).
*   *Evita crear templates de tipo `.liquid`* para páginas generales. Los templates `.json` permiten arrastrar y soltar secciones en el admin de Shopify, mientras que los `.liquid` bloquean la maquetación.
*   **Estructura típica de un `templates/index.json`:**
    ```json
    {
      "name": "Home Page",
      "sections": {
        "hero_banner": {
          "type": "custom-hero",
          "settings": {
            "title": "Cosmética Consciente"
          }
        },
        "product_grid": {
          "type": "featured-collection",
          "settings": {
            "collection": "novedades",
            "products_to_show": 4
          }
        }
      },
      "order": ["hero_banner", "product_grid"]
    }
    ```

### 📂 `sections/`
Los bloques de construcción clave de tu tema. Son archivos `.liquid` individuales que contienen:
1.  **Marcado HTML / Liquid:** La estructura del componente.
2.  **CSS/JS específico (opcional):** Con etiquetas `<style>` o `<script>`.
3.  **Esquema JSON (`{% schema %}`):** Define los campos configurables en el editor de Shopify (textos, imágenes, colores, colecciones) y declara si soporta **Blocks** para anidar elementos (como diapositivas dentro de un slider).

### 📂 `snippets/`
Archivos `.liquid` pequeños que se incluyen dentro de layouts, secciones o templates mediante el tag `{% render 'nombre-del-snippet' %}`.
*   **Uso:** Excelente para componentes atómicos que se repiten con frecuencia (ej: `card-product.liquid`, `icon-cart.liquid`, `social-icons.liquid`).
*   *Nota:* No pueden contener etiquetas `{% schema %}` ni recibir settings directamente del editor, solo reciben variables pasadas explícitamente en el `render`.

### 📂 `assets/`
Archivos estáticos.
*   **Truco Pro:** Puedes usar Liquid dentro de archivos CSS y JS renombrándolos a `.css.liquid` o `.js.liquid`. Esto permite inyectar variables de color o tipografías directamente desde la configuración del comerciante:
    ```css
    body {
      background-color: {{ settings.color_background }};
    }
    ```
*   **Advertencia de Rendimiento:** Es mejor utilizar CSS puro con Variables CSS nativas definidas en el layout, reduciendo el procesamiento en servidor de archivos estáticos pesados.

---

## 3. Flujo de Trabajo y CLI de Shopify (Shopify CLI v3)

El desarrollo local de Shopify Themes se gestiona mediante la consola usando comandos robustos que conectan tus archivos locales con un servidor de previsualización en la nube de Shopify.

### 3.1 Inicializar un Tema desde Cero
Shopify recomienda partir de una base limpia. Usamos el repositorio **Skeleton Theme** (ultra-minimalista enfocado en rendimiento y OS 2.0) en lugar de Dawn si buscamos un control total del diseño sin código heredado masivo:

```powershell
# Clona el Skeleton Theme en una nueva carpeta local
shopify theme init my-premium-theme --clone-url https://github.com/shopify/skeleton-theme
```

### 3.2 El Servidor de Desarrollo Local
Para previsualizar tus cambios en tiempo real con datos de producto reales de tu tienda de pruebas:

```powershell
# Inicia el túnel local hacia la tienda de desarrollo
shopify theme dev --store tu-tienda-prueba.myshopify.com
```

> [!IMPORTANT]
> **El Servidor de Desarrollo Local genera 3 enlaces vitales:**
> 1. `http://127.0.0.1:9292` -> Vista previa local con **Hot-Reloading** de CSS y secciones modulares (Óptimo para Chrome).
> 2. Enlace al **Theme Editor** -> Permite probar la integración de tus esquemas JSON y configuraciones visuales en tiempo real.
> 3. Enlace Compartible -> Una URL segura de Shopify para mostrar el progreso a terceros.

### 3.3 Sincronización Bidireccional (Crucial para el Flujo)
Cuando el usuario o tú modificáis configuraciones en el Theme Editor web, esos datos se guardan en el servidor de Shopify (dentro de `config/settings_data.json` o plantillas `.json`). Si editas código localmente al mismo tiempo, corres el riesgo de sobreescribir estos cambios.

Para evitarlo, inicia el servidor con la flag de sincronización:
```powershell
shopify theme dev --store tu-tienda-prueba.myshopify.com --theme-editor-sync
```
O realiza una descarga manual de las configuraciones del editor antes de commitear:
```powershell
shopify theme pull --store tu-tienda-prueba.myshopify.com
```

### 3.4 Puesta en Producción (Push y Publish)
Una vez terminado el diseño:

```powershell
# Sube el código a una ranura de desarrollo inédita (sin pisar el tema en vivo)
shopify theme push --unpublished --theme "Ritual - Fase 1"

# Publica un tema específico para ponerlo en vivo en la tienda
shopify theme publish
```

---

## 4. Motor de Plantillas Liquid: Reglas y Sintaxis Esenciales

Liquid es un lenguaje de plantillas seguro que no ejecuta código malicioso en el servidor de Shopify. Se divide en tres conceptos clave:

### 4.1 Objetos e Inyecciones (Output)
Encerrados entre llaves dobles `{{ }}`. Muestran contenido dinámico del backend de Shopify.
```liquid
<h1>{{ product.title }}</h1>
<span class="price">{{ product.price | money }}</span>
```

### 4.2 Etiquetas (Tags)
Encerradas entre llaves y porcentajes `{% %}`. Controlan la lógica, iteraciones, condicionales y flujos.
```liquid
{% if product.available %}
  <button class="btn btn--primary">Añadir al Carrito</button>
{% else %}
  <button class="btn btn--disabled" disabled>Agotado</button>
{% endif %}
```

### 4.3 Filtros Vitales para Frontend de Alta Calidad
Los filtros modifican la salida y se aplican mediante una barra vertical `|`.

*   **Rendimiento en Imágenes (`image_url` + `image_tag`):**
    *Nunca inyectes la URL de imagen original en bruto*. Shopify generaría imágenes gigantescas arruinando la velocidad de carga. Usa renderizado dinámico responsivo:
    ```liquid
    {{ product.featured_image | image_url: width: 800 | image_tag: 
       widths: '375, 550, 750, 1100, 1500',
       sizes: '(min-width: 1200px) 50vw, 100vw',
       loading: 'lazy',
       class: 'product-card__image'
    }}
    ```
*   **Localización y Textos Traducibles (`t`):**
    Permite cambiar copys estáticos desde el editor de traducciones sin tocar código:
    ```liquid
    <label for="ContactFormEmail">{{ 'templates.contact.form.email' | t }}</label>
    ```
*   **Formateo de Moneda (`money`):**
    ```liquid
    <span>{{ variant.price | money }}</span>  <!-- Imprime: $150.00 MXN o €12.00 -->
    ```

---

## 5. Metodología "Design-to-Code": Traduciendo tus Diseños Visuales a Liquid

Cuando diseñas layouts espectaculares (en Figma, Photoshop o imágenes generadas por IA) y deseas recrearlos a nivel pixel-perfect con calidad premium, la arquitectura estructurada de Shopify debe respetar estrictos criterios de diseño.

### 5.1 Anatomía de una Sección Premium
Para que una sección nativa luzca cara y exclusiva, estructuramos su archivo `.liquid` (por ejemplo, `sections/premium-hero.liquid`) en tres partes claramente delimitadas: **Estructura semántica**, **CSS encapsulado con variables HSL**, y **Esquemas JSON robustos**.

Aquí tienes el patrón maestro de código para una sección interactiva:

```liquid
{%- comment -%}
  SECCIÓN: Premium Hero con Layout Dinámico y Hardware-Accelerated Motion
{%- endcomment -%}

{%- style -%}
  /* CSS Variables locales calibradas en HSL */
  .premium-hero-{{ section.id }} {
    --hero-bg: {{ section.settings.bg_color }};
    --hero-text: {{ section.settings.text_color }};
    --hero-accent: {{ section.settings.accent_color }};
    --grid-gap: {{ section.settings.spacing }}px;
    
    background-color: var(--hero-bg);
    color: var(--hero-text);
    padding-top: {{ section.settings.padding_top }}px;
    padding-bottom: {{ section.settings.padding_bottom }}px;
  }
{%- endstyle -%}

{{ 'section-premium-hero.css' | asset_url | stylesheet_tag }}

<section class="premium-hero premium-hero-{{ section.id }} {% if section.settings.full_width %}hero--full{% endif %}" id="Hero-{{ section.id }}">
  <div class="hero__container page-width">
    
    <div class="hero__grid">
      <!-- Columna Izquierda: Contenido Editorial -->
      <div class="hero__content">
        {%- if section.settings.kicker != blank -%}
          <span class="hero__kicker small-caps text-accent">{{ section.settings.kicker }}</span>
        {%- endif -%}
        
        <h2 class="hero__title font-heading-fluid">
          {{ section.settings.title | escape }}
        </h2>
        
        {%- if section.settings.description != blank -%}
          <div class="hero__description rte">
            {{ section.settings.description }}
          </div>
        {%- endif -%}

        <!-- Renderización de bloques dinámicos de botones (Blocks) -->
        <div class="hero__actions">
          {%- for block in section.blocks -%}
            {%- case block.type -%}
              {%- when 'button' -%}
                <a href="{{ block.settings.link }}" 
                   class="btn {% if block.settings.primary %}btn--action{% else %}btn--secondary{% endif %}" 
                   {{ block.shopify_attributes }}>
                  <span>{{ block.settings.label }}</span>
                </a>
            {%- endcase -%}
          {%- endfor -%}
        </div>
      </div>

      <!-- Columna Derecha: Imagenes con Relación de Aspecto Fija y Carga Lazy -->
      <div class="hero__media-wrapper">
        {%- if section.settings.image != blank -%}
          <div class="hero__media-ratio" style="--aspect-ratio: {{ section.settings.image.aspect_ratio }};">
            {{ section.settings.image | image_url: width: 1200 | image_tag:
               widths: '400, 600, 800, 1200',
               sizes: '(min-width: 990px) 50vw, 100vw',
               class: 'hero__image',
               loading: 'lazy'
            }}
          </div>
        {%- else -%}
          <!-- Placeholder de Shopify si no hay imagen asignada -->
          {{ 'lifestyle-1' | placeholder_svg_tag: 'placeholder-svg hero__placeholder' }}
        {%- endif -%}
      </div>
    </div>

  </div>
</section>

{% schema %}
{
  "name": "⚡ Premium Hero Banner",
  "tag": "section",
  "class": "section-hero-banner",
  "limit": 1,
  "settings": [
    {
      "type": "header",
      "content": "Alineación y Contenido"
    },
    {
      "type": "checkbox",
      "id": "full_width",
      "label": "Ancho Completo",
      "default": false
    },
    {
      "type": "image_picker",
      "id": "image",
      "label": "Imagen de Banner"
    },
    {
      "type": "text",
      "id": "kicker",
      "label": "Kicker (Subtítulo Superior)",
      "default": "EDICIÓN LIMITADA"
    },
    {
      "type": "text",
      "id": "title",
      "label": "Título Principal",
      "default": "Piel Radiante de Forma Natural"
    },
    {
      "type": "richtext",
      "id": "description",
      "label": "Descripción",
      "default": "<p>Cosméticos elaborados con ingredientes 100% orgánicos y activos botánicos de alta pureza.</p>"
    },
    {
      "type": "header",
      "content": "Ajustes de Color HSL"
    },
    {
      "type": "color",
      "id": "bg_color",
      "label": "Color de Fondo",
      "default": "#1a1c1a"
    },
    {
      "type": "color",
      "id": "text_color",
      "label": "Color de Texto",
      "default": "#f4f4f4"
    },
    {
      "type": "color",
      "id": "accent_color",
      "label": "Color de Acento",
      "default": "#d9b68a"
    },
    {
      "type": "header",
      "content": "Espaciado y Retícula"
    },
    {
      "type": "range",
      "id": "spacing",
      "min": 16,
      "max": 80,
      "step": 4,
      "unit": "px",
      "label": "Separación del Grid",
      "default": 32
    },
    {
      "type": "range",
      "id": "padding_top",
      "min": 0,
      "max": 120,
      "step": 10,
      "unit": "px",
      "label": "Margen Superior",
      "default": 60
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "min": 0,
      "max": 120,
      "step": 10,
      "unit": "px",
      "label": "Margen Inferior",
      "default": 60
    }
  ],
  "blocks": [
    {
      "type": "button",
      "name": "Botón de Acción",
      "limit": 2,
      "settings": [
        {
          "type": "text",
          "id": "label",
          "label": "Texto del Botón",
          "default": "Comprar Ahora"
        },
        {
          "type": "url",
          "id": "link",
          "label": "Enlace"
        },
        {
          "type": "checkbox",
          "id": "primary",
          "label": "Estilo Primario",
          "default": true
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "⚡ Premium Hero Banner",
      "blocks": [
        {
          "type": "button",
          "settings": {
            "label": "Explorar Colección",
            "primary": true
          }
        }
      ]
    }
  ]
}
{% endschema %}
```

---

## 6. Los Pilares del Diseño Exclusivo (Design Spine)

Para evitar que tu tienda Shopify tenga ese aspecto "genérico de plantilla de $30 USD", el desarrollo del frontend bajo este manual adopta directrices de excelencia estética.

### 6.1 Paleta de Color Calibrada (HSL Armonioso)
Baneamos por completo los colores planos (`#ffffff`, `#000000`, `#ff0000`). En su lugar, el CSS de tu tema debe trabajar bajo un espacio de color **HSL calibrado** para asegurar transiciones sutiles y escalas de contrastes elegantes.

```css
/* Definición en assets/theme.css */
:root {
  /* Marca Core: Verde Orgánico profundo y Arena Muted */
  --color-primary-h: 120;
  --color-primary-s: 15%;
  --color-primary-l: 12%; /* #1a201a */
  
  --color-accent-h: 36;
  --color-accent-s: 50%;
  --color-accent-l: 75%; /* #d9b68a */

  --color-neutral-bg-h: 60;
  --color-neutral-bg-s: 4%;
  --color-neutral-bg-l: 96%; /* Arena suave */

  /* Inyección de Variables HSL Activas */
  --bg-primary: hsl(var(--color-neutral-bg-h), var(--color-neutral-bg-s), var(--color-neutral-bg-l));
  --text-primary: hsl(var(--color-primary-h), var(--color-primary-s), var(--color-primary-l));
  --brand-accent: hsl(var(--color-accent-h), var(--color-accent-s), var(--color-accent-l));
}
```

### 6.2 Tipografía Fluida y Jerarquía Editorial
Implementamos tipografía que se adapta matemáticamente al tamaño del viewport sin saltos bruscos en breakpoints estáticos (sin amontonar textos en móviles).
```css
:root {
  /* Cálculo fluido: min-size 32px, max-size 54px */
  --font-size-hero-fluid: clamp(2rem, 4vw + 1rem, 3.375rem);
  --font-size-heading-fluid: clamp(1.5rem, 3vw + 0.8rem, 2.5rem);
  --font-size-body: 1rem;
  
  --font-family-serif: 'Outfit', 'Inter', system-ui, sans-serif;
  --font-family-heading: 'Cinzel', 'Playfair Display', Georgia, serif;
}

h1, .font-heading-fluid {
  font-family: var(--font-family-heading);
  font-size: var(--font-size-hero-fluid);
  line-height: 1.15;
  letter-spacing: -0.02em;
}
```

### 6.3 Relación de Aspecto Fija en Imágenes (Aspect Ratio Control)
Uno de los fallos estéticos más comunes en e-commerce es la desalineación de tarjetas debido a imágenes de distintos tamaños. Forzamos un contenedor con relación de aspecto estricta y rellenamos con `object-fit: cover`.

```css
.hero__media-ratio {
  position: relative;
  width: 100%;
  /* Fallback a 16:9 si la variable no está inyectada */
  padding-bottom: calc(100% / (var(--aspect-ratio, 1.777))); 
  overflow: hidden;
  border-radius: 8px;
}

.hero__image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

/* Efecto Hover Premium */
.hero__media-ratio:hover .hero__image {
  transform: scale(1.04);
}
```

### 6.4 Micro-Interacciones de Alta Gama (Custom Springs)
Las animaciones deben sentirse orgánicas y rápidas. Reemplazamos los típicos `ease-in-out` lentos y pesados por físicas de tipo resorte (**Spring Curves**) que aceleran mediante la tarjeta gráfica (`transform` e `opacity` exclusivamente).

```css
/* Botón premium con micro-movimiento */
.btn {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 1rem 2.5rem;
  border: 1px solid var(--text-primary);
  background: transparent;
  color: var(--text-primary);
  text-decoration: none;
  font-weight: 500;
  overflow: hidden;
  transition: color 0.4s cubic-bezier(0.19, 1, 0.22, 1);
  will-change: transform;
}

.btn::before {
  content: '';
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: var(--text-primary);
  transition: transform 0.5s cubic-bezier(0.25, 1, 0.5, 1);
  z-index: -1;
  transform: translateY(0);
}

.btn:hover::before {
  transform: translateY(-100%);
}

.btn:hover {
  color: var(--bg-primary);
  transform: translateY(-2px);
}

.btn:active {
  transform: translateY(0);
}
```

---

## 7. Checklist de SEO y Accesibilidad (A11y) Nativos

Shopify cuenta con excelentes herramientas para SEO, pero debemos configurarlas correctamente en la estructura base del tema.

- [ ] **Estructura Semántica del DOM:** Asegura que solo exista un `<h1>` por página (generalmente el título del producto, de la colección, o el título de la marca en la Home).
- [ ] **Etiquetas Alt Dinámicas:** Toda imagen cargada mediante Liquid debe incluir su texto alternativo dinámico provisto por el panel de administración:
  ```liquid
  alt="{{ image.alt | escape }}"
  ```
- [ ] **Esquema de Datos Estructurados (JSON-LD):** Incluido en la cabecera para indexar productos en Google Rich Snippets:
  ```liquid
  <script type="application/ld+json">
    {
      "@context": "http://schema.org",
      "@type": "Product",
      "name": {{ product.title | json }},
      "image": {{ product.featured_image | image_url: width: 1000 | json }},
      "description": {{ product.description | strip_html | json }},
      "offers": {
        "@type": "Offer",
        "price": {{ product.selected_or_first_available_variant.price | money_without_currency | json }},
        "priceCurrency": {{ shop.currency | json }},
        "availability": "http://schema.org/{% if product.available %}InStock{% else %}OutOfStock{% endif %}"
      }
    }
  </script>
  ```
- [ ] **Rutas Semánticas de Foco:** Cualquier modal de carrito (`drawer`) o menú desplegable debe poder cerrarse con la tecla `ESC` y soportar navegación por teclado (`tabindex="0"`).

---

## ⚠️ REGLAS DE ORO & ERRORES CRÍTICOS EN THEMES

> **Lee esto antes de realizar cualquier commit.**

### 🚨 Regla 1: El Conflicto de Sobreescritura del Editor
*   **Problema:** Sincronizar código local con `shopify theme push` puede machacar la configuración que el comerciante hizo en caliente sobre el Editor.
*   **Mitigación:** Integra `shopify theme dev --theme-editor-sync` en tu workflow de terminal o corre `git pull` de la rama de desarrollo antes de arrancar.

### 🚨 Regla 2: Olvidar los `shopify_attributes` en Bloques
*   **Problema:** Al programar un bucle de bloques (`{% for block in section.blocks %}`), si olvidas inyectar el parámetro `{{ block.shopify_attributes }}` dentro de la etiqueta HTML contenedora del bloque, el Editor de Shopify no podrá enfocar ni seleccionar dicho elemento visualmente al hacer clic sobre él.
*   **Ejemplo Correcto:**
    ```liquid
    <div class="card" {{ block.shopify_attributes }}>
    ```

### 🚨 Regla 3: No prever el "CLS" (Cumulative Layout Shift) en Fuentes
*   **Problema:** Al cargar tipografías elegantes (como fuentes de Google Fonts), la página se renderiza primero con tipografía de sistema y luego da un "salto" visual cuando la fuente premium carga. Esto penaliza tu puntuación de rendimiento Lighthouse Core Web Vitals.
*   **Mitigación:** Inserta etiquetas `<link rel="preconnect">` en tu `theme.liquid` para Google Fonts o aloja los archivos `.woff2` directamente en la carpeta `/assets` del tema precargándolos en el `<head>`:
    ```liquid
    <link rel="preload" href="{{ 'Cinzel-Regular.woff2' | asset_url }}" as="font" type="font/woff2" crossorigin>
    ```

---

*Manual técnico de arquitectura de Shopify Themes y diseño premium. Creado en Mayo de 2026 para el desarrollo pixel-perfect.*
