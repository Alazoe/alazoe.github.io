# AviVet — Sitio GitHub Pages

Sitio estático desplegado en **avivet.cl** (GitHub Pages, repo `Alazoe/alazoe.github.io`).
Clon local en `~/alazoe-github-io/` (no usar los clones de `~/Documents`). Push a `main` publica automáticamente.

## Aplicación principal: Calendarios de Vacunación (`vacunas/index.html`)

Un solo archivo HTML+JS (~2100 líneas). Sin backend, sin build. Todo el catálogo vive en el JS del archivo.

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

`marek`, `gumboro`, `salmonella`, `bronquitis`, `newcastle`, `laringotraqueitis`, `viruela`, `encefalomielitis`, `eds`, `coriza`, `coccidiosis`, `hci` (Hepatitis por Cuerpos de Inclusión, solo Angavac)

### Flujo de usuario

1. Selecciona enfermedades (tarjetas toggle)
2. Elige vacuna por enfermedad (agrupadas por tipo — vectorizadas primero). Cada tarjeta muestra su laboratorio; el filtro **Laboratorio** (Todos · Ceva · MSD · Boehringer · Otros) deja ver uno solo para armar el programa con un laboratorio
3. Ajusta días con sliders; se muestra la recomendación de la FT
4. "Generar calendario" (o "Calendario AviVet" / "Programa Nutringen" / **"Plan Ceva" / "Plan MSD"**) → tabla con botones ±7d por fila
5. **Revisión técnica** automática sobre la tabla (`revisarCalendario()`), recalculada en cada `renderTabla()`
6. Exportar PDF (jsPDF + autotable vía CDN), incluye la revisión técnica y el anexo ND/BI

Pestañas: Generar Calendario · Estado de Fichas Técnicas · Resumen de Fichas (por enfermedad, con casilla "revisada" en localStorage y PDF).

### Revisión técnica del calendario

`revisarCalendario(cal, inicioPosturaDia)` devuelve alertas `{nivel, filas, dias, titulo, texto, fuente}` con nivel `conflicto` / `aviso` / `nota` y fuente `FT` o `Criterio técnico` (siempre declarar cuál). Reglas:

- Edad mínima y rango por dosis (`RANGOS`), LT-VAX ±3 días de vivas ND/BI, un solo vector HVT (`HVT_VECTOR`)
- Inactivadas con ND/BI: viva previa del mismo agente, ≥4 semanas antes
- Postura: `postura:false` desde el inicio de postura (campo "Inicio de postura", semana, default 20) y días mínimos previos en `PRE_POSTURA{}`
- Avipro Salmonella Vac E: otros productos a ≤14 días (FT SAG; la FT europea lo limita a antibacterianos ±3 días)
- Mismo día: dos vivas respiratorias (distinto laboratorio vía `labDe()`), salvo pares en `COMPATIBLES[]` declarados por FT/laboratorio; misma enfermedad dos veces solo entre vacunas de la misma clase (viva/inactivada)
- Nota: vivas respiratorias a <10 días entre sí
- Nota única: pares de vacunas a ≤14 días. Es la cláusula estándar de casi todas las FT SAG («no administrar otros productos 14 días previos y posteriores», por falta de estudios); no cuenta los pares de `COMPATIBLES` ni Avipro (que tiene su propio aviso)

Al agregar una vacuna con restricciones de su FT: sumarla a `PRE_POSTURA`, `COMPATIBLES` o `HVT_VECTOR` según corresponda.

### Planes por laboratorio (`PLANES_LAB`)

Programas de pollas de reemplazo (postura semana 20) con vacunas de **un solo laboratorio**, días dentro de los rangos de sus FT y combinaciones declaradas en `COMPATIBLES`. Al cambiarlos, cargarlos y verificar que `revisarCalendario()` no dé conflictos ni avisos.

- **Ceva**: Vectormune HVT NDV + Novamune (planta) · Mass L + Ibird spray d1 · Immucox 5 d3 · BI L d21/45/77 · Ibird d63/91 · FP-LT+AE y Corymune 4K d63 · Corymune 7K d105. Ceva no tiene Salmonella viva: la cubren 4K/7K.
- **MSD**: Innofusion ILT-IBD + Rismavac mezcladas (planta) · IB Ma5 + IB 4-91 spray d1 · ND Clone 30 d21 · Ma5 + Clone 30 d42 · 4-91 d63/105 · Salenvac ETC d56/98 · AE+POX y Coryza d70 · Coryza d105 · IB Multi+ND+EDS d112. MSD no tiene coccidiosis con FT cargada.
- **Boehringer**: no se arma plan propio. En Chile no tiene inactivadas ND/BI/EDS, ni Salmonella, Coriza, Viruela o AE.

### Boehringer Ingelheim (FT SAG, octubre 2026)

13 registros aviares en el SAG (importador Boehringer Ingelheim Ltda.). Rótulo/inserto aprobado: `https://medicamentos.sag.gob.cl/_CARGAS/ETIQUETAS/<N>-B.pdf` (el 2775 está como `2775-BP.pdf`). PDFs guardados en `~/AviVet_Vacunas/fichas_tecnicas/usadas/`.

| id | Vacuna | Reg. SAG |
|----|--------|----------|
| `bi_nd_lasota` / `bi_nd_b1` | Newcastle La Sota / B1 (sin nombre comercial) | 1169-B / 1170-B |
| `bi_nd_b1_ib_mass` / `bi_ib_mass` | ND B1 + BI Mass / BI Mass | 1173-B / 1176-B |
| `bi_ib_h120` | BI H120 comprimido efervescente | 2775-B |
| `gallivac_ib88_neo` | Gallivac IB88 Neo (CR88 = 793/B) | 2731-B |
| `lt_blen` | LT-Blen (Hudson) | 1177-B |
| `bursa_blen_m` | Bursa Blen M (G-87, FT solo broiler) | 1179-B |
| `vaxxitek_hvt_ibd` | Vaxxitek HVT+IBD (vector HVT) | 2757-B |
| `prevexxion_rn` | Prevexxion RN (Marek serotipo 1) | 2767-B |
| `bi_marek_hvt` / `bi_marek_hvt_sb1` | Marek HVT / HVT+SB1 congeladas | 1172-B / 1175-B |
| `angavac` | Angavac (adenovirus 4, HCI) | 1927-B |

Ese mismo mes se agregaron desde FT SAG: CEVAC MD HVT (2446-B), Cevac Vitapest L (2770-B), Cevac Vitabron L (2686-B), CEVAC Bron 120 L (1662-B), Innofusion ILT-IBD (2758-B), Innofusion ND-ILT (2768-B) y Nobilis Salenvac ETC (2558-B). Nobilis IB 4-91 pasó a FT SAG 2703-B. INNOVAX ILT es de MSD/Intervet (antes figuraba como Zoetis).

Faltan en el catálogo (registradas en el SAG, sin cargar): Ceva Nextmune (FT solo broiler), Gumboro Lukert, CAV, Reomune 3; MSD Paracox 8, Coccivac D2, Fortegra, Nobilis CAV P4, REO, Rhino CV, TRT, Multriva, IB+G+ND, Breedervac Reo Plus.

### Reglas al editar el catálogo

- Agregar vacuna nueva: entrada en `CATALOGO[]` + entrada en `RANGOS{}` con el mismo `id`.
- `id` en snake_case, único. `tipo` debe ser exactamente: `"Viva"`, `"Viva (congelada)"`, `"Inactivada"`, `"Vectorizada"`, `"Complejo inmune"`.
- Vacunas sin FT SAG disponible van a `SIN_FT[]`, no a `CATALOGO[]`. Excepción: si hay ficha del laboratorio (no SAG) con datos verificados, va a `CATALOGO[]` con `ft_pendiente:true` y `reg_sag:"—"`; así es seleccionable pero cuenta como "Sin FT". Hoy no hay ninguna (Nobilis IB 4-91 lo fue hasta tener la FT SAG 2703-B).
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

Están en `SIN_FT[]` en el código (o en `CATALOGO[]` con `ft_pendiente:true`). Pasar a `CATALOGO[]` normal cuando se obtenga la FT SAG:

| Vacuna | Tipo | Proveedor | Protege | Estado |
|--------|------|-----------|---------|--------|
| Corymune 4 (autovacuna) | Inactivada | Autovacuna | Coriza | `SIN_FT` |

---

## Repositorios

| Repo | URL | Propósito |
|------|-----|-----------|
| `Alazoe/alazoe.github.io` | github.com/Alazoe/alazoe.github.io | Sitio producción (este repo) |
| `Alazoe/AviVet_Vacunas` | github.com/Alazoe/AviVet_Vacunas | Espejo Flask local (puerto 5001): copiar `vacunas/index.html` a `templates/index.html` tras cada cambio |

## Workflow habitual

```bash
# Editar en
~/alazoe-github-io/vacunas/index.html

# Publicar
git -C ~/alazoe-github-io add vacunas/index.html
git -C ~/alazoe-github-io commit -m "mensaje"
git -C ~/alazoe-github-io push origin main
```
