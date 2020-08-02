https://microservices.io/patterns/data/saga.html

## Contexto

Ha aplicado la base de datos por patrón de servicio. Cada servicio tiene su propia base de datos. Sin embargo, algunas transacciones comerciales abarcan varios servicios, por lo que necesita un mecanismo para implementar transacciones que abarquen servicios. Por ejemplo, imaginemos que está construyendo una tienda de comercio electrónico donde los clientes tienen un límite de crédito. La aplicación debe garantizar que un nuevo pedido no supere el límite de crédito del cliente. Dado que los pedidos y los clientes se encuentran en diferentes bases de datos propiedad de diferentes servicios, la aplicación no puede simplemente usar una transacción ACID local.

## Problema

¿Cómo implementar transacciones que abarcan servicios?

## Solución

Implementar cada transacción comercial que abarque múltiples servicios es una saga. Una saga es una secuencia de transacciones locales. Cada transacción local actualiza la base de datos y publica un mensaje o evento para activar la próxima transacción local en la saga. Si una transacción local falla porque viola una regla comercial, la saga ejecuta una serie de transacciones compensatorias que deshacen los cambios realizados por las transacciones locales anteriores.
