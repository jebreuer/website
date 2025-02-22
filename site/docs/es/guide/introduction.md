---
prev: ../guide/
next: ./getting-started.md
---

# Introducción

Un bot de Telegram es una cuenta de usuario especial que está automatizada por un programa.
Cualquiera puede crear un bot de Telegram, el único prerrequisito es que sepas un poco de codificación.

> Si ya sabes cómo crear bots, dirígete a [Cómo empezar](./getting-started.md)

grammY es una biblioteca que hace que sea súper sencillo escribir un bot de este tipo.

## Cómo escribir un bot

Antes de empezar a crear tu bot, familiarízate con lo que los bots de Telegram pueden y no pueden hacer.
Consulta la [Introducción para desarrolladores](https://core.telegram.org/bots) del equipo de Telegram.

Al crear tu bot de Telegram, crearás un archivo de texto con el código fuente de tu bot.
(También puedes copiar uno de nuestros archivos de ejemplo).
Define _lo que tu bot realmente hace_, es decir, "cuando un usuario envía este mensaje, responde con esto", etc.

Entonces puedes ejecutar ese archivo fuente.
Tu bot funcionará ahora, hasta que dejes de ejecutarlo.

Ya has terminado...

## Cómo mantener un bot en funcionamiento

...excepto, si te tomas en serio tu proyecto de bot.
Si detienes tu bot (o apagas tu ordenador), tu bot deja de responder, por lo que ya no reaccionará a ningún mensaje.

> Omite esta sección si sólo quieres jugar con los bots, y [continúa aquí abajo con los requisitos previos](#requisitos-previos-para-empezar) para empezar.

En pocas palabras, si quieres que el bot esté en línea todo el tiempo, tienes que mantener un ordenador funcionando las 24 horas del día.
Como lo más probable es que no quieras hacer eso con tu portátil, debes subir tu código a un _hosting provider_ (en otras palabras, el ordenador de otra persona, también conocido como _servidor_), y dejar que esa gente lo ejecute por ti.

Hay innumerables compañías que te permiten ejecutar tu bot de Telegram de forma gratuita.
Esta documentación cubre un número de diferentes proveedores de alojamiento que sabemos que funcionan bien con grammY (comprueba la sección [Alojamiento](../hosting/comparison.md)).
Al final, sin embargo, la elección de qué proveedor elegir depende de ti.
Recuerda que ejecutar tu código en otro lugar significa que quien sea el dueño de ese "lugar" tiene acceso a todos tus mensajes y a los datos de tus usuarios, así que deberías elegir un proveedor en el que puedas confiar.

Aquí hay un diagrama (simplificado) de cómo se verá la configuración al final cuando Alice se ponga en contacto con tu bot:

```asciiart:no-line-numbers
_________    envía un mensaje     ____________                    ____________
| Alice | —>   de Telegram     —> | Telegram | —> HTTP request —> | your bot |
—————————       a tu bot          ————————————                    ————————————

 un teléfono                 servidores de Telegram               tu portátil,
                                                                mejor: un servidor


|____________________________________________|                   |___________|
                    |                                                  |
        Responsabilidad de Telegram                             tu responsabilidad
```

Del mismo modo, tu bot puede hacer peticiones HTTP a los servidores de Telegram para enviar mensajes de vuelta a Alice.
(Si nunca has oído hablar de HTTP, puedes pensar en él como los paquetes de datos que se envían a través de Internet, por ahora).

## Lo que grammY hace por ti

Los bots interactúan con Telegram a través de peticiones HTTP.
Cada vez que tu bot envía o recibe mensajes, las peticiones HTTP van y vienen entre los servidores de Telegram y tu servidor/ordenador.

En su núcleo, grammY implementa toda esta comunicación por ti, así que puedes simplemente escribir `sendMessage` en tu código y un mensaje será enviado.
Además, hay una variedad de otras cosas útiles que grammY hace para simplificar la creación de tu bot.
Las conocerás a medida que vayas avanzando.

## Requisitos previos para empezar

> Sáltate el resto de esta página si ya sabes cómo desarrollar una aplicación Deno o Node.js, y [empieza](./getting-started.md).

Aquí hay algunas cosas interesantes sobre la programación---cosas que son esenciales para la codificación, pero que rara vez se explican porque la mayoría de los desarrolladores piensan que son evidentes.

En la siguiente sección, crearás un bot escribiendo un archivo de texto que contiene el código fuente en el lenguaje de programación [TypeScript](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html).
La documentación de grammY no te enseñará a programar, así que esperamos que te enseñes a ti mismo.
Sin embargo, recuerda: ¡crear un bot de Telegram con grammY es una buena manera de aprender a programar! :rocket:

::: tip Aprender a codificar
Puedes empezar a aprender TypeScript con el [tutorial oficial](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html) escrito por el equipo de TypeScript, y luego avanzar desde ahí.
No pases más de 30 minutos leyendo cosas en internet, luego vuelve aquí, (lee el resto de la sección) y [empieza](./getting-started.md).

Si ves una sintaxis desconocida en los documentos, o si recibes un mensaje de error que no entiendes, búscalo en Google---la explicación ya está en Internet (por ejemplo, en Stack Overflow).
:::

::: danger No aprender a codificar
Ahórrate algo de tiempo viendo [este vídeo de 34 segundos de duración](https://youtu.be/8RtGlWmXGhA).
:::

Al elegir grammY, ya te has decidido por un lenguaje de programación, concretamente TypeScript.
Pero, ¿qué pasa una vez que has creado tu código TypeScript, cómo va a empezar a ejecutarse?
Para ello, necesitas instalar algún software que sea capaz de _ejecutar_ tu código.
Este tipo de software se llama _entorno de ejecución_.
Toma tus archivos de código fuente y realmente hace lo que está programado en ellos.

Para nosotros, hay dos entornos de ejecución para elegir, [Deno](https://deno.com/runtime) y [Node.js](https://nodejs.org).
(Si ves que la gente lo llama _Node_, es que son demasiado perezosos para escribir ".js", pero significan lo mismo).

> El resto de esta sección te ayuda a decidir entre estas dos plataformas.
> Si ya sabes lo que quieres usar, salta a los [requisitos previos para Node.js](#requisitos-previos-para-node-js) o los [de Deno](#requisitos-previos-para-deno).

Node.js es la tecnología más antigua y madura.
Si necesitas conectarte a una base de datos extraña o hacer otras cosas de bajo nivel relacionadas con el sistema, es muy probable que puedas hacerlo con Node.js.
Deno es relativamente nuevo, por lo que a veces aún le falta soporte para algunas cosas avanzadas.
Hoy en día, la mayoría de los servidores utilizan Node.js.

Por otro lado, Deno es significativamente más fácil de aprender y usar.
Si aún no tienes mucha experiencia en programación, **tiene sentido empezar con Deno**.

Incluso si has escrito código para Node.js antes, deberías considerar darle una oportunidad a Deno.
Muchas cosas que son difíciles en Node.js son sencillas en Deno.

Deno

- es mucho más fácil de instalar,
- no requiere que configures nada sobre tu proyecto,
- utiliza mucho menos espacio en disco,
- tiene herramientas de desarrollo superiores incorporadas y una gran integración con el editor,
- es mucho más seguro, y
- tiene muchas más ventajas que no caben aquí.

Desarrollar código con Deno es también mucho más divertido.
Al menos, esa es nuestra opinión.

Sin embargo, si tienes una razón para usar Node.js, por ejemplo porque ya lo conoces bien, ¡entonces está completamente bien!
Nos estamos asegurando de que grammY funcione igual de bien en ambas plataformas, y no estamos cortando ninguna esquina.
Por favor, elige lo que creas que es mejor para ti.

### Requisitos previos para Deno

Antes de empezar a crear tu bot, vamos a dedicar unos minutos a una configuración adecuada para el desarrollo de software.
Esto significa instalar algunas herramientas.

#### Preparando tu máquina para el desarrollo

[Instala Deno](https://deno.com/manual/getting_started/installation#download-and-install) si aún no lo has hecho.

También necesita un editor de texto que se adapte bien a la codificación.
El mejor para Deno es Visual Studio Code, a menudo llamado simplemente VS Code.
[Instálelo.](https://code.visualstudio.com/)

A continuación, debe conectar VS Code y Deno.
Es muy sencillo: Existe una extensión para VS Code que lo hace todo automáticamente.
Puedes instalarla [como se describe aquí](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno).

Su sistema ya está listo para el desarrollo de bots. :tada:

#### Desarrollando un Bot

Crea un nuevo directorio en algún lugar.
Contendrá tu proyecto bot.
Abre este nuevo directorio en VS Code.

```sh
mkdir ./my-bot
cd ./my-bot
code .
```

> Si estás en macOS y el comando `code` no está disponible, simplemente abre VS Code, pulsa `Cmd+Shift+P`, escribe "shell command", y pulsa Enter.

En VS Code, convierte este directorio vacío en un proyecto Deno.
Pulsa `Ctrl+Shift+P`, escribe "deno init", y pulsa Enter.
En la parte inferior derecha de su editor debería aparecer la versión de Deno instalada en su sistema.

Su entorno de desarrollo Deno está listo.
Ahora puede empezar a escribir su bot.
Esto se explica en la página siguiente.

Una última cosa:
Después de haber creado su bot, por ejemplo en un archivo llamado `bot.ts`, puede ejecutarlo escribiendo `deno run --allow-net bot.ts` en su terminal.
(Sí, escribir software significa usar mucho el terminal, acostúmbrate).
Puedes parar el bot de nuevo con `Ctrl+C`.

¿Listo?
¡[Empieza](./getting-started.md#introduccion-a-deno)! :robot:

### Requisitos previos para Node.js

Vas a escribir tu bot en TypeScript, pero, al contrario que Deno, Node.js no puede ejecutar TypeScript.
En su lugar, una vez que tengas un archivo fuente (por ejemplo, llamado `bot.ts`), vas a _compilarlo_ a JavaScript.
Entonces tendrás dos archivos: tu `bot.ts` original, y un `bot.js` generado, que a su vez puede ser ejecutado por Node.js.
Los comandos exactos para todo esto serán introducidos en la siguiente sección cuando realmente crees un bot, pero es importante saber que estos pasos son necesarios.

Para poder ejecutar el archivo `bot.js`, tienes que tener instalado [Node.js](https://nodejs.org/en/).

En resumen, esto es lo que tienes que hacer para Node.js:

1. Crea un archivo fuente `bot.ts` con código TypeScript, por ejemplo usando [VS Code](https://code.visualstudio.com/) (o cualquier otro editor de código).
2. Compila el código ejecutando un comando en tu terminal. Esto genera un archivo llamado `bot.js`.
3. Ejecuta `bot.js` usando Node.js, de nuevo desde tu terminal.

Cada vez que modifiques tu código en `bot.ts`, debes reiniciar el proceso de Node.js.
Pulsa `Ctrl+C` en tu terminal para detener el proceso.
Esto detendrá tu bot.
Entonces, necesitas repetir los pasos 2 y 3.

::: tip Espera, ¿qué?

Instalar Node.js y configurar todo correctamente lleva mucho tiempo.
Si nunca has hecho esto antes, deberías esperar encontrarte con un montón de problemas confusos que son difíciles de solucionar.

Por eso esperamos que sepas cómo configurar tu sistema, o que seas capaz de aprender por ti mismo.
(Instalar Node.js _de la forma correcta_ es tan complicado que no cabe en esta página).

Si te sientes perdido en este punto, deberías dejar Node.js y usar [Deno](#requisitos-previos-para-deno) en su lugar.

:::

¿Aún confiado?
¡Genial!
¡[Empieza](./getting-started.md#comenzando-en-node-js)! :robot:
