# Blog con MkDocs Material

Un blog simple y elegante creado con MkDocs Material.

## 🚀 Características

- Diseño moderno y responsive
- Tema Material Design
- Sintaxis Markdown para contenido
- Búsqueda integrada
- SEO optimizado
- Fácil personalización

## 📦 Instalación

```bash
# Clonar el repositorio
git clone https://github.com/vladimir1284/blog

# Instalar dependencias
pip install -r requirements.txt
```

## 🛠️ Desarrollo

```bash
# Servidor local
mkdocs serve

# Construir sitio
mkdocs build
```

## 📝 Uso

1. Edita los archivos Markdown en la carpeta `docs/`
2. Añade posts nuevos en `docs/blog/posts/`
3. Configura el sitio en `mkdocs.yml`
4. Ejecuta `mkdocs serve` para previsualizar
5. Publica con `mkdocs build`

## 📁 Estructura del Proyecto

```
├── docs/
│   ├── assets/
│   │   └── img/
│   │       ├── blog.png
│   │       └── logo.png
│   ├── blog/
│   │   ├── index.md          # Página principal del blog
│   │   └── posts/
│   │       └── myfirst.md    # Artículos del blog
│   └── index.md              # Página de inicio
├── mkdocs.yml                # Configuración
├── README.md
└── requirements.txt          # Dependencias
```

## 📄 Contenido

- **Página principal**: `docs/index.md`
- **Blog**: `docs/blog/index.md` 
- **Posts**: `docs/blog/posts/`
- **Imágenes**: `docs/assets/img/`

## 🌐 Despliegue

```bash
# GitHub Pages
mkdocs gh-deploy
```
