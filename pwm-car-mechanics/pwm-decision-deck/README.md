# pwm-decision-deck

Presentación en español que explica y defiende la decisión **PWM-002** del proyecto *PWM Learning Site*: **Vite como bundler con salida completamente estática** — no SSG, no SPA, no SSR.

Construida con [Slidev](https://sli.dev) — que, convenientemente, corre sobre Vite.

## Ejecutar

```bash
pnpm install
pnpm dev      # http://localhost:3030
pnpm build    # genera dist/
```

## Despliegue (GitHub Pages)

El workflow `.github/workflows/deploy.yml` construye y publica la presentación en cada push a `main`.

Primera vez: en el repositorio, *Settings → Pages → Source: GitHub Actions*.

## Contenido

- `slides.md` — las 12 diapositivas
- Notas del presentador incluidas como comentarios HTML

## Contexto

Creada como experimento de creación de presentaciones con **OpenCode** y skills de ingeniería (grilling → domain-modeling → implement). Forma parte del proyecto *pwm-car-mechanics*: lecciones interactivas de PWM en sistemas automotrices más un Teaching Deck interactivo.
