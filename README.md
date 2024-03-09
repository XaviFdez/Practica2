# Práctica 2 : INTERRUPCIONES

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
Respecto al funcionamiento del código ,de esta parte de la práctica consiste en la interrupción por GPIO, cómo se puede ver, las interrupciones en el código estan inicializadas en el Pin 18, para que se generen las interrupciones, el programa mostrará las interrupciones que han sucedido.
Cuando pasa 1 minuto o lo que es igual a 60000 milisegundos se desvinculan las interrupciones en el Pin asignado.

El funcionamiento práctico consiste en poner el cable en el puerto G18 (PIN) de la ESP32 y la otra parte del cable junto al Pin GND, de esta manera se generarán las interrupciones,mostrándose el parpadeo del led del procesador.

Salidas que se obtienen en impresión en serie: 
Se obtienen 2 salidas:
- La de pulsaciones del botón.
- La de desvinculación a los 60000 segundos.

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
La segunda parte, se basa en la interrupción por timer o por temporizador, en relación al código, el programa muestra las interrupciones generadas por el timer, que en este caso está a 1000000 microsegundos que es igual a cada 1 segundo, cada interrupción.

El total de interrupciones las mostrará por el puerto serie.
