#include <LedControl.h>

// Configuração da Matriz: DIN, CLK, CS, número de módulos
LedControl lc = LedControl(12, 11, 10, 4);

// DEFINIÇÃO DOS PINOS DOS JOYSTICKS
#define xPin1 A2 // Joystick do Jogador 1 (Raquete Esquerda)
#define xPin2 A1 // Joystick do Jogador 2 (Raquete Direita)
#define buttonPin 2

float Bola_Lin = 4, Bola_Col = 15; 
float Vel_Lin = 0.5; 
float Vel_Col = -0.5;

int posAntiga[2] = {-1, -1};
int pontos[2] = {0, 0}; 
const int LIMITE_PONTOS = 3; 

// Fonte de números 0-3
const byte numeros[4][5] = {
  {B01110, B10001, B10001, B10001, B01110}, // 0
  {B00100, B01100, B00100, B00100, B01110}, // 1
  {B11110, B00001, B01110, B10000, B11111}, // 2
  {B11110, B00001, B01110, B00001, B11110}  // 3
};

// Desenhos das letras para animação final
const byte letraW[5] = {B10001, B10001, B10101, B10101, B01010}; // W (Winner)
const byte letraL[5] = {B10000, B10000, B10000, B10000, B11111}; // L (Loser)

void setup() {
  for (int m = 0; m < 4; m++) { 
    lc.shutdown(m, false);   
    lc.setIntensity(m, 4);
    lc.clearDisplay(m);
  }
  
  pinMode(buttonPin, INPUT_PULLUP);
  randomSeed(analogRead(A0)); 
  Serial.begin(9600);
}

void desenhaPontos() {
  for(int m = 0; m < 4; m++) lc.clearDisplay(m); 
  for (int i = 0; i < 5; i++) {
    lc.setRow(2, i + 1, numeros[pontos[0]][i]); 
    lc.setRow(1, i + 1, numeros[pontos[1]][i]); 
  }
}

void animacaoFinal(int vencedor) {
  for(int m = 0; m < 4; m++) lc.clearDisplay(m);
  for (int r = 0; r < 3; r++) {
    for (int i = 0; i < 5; i++) {
      if (vencedor == 0) { 
        lc.setRow(3, i + 1, letraW[i]); 
        lc.setRow(0, i + 1, letraL[i]); 
      } else { 
        lc.setRow(0, i + 1, letraW[i]); 
        lc.setRow(3, i + 1, letraL[i]); 
      }
    }
    delay(500);
    for(int m = 0; m < 4; m++) lc.clearDisplay(m);
    delay(300);
  }
}

void ligaLed(float linha, float coluna, bool liga) { 
  int c = round(coluna);
  int l = round(linha);
  if (c >= 0 && c <= 31 && l >= 0 && l <= 7) {
    lc.setLed(3 - (c / 8), l, c % 8, liga); 
  }
}

void raquetes(int jogador, int posAtual) {  
  int coluna = (jogador == 0) ? 0 : 31;
  if (posAtual != posAntiga[jogador] || round(Bola_Col) == coluna) {
    lc.setColumn(3 - (coluna / 8), coluna % 8, B00000000); 
    ligaLed(posAtual - 1, coluna, true); 
    ligaLed(posAtual,     coluna, true); 
    ligaLed(posAtual + 1, coluna, true);
    posAntiga[jogador] = posAtual;
  }
}

void processaPonto(int jogadorVencedor) {
  pontos[jogadorVencedor]++;
  
  if (pontos[jogadorVencedor] >= LIMITE_PONTOS) {
    animacaoFinal(jogadorVencedor);
    pontos[0] = 0;
    pontos[1] = 0;
  } else {
    desenhaPontos();
    delay(2000);
  }

  lc.clearDisplay(1);
  lc.clearDisplay(2);
  
  Bola_Lin = 4; 
  Bola_Col = 15;
  // Define direção inicial da bola e reseta velocidade vertical
  Vel_Col = (jogadorVencedor == 0) ? 0.5 : -0.5; 
  Vel_Lin = (random(0, 2) == 0) ? 0.4 : -0.4; 
  
  posAntiga[0] = -1; 
  posAntiga[1] = -1;
}

void loop() {
  int yRaq1 = map(analogRead(xPin1), 0, 1023, 1, 6); 
  int yRaq2 = map(analogRead(xPin2), 0, 1023, 1, 6); 

  ligaLed(Bola_Lin, Bola_Col, false);

  Bola_Lin += Vel_Lin;
  Bola_Col += Vel_Col;

  // Ressalto teto/chão
  if (Bola_Lin <= 0 || Bola_Lin >= 7) Vel_Lin *= -1; 
  
  // COLISÃO JOGADOR 1 (ESQUERDA)
  if (Bola_Col <= 0) { 
    if (round(Bola_Lin) >= yRaq1 - 1 && round(Bola_Lin) <= yRaq1 + 1) {
      Vel_Col = abs(Vel_Col); // Garante movimento para a DIREITA (+)
      
      if (round(Bola_Lin) != yRaq1){
        Vel_Col *= random(8, 15) / 10.0; // Variação leve
      }
      Vel_Col = max(min(Vel_Col, 2.0), 0.4); // Limita velocidade positiva
      Bola_Col = 1; 
    } else {
      processaPonto(1); 
    }
  } 
  // COLISÃO JOGADOR 2 (DIREITA)
  else if (Bola_Col >= 31) { 
    if (round(Bola_Lin) >= yRaq2 - 1 && round(Bola_Lin) <= yRaq2 + 1) {
      Vel_Col = -abs(Vel_Col); // Garante movimento para a ESQUERDA (-)
      
      if (round(Bola_Lin) != yRaq2 ){
        // Calculamos o aumento baseado no valor absoluto para não inverter o sinal
        float magnitude = abs(Vel_Col) * (random(8, 15) / 10.0);
        magnitude = max(min(magnitude, 2.0), 0.4);
        Vel_Col = -magnitude; // Reaplica o sinal negativo
      }
      Bola_Col = 30;
    } else {
      processaPonto(0);
    }
  }

  raquetes(0, yRaq1); 
  raquetes(1, yRaq2);
  ligaLed(Bola_Lin, Bola_Col, true); 
  
  delay(35); 
}
