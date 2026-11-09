# Valoración de ISA por DCF en Python + plantilla DCF genérica

Proyecto final de **Renta Variable 2026 (EIA / TBS)**: valoración de **Interconexión Eléctrica S.A. E.S.P. (ISA)** con las 9 etapas de Damodaran.
La valoración se hizo originalmente en Excel con fórmulas vivas. Este repositorio la pasa a Python en dos notebooks:

| Notebook | Qué hace |
|---|---|
| [`notebooks/01_ISA_DCF.ipynb`](notebooks/01_ISA_DCF.ipynb) | Réplica exacta del modelo de ISA. Da **$23.763 por acción**, el mismo valor del Excel, y lo verifica con un `assert`. |
| [`notebooks/02_Plantilla_DCF.ipynb`](notebooks/02_Plantilla_DCF.ipynb) | Plantilla genérica para valorar cualquier empresa no financiera por DCF del FCFF, con chequeos de sensatez, sensibilidades y decisión. |

Equipo: Gregorio Gómez, Cipriano Gómez y Miguel Vallejo. Fecha de corte: 30-oct-2026.

## Cómo correrlo

```bash
pip install -r requirements.txt
jupyter lab notebooks/
```

Los notebooks leen los CSV de `data/`. Para probar otro escenario, cambie el CSV y corra todo de nuevo.

## Notebook 1 — DCF de ISA

Cada sección corresponde a una hoja del Excel:

1. **Riesgo y retorno (E2):** retornos mensuales de ISA y del MSCI COLCAP a 60 meses: volatilidad, Sharpe y drawdown.
2. **Moneda y riesgo país (E3):**
   - Valoramos en COP nominal.
   - Rf = Treasury 10 años − default de EE. UU. (4,70%), pasada a COP con Fisher.
   - CRP ponderada por los ingresos de cada país (2,64%, λ = 1).
3. **Beta (E4):**
   - La regresión contra el COLCAP da 1,10 (R² 0,55) y la usamos solo como contraste.
   - La beta que usamos es la bottom-up de Damodaran por negocio (Energía 0,46, Vías 0,55, Telecom 0,50), re-apalancada con Hamada: **0,756**.
4. **Proyección 2026E-2036E (hoja Proyeccion):**
   - Ingresos por segmento y país, indexados a su inflación y moneda, más rendimiento sobre el capex del año anterior.
   - Desde 2031 hay erosión regulatoria.
   - Sobre esa base se calculan EBITDA, NOPAT, reinversión, FCFF, ROIC, utilidad por acción, dividendos y deuda neta.
   - También incluye el efecto de conversión de 2026 (revaluación del peso) y la consolidación de IE Madeira.
5. **Costo de capital (E5):** Ke COP 13,03%, Kd COP 10,31% antes de impuestos, D/(D+E) 46,6% y **WACC 10,18%**.
6. **DCF (E7):**
   - Se descuentan los FCFF 2027E-2036E y un valor terminal con g = 4,7% (0% real) y reinversión = g / RONIC.
   - El puente va de la firma al patrimonio: co-controladas, caja, deuda y minoritarios a mercado.
   - Al final se suma la cuota de dividendo de diciembre y el valor se trae a la fecha de corte.
   - Verificaciones incluidas: Fisher en USD, conciliación FCFF-FCFE, DDM, matriz WACC × g, y el WACC y el g implícitos en el precio.
7. **Decisión (E9):** valor justo = 70% DCF + 30% comparables; precio objetivo a 12 meses.

## Notebook 2 — Plantilla genérica

Llene dos archivos:

- `data/plantilla_supuestos.csv`: supuestos escalares: precio, acciones, tasas, beta desapalancada, deuda, caja, g y RONIC. Si llena `wacc_manual`, ese WACC reemplaza el calculado.
- `data/plantilla_proyeccion.csv`: drivers año a año: crecimiento de ingresos, margen EBIT, D&A / ingresos, capex / ingresos y Δ capital de trabajo / Δ ingresos.

El notebook calcula:

- El Ke por CAPM con riesgo país, el Kd sintético y el WACC en moneda local.
- La proyección del FCFF y el valor terminal con reinversión consistente (g / RONIC).
- El puente de la firma al valor por acción.
- Seis **chequeos de sensatez**: g < WACC; g real ≤ 2%; RONIC cerca del WACC; valor terminal < 80%; último crecimiento ≈ g; capex ≥ D&A.
- La matriz WACC × g y un gráfico tornado.
- El WACC y el g implícitos en el precio, y una recomendación con umbral de ±15%.

Los datos que trae son de una **empresa ficticia** ("Empresa Ejemplo S.A."); reemplácelos por los de su empresa.

## Datos (`data/`)

| Archivo | Contenido |
|---|---|
| `isa_supuestos.csv` | Supuestos escalares de ISA, con su unidad, descripción y fuente: tasas, primas de riesgo, betas, impuestos, rendimientos del capex, erosión, IE Madeira, mercado y base 2025. |
| `isa_proyeccion_anual.csv` | Insumos año a año 2025A-2036E: inflaciones, tasas de cambio, plan de capex por segmento, ingresos 2025 por segmento, margen, concesiones, costo financiero y RBSE. Una celda vacía significa que el modelo la calcula. |
| `isa_conversion_2026.csv` | Tasas de cierre de dic-25 por moneda y los pesos usados para convertir el balance a la tasa spot de la fecha de corte. |
| `isa_mercado_mensual.csv` | Precios mensuales y total return de ISA y del MSCI COLCAP, IPC de Colombia y tasa BanRep (nov-2021 a oct-2026). |
| `plantilla_supuestos.csv`, `plantilla_proyeccion.csv` | Insumos de ejemplo de la plantilla. |

Fuentes: estados financieros de ISA (2025 y 2T26), Bloomberg (precios, tasas, consenso), Damodaran (ERP, CRP, betas por sector, spreads por rating) y el plan de inversión de ISA.

## Hallazgos

| | COP por acción |
|---|---:|
| **DCF del FCFF (valor intrínseco)** | **23.763** |
| FCFE (conciliado) | 23.460 |
| DDM | 16.647 |
| Comparables (mediana núcleo) | 26.709 |
| Valor justo (70% DCF + 30% comparables) | 24.647 |
| **Precio objetivo a 12 meses** | **26.800** |
| Precio de mercado (30-oct-2026) | 27.220 |

**Recomendación: VENDER.** El retorno total esperado a 12 meses es 2,4% (−1,5% de precio + 4,0% de dividendo), muy por debajo del Ke de 13,0%.

1. **El DCF está 12,7% por debajo del precio.**
   - La acción descuenta un WACC de 9,9% (vs. 10,18%) o un g a perpetuidad de 7,0% nominal (2,2% real).
   - Ninguno de los dos se justifica para un negocio regulado que crece con la inflación.
   - El mercado parece pagar una opción de control: un múltiplo de toma de control de ~9,65x EBITDA daría ~$38.700.
2. **ISA es un "bono indexado con opción de crecimiento".**
   - El ROIC proyectado (10,5%-11,6%) queda apenas por encima del WACC (10,2%).
   - Por eso el crecimiento crea poco valor: el plan de $28 billones aumenta el tamaño, pero no mucho el valor por acción.
3. **El valor terminal pesa 64,5% del valor operativo,** así que el resultado depende mucho del WACC.
   - WACC ± 0,5pp y g ± 0,5pp dan un rango de $18.170 a $30.478.
   - El precio de mercado solo se alcanza con un WACC cercano a 9,9%.
4. **La beta importa.**
   - La regresión contra el COLCAP (1,10) mide el riesgo frente a un índice local donde ISA pesa ~7%.
   - La bottom-up (0,756) refleja mejor un negocio regulado diversificado en 5 países.
5. **Consistencia interna:**
   - El DCF en USD convertido al spot da el mismo valor que en COP: Fisher se cumple con diferencia 0.
   - El FCFE difiere del FCFF en ~1%, porque la deuda neta proyectada cambia la estructura de capital mientras el WACC la supone constante.
   - El DDM da menos: solo valora el dividendo (payout de 50%), y la utilidad retenida se reinvierte a un ROE (~12,8% en 2036E) que apenas iguala el Ke (13,0%), así que casi no agrega valor.
6. **ISA rindió menos que su índice.** En los 60 meses, ISA tuvo más volatilidad que el COLCAP (29% vs. 20% anual) y menor retorno (14% vs. 22% aritmético anual, con dividendos).

> Trabajo académico. No es una recomendación de inversión.
