https://microservices.io/patterns/data/saga.html

## Contexto

Ha aplicado la base de datos por patrón de servicio. Cada servicio tiene su propia base de datos. Sin embargo, algunas transacciones comerciales abarcan varios servicios, por lo que necesita un mecanismo para implementar transacciones que abarquen servicios. Por ejemplo, imaginemos que está construyendo una tienda de comercio electrónico donde los clientes tienen un límite de crédito. La aplicación debe garantizar que un nuevo pedido no supere el límite de crédito del cliente. Dado que los pedidos y los clientes se encuentran en diferentes bases de datos propiedad de diferentes servicios, la aplicación no puede simplemente usar una transacción ACID local.

## Problema

¿Cómo implementar transacciones que abarcan servicios?

## Solución

Implementar cada transacción comercial que abarque múltiples servicios es una saga. Una saga es una secuencia de transacciones locales. Cada transacción local actualiza la base de datos y publica un mensaje o evento para activar la próxima transacción local en la saga. Si una transacción local falla porque viola una regla comercial, la saga ejecuta una serie de transacciones compensatorias que deshacen los cambios realizados por las transacciones locales anteriores.

![imagen](https://user-images.githubusercontent.com/9199380/89132509-b013b380-d4e2-11ea-9f06-905584dbfe87.png)

Hay dos formas de coordinación de sagas:

     Coreografía: cada transacción local publica eventos de dominio que desencadenan transacciones locales en otros servicios
     Orquestación: un orquestador (objeto) le dice a los participantes qué transacciones locales deben ejecutar
     
![imagen](https://user-images.githubusercontent.com/9199380/89132540-efda9b00-d4e2-11ea-84e5-4e65b046068f.png)

Una aplicación de comercio electrónico que utiliza este enfoque crearía un pedido utilizando una saga basada en coreografía que consta de los siguientes pasos:

     El servicio de pedidos recibe la solicitud POST / pedidos y crea un pedido en estado PENDIENTE
     Luego emite un evento de pedido creado
     El controlador de eventos del Servicio de atención al cliente intenta reservar crédito
     Luego emite un evento que indica el resultado
     El controlador de eventos de OrderService aprueba o rechaza el pedido
     
 ![imagen](https://user-images.githubusercontent.com/9199380/89132571-29130b00-d4e3-11ea-9bd8-4d9ac7e4d880.png)
 
 Una aplicación de comercio electrónico que utiliza este enfoque crearía un pedido utilizando una saga basada en la orquestación que consta de los siguientes pasos:

     El Servicio de pedidos recibe la solicitud POST / pedidos y crea el orquestador de la saga Crear pedido
     El orquestador de la saga crea una orden en estado PENDIENTE
     Luego envía un comando de Reserva de Crédito al Servicio al Cliente
     El Servicio al Cliente intenta reservar crédito
     Luego envía un mensaje de respuesta que indica el resultado
     El orquestador de la saga aprueba o rechaza la orden
     
##Contexto resultante

Este patrón tiene los siguientes beneficios:

     Permite que una aplicación mantenga la consistencia de datos en múltiples servicios sin usar transacciones distribuidas

Esta solución tiene los siguientes inconvenientes:

     El modelo de programación es más complejo. Por ejemplo, un desarrollador debe diseñar transacciones compensatorias que deshacen explícitamente los cambios realizados anteriormente en una saga.

También hay que abordar los siguientes problemas:

     Para ser confiable, un servicio debe actualizar atómicamente su base de datos y publicar un mensaje / evento. No puede usar el mecanismo tradicional de una transacción distribuida que abarca la base de datos y el intermediario de mensajes. En su lugar, debe usar uno de los patrones enumerados a continuación.

