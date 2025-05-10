---
title: 'Desarrollando App Pomodoro con JavaScript'
description: 'Explora el proceso de desarrollo de Focushroom, una aplicación Pomodoro construida con JavaScript. Descubre cómo implementar ciclos automáticos, sonidos relajantes y almacenamiento local para mejorar la productividad.'
pubDate: 'may 05 2025'
heroImage: '/blog5-hero.jpg'
keywords: 'App Pomodoro con JavaScript, Desarrollo de aplicaciones de productividad, Implementación de temporizador en JavaScript, Implementación de cronómetro en JavaScript, Uso de localStorage en aplicaciones web, Integración de sonidos en aplicaciones web, Diseño de interfaces para aplicaciones de productividad, App Pomodoro con React, Implementación de cronómetro con React, Implementación de temporizador con React'
---

_Explorá el proceso de desarrollo de Focushroom, una aplicación Pomodoro construida con la librería de JavaScript: ReactJS. Descubrí cómo implementar ciclos automáticos, sonidos y almacenamiento local para mejorar la productividad._

### Índice

-   [Implementación del temporizador en Javascript](#implementación-del-temporizador-en-javascript)
-   [Manejo de ciclos automáticos](#manejo-de-ciclos-automáticos)
-   [Integración de sonidos](#integración-de-sonidos)
-   [Identificador de progreso](#identificador-de-progreso)
-   [Mi aplicación pomodoro](#mi-aplicación-pomodoro)

---

## Implementación del temporizador en Javascript

Luego de días de estudio _intenso_ para un parcial de la facultad, reaparezco con **excelentes** actualizaciones sobre Focushroom. Aunque me volví **un poquito loca** con el temporizador, los ciclos, etc; me las pude ingeniar para resolverlo, además de _otras cuantas cosas_ que agregué.

Para poder implementar un pomodoro en tu aplicación con la librería de React vas a tener que amigarte con `useState` y `useEffect`. Primero vamos a crear estados para los datos del temporizador que queremos mostrar, en mi caso solo minutos y segundos, esto va dentro de la función del componente, pero antes del `return`:

```js
const [minutes, setMinutes] = useState(25);
const [seconds, setSeconds] = useState(0);
```

(En mi aplicación los minutos no están _hardcodeados_, sino que los elije el usuario en una pantalla previa al temporizador y recupero esa información con el uso de [rutas dinámicas](https://albanablog.netlify.app/blog/blog-4)).

También es importante ofrecerle al usuario la **posibilidad de pausar el temporizador**, para eso vamos a crear otro estado para saber si está corriendo o no:

```js
const [isRunning, setIsRunning] = useState(true);
```

Es muy específico de cada proyecto donde quieras implementar un temporizador, pero en mi caso yo inicio el estado en `true` porque quiero que la cuenta regresiva **inicie al instante**. Pero igualmente, necesito el estado para tener la funcionalidad de pausar y reanudar el pomodoro.

Es momento de hacer uso del hook `useEffect`, esta sería la estructura base del mismo:

```js
useEffect(() => {
	if (!isRunning) return;

	const interval = setInterval(() => { ... }, 1000);

	return () => clearInterval(interval);
}, [isRunning]);
```

Vamos a depender del cambio de la variable `isRunning` para saber si **renderizamos el código o no**. Entonces si la misma es falsa, salimos del `useEffect`. Si nuestro pomodoro tiene la bandera de que está corriendo en verdadera, entonces vamos a querer crear un intervalo que ejecute un código cada _1000 milisegundos = 1 segundo_. La función `setInterval` nos devuelve una especie de **ID** de nuestro intervalo que vamos a almacenar en una variable.

Antes de correr el hook otra vez, al final voy a tener una función de limpieza (_cleanup function_): `clearInterval`, para luego poder limpiar el intervalo, necesario para evitar que queden múltiples intervalos activos al mismo tiempo. **Por eso necesitamos del ID**, para saber cuál debemos limpiar.

> Dato importante: siempre debes tener una cleanup function para los intervalos.

Dentro del intervalo lo que queremos hacer segundo a segundo, (como comenzamos de atrás hacia adelante porque es un temporizador), es evaluar primero cuando **los segundos NO son cero**.

Tenemos que ir restando los segundos, así que haciendo uso de las variables que **desestructuramos** del `useState` vamos a settear los segundos con una función _lambda_ muy sencilla, tomando el último valor del set le restamos 1:

```js
const interval = setInterval(() => {
	if (seconds > 0) {
		setSeconds((s) => s - 1);
	}
}, 1000);
```

Pero si los segundos están en 0 quiere decir que ahora tenemos que restarle a los minutos (**siempre y cuando no sea 0 tampoco**), y los segundos tienen que estar ahora en 59:

```js
const interval = setInterval(() => {
	if (seconds > 0) {
		setSeconds((s) => s - 1);
	} else {
		if (minutes > 0) {
			setMinutes((m) => m - 1);
			setSeconds(59);
		}
	}
}, 1000);
```

Simplemente, ahora sabemos que cuando los segundos sean 0 y a su vez los minutos, entonces **el temporizador llegó al final**:

```js
const interval = setInterval(() => {
	if (seconds > 0) {
		setSeconds((s) => s - 1);
	} else {
		if (minutes > 0) {
			setMinutes((m) => m - 1);
			setSeconds(59);
		} else {
			setIsRunning(!isRunning);
			clearInterval(interval);
		}
	}
}, 1000);
```

## Manejo de ciclos automáticos

Ahora que tenés un **temporizador funcionando**, para que sea realmente un pomodoro, tenemos que iterar entre el tiempo de **enfoque** y el tiempo de **descanso** -que no necesariamente tienen la misma cantidad de minutos-; además de la cantidad de veces que se van a repetir estos ciclos: _1 ciclo = tiempo enfoque + tiempo descanso_. Nuevamente, haciendo uso de las **rutas dinámicas**, recupero los minutos de descanso y la cantidad de ciclos/vueltas.

**Mi idea** para el temporizador de pomodoro es que inicie el tiempo de descanso _automáticamente_ una vez que se termina el tiempo de enfoque. Así que primero vamos a crear algunas constantes más, así es como se vería con el uso de **rutas dinámicas** y sus parámetros, pero simplemente se pueden _hardcodear_ los minutos y ciclos:

```js
const focusTime = Number(routeParams.focusTime);
const breakTime = Number(routeParams.breakTime);
const cycles = Number(routeParams.cycles);

const [minutes, setMinutes] = useState(focusTime);
const [seconds, setSeconds] = useState(0);

const [isRunning, setIsRunning] = useState(true);

const [currentCycle, setCurrentCycle] = useState(1); /* inicia en el ciclo 1 */
const [currentState, setCurrentState] = useState(true); /* true: focus -- false: break */
```

Ahora la lógica cuando los segundos y minutos son 0 al mismo tiempo **cambia un poco**, ya que primero tenemos que evaluar si nos encontramos en estado de focus (`true`) entonces debemos cambiarlo a estado de break (`false`). Pero igualmente seguimos con la estructura que estábamos trabajando:

```js
const interval = setInterval(() => {
	if (seconds > 0) {
		setSeconds((s) => s - 1);
	} else {
		if (minutes > 0) {
			setMinutes((m) => m - 1);
			setSeconds(59);
		} else {
			/* si estamos en break recién ahi sumamos el ciclo */
			/* ya que -> 1 ciclo = focus + break */
			!currentState && setCurrentCycle((c) => c + 1);

			/* setteamos el estado (el contrario al actual) */
			/* e iniciamos los minutos y segundos */
			setCurrentState((prev) => {
				setMinutes(prev ? breakTime : focusTime);
				setSeconds(0);
				return !prev;
			});

			/* setIsRunning(!isRunning);
			clearInterval(interval); */
		}
	}
}, 1000);
```

Si los minutos y segundos llegan a cero y llegamos al total de ciclos, vamos a querer ahora sí, **detener la ejecución del temporizador**. Personalmente, la preferencia que tengo con la técnica pomodoro, es **evitar el último break**, lo considero **innecesario** ya que igualmente voy a dejar de enfocarme en la tarea de todas formas. Así que, en este caso, hice que _obvie el último tiempo de break_:

```js
const interval = setInterval(() => {
	if (seconds > 0) {
		setSeconds((s) => s - 1);
	} else {
		if (minutes > 0) {
			setMinutes((m) => m - 1);
			setSeconds(59);
		} else {
			/* si estamos en focus y la cantidad de ciclos */
			/* actuales es la misma que los ciclos totales */
			if (currentState && currentCycle === cycles) {
				setIsRunning(!isRunning);
				clearInterval(interval);
				return;
			}

			!currentState && setCurrentCycle((c) => c + 1);
			setCurrentState((prev) => {
				setMinutes(prev ? breakTime : focusTime);
				setSeconds(0);
				return !prev;
			});
		}
	}
}, 1000);
```

**¡Y listo! Ya sabes cómo hacer un temporizador/cronómetro pomodoro _y encima con React_.**

## Integración de sonidos

Si queres agregarle un toque **personal, divertido y diferenciador**, qué mejor que integrar sonidos a tu pomodoro. Siempre te recomiendo **pensar como usuario**, ¿te gusta que haya **música** en tiempo de enfoque?, ¿te molesta no darte cuenta **cuándo terminó** el tiempo de enfoque o descanso?, etc. Por eso, por mis preferencias y necesidades, simplemente utilicé dos audios: para avisar que **cambió** el temporizador, y **música relajante** solamente durante el tiempo de break.

Tuve algunas complicaciones para implementarlo con React y el uso de `useEffect`. Me presentó **dificultades** a la hora de reproducir el audio y que no quedé infinitamente en loop. Junto con **ChatGPT** y algunos **foros** con recomendaciones de devs, llegué a esta solución para que vos también puedas utilizar **sonidos en tu app pomodoro** con React.

El primer paso importante es llamar a los audios con el uso de la función `useRef` y luego creando una instancia de `Audio`:

```js
const audioChangeRef = useRef(new Audio('/change-state.mp3'));
```

La función `useRef` devuelve un **objeto de referencia mutable** cuya propiedad `.current` se inicializa con el argumento pasado entre paréntesis. De esta forma nos aseguramos que el objeto devuelto vaya a **persistir** durante toda la vida útil del componente.

Entonces dentro de otro `useEffect` vamos a manejar este objeto:

```js
useEffect(() => {
	const audio = audioChangeRef.current;
	if (!isRunning) return;
	if (minutes === 0 && seconds === 0) {
		audio.currentTime = 0;
		audio.volume = 0.5;
		audio.play();
	}
}, [isRunning, minutes, seconds]);
```

Básicamente queremos reproducir el audio **siempre y cuando** el pomodoro esté corriendo, y **específicamente** cuando los minutos y los segundos sean 0 (**no importa el estado** si es focus o break).

Viéndolo así resuelta quizás **fácil** o incluso **intuitivo**, pero sinceramente **me costó unas horas** llegar a esta solución. Pero ahora que lo sé simplemente me **agiliza muchísimo** cualquier otro proyecto que tenga que encarar la implementación de **audios y sonidos**.

## Identificador de progreso

Algo sumamente importante para la **UX del pomodoro**, es saber cuánto falta de manera **visual**, poder hacer una comparación -más allá de los minutos transcurridos- para conocer el **progreso** de la sesión.

Por eso te quiero compartir este **código de CSS** que siempre utilizo para mis **barras de progreso**.

Así me gusta estructurar mi HTML:

```html
<span class="progress-bar">
	<span class="progress-bar__fill" data-progress=""></span>
</span>
```

Y para los estilos del "fondo" de la barra de progreso, me gusta hacerlos así (algunos pueden variar):

```css
.progress-bar {
	display: block;
	background-color: var(--secondary);
	width: 100%;
	height: var(--6);
	position: relative;
	margin-block: var(--4);
}
```

Lo interesante es el uso del atributo `data-progress` de manera dinámica en nuestro CSS:

> Advertencia: posiblemente te muestre un error de sintaxis en tu IDE, pero funciona correctamente.

```css
.progress-bar__fill {
	background-color: var(--primary);
	position: absolute;
	top: 0;
	left: 0;
	width: attr(data-progress %);
	height: inherit;
}
```

**¿Conocías la función de CSS `attr()`?** Te dejo la página de [mdn web docs](https://developer.mozilla.org/en-US/docs/Web/CSS/attr) para que puedas saber más al respecto.

De esta forma, el valor del atributo `data-progress` ahora se va a transformar en un **porcentaje** y se verá reflejado en el ancho del `span`. **_Simplemente increíble_**.

Esta implementación se **potencia** muchísimo con el uso de React, ya que en mi caso, **creé un componente** con la estructura HTML que viste más arriba y sus repectivos estilos; le pasé como `prop` el valor del progreso, y dentro de mi componente pomodoro (donde muestro el temporizador) agrego la barra con el **progreso actual calculándolo** con la siguiente regla de tres simples, (haciendo uso de `useState` para que se actualicen los valores):

```js
String(((minutes * 60 + seconds) * 100) / (focusTime * 60));
```

Así de sencillo ;)

## Mi aplicación pomodoro

**¡Ya está terminado el MVP de Focushroom!**

-   Disponible en versión móvil (Android):
    <a href="/focushroom.apk" download="focushroom_v1.apk">Descargar APK</a>

-   Disponible en versión local (web):
    [Instrucciones de descarga](https://github.com/albana-meloni/focushroom?tab=readme-ov-file#%EF%B8%8F-instalaci%C3%B3n-local)

Me siento **orgullosa** de decir que ahora mismo estoy utilizando **mi propio pomodoro** para escribir esta entrada de blog. Simplemente es un **logro grandísimo** haber comenzado este proyecto y tener el primer **resultado** en tan poco tiempo y con una satisfacción tan grande. **¡Próximamente se acercan actualizaciones!**

![Screenshot de Focushroom en versión 1.0](/focushroom-screenshot.png)

---

Gracias por leerme de nuevo hoy, valoro mucho que hayas llegado hasta el final. ¡No es casualidad que estés acá! Deja un comentario debajo o podemos tener una buena conversación vía [mail](mailto:melonialbana@gmail.com) :)

_Albana, frontend developer._
