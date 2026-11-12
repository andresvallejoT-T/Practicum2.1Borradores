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
