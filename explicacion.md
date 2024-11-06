Proyecto:cortinas
El proyecto que quiero realizar son unas cortinas automáticas con un sensor de luz y otro sensor ultrasonido para detectar el porcentaje de apertura y que sea con comunicación i2f
las conexiones serian:
el esp 32  conectado a los sensores y al mismo tiempo a los motores paso a paso para que se puedan regular automáticamente 
Materiales a usar:
2 motores paso a paso (bipolares)
1 fotocélula
1 sensor ultrasónico
1 ESP32
¿Por qué estos materiales?
2 motores paso a paso (bipolares): Estos motores serían para mover la cortina horizontalmente. Sé que se puede hacer con uno solo, pero es para más potencia. Además, son bipolares para que puedan abrir y cerrar las cortinas.
1 fotocélula: Es para detectar la cantidad de lúmenes que entran a la habitación y así determinar si se cierra o se abre.
1 sensor ultrasónico: Es para que la cortina no siga avanzando y para que se pueda saber el porcentaje de la cortina que está abierta o cerrada y controlarlo.
1 ESP32: Es para la parte de comunicación y para mover los motores.
Diagrama de flujo:

El flujo de tiempo comenzaría con un sensor de luz, el LDR 03, que está conectado a una resistencia y un potenciómetro para ajustar la sensibilidad con respecto a la luz que recibe el sensor LDR. Además,
aunque no sea visible, también incluye un sensor ultrasónico conectado al ESP32 y dos motores paso a paso bipolares. Estos motores están conectados a un puente H L298N. Cabe destacar que el puente H solo puede controlar un motor con sus dos bobinas, pero al requerir solo una bobina por motor, se utiliza de esta manera por motivos económicos.


![Captura](https://github.com/user-attachments/assets/b439d339-f2a9-46ab-a5eb-7009a3db3d98)


En este esquema podemos ver cómo funcionará la información transmitida en el proyecto. Como se observa, el sensor ultrasónico y el LDR envían información al ESP32, y esta información se transmite al L298N, que controla las bobinas de cada motor. Los motores se mueven en función de la información proporcionada por los sensores.
También se puede observar que el puente H debe estar conectado a una fuente externa de 12V, ya que el ESP32 no puede manejar esos valores, y con esta fuente es capaz de controlar los movimientos.
![Captura2](https://github.com/user-attachments/assets/94954009-b6e2-4e5a-aa06-bf1355b82f88)

Código:
#include <AccelStepper.h>


// Definición de pines
#define MOTOR_OPEN_DIR 25 // Cambia según tu conexión
#define MOTOR_OPEN_STEP 26
#define MOTOR_CLOSE_DIR 27
#define MOTOR_CLOSE_STEP 14
#define LDR_PIN 34 // Pin analógico
#define TRIG_PIN 32
#define ECHO_PIN 33


// Parámetros de la cortina
const int LDR_THRESHOLD = 300; // Valor de luz para cerrar
const int DISTANCE_THRESHOLD = 10; // Distancia para fin de carrera (en cm)


// Inicializar motores
AccelStepper motorOpen(AccelStepper::DRIVER, MOTOR_OPEN_STEP, MOTOR_OPEN_DIR);
AccelStepper motorClose(AccelStepper::DRIVER, MOTOR_CLOSE_STEP, MOTOR_CLOSE_DIR);


void setup() {
  Serial.begin(115200);
  motorOpen.setMaxSpeed(1000);
  motorClose.setMaxSpeed(1000);
 
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
}


void loop() {
  int ldrValue = analogRead(LDR_PIN);
  long distance = getDistance();


  if (ldrValue < LDR_THRESHOLD) {
    closeCurtains();
  } else {
    if (distance > DISTANCE_THRESHOLD) {
      openCurtains();
    }
  }
 
  delay(1000); // Espera un segundo antes de volver a leer
}


void openCurtains() {
  motorOpen.moveTo(2000); // Ajustar según la longitud de la cortina
  motorOpen.runToPosition();
}


void closeCurtains() {
  motorClose.moveTo(2000); // Ajustar según la longitud de la cortina
  motorClose.runToPosition();
}


long getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
 
  long duration = pulseIn(ECHO_PIN, HIGH);
  long distance = (duration / 2) * 0.0343; // Convertir a cm
  return distance;
}





