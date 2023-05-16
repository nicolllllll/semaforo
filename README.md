//Declarando arreglo para pines
const int ledPin[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13};

//tiempo esta dados en milisegundos
const int changeRedToYellow    = 5000;
const int changeYellowToGreen  = 200;
const int changeGreenToRed     = 5000;

// estado inicial para sensores
int sensoInfrarrojo_1 = 0;
int sensoInfrarrojo_2 = 0;
int sensoInfrarrojo_3 = 0;
int sensoInfrarrojo_4 = 0;

//tiempo  para que el semaforo vuelva a la normalidad (funci�n  millis())
unsigned long timeSemaforo = 0;

//modificar tiempo sem�foro contrancon
int endTimeSemaforoSensor  = 10;

//instrucci�n extraida de foro de arduino  : https://forum.arduino.cc/t/reiniciar-millis/576475/2, reset var millis()->timeSemaforo
extern volatile unsigned long timer0_millis;

//main : punto configuraci�n, aquivalente
void setup() {

  //iniciar puerto serie
  Serial.begin(9600);

  //configurando pines como salida
  for (int i = 0; i <= 13; i++) {
    pinMode(ledPin[i], OUTPUT);
  }

  //configurando pin como entrada [infrarojo->1]->sem�foro 1
  pinMode(ledPin[8], INPUT);

  //configurando pin como entrada [infrarojo->2] ->sem�foro 1
  pinMode(ledPin[9], INPUT);

  //configurando pin como entrada [infrarojo->2] ->sem�foro 2
  pinMode(ledPin[12], INPUT);

  //configurando pin como entrada [infrarojo->2] ->sem�foro 2
  pinMode(ledPin[13], INPUT);

}

void loop() {

  //leyendo sensores infrarojos para sem�foro 1
  sensoInfrarrojo_1 = digitalRead(ledPin[8]);
  sensoInfrarrojo_2 = digitalRead(ledPin[9]);
  sensoInfrarrojo_3 = digitalRead(ledPin[12]);
  sensoInfrarrojo_4 = digitalRead(ledPin[13]);

 //evalua los casos de uso
  int StateSensor =  sensorState (sensoInfrarrojo_1,  sensoInfrarrojo_2, sensoInfrarrojo_3, sensoInfrarrojo_4);

  Serial.print("a1 :: ");
  Serial.print(StateSensor);
  Serial.println();

  switch (StateSensor) {
    case 1:
      //cambian estado de pines
      digitalWrite(ledPin[7], HIGH);
      digitalWrite(ledPin[2], HIGH);
      digitalWrite(ledPin[6], LOW);
      digitalWrite(ledPin[3], LOW);
      digitalWrite(ledPin[5], LOW);
      digitalWrite(ledPin[4], LOW);

      //simulaci�n encendido de pantalla
      digitalWrite(ledPin[11], HIGH);

      //inici conteo de tiempo con la funcion  millis de arduino
      timeSemaforo = millis() / 1000;

      //si el tiempo de millis es igual alde la variable cambia de estado
      if (endTimeSemaforoSensor == timeSemaforo ) {

        //apagando pines del 2 al 7
        lowLed();

        //se repite dos veces para darle paso al sem�foro 2
        for (int i = 1; i <= 2; i++) {

          //apagando pines del 2 al 7
          lowLed();

          // inicia secuencia normal solo dos veces
          semaforoNormal();
        }

        //se reinicia la variable de la funci�n  millis de arduino
        setMillis(timeSemaforo);

      }
      break;
    case 2:
      //lowLed();
      //cambian estado de pines
      digitalWrite(ledPin[5], HIGH);
      digitalWrite(ledPin[4], HIGH);
      digitalWrite(ledPin[6], LOW);
      digitalWrite(ledPin[3], LOW);
      digitalWrite(ledPin[7], LOW);
      digitalWrite(ledPin[2], LOW);

      //simulaci�n encendido de pantalla
      digitalWrite(ledPin[10], HIGH);

      //inici conteo de tiempo con la funcion  millis de arduino
      timeSemaforo = millis() / 1000;

      //si el tiempo de millis es igual alde la variable cambia de estado
      if (endTimeSemaforoSensor == timeSemaforo ) {

        //apagando pines del 2 al 7
        lowLed();

        //se repite dos veces para darle paso al sem�foro 2
        for (int i = 1; i <= 2; i++) {

          //apagando pines del 2 al 7
          lowLed();

          // inicia secuencia normal solo dos veces
          semaforoNormal();
        }

        //se reinicia la variable de la funci�n  millis de arduino
        setMillis(timeSemaforo);

      }
      break;

      case 3:
      //apagando pines del 2 al 7
      lowLed();

      // inicia secuencia normal si los dos sensores  no estan en estado HIGH
      semaforoNormal();

      //se reinicia la variable de la funci�n  millis de arduino
      setMillis(timeSemaforo);

      break;


    default:
      //apagando pines del 2 al 7
      lowLed();

      // inicia secuencia normal si los dos sensores  no estan en estado HIGH
      semaforoNormal();

      //se reinicia la variable de la funci�n  millis de arduino
      setMillis(timeSemaforo);
  }

}


int sensorState (int s1,  int s2, int s3, int s4) {

    // TABLA DE ESTADOS FUNCIONALES DEL SEM�FORO
    //************************************************************
    //state 2 = 1100
    //          1110
    //          1101
    //state 1 = 0011
    //          0111
    //          1011
    //*************************************************************
    //sensores infrarojos   trabajan a la inversa de activan el LOW
    //caso 1 de uso
    if(s1 == LOW && s2 == LOW && s3 == HIGH && s4 == HIGH){
      return 1;
    }
    if(s1 == LOW && s2 == LOW && s3 == HIGH && s4 == LOW){
      return 1;
    }
    if(s1 == LOW && s2 == LOW && s3 == LOW && s4 == HIGH){
      return 1;
    }
    //caso 1 de uso
    if(s1 == HIGH && s2 == HIGH && s3 == LOW && s4 == LOW){
      return 2;
    }
    if(s1 == LOW && s2 == HIGH && s3 == LOW && s4 == LOW){
      return 2;
    }
    if(s1 == HIGH && s2 == LOW && s3 == LOW && s4 == LOW){
      return 2;
    }
    //caso 3 de uso ( por defecto)
    return 3; 

}

//extraida de foro de arduino  : https://forum.arduino.cc/t/reiniciar-millis/576475/2, reset var millis()->timeSemaforo
void setMillis(unsigned long new_millis) {
  uint8_t oldSREG = SREG;
  cli();
  timer0_millis = new_millis;
  SREG = oldSREG;
}

//funci�n encargada de poner pines en estado bajo
void lowLed() {
  for (int i = 2; i <= 7; i++) {
    digitalWrite(ledPin[i], LOW);
  }
}

//funci�n encargada de secuencia del sem�foro en estado normal
void semaforoNormal() {
  //simulaci�n encendido de pantalla
  digitalWrite(ledPin[10], HIGH);

  //cambiando estado de los pines HIGH/LOW
  digitalWrite(ledPin[4], HIGH);
  digitalWrite(ledPin[5], HIGH);
  delay(changeRedToYellow);

  digitalWrite(ledPin[3], HIGH);
  delay(changeYellowToGreen);

  //simulaci�n apagado de pantalla
  digitalWrite(ledPin[10], LOW);

  //simulaci�n encendido de pantalla
  digitalWrite(ledPin[11], HIGH);

  //cambiando estado de los pines HIGH/LOW
  digitalWrite(ledPin[7], HIGH);
  digitalWrite(ledPin[4], LOW);
  digitalWrite(ledPin[3], LOW);
  digitalWrite(ledPin[5], LOW);
  digitalWrite(ledPin[2], HIGH);
  delay(changeYellowToGreen);

  //cambiando estado de los pines HIGH/LOW
  digitalWrite(ledPin[7], HIGH);
  delay(changeGreenToRed);

  //cambiando estado de los pines HIGH/LOW
  digitalWrite(ledPin[6], HIGH);


  delay(changeYellowToGreen);
  digitalWrite(ledPin[11], LOW);

}
