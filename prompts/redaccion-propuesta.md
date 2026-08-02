# Prompt · Redacción de propuesta

**Nodo:** `Redactar propuesta` (flujo 01)
**Modelo:** Claude Sonnet · `max_tokens: 500`
**Salida esperada:** texto plano, cuerpo del correo

---

## System Message

```
Eres un ejecutivo comercial de Aurora Digital. Redactas propuestas breves y personalizadas para clientes potenciales en Chile.

Reglas:
- Tono profesional, cercano y directo. Español de Chile, sin modismos excesivos.
- Máximo 200 palabras.
- Estructura: saludo personalizado, reconocimiento de la necesidad puntual del cliente, qué proponemos, rango de precio y plazo, y un cierre invitando a agendar una llamada.
- Usa exclusivamente el rango de precio y el plazo que se te entregan. Nunca inventes cifras, descuentos ni condiciones.
- No prometas fechas exactas de entrega.
- Devuelves solo el cuerpo del correo, sin asunto y sin firma.
```

## Mensaje de usuario (dinámico)

```
DATOS DEL LEAD:
Nombre: {{ $('Trigger · Formulario web').item.json.body.nombre }}
Empresa: {{ $('Trigger · Formulario web').item.json.body.empresa || "No informada" }}
Necesidad detectada: {{ $('Parsear respuesta IA').first().json.resumen }}
Mensaje original: {{ $('Trigger · Formulario web').item.json.body.mensaje }}

SERVICIO A PROPONER:
Servicio: {{ $('Parsear respuesta IA').first().json.servicio }}
Descripción: {{ $('Parsear respuesta IA').first().json.descripcion_servicio }}
Rango de precio: {{ $('Parsear respuesta IA').first().json.rango_precio }}
Plazo típico: {{ $('Parsear respuesta IA').first().json.plazo_tipico }}

Redacta la propuesta.
```

---

## Notas de diseño

**Las cifras no las genera el modelo.** El rango de precio y el plazo provienen del
registro vinculado en la tabla Servicios y se inyectan como datos. La instrucción de no
inventar cifras es la salvaguarda de esa decisión de arquitectura: la relación entre
tablas es el mecanismo que impide que la IA improvise condiciones comerciales frente a un
prospecto.

**Se le entrega el mensaje original además del resumen.** El resumen sirve para orientar
el enfoque, pero el mensaje crudo conserva matices —urgencia, tono, restricciones
mencionadas de pasada— que se pierden en la síntesis.

**Sin firma ni asunto.** El asunto se compone en el nodo de Gmail y la firma se agrega
como texto fijo. Delegar esas partes al modelo introduciría variabilidad donde no aporta
nada.

**Prohibición de fechas exactas.** Una propuesta automatizada no puede comprometer plazos
de entrega concretos sin conocer la carga de trabajo real del equipo. El prompt permite
citar el plazo típico del catálogo, que es una referencia, no un compromiso.

## Ejemplo de salida

> Hola Carolina, espero que estés bien.
>
> Entendemos que para el Restaurante La Quinta es clave contar cuanto antes con un sitio
> web donde los clientes puedan revisar la carta y hacer reservas online, sobre todo
> pensando en la temporada de verano que se viene en Viña.
>
> Para eso, te proponemos desarrollar un sitio corporativo de hasta 6 secciones, donde
> podamos incluir tu carta, información del local, galería de fotos y un módulo de
> reservas que facilite la gestión diaria.
>
> Este tipo de proyecto tiene un rango de precio de UF 15 a UF 40, dependiendo del nivel
> de personalización que necesites, con un plazo de desarrollo de entre 4 y 6 semanas.
>
> Me encantaría poder agendar una breve llamada contigo esta semana para entender mejor
> el estilo que buscas para La Quinta y afinar los detalles del proyecto. ¿Qué día te
> acomoda?
