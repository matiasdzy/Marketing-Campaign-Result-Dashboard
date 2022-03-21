# RESULTADOS DE CAMPAÑAS DE MARKETING - MAVEN ANALYTICS
## TEMÁTICA SELECCIONADA
Se analizan los resultados de una campaña de marketing digital de 2240 clientes de [Maven Marketing](https://www.mavenanalytics.io/data-playground?order=fields.numberOfRecords&pageSize=20&search=Marketing%20Campaign%20Results) realizada durante el período de 2012 a 2014. 
Los resultados incluyen perfiles de clientes, preferencias de productos, éxitos/fracasos de campaña y rendimientos del canal.
Se pretende analizar qué factores están relacionados con el perfil del cliente medio, identificar cuál es la campaña más exitosa, cuáles son los canales de menor rendimiento, cuales factores se relacionan con el número de compras web y cuales productos tienen mayor preferencia.

## OBJETIVOS

>- <p style="text-align: justify;">Conocer modelo de negocios de la empresa Maven Marketing e identificar información requerida por el usuario para facilitar el proceso de toma de decisiones.</p>
>- <p style="text-align: justify;">Generar un modelo relacional y transformar los datos para que puedan ser usados por la herramienta de visualización.</p>
>- <p style="text-align: justify;">Diseñar KPIs que permitan evaluar de manera rápida e intuitiva los resultados de las campañas de marketing.</p>
>- <p style="text-align: justify;">Implementar técnicas de storytelling para la presentación de los datos trabajados y validar la información con el usuario final del tablero.</p>

## DIAGRAMA ENTIDAD RELACIÓN EER
<p style="text-align: justify;">A partir del dataset con tabla unica, se procedió a normalizar la tabla e identificar las relaciones entre cada una de las tablas. A continuación se presenta, un esquema de los resultados.</p>

![Imgur](https://imgur.com/O9XGe8Q.png?1)

<p style="text-align: justify;">Posteriormente se realizó un listado de las tablas, donde se identificaron los tipos de clave, nombre del campo y tipo de dato como se muestra en el siguiente ejemplo:</p>

![Tabla](https://i.imgur.com/m9vSlbl.png)


## MOCKUPS

<p style="text-align: justify;">Se crearon prototipos del dashboard que sirven para dilusidar el resultado esperado. Se presentan los prototipos de cada solapa:</p>

![Imgur](https://i.imgur.com/vEkqbsX.png?1)

![Imgur](https://i.imgur.com/8XEBgfe.png?1)

![Imgur](https://i.imgur.com/DVpogXp.png?1)
	
## LIMPIEZA Y MODELADO DE DATOS
<p style="text-align: justify;">La normalización y limpieza de datos de todas las tablas se realizo con PowerQuery. Se realizaron las conexiones en las tablas obteniendose como resultado el siguiente modelo de datos tipo estrella: </p>

![Imgur](https://i.imgur.com/tJM1FJc.png?2)

## MEDIDAS CALCULADAS
<p style="text-align: justify;">Se crearon una serie de medidas calculadas necesarias para realizar calculos con los datos que permitan obtener indormación relevante en cada solapa, yendo de lo más general a los más específico. Algunas de las medidas realizadas fueron: </p>
##### PRIMER SOLAPA 
###### Ticket Promedio: 
<p style="text-align: justify;">Es un indicador clave ya que siempre será más fácil incrementar el importe medio de cada pedido antes que captar nuevos clientes, además, las acciones que puedes realizar para incrementar tu ticket medio suelen ser mucho más baratas que acciones de captación como Cross-selling y Upselling.</p>

~~~
ticket_promedio =
VAR productos_vendidos =
	SUM ( Productos[Cant_producto] )
VAR compras_totales = [cant_compras_total]
RETURN
	DIVIDE ( productos_vendidos, compras_totales )
~~~

###### Porcentaje de conversión: 
<p style="text-align: justify;">Esta medida tiene como objetivo informar sobre el porcentaje de visitantes de la página web realizan al menos una compra. Es un indicador clave ya que se relaciona con la experiencia del cliente en el proceso de compra. Representa una métrica clave ya que de su valor se desprende la decisión de lanzar o no campañas de captación, el valor mínimo recomendado es del 3 al 5%.</p>

~~~
%conversion = 
VAR ComprasMes =
    CALCULATE ( COUNT ( Compras[ID] ), Compras[Recency] < 30 )
VAR NumeroVisitasMes =
    SUM ( Compras[NumWebVisitsMonth] )
VAR NuevasCompras = ComprasAdd[Valor ComprasAdd]
RETURN
    ( ( ComprasMes + ComprasAdd[Valor ComprasAdd] ) / NumeroVisitasMes )
~~~

###### Acumulado total de productos: 
<p style="text-align: justify;">Esta medida tiene como objetivo formar la línea del eje secundario del Pareto que indica el monto acumulado, lo que permite mostrar cuales 20% de productos representan el 80% del monto recaudado.</p>

~~~
% Total acumulado productos = 
VAR acumulado =
    CALCULATE (
        [Ventas por producto],
        TOPN (
            [Ranking productos],
            ALLSELECTED ( Productos[Producto] ),
            [Ventas por producto], DESC
        )
    )
VAR total =
    CALCULATE (
        [Ventas por producto],
        ALLSELECTED ( Productos[Producto] )
    )
RETURN
    DIVIDE ( acumulado, total )
~~~

###### Variación anual de subscriptores: 
<p style="text-align: justify;">Esta medida tiene como objetivo informar sobre la variación de subscriptores respecto del año pasado</p>

~~~
% Var. subs = 
VAR SubsAnuales = COUNT(Consumidor[Dt_Customer])
VAR SubsLY = CALCULATE( Medidas[Subsripciones], SAMEPERIODLASTYEAR(Calendario[Date]))
RETURN
    IF (
        ISBLANK ( SubsLY ),
        BLANK (),
        DIVIDE ( ( SubsAnuales - SubsLY ), SubsLY, 0 )
    )
~~~

##### SEGUNDA SOLAPA 
###### Nuevos clientes: 
<p style="text-align: justify;">Esta medida informa cuantos clientes nuevos se incorporaron a la compañía en los últimos 30 días.</p>

~~~
Nuevos Clientes = 
CALCULATE(COUNT(Consumidor[Dt_Customer]), Consumidor[Antiguedad] < 30 )
~~~

##### TERCER SOLAPA 
###### % Total acumulado (cantidad): 
<p style="text-align: justify;">Esta medida tiene como objetivo mostrar cuales 20% de los tipos de clientes abarcan el 80 % de la cantidad de compras. Para calcular esta medida, previamente se debe calcular el ranking de los tipos de clientes con mayor cantidad de ventas ordenado de mayor a menor a través de la siguiente medida:</p>

~~~
Ranking clusteres = 
RANKX (
    ALLSELECTED ( Consumidor[Tipos de consumidores] ),
    [cant_compras_total],
    ,
    DESC
)
~~~

<p style="text-align: justify;">Luego, se procede a calcular la medida que acumula los valores de ese ranking para obtener la línea acumulada del gráfico secundario.</p>

~~~
% Total acumulado (cantidad) = 
VAR acumulado =
    CALCULATE (
        [cant_compras_total],
        TOPN (
            [Ranking clusteres],
            ALLSELECTED ( Consumidor[Tipos de consumidores] ),
            [cant_compras_total], DESC
        )
    )
VAR total =
    CALCULATE (
        [cant_compras_total],
        ALLSELECTED ( Consumidor[Tipos de consumidores] )
    )
RETURN
    DIVIDE ( acumulado, total )
~~~

## CLUSTERIZACIÓN DE CLIENTES
<p style="text-align: justify;">Se realizó un clúster a partir de datos de cantidad de productos comprados, monto de los productos, ingresos de los clientes y edad.Esto permite direccionar campañas y productos al agrupar a los clientes en clientes malos, regulares, buenos y excelentes. 
**Malo:** Cliente que compra poco y por un bajo monto.
**Regular:** Cliente que compra una cantidad moderada y monto moderado.
**Bueno:** Cliente que compra una mayor cantidad de productos, pero su diferencia principal es el monto de compra.
**Excelente:** Cliente que compra una amplia cantidad de productos con los montos más altos.</p>

![Imgur](https://i.imgur.com/QVjzL5E.png?1)

## RESULTADOS FINALES
##### CARÁTULA
<p style="text-align: justify;">Esta solapa permite identificar las distintas solapas así como también los datos de responsables y última actualización</p>

![Imgur](https://i.imgur.com/7aq2fqA.png)

------------

##### GLOSARIO
<p style="text-align: justify;">Esta solapa permite explicar los distintos términos utilizados a lo largo del informe</p>

![Imgur](https://i.imgur.com/FWzkLfq.png)

------------

##### PRIMER SOLAPA  (PODUCTOS Y CANALES)
<p style="text-align: justify;">Esta solapa permite identificar de manera general, la situación actual de las campañas y el relacionamiento de las ventas de los diferentes productos de acuerdo con el país, canal de venta, ingresos y educación.</p>

![Imgur](https://i.imgur.com/shHHlOr.png)

------------

##### SEGUNDA SOLAPA  (CLIENTES)
<p style="text-align: justify;">Permite conocer de manera más especifica que la solapa anterior a los consumidores ya que en las diferentes gráficas y filtros observamos todas las variables que los identifican.</p>

![Imgur](https://i.imgur.com/S8XPErh.png)


------------

##### TERCER SOLAPA  (CAMPAÑAS)
<p style="text-align: justify;">En esta última solapa se visualiza el desempeño de cada una de las campañas de acuerdo con distintas variables.</p>

![Imgur](https://i.imgur.com/1anZYE0.png)

------------

## CONCLUSIONES
- <p style="text-align: justify;">Se logró abordar un proyecto de data analytics de forma integral, permitiendo conocer el modelo de negocios de la empresa Maven Marketing a través de un tablero que muestra información de lo más general a lo más específico.</p>

- <p style="text-align: justify;">De la solapa “Productos y Canales” se observa que el 80% del monto facturado se debe al producto 1 (vino) y al producto 3 (carnes). El mercado que más consume estos productos es el español y lo realiza a través de tiendas físicas.</p>

- <p style="text-align: justify;">De la solapa “Clientes” y “Campañas” se observa que los clientes buenos y excelentes representa el 82,06 % del monto facturado, estos clientes compran los entre 10 y 30 productos con los montos más elevados, son graduados y la mayor cantidad de ellos tienen entre 35 y 50 años. </p>

- <p style="text-align: justify;">Se observa que los clientes regulares y malos compran muchos productos pero no son de valor representativo, esto debe tenerse en cuenta para reducir costos, ya que estas compras son las que contribuyen en mayor medida en los costos de comercialización.</p>

------------

## LINEAMIENTOS FUTUROS
<p style="text-align: justify;">Se recomienda avanzar en análisis predictivo de cantidad de subscriptores y tipo de los mismos para poder planificar con antelación estrategias que permitan a la compañía satisfacer las necesidades de los clientes.</p>
