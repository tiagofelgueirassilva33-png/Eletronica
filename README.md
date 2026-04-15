#include <LedControl.h>

// Pinos: DIN, CLK, CS, Número de módulos
LedControl lc = LedControl(12, 11, 10, 4);

#define xPin1 A0 
#define xPin2 A1 
#define buttonPin 2

float Bola_Lin = 4, Bola_Col = 15; 
float Vel_Lin = 0.5; 
float Vel_Col = -0.5;

int posAntiga[2] = {-1, -1};
int pontos[2] = {0, 0}; 
const int LIMITE_PONTOS = 3; 

// Fonte de números 0-9
const byte numeros[10][5] = {
  {B01110, B10001, B10001, B10001, B01110}, // 0
  {B00100, B01100, B00100, B00100, B01110}, // 1
  {B11110, B00001, B01110, B10000, B11111}, // 2
  {B11110, B00001, B01110, B00001, B11110}, // 3
  {B10001, B10001, B11111, B00001, B00001}, // 4
  {B11111, B10000, B11110, B00001, B11110}, // 5
  {B01110, B10000, B11110, B10001, B01110}, // 6
  {B11111, B00001, B00010, B00100, B01000}, // 7
  {B01110, B10001, B01110, B10001, B01110}, // 8
  {B01110, B10001, B01111, B00001, B01110}  // 9
};

void setup() {
  for (int m = 0; m < 4; m++) { 
    lc.shutdown(m, false);   
    lc.setIntensity(m, 4);
    lc.clearDisplay(m);
  }
  pinMode(buttonPin, INPUT_PULLUP);
  randomSeed(analogRead(A2)); 
  Serial.begin(9600);
}

// Mostra a pontuação apenas no intervalo
void desenhaPontos() {
  for(int m=0; m<4; m++) lc.clearDisplay(m); // Limpa o campo para o placar

  for (int i = 0; i < 5; i++) {
    lc.setRow(2, i + 1, numeros[pontos[0]][i]); // Matriz do Jogador 1
    lc.setRow(1, i + 1, numeros[pontos[1]][i]); // Matriz do Jogador 2
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
  
  desenhaPontos(); // Mostra o placar
  delay(2000);    // Pausa de 2 segundos

  if (pontos[0] >= LIMITE_PONTOS || pontos[1] >= LIMITE_PONTOS) {
    pontos[0] = 0;
    pontos[1] = 0;
    // Podes adicionar aqui um efeito de piscar se quiseres celebrar a vitória
  }

  // Limpa as matrizes centrais e reinicia o campo
  lc.clearDisplay(1);
  lc.clearDisplay(2);
  
  Bola_Lin = 4; 
  Bola_Col = 15;
  Vel_Col *= -1; 
  posAntiga[0] = -1; 
  posAntiga[1] = -1;
}

void loop() {
  // Ajuste do mapeamento para os teus joysticks
  int xVal1 = map(analogRead(xPin1), 10, 1010, 1, 6); 
  int xVal2 = map(analogRead(xPin2), 10, 1010, 1, 6); 

  ligaLed(Bola_Lin, Bola_Col, false); // Apaga rastro

  Bola_Lin += Vel_Lin;
  Bola_Col += Vel_Col;

  // Colisão com bordas horizontais
  if (Bola_Lin <= 0 || Bola_Lin >= 7) Vel_Lin *= -1; 
  
  // Lógica de colisão com raquetes e pontos
  if (Bola_Col <= 0) { 
    if (round(Bola_Lin) >= xVal1 - 1 && round(Bola_Lin) <= xVal1 + 1) {
      Vel_Col *= -1;
      Bola_Col = 1;
    } else {
      processaPonto(1); // Jogador 2 pontua
    }
  } 
  else if (Bola_Col >= 31) { 
    if (round(Bola_Lin) >= xVal2 - 1 && round(Bola_Lin) <= xVal2 + 1) {
      Vel_Col *= -1;
      Bola_Col = 30;
    } else {
      processaPonto(0); // Jogador 1 pontua
    }
  }

  raquetes(0, xVal1); 
  raquetes(1, xVal2);
  ligaLed(Bola_Lin, Bola_Col, true); 
  
  delay(30); 
}
