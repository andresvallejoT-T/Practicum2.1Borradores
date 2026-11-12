# Practicum2.1Borradores

## Catálogo de fuentes

Dos fuentes principales y dos de apoyo, consultadas el 11 de noviembre de 2026.

| Campo   | F1 · INEC   | F2 · SRI                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Conjunto de datos | [Estimaciones y Proyecciones de Población, Revisión 2024](https://www.ecuadorencifras.gob.ec/proyecciones-poblacionales/) | [Catastro RUC por provincia](https://www.sri.gob.ec/datasets), archivo de Loja (SRI_RUC_Loja.zip)  |
| Publicador   | Instituto Nacional de Estadística y Censos  | Servicio de Rentas Internas  |
| Qué contiene  | Población por cantón y año, por sexo y edad, ajustada al Censo 2022 con proyecciones hasta el 2050 | Un registro por contribuyente: provincia, cantón, estado, actividad y fechas |
| Licencia    | [CC BY 4.0](https://www.ecuadorencifras.gob.ec/informacion-censal-cantonal/) |  [CC BY 4.0](https://www.ecuadorencifras.gob.ec/informacion-censal-cantonal/) |
| Formato      | Excel (.xlsx) | CSV comprimido en .zip |
| Estructura     | Tabla agregada, ancha  | Registro a registro, larga `nota: separador` "&#124;" |
| Unidad       | Personas  | Contribuyentes (conteo)  |
| Versión           | Revisión 2024, base Censo 2022  | Version 1/11/2026 (Ultimo registro actualizado)    |
| Diccionario       | Dentro del Excel metadas[^1]   | [Diccionario de variables RUC](https://www.sri.gob.ec/o/sri-portlet-biblioteca-alfresco-internet/descargar/a425b664-828d-4ef4-a103-fe689237d25f/SRI_Diccionario_Catastro.xlsx) |
                                                                                                                                       
[^1] sin embargo muestra indicadores y formulas mas no se encontro un diccionario y solo se puede inferir


## Análisis de compatibilidad

Las fuentes son compatibles a nivel de cantón y año. Las diferencias son de formato, de nivel de detalle y de cómo nombran los territorios; todas se resuelven en la transformación.


| Aspecto                       | F1 · INEC                                              | F2 · SRI                                               | Compatible         | Regla de transformación                                                                                    |
| ----------------------------- | ------------------------------------------------------ | ------------------------------------------------------ | ------------------ | ---------------------------------------------------------------------------------------------------------- |
| Formato de archivo            | Excel con títulos, notas y encabezados en varias filas | CSV dentro de un .zip                                  | Con transformación | Descomprimir el .zip; leer el Excel saltando las filas de título y notas                                   |
| Nivel de detalle              | Agregado: un total por cantón                          | Detallado: una fila por contribuyente                  | Con transformación | Contar contribuyentes por cantón (groupby) antes de unir                                                   |
| Forma de la tabla             | Ancha: los años o grupos van en columnas               | Larga: una fila por registro                           | Con transformación | Pasar el Excel a formato largo (melt)                                                                      |
| Identificación del territorio | Código DPA y nombre                                    | Solo nombre de provincia y cantón, en texto            | No directo         | Normalizar nombres (mayúsculas, sin tildes, sin espacios dobles) y mapearlos al código DPA con la tabla A1 |
| Nombres de cantón             | Nombre oficial                                         | Puede variar en tildes, Ñ o abreviaturas               | Con transformación | Tabla de equivalencias para los casos que no crucen; registrarlos como error                               |
| Fecha                         | Año (entero)                                           | Fechas de inicio, suspensión o reinicio de actividades | Con transformación | Convertir a fecha estándar (AAAA-MM-DD) y extraer el año                                                   |
| Corte temporal                | Serie anual                                            | Foto del catastro al día de la descarga                | Parcial            | Cruzar el conteo actual con la población del año actual; los años pasados solo como aproximación           |
| Estado del registro           | No aplica                                              | Activo, suspendido u otros                             | Con transformación | Contar solo contribuyentes activos para el indicador principal                                             |
| Unidad                        | Personas                                               | Contribuyentes                                         | Sí                 | Guardar la unidad por indicador                                                                            |
| Codificación y separador      | Excel, sin problema                                    | Por confirmar (tildes y Ñ)                             | Por verificar      | Detectar codificación y separador al leer                                                                  |

**Esquema común propuesto**

Una fila por observación, igual para las dos fuentes:

| Columna    | Tipo    | Ejemplo                       |
| ---------- | ------- | ----------------------------- |
| codigo_dpa | texto   | 1101 (cantón Loja)            |
| anio       | entero  | 2026                          |
| indicador  | texto   | poblacion_total / ruc_activos |
| valor      | decimal | número                        |
| unidad     | texto   | personas / contribuyentes     |
| fuente_id  | entero  | 1 / 2                         |

Indicador integrado: RUC activos por cada 1.000 habitantes, por cantón, calculado a partir de las dos filas.

**Validaciones que salen de este análisis**

- Cada cantón del SRI debe encontrar su código DPA; los que no, se registran como error.
- La provincia de Loja debe tener 16 cantones en ambas fuentes.
- valor no nulo y mayor o igual a cero.
- La suma de población de los cantones debe igualar el total provincial del mismo Excel.
- Sin duplicados de (codigo_dpa, anio, indicador, fuente_id) ni de número de RUC dentro del catastro.

Los nombres exactos de columnas, el separador y la codificación del CSV del SRI no se pudieron abrir desde aquí. Hay que confirmarlos con el diccionario de variables al descargar; si algo cambia, se ajusta esta tabla.

## Riesgos y pendientes

| Riesgo                                              | Mitigación                                                                              |
| --------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Nombres de cantón del SRI que no cruzan con el INEC | Tabla de equivalencias y registro de cada caso en la bitácora de errores                |
| El SRI reemplaza el archivo sin aviso               | Guardar cada descarga con fecha y hash; el hash evita cargar dos veces el mismo archivo |
| Archivos grandes al pasar a todo el Ecuador         | Empezar con Loja; para el resto, leer por partes (chunksize)                            |
| El catastro es una foto actual, no una serie        | Documentarlo como limitación en el informe experimental                                 |
| Licencia del SRI no explícita                       | Citar al SRI como fuente y preguntar al profe si basta                                  |

**Pendientes antes del 15 de noviembre**

- ☐ Descargar SRI_RUC_Loja.zip, el diccionario del RUC y el Excel de proyecciones cantonales del INEC
- ☐ Descargar la tabla de códigos DPA del INEC
- ☐ Confirmar columnas, separador y codificación del CSV del SRI
- ☐ Contar filas, cantones y valores vacíos de cada archivo
- ☐ Probar el cruce de los 16 cantones de Loja
- ☐ Generar el diagrama entidad-relación en draw.io

**Fuentes consultadas**

- [INEC · Información censal y códigos DPA](https://www.ecuadorencifras.gob.ec/informacion-censal-cantonal/)
- [INEC · Estimaciones y proyecciones de población](https://www.ecuadorencifras.gob.ec/proyecciones-poblacionales/)
- [SRI · Datasets](https://www.sri.gob.ec/datasets)
- [El Comercio · Portal de datos abiertos del SRI](https://www.elcomercio.com/actualidad/negocios/sri-impulsa-el-acceso-ciudadano-a-la-informacion-con-su-portal-de-datos-abiertos-en-ecuador/)
