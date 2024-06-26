# Práctica 2 : INTERRUPCIONES
## Introducción de la práctica

En esta segunda práctica se trabajan las interrupciones, dispondremos de 2 leds para llevarla a cabo, además del microcontrolador ESP32.

La práctica incluye de 2 partes:

**- Práctica A:**
En esta parte se estudian las interrupciones por GPIO.

**- Práctica B:**
En esta otra parte las interrupciones se estudian por timer.

# 1.-Práctica A

```c++
struct Button {
const uint8_t PIN;
uint32_t numberKeyPresses;
bool pressed;
};
Button button1 = {18, 0, false};
void IRAM_ATTR isr() {
button1.numberKeyPresses += 1;
button1.pressed = true;
}
void setup() {
Serial.begin(115200);
pinMode(button1.PIN, INPUT_PULLUP);
attachInterrupt(button1.PIN, isr, FALLING);
}
void loop() {
if (button1.pressed) {
Serial.printf("Button 1 has been pressed %u times\n",
button1.numberKeyPresses);
button1.pressed = false;
}

//Detach Interrupt after 1 Minute
static uint32_t lastMillis = 0;
if (millis() - lastMillis > 60000) {
lastMillis = millis();
detachInterrupt(button1.PIN);
Serial.println("Interrupt Detached!");
}
}
```
### Funcionamiento y salidas:

Respecto al funcionamiento del código ,de esta parte de la práctica consiste en la interrupción por GPIO, cómo se puede ver, las interrupciones en el código estan inicializadas en el Pin 18, para que se generen las interrupciones, el programa mostrará las interrupciones que han sucedido.
Cuando pasa 1 minuto o lo que es igual a 60000 milisegundos se desvinculan las interrupciones en el Pin asignado.

El funcionamiento práctico consiste en poner el cable en el puerto G18 (PIN) de la ESP32 y la otra parte del cable junto al Pin GND, de esta manera se generarán las interrupciones,mostrándose el parpadeo del led del procesador.

Salidas que se obtienen en impresión en serie: 
````
Button 1 has been pressed 1 times
Button 1 has been pressed 2 times
Button 1 has been pressed 3 times
Button 1 has been pressed 4 times
````
### Especificaciones del código
El código proporciona una estructura de datos donde incluyen las siguientes variables:
- El PIN al que está conectado (*const uint8_t PIN;*)
- Contador de veces (*uint32_t numberKeyPresses;*)
- Indicador si el botón está presionado (*bool pressed;*)

Además también se disponen de 3 funciones:
- *Función de interrupción (ISR):* Tiene la funcionalidad de incrementar el contador de presiones del botón y establece el indicador pressed como verdadero.
- *Función setup():* se configura el pin del botón como entrada con resistencia de pull-up interna y se adjunta la ISR al pin del botón.
- *Función loop():* comprueba si el botón ha sido presionado.

# 2.-Práctica B

```c++
volatile int interruptCounter;
int totalInterruptCounter;
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;
void IRAM_ATTR onTimer() {
portENTER_CRITICAL_ISR(&timerMux);
interruptCounter++;
portEXIT_CRITICAL_ISR(&timerMux);
}
void setup() {
Serial.begin(115200);
timer = timerBegin(0, 80, true);
timerAttachInterrupt(timer, &onTimer, true);
timerAlarmWrite(timer, 1000000, true);
timerAlarmEnable(timer);
}
void loop() {
if (interruptCounter > 0) {
portENTER_CRITICAL(&timerMux);
interruptCounter--;
portEXIT_CRITICAL(&timerMux);
totalInterruptCounter++;
Serial.print("An interrupt as occurred. Total number: ");
Serial.println(totalInterruptCounter);
}
}
```
### Funcionamiento y salidas:

La segunda parte, se basa en la interrupción por timer o por temporizador, en relación al código, el programa muestra las interrupciones generadas por el timer, que en este caso está a 1000000 microsegundos que es igual a cada 1 segundo, cada interrupción.

La salida por el puerto serie es: 
```
An interrupt as occurred. Total number: 1
An interrupt as occurred. Total number: 2
An interrupt as occurred. Total number: 3
```
### Especificaciones del código
Esta parte del codigo tambien incluye las siguientes funcionalidades:
- *Función de Interrupción del Temporizador (ISR)* : Se encarga de incrementar las interrupciones dentro de una sección crítica.
- *Función setup():* se configura el temporizador para generar una interrupción cada 1 segundo y, finalmente, se habilitan las interrupciones del temporizador.
- *Función loop():* se comprueba si se ha producido alguna interrupción del temporizador. Si se ha producido, se decrementa interruptCounter, se incrementa totalInterruptCounter.
