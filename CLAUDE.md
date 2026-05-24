# AviVet — Sitio GitHub Pages

Sitio estático desplegado en **avivet.cl** (GitHub Pages, repo `Alazoe/alazoe.github.io`).
Clon local en `/tmp/avivet-site/`. Push a `main` publica automáticamente.

## Aplicación principal: Calendarios de Vacunación (`vacunas/index.html`)

Un solo archivo HTML+JS (~1300 líneas). Sin backend, sin build. Todo el catálogo vive en el JS del archivo.

### Estructura del JS

| Objeto | Descripción |
|--------|-------------|
| `ENFERMEDADES[]` | Tarjetas seleccionables. Cada una tiene `id`, `nombre`, `desc`. |
| `CATALOGO[]` | Vacunas con FT SAG. Campos: `id`, `nombre`, `tipo`, `protege[]`, `cepa`, `via_label`, `edad_min`, `programa`, `resguardo`, `almacenamiento`, `proveedor`, `reg_sag`, `postura`, `notas`. |
| `RANGOS{}` | `vacunaId → [{label, min, max, default, nota}]`. Controla los sliders. |
| `SIN_FT[]` | Vacunas conocidas sin FT cargada. Aparecen en la pestaña "Estado FT". |
| `ENF_NOMBRES{}` | `enfId → nombre display`. |
| `TIPO_GRUPOS[]` | Define los 4 grupos visuales del selector: Vectorizadas, Complejos, Vivas, Inactivadas. |

### Enfermedades disponibles (IDs)

`marek`, `gumboro`, `salmonella`, `bronquitis`, `newcastle`, `laringotraqueitis`, `viruela`, `encefalomielitis`, `eds`, `coriza`, `coccidiosis`

### Flujo de usuario

1. Selecciona enfermedades (tarjetas toggle)
2. Elige vacuna por enfermedad (agrupadas por tipo — vectorizadas primero)
3. Ajusta días con sliders; se muestra la recomendación de la FT
4. "Generar calendario" → tabla con botones ±7d por fila para mover semanas
5. Exportar PDF (jsPDF + autotable vía CDN)

### Reglas al editar el catálogo

- Agregar vacuna nueva: entrada en `CATALOGO[]` + entrada en `RANGOS{}` con el mismo `id`.
- `id` en snake_case, único. `tipo` debe ser exactamente: `"Viva"`, `"Viva (congelada)"`, `"Inactivada"`, `"Vectorizada"`, `"Complejo inmune"`.
- Vacunas sin FT SAG disponible van a `SIN_FT[]`, no a `CATALOGO[]`.
- Para vacunas de dosis única en día fijo (ej. día 1): usar `min:1, max:1, default:1` en RANGOS. El slider se renderiza deshabilitado.
- Al agregar enfermedad nueva: agregar a `ENFERMEDADES[]`, `ENF_NOMBRES{}` y el CSS `.enf-tag.tag-{id}`.

---

## Calendario de referencia — AGRICOVIAL (pollas reemplazo postura)

Programa real usado en Chile. Útil para validar rangos y coherencia de días.

### Versión actual

| Día | Sem | Enfermedad | Vacuna | Vía | Laboratorio |
|-----|-----|-----------|--------|-----|-------------|
| 1 | 0 | Marek + Laringotraqueitis | Innovax ILT-SB1 | S.C. | MSD |
| 1 | 0 | Gumboro | Novamune | spray | MSD |
| 1 | 0 | Bronquitis | Nobilis MA S | spray | Elanco |
| 1 | 0 | Salmonella (S.E.) | Salmonella Vac E | spray | Elanco |
| 28 | 4 | Bronquitis | IB 4/91 | spray | Ceva |
| 42 | 6 | Newcastle | ND C2 | spray | MSD |
| 42 | 6 | Salmonella (S.E.) | Salmonella Vac E | agua | Elanco |
| 42 | 6 | Bronquitis | Nobilis Ma5 | spray | MSD |
| 63 | 9 | Coriza | Coriza autovacuna | I.M. | Autovacuna |
| 63 | 9 | Viruela + LT + Encefalomielitis | Vectormune FP LT AE | Punción alar | Ceva |
| 63 | 9 | Bronquitis + Newcastle | IBI L | spray | Ceva |
| 98 | 14 | Coriza + ND + IBV + EDS + S.E. | Cevac Corymune 7 | I.M. | Ceva |
| 105 | 15 | **Traslado** | — | — | — |
| 126 | 18 | Bronquitis | Ibird | spray | Ceva |

### Propuesta AGRICOVIAL

| Día | Sem | Enfermedad | Vacuna | Vía | Laboratorio |
|-----|-----|-----------|--------|-----|-------------|
| 1 | 0 | Marek + Laringotraqueitis | Innovax ILT-SB1 | S.C. | MSD |
| 1 | 0 | Gumboro | Novamune | spray | MSD |
| 1 | 0 | Bronquitis | IB 4/91 | S.C. | MSD |
| 1 | 0 | Salmonella (S.E.) | Salmonella Vac E | spray | Elanco |
| 28 | 4 | Bronquitis + Newcastle | BI L | spray | Ceva |
| 42 | 6 | Salmonella (S.E.) | Salmonella Vac E | agua | Elanco |
| 56 | 8 | Bronquitis | Mass L | spray | Ceva |
| 63 | 9 | Coriza A,B,C + Salmonella | Corymune 4 | I.M. | Ceva |
| 63 | 9 | Viruela + LT + Encefalomielitis | Vectormune FP LT AE | Punción alar | Ceva |
| 84 | 12 | Bronquitis + Newcastle | BI L | spray | Ceva |
| 98 | 14 | Coriza + ND + IBV + EDS + S.E. | Corymune 7 | I.M. | Ceva |
| 105 | 15 | **Traslado** | — | — | — |
| 126 | 18 | Bronquitis | Ibird | spray | Ceva |

---

## Plan de monitoreo epidemiológico (referencia CEVA)

| Enfermedad / Vacuna | Tipo | Condición | Edad | N° muestras | Muestra | Análisis | Objetivo |
|---------------------|------|-----------|------|-------------|---------|---------|---------|
| Bronquitis Infecciosa | Monitoreo | Con signología | * | 5 | Órganos (gallinas) | IBV RT-PCR | Screening + tipificación cepa campo |
| Bronquitis Infecciosa | Monitoreo | Con/sin signología | Todas | 20 | Sangre | ELISA IBV | Screening serológico plantel |
| Bronquitis Infecciosa | Monitoreo | Sin signología | 13 sem | 20 | Sangre | ELISA IBV | Pre-vacunación inactivada |
| Coriza Infecciosa | Monitoreo | Con signología | * | 5 | Gallinas/cabezas | Microbiología + PCR | Aislamiento y tipificación cepa |
| Mycoplasma Sinoviae | Monitoreo | Con signología | * | 3 pool×5 | Tórulas traqueales | PCR MS | Detección (edad de infección) |
| Mycoplasma Sinoviae | Monitoreo | Con/sin signología | 15 y 26 sem | 20 | Sangre | ELISA MS | Contagio en primeras semanas postura |
| Vectormune FP LT AE | Seguimiento | Lote vacunado | 13 sem | 20 | Sangre | ELISA ILT-Idvet | Rendimiento vacunal LT |
| Corymune 4-7 | Seguimiento | Lote vacunado | 18 sem | 20 | Sangre | ELISA ND, IBV, S.E. | Respuesta serológica prevacunación postura |

---

## Vacunas presentes en el calendario de referencia sin FT SAG cargada

Están en `SIN_FT[]` en el código. Agregar a `CATALOGO[]` cuando se obtenga la FT oficial:

| Vacuna | Tipo | Proveedor | Protege |
|--------|------|-----------|---------|
| Innovax ILT-SB1 | Vectorizada | MSD / Intervet Chile | Marek + Laringotraqueitis |
| Nobilis IB 4/91 | Viva | MSD / Intervet Chile | Bronquitis (cepa 4/91) |
| Salmonella Vac E | Viva | Elanco | Salmonella |
| Corymune 4 (autovacuna) | Inactivada | Autovacuna | Coriza |

---

## Repositorios

| Repo | URL | Propósito |
|------|-----|-----------|
| `Alazoe/alazoe.github.io` | github.com/Alazoe/alazoe.github.io | Sitio producción (este repo) |
| `Alazoe/AviVet_Vacunas` | github.com/Alazoe/AviVet_Vacunas | Proyecto Flask local (puerto 5001) |

## Workflow habitual

```bash
# Editar en
/tmp/avivet-site/vacunas/index.html

# Publicar
git -C /tmp/avivet-site add vacunas/index.html
git -C /tmp/avivet-site commit -m "mensaje"
git -C /tmp/avivet-site push origin main
```
