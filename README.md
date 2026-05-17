# ⚡ Ritual Cosmético - Shopify Theme (OS 2.0)

Este es el repositorio oficial del **Tema de Shopify nativo** para **Ritual Cosmético**. Está desarrollado bajo el estándar **Online Store 2.0 (OS 2.0)** utilizando **Liquid**, HTML5 semántico, CSS Vanilla modular (con variables HSL dinámicas) y físicas de animación fluidas.

Este repositorio está estructurado para integrarse directamente con la **conexión nativa de Shopify + GitHub**, lo que permite previsualizar y sincronizar cambios automáticamente en tiempo real.

---

## 📂 Estructura del Proyecto

El repositorio mantiene la estructura oficial requerida por Shopify en su raíz:

```
shopify-theme/
├── assets/             # Hojas de estilo (.css), scripts (.js), imágenes y SVGs
├── config/             # Configuración general y esquemas de Theme Settings
├── layout/             # Envoltorios globales (theme.liquid)
├── locales/            # Archivos JSON de traducción multi-idioma
├── sections/           # Módulos Liquid reutilizables y configurables en el editor
├── snippets/           # Fragmentos pequeños de código Liquid reutilizable
└── templates/          # Estructuras JSON para cada tipo de página
```

---

## 🛠️ Flujo de Trabajo Local (Desarrollo)

Para trabajar localmente en este tema, asegúrate de tener instalado el [Shopify CLI](https://shopify.dev/docs/api/shopify-cli).

### 1. Iniciar Servidor de Desarrollo Local
Para previsualizar tus cambios locales con datos reales de la tienda en tiempo real:

```bash
shopify theme dev --store tu-tienda-prueba.myshopify.com --theme-editor-sync
```

> 💡 **Nota:** Esto iniciará un túnel local en `http://127.0.0.1:9292` con soporte para **Hot-Reloading** de CSS y secciones. La flag `--theme-editor-sync` asegura que los cambios que realices en el Theme Editor web de Shopify se descarguen automáticamente a tus archivos locales en caliente.

### 2. Descargar Cambios Manualmente (Pull)
Si realizaste ediciones en el Theme Editor web y deseas traerlas a local de forma manual:

```bash
shopify theme pull --store tu-tienda-prueba.myshopify.com
```

### 3. Subir Código (Push)
Para subir tus cambios locales a una versión en desarrollo inédita (sin pisar el tema en vivo):

```bash
shopify theme push --unpublished --theme "Ritual - Fase 1"
```

### 4. Publicar Tema en Vivo (Publish)
Una vez validado el tema en desarrollo y listo para producción:

```bash
shopify theme publish
```

---

## 🎨 Pilares del Diseño Exclusivo (Design Taste)

Este tema implementa una arquitectura estética de alta gama:
*   **Colores HSL Dinámicos:** Evita contrastes planos agresivos usando una paleta armónica calibrada.
*   **Relación de Aspecto en Imágenes:** Uso riguroso de `aspect-ratio` y `object-fit: cover` en componentes para evitar saltos de maquetación (CLS) y mantener simetría total.
*   **Micro-interacciones Fluidas:** Físicas de tipo resorte (Spring Curves) aplicadas exclusivamente mediante aceleración por GPU (`transform` e `opacity`).
*   **SEO & Accesibilidad (A11y):** Marcado HTML semántico, etiquetas descriptivas `alt` dinámicas, y estructura estructurada de datos (JSON-LD) precargada en servidor.

---
*Desarrollado para Ritual Cosmético - Mayo 2026.*
