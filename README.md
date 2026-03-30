# Eletronica
## Projeto Ping Pong
#include <LedControl.h>
LedControl lc = LedControl(12, 11, 10, 4);

#define xPin A0
#define yPin A1
#define buttonPin 2

float Bola_Lin = 4, Bola_Col = 15;
float Vel_Lin = 1.6;
float Vel_Col = -0.6;

unsigned long delaytime = 200;



void setup() {

  for (int m = 0; m < 4; m++) {
    lc.shutdown(m, false);
    /* Set the brightness to a medium values */
    lc.setIntensity(m, 8);
    /* and clear the display */
    lc.clearDisplay(m);    
  }

  //  joystick
  pinMode(buttonPin, INPUT_PULLUP);

  randomSeed(analogRead(A2));
  Serial.begin(9600);

}

void raquetes(int raquete, int pos) {
  lc.setColumn(3-raquete*3, raquete*7, B00000000);
  ligaLed(pos - 1,  raquete*31, true);
  ligaLed(pos,  raquete*31, true);
  ligaLed(pos + 1,  raquete*31, true);
  
}




void ligaLed(float linha, float coluna, bool liga) {
  lc.setLed(3 - round(coluna) / 8, round(linha), round(coluna) % 8, liga);
}

/*void joystick(int xPin = A0, int yPin = A1, int buttonPin = 2, int xVal,int yVal, int buttonState){
  
  xVal = analogRead(xPin);
  yVal = analogRead(yPin);
  buttonState = digitalRead(buttonPin);


  Serial.print("X: ");
  Serial.print(xVal);
  Serial.print(" | Y: ");
  Serial.print(yVal);
  Serial.print(" | Button: ");
  Serial.print(buttonState);


}
*/
  



void loop() {

  // definir posição da raquete
  // ler os valores do X do joystick
  int xVal1 = analogRead(xPin1);
  int xVal2 = analogRead(xPin2);
  
  Serial.println(xVal1);
  raquetes(0, map(xVal1, 0,1010, 0,7));
  raquetes(1, map(xVal2, 0,1010, 0,7));

  ligaLed(Bola_Lin, Bola_Col, false);

  if (Bola_Lin + Vel_Lin > 7 || Bola_Lin + Vel_Lin < 0) {
    Vel_Lin *= -1;
  }
  Bola_Lin = Bola_Lin + Vel_Lin;

  if (Bola_Col + Vel_Col > 31 || Bola_Col + Vel_Col < 0) {
    Vel_Col *= -1;
    Vel_Lin *= random(2, 20) / 10;
  }
  Bola_Col += Vel_Col;


  ligaLed(Bola_Lin, Bola_Col, true);
  delay(40);
}
