---
title: 'Desarrollando App Pomodoro con JavaScript'
description: 'Explora el proceso de desarrollo de Focushroom, una aplicación Pomodoro construida con JavaScript. Descubre cómo implementar ciclos automáticos, sonidos relajantes y almacenamiento local para mejorar la productividad.'
pubDate: 'may 05 2025'
heroImage: '/blog5-hero.jpg'
keywords: 'App Pomodoro con JavaScript, Desarrollo de aplicaciones de productividad, Implementación de cronómetro en JavaScript, Uso de localStorage en aplicaciones web, Integración de sonidos en aplicaciones web, Diseño de interfaces para aplicaciones de productividad'
---

Luego de días de estudio _intenso_ para un parcial de la facultad, reaparezco con **excelentes** actualizaciones sobre Focushroom.

Aunque me volví **un poquito loca** con el cronómetro, los ciclos, etc; me las pude ingeniar para resolverlo, además de _otras cuantas cosas_ que agregué:

- Antes tenía el problema que no comenzaba solo y luego tuve el problema de que no se detiene solo, _parecía un chiste_. Sin embargo, quiero compartir mi código por si te sirve la manera que lo solucioné o tenés recomendaciones para hacerlo mejor (comparto autoría con algunas personas en StackOverflow y mi íntimo amigo ChatGPT).

```js
/* routeParams vienen de los parámetros dinámicos de la url donde recopilo información sobre el pomodoro (lo explico en el blog anterior) */
const focusTime = Number(routeParams.focusTime);
const breakTime = Number(routeParams.breakTime);
const cycles = Number(routeParams.cycles);
const [minutes, setMinutes] = useState(focusTime);
const [seconds, setSeconds] = useState(0);
const [currentCycle, setCurrentCycle] = useState(1);
const [currentState, setCurrentState] = useState(true); /* true: focus / false: break */
const [isRunning, setIsRunning] = useState(true);

useEffect(() => {
	if (!isRunning) return;

	const interval = setInterval(() => {
		if (seconds > 0) {
			setSeconds((s) => s - 1);
		} else {
			if (minutes > 0) {
				setMinutes((m) => m - 1);
				setSeconds(59);
			} else {
				if (currentState && currentCycle === cycles) {
					setIsRunning(!isRunning);
					clearInterval(interval);
					/* finalizó el pomodoro */
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

	return () => clearInterval(interval);
}, [isRunning, seconds, minutes, focusTime, breakTime, cycles, currentCycle, currentState]);
```

- Solucioné **otro problema** que tenía con los ciclos, anteriormente yo sumaba un ciclo dentro del `setCurrentState` y tenía el problema que en el segundo ciclo la variable `currentCycle` tenía un valor de 3 en vez de 2. No sé por qué (sinceramente) pero incluso los `console.log` se duplicaban dentro. Entonces cuando me di cuenta que el problema era ese, enseguida sumé al ciclo por fuera y funciona a la perfección.

*(De hecho, ahora mismo estoy utilizando **mi** pomodoro para escribir esta entrada de blog).*

- Agregué una barra de progreso, identificador de tiempo de enfoque o descanso, un botón de pausa, un poco de estilos y claramente un cronómetro **grande** para que puedas ver incluso con el celular lejos.

- **Implementé sonidos** para el cambio de tiempo de enfoque a descanso o viceversa; además de un **soundtrack relajante** únicamente durante el tiempo de descanso.

- Utilicé `localStorage` para el contador de _mushpoints_.

Y por ahora no tengo más, pero sé que mis **próximos pasos** a seguir son:

1. Posiblidad de desactivar el sonido.
2. Pantalla de finalización con la obtención de los mushpoints y posibilidad de reiniciar.
3. Historial de sesiones pomodoro.
4. Sistema de compras de la casa con los mushpoints.

Tengo muchas más cosas planeadas, **realmente no puedo esperar a lanzar el MVP de Focushroom**, por ahora te puedo dar un pequeño adelanto:

![Screenshot de Focushroom en versión beta](/screenshot-focushroom(1).png)

Cuando consiga mi objetivo lo vas a ver claramente reflejado en el título de la entrada de blog, por el momento me mantengo con mis **analogías**.

---

Gracias por leerme de nuevo hoy, valoro mucho que hayas llegado hasta el final. ¡No es casualidad que estés acá! Deja un comentario debajo o podemos tener una buena conversación vía [mail](mailto:melonialbana@gmail.com) :)

_Albana, frontend developer._
