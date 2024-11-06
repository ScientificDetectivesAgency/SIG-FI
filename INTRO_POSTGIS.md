## Análisis de datos geográficos con PostGis

En el contexto actual, la capacidad de analizar grandes volúmenes de datos es esencial en ambitos como la ingeniería. La gestión y el análisis de datos geoespaciales son fundamentales para la toma de decisiones informadas en proyectos de exploración, evaluación y monitoreo de recursos naturales, ente muchas otras. En este ejercicio práctico, se presentará a modo de introducción el uso de SQL (Structured Query Language) junto con PostGIS, una extensión de PostgreSQL que añade soporte para datos geoespaciales.

1. __Preparando la base de datosp__
   - Manejadores de bases de datos
   - Creación de bases de datos
   - Extensión PostGIS 
   - Esquemas
2. __Consultas Básicas__
   - SELECT: selección de datos.
   - WHERE: filtrado de resultados.
   - ORDER BY y LIMIT: ordenamiento y paginación de resultados.
3. __Funciones de Agregación__
  - Uso de funciones como COUNT, SUM, AVG, MIN, MAX.
  - Agrupamiento de resultados con GROUP BY.
4. __Uniones y Subconsultas__
  - JOIN: combinando tablas.
  - Subconsultas: consultas dentro de consultas.
5. __Pre-procesamiento de datos__
   - Modificación de columanas, tipos de datos, etc.
   - Timestamp
6. Operadores espaciales
_Tarea:_ Busca las siguientes funciones y analiza cómo funcionan 
  - St_Intersects
  - St_Contains
  - St_Touches
  - St_Crosses
  - St_DWithin
# Manejadores de bases de datos y PostGIS 
En esta parte de la práctica vamos a comenzar a usar el manejador de bases de datos a través del programa DBeaver, este nos permite tener una interfáz amigable con la que podamos interactuar y hacer las consultas a nuestra base de datos. Primero vamos a comenzar con los tres pasos que indispensables para comenzar a trabajar con sql: 1) conectar el manejador a la interfáz, 2) crear una base de datos de trabajo, 3) conectarse a la base y crear la extensión de PostGIS. 

## Consultas básicas ## 

Vamos a trabajar con datos de población, esto quiere decir que los datos estarán divididos en dos: 1) los datos de las geometrías de los municipios y 2) los datos de población asociados al 
censo de Población y Vivienda INEGI, 2020. Lo primero que tenemos que hacer es [descargar los datos](https://github.com) y subirlos al manejador de la base de datos. 

Cuando estás trabajando con nuevos datos, es necesario que comiences a explorar las tablas con las que vas a trabajar. El comando _SELECT_ normalmente te ayuda en esta tarea. Cuando usas _SELECT_ seguido de un _*_ y un _FROM_ estás pidiendo todas las columnas de una tabla determinada. Recuerda que puedes limitar, tanto las columnas como las filas, para poder definir las filas puedes usar el comando _WHERE_ si es basado en una condición o un limit si sólo te importa ver un _n_ numero de filas.    

```sql
SELECT * FROM esquema.tabla; 
```
```sql
SELECT * FROM esquema.tabla LIMIT 10; 
```
```sql
SELECT * FROM esquema.tabla WHERE columna = 'ATRIBUTO'; 
```
```sql
SELECT * FROM esquema.tabla WHERE columna = 'ATRIBUTO'OR columna2 = 'OTRO ATRIBUTO'; 
```
```sql
SELECT * FROM esquema.tabla WHERE columna = 'ATRIBUTO' AND columna2 = 'OTRO ATRIBUTO'; 
```
```sql
SELECT * FROM esquema.tabla WHERE columna IN( 'ATRIBUTO','OTRO ATRIBUTO'); 
```
Además de poder consutar los datos por limites o por atributos, también es posibles ordenar los datos ya sea de forma ascendente o descendente: 

```sql
SELECT * FROM esquema.tabla WHERE columna = 'ATRIBUTO' ORDER BY columna ASC; 
```
Ahora vamos a comenzar a trabajar con la tabla que contienen datos de población, vamos a usar funciones de agregación para hacer operaciones sobre los datos.
Las funciones de agregación siempre van despues del select y deben agruparse sobre una categoría. Por ejemplo: 

¿Cúantos municipios hay por cada Entidad Federativa? 

```sql
select count(*) as municipios, fm.entidad_nombre 
from fact_municipio2020 fm
group by fm.entidad_nombre; 
```
Ahora lo ordenamos
```sql
select count(*) as municipios, fm.entidad_nombre 
from fact_municipio2020 fm
group by fm.entidad_nombre order by count(*) asc; 
```
¿Cuál es el municipio con mayor población en hogares censales indígenas? 
```sql
select phog_ind::int, municipio_nombre
from fact_municipio2020
where phog_ind!='*'
order by phog_ind::int desc;
```
¿Ahora cuál es el estado con mayor población en hogares censales indígenas?
```sql
select sum(a.phog_ind::int) hog_indigenas, a.entidad_nombre 
from fact_municipio2020 a
where a.phog_ind!='*' 
group by a.entidad_nombre
order by sum(a.phog_ind::int) desc;
```
__PUNTO EXTRA:__ La columna _tothog_ contiene los datos del total de hogares, ¿Cómo calcularías la proporción de hogares indigenas del total de hogares por Estado?

Ahora vamos a trabajar con funciones espaciales y para eso es necesario asignar la geometría a la tabla de datos, para ello haremos un join. 
```sql
select b.geom, a.* 
from since.fact_municipio2020 a
join since.dim_municipio2020 b 
on a.municipio_cvegeo = b.municipio_cvegeo;
```
Ahora vamos a crear la tabla integrada 
```sql
create table municipios as
select b.geom, a.* 
from since.fact_municipio2020 a
join since.dim_municipio2020 b 
on a.municipio_cvegeo = b.municipio_cvegeo;
```
Con los datos de inmuebles que se encuentran en la carpeta de datos vamos a estimar el numero de inmuebles en renta y venta que se encuentran disponibles en los municipios de la Zona Metropolitana
del Valle de México. Para ello vamos a intersectar el polígono de la Zona Metropolitana con los municipios de la República. 
```sql
select a.*
from municipios a 
join zmvm b
on st_intersects(a.geom, b.geom);
```
Ejercicio: 

1. Carga los datos de inundaciones y edificios dañados por el sismo de 2017 de la Ciudad de México
2. Calcula el promedio de rentas por colonia en la Ciudad de México
3. Compara el precio de las Ventas con el riesgo de inundaciones
4. Agrega una columna que se llame avg_rentas y avg_ventas a la capa de inundaciones
5. Actualizala con información de inmuebles
6. Crea una capa sólo con las colonias con riesgo Alto



















