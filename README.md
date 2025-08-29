# poemSampler

## Acerca del proyecto

Grupo: 01: maincraf

Integrantes:
  - [Camila Delgado](https://github.com/notcaamila)
  - [Braulio Figueroa](https://github.com/brauliofigueroa2001)
  - [Santiago Gaete](https://github.com/santiagoClifford)
  - [Camila Parada](https://github.com/Camila-Parada)
 
***
## Presentación textual

* BORRAR AL ÚLTIMO LA INDICACIÓN * Plantea aquí el problema de diseño que abordaste. Menciona el texto de referencia.

poemSampler comienza como una interfaz que permite funsionar estractos de distintos poemas, para crear tus propios "remixes". Debido a limitaciones técnicas, el proyecto fue iterando a medida que avanzábamos en el código:

- Versión 1: poema scrollea automáticamente. Algunas palabras se ven destacadas. Al escribir una de esas palabras, aprece una imagen en un segunda pantalla.

- Versión 2: el scroll se controla con el potenciómetro. Al presionar un botón, en una segunda pantalla se ve una imagen asociada al refrán.

- Versión 3: con el potenciómetro controlas el scroll de los refranes. Al presionarl botón, aparece una imagen respecto al refrán y el texto se va. Al usar el potenciómetro, vuelve a verse el texto y la imagen desaparece. La imagenes son combinables entre sí.

- Versión 4: el potenciómetro permite scrollear entre los elementos de la lista. Con el botón vas turnando entre la lista de texto(void modoTexto()) y la lista de imagenes (void modoHd).

El usuario puede navegar entre los refranes seleccionados por el equipo usando el potenciómetro, ya sea de manera textual, o de manera visual. Para ir cambiando entre ambas maneras, se usa el botón.

***
## Inputs y outputs

* BORRAR AL ÚLTIMO LA INDICACIÓN * ¿Cuál es la interacción? ¿Qué ofrece la máquina de vuelta?

***
## Bocetos de planificación

Este proceso ha sido complejo, puesto que a pesar de la unión de ideas múltiples no todos los aspectos resultaron viables principalmente por aspectos técnicos: conectar 2 pantallas a la vez (requiere un bus I2C o cambiar de posición la resistencia del adress), destacar con amarillo secciones del texto (imposible puesto que las pantallas ya vienen con un color establecido que no se puede configurar), entre otros ajustes. Ante las adversidades seguimos construyendo ideas para ponerlas a prueba.

![croquis ideas individuales](./imagenes/boceto-v1.jpg)

▼ Primer boceto para organizar las ideas. Se observa una idea de navegación de un compositor(mezclador) de texto. Obtenido de: creación personal.

![croquis poemSampler-v1](./imagenes/boceto-v2.jpg)

▼ Segundo boceto. Se mezclan las ideas propuestas sobre el proyecto. Obtenido de: creación personal.

![Diagrama estructura](./imagenes/croquis-focus.jpeg) 

▼ Tercer boceto. Esquema conceptual sobre el funcionamiento y la interface. Obtenido de: creación personal.

![pseudo flujo](./imagenes/pseudoFlujo.png)

▼ Diagrama de flujo del funcionamiento. Obtenido de: creación personal.

***
## Etapas del código

1- Inclusión de librerías y creación de variables con los parámetros visuales

```cpp
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Wire.h>
#define screenW 128
#define screenH 64
#define oledReset -1
#define screenAdress 0x3C
Adafruit_SSD1306 display(screenW, screenH, &Wire, oledReset);
```

2- Variables específicas del código: Botón y Potenciometro

```cpp
//variable que lleva la cuenta de cuántas veces se ha presionado el botón
int botonComputo = 0;
//pin del botón
int botonPin = 2;
//pin del potenciómtro
int potePin = A0;
```

3- Listado del contenido textual: Refranes a la chilena

```cpp
char* refranes[] = { "perro ke ladra no muerde",
                     "se le escapan lo' enanito' pal' bosqe",
                     "kear como xaleca 'e mono",
                     "pasar gato x liebre",
                     "cria cuervo' y te sacaran lo' ojo'"};
```

4- Matrices de bits (Bitmaps): códigos hexadecimales de las imágenes a usar
  
(contenido internoborrado para hacer este documento más legibble)

```cpp
const unsigned char bitmapPerro[] PROGMEM = {
};
const unsigned char bitmapEnano[] PROGMEM = {
};
const unsigned char bitmapMono[] PROGMEM = {
};
const unsigned char bitmapGato[] PROGMEM = {
};
const unsigned char bitmapCuervo[] PROGMEM = {
};
```
5- Variable contenedora de los Bitmaps

```cpp
const int bitmapArrayLength = 5;
const unsigned char* bitmapArray[5] = {
  bitmapPerro, bitmapEnano, bitmapMono, bitmapGato, bitmapCuervo
};
```

6- "void setup": iniciando el programa

```cpp
void setup() {
  //<https://docs-arduino-cc.translate.goog/language-reference/en/functions/communication/serial/begin/>
  Serial.begin(9600);
  //extraído de la librería de adafruit. En caso de que falle la conexión, imprime un mensaje.
  if (!display.begin(SSD1306_SWITCHCAPVCC, screenAdress)) {
    Serial.println(F("SSD1306 allocation failed"));
    for (;;)
      ;
  }
  // configurar el pin para recibir datos.
  pinMode(botonPin, INPUT_PULLUP);

  //setear el display
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.display();
}
```

7- "void loop": procesos repetitivos

La variable votoComputo lleva la cuenta de cuántas veces fue presionado el botón. En los números pares está en modoPoeta(), en los números impares en modoHd().

```cpp
void loop() {
  //contar las veces que se ha presionado el botón <https://forum.arduino.cc/t/counting-button-presses/119881/4>
  if (digitalRead(botonPin) == LOW) {
    botonComputo++;
    delay(200);
  }
  //<https://docs.arduino.cc/language-reference/en/structure/arithmetic-operators/remainder>
  //<https://github.com/clifford1one/voluntadGuiada/blob/main/code/interact.js>
  if (botonComputo % 2 == 0) {
    //números pares
    modoPoeta();
  } else {
    //números impares
    modoHd();
  }
}
```
8- "void modoPoeta": función que contiene los refranes textuales

```cpp
//el modo donde se leen los refranes
void modoPoeta() {
  display.stopscroll();
  display.clearDisplay();
  int poteValue = analogRead(potePin);
  //<https://www.w3schools.com/cpp/cpp_arrays_loop.asp>
  //<https://www.youtube.com/watch?v=RXWO3mFuW-I&t=303s>
  //poteValue lo divido en 205, porque quiero que el pote se deivida en 5 "secciones".
  //1023/5 = 204,6
  int fraccionPote = poteValue / 205;
  //<https://docs.arduino.cc/language-reference/en/functions/math/constrain>
  fraccionPote = constrain(fraccionPote, 0, 4);
  display.setCursor(0, 20);
  display.println(refranes[fraccionPote]);
  display.display();
}
```

9- "void modoHd": función que contiene las imágenes basadas en los refranes

```cpp

void modoHd() {
  display.stopscroll();
  display.clearDisplay();
  int poteValue = analogRead(potePin);
  int fraccionPote = poteValue / 205;
  fraccionPote = constrain(fraccionPote, 0, 4);
  display.drawBitmap(0, 0, bitmapArray[fraccionPote], screenW, screenH, SSD1306_WHITE);
  display.display();
}
```

***
## Roles del equipo

1- El Notch (Big boss): Santiago "Clifford" Gaete
Desarrollador principal del proyecto. Unificador de ideas. Lo que imagina se lleva a cabo. Sueña en lenguajes de programación.

2- Allays (investigación): Camila "Mille" Parada
Investigadora de códigos. Analista de viabilidad y riesgos de circuito. Coleccionista de micro controladores. 

3- El Notch (Big boss): Santiago "Clifford".

***
## Fotografías y videos del proyecto funcionado

Subir fotos y videos

El video debe estar subido a youtube y mencionado en un enlace para ahorrar espacio en el repositorio

***
## Bibliografía

Citas en APA de repositorios y enlaces de los cuales se inspiraron. Bibliotecas, tutoriales, etc.

1- Consulta de información

- Arduino cc (2012, September 12). counting button presses. Arduino Forum. https://forum.arduino.cc/t/counting-button-presses/119881/4‌
- Arduino docs (2024). #define .Arduino.cc. https://docs.arduino.cc/language-reference/en/structure/further-syntax/define/
- Arduino docs (2024). string, Arduino.cc. https://docs.arduino.cc/language-reference/en/variables/data-types/string/
- Arduino docs (2024). Serial.begin(). Arduino.cc. https://docs.arduino.cc/language-reference/en/functions/communication/Serial/begin/
- Arduino docs (2025). % (remainder). Arduino.cc. https://docs.arduino.cc/language-reference/en/structure/arithmetic-operators/remainder
- Arduino docs (2025). constrain(). Arduino.cc. https://docs.arduino.cc/language-reference/en/functions/math/constrain
- W3schools (n.d.). C++ Loop Through an Array. Www.w3schools.com. https://www.w3schools.com/cpp/cpp_arrays_loop.asp
- 
‌
2- Videos

- The Coding Train (2025). 7.2: Arrays and Loops - p5.js Tutorial. Youtu.be. https://youtu.be/RXWO3mFuW-I?si=1Iiri9lDNYRLTjQS
- 

3- Inspiración

- Wokwi ESP32, STM32, Arduino Simulator. (2019). arduino_oled_animation__upir. Wokwi.com. https://wokwi.com/projects/374294166215201793
- 

4- Recursos

- Adafruit. (2020, January 11). adafruit/Adafruit_SSD1306. GitHub. https://github.com/adafruit/Adafruit_SSD1306
- Adafruit. (2021, October 14). Adafruit GFX Library. GitHub. https://github.com/adafruit/Adafruit-GFX-Library
- Arduino docs (2024). Wire. Arduino.cc. https://docs.arduino.cc/language-reference/en/functions/communication/wire/
- Clifford1one. (2025). voluntadGuiada/code/interact.js. GitHub. https://github.com/clifford1one/voluntadGuiada/blob/main/code/interact.js
- image2cpp. (n.d.). Javl.github.io. https://javl.github.io/image2cpp/

‌

‌
