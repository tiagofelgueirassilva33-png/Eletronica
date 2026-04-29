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

// Mostra o placar nos módulos centrais
void desenhaPontos() {
  for(int m = 0; m < 4; m++) lc.clearDisplay(m); 

  for (int i = 0; i < 5; i++) {
    lc.setRow(2, i + 1, numeros[pontos[0]][i]); // J1 na Matriz 2
    lc.setRow(1, i + 1, numeros[pontos[1]][i]); // J2 na Matriz 1
  }
}

// Animação de Vitória e Derrota
void animacaoFinal(int vencedor) {
  for(int m = 0; m < 4; m++) lc.clearDisplay(m);

  // Pisca as letras 3 vezes para o final do jogo
  for (int r = 0; r < 3; r++) {
    for (int i = 0; i < 5; i++) {
      if (vencedor == 0) { // Jogador 1 venceu (Esquerda)
        lc.setRow(3, i + 1, letraW[i]); // W na ponta esquerda
        lc.setRow(0, i + 1, letraL[i]); // L na ponta direita
      } else { // Jogador 2 venceu (Direita)
        lc.setRow(0, i + 1, letraW[i]); // W na ponta direita
        lc.setRow(3, i + 1, letraL[i]); // L na ponta esquerda
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
  
  // Se alguém atingir o limite (3), faz a animação final de Winner/Loser
  if (pontos[jogadorVencedor] >= LIMITE_PONTOS) {
    animacaoFinal(jogadorVencedor);
    pontos[0] = 0;
    pontos[1] = 0;
  } else {
    // Caso contrário, apenas mostra o placar normal por 2 segundos
    desenhaPontos();
    delay(2000);
  }

  // Reinicia o campo e a bola
  lc.clearDisplay(1);
  lc.clearDisplay(2);
  
  Bola_Lin = 4; 
  Bola_Col = 15;
  Vel_Col = (jogadorVencedor == 0) ? 0.5 : -0.5; // Lança a bola para quem perdeu o ponto
  posAntiga[0] = -1; 
  posAntiga[1] = -1;
}

void loop() {
  // Leitura dos Joysticks
  int yRaq1 = map(analogRead(xPin1), 0, 1023, 1, 6); 
  int yRaq2 = map(analogRead(xPin2), 0, 1023, 1, 6); 

  // Apaga rastro da bola
  ligaLed(Bola_Lin, Bola_Col, false);

  // Move a bola
  Bola_Lin += Vel_Lin;
  Bola_Col += Vel_Col;

  // Ressalto teto/chão
  if (Bola_Lin <= 0 || Bola_Lin >= 7) Vel_Lin *= -1; 
  
  // Colisão Raquete Jogador 1 (Esquerda)
  if (Bola_Col <= 0) { 
    if (round(Bola_Lin) >= yRaq1 - 1 && round(Bola_Lin) <= yRaq1 + 1) {
      Vel_Col *= -1;
      // Tua lógica de física: variação de velocidade se não bater no centro
      if (round(Bola_Lin) != yRaq1){
        Vel_Col *= random(5, 20) / 10.0;
        Vel_Col = max(min(Vel_Col, 2.5), 0.25);
      }
      Bola_Col = 1;
    } else {
      processaPonto(1); // Ponto para o Jogador 2
    }
  } 
  // Colisão Raquete Jogador 2 (Direita)
  else if (Bola_Col >= 31) { 
    if (round(Bola_Lin) >= yRaq2 - 1 && round(Bola_Lin) <= yRaq2 + 1) {
      Vel_Col *= -1;
      if (round(Bola_Lin) != yRaq2 ){
        Vel_Col *= random(5, 20) / 10.0;
        Vel_Col = max(min(Vel_Col, 2.5), 0.25);
      }
      Bola_Col = 30;
    } else {
      processaPonto(0); // Ponto para o Jogador 1
    }
  }

  // Atualiza raquetes e bola
  raquetes(0, yRaq1); 
  raquetes(1, yRaq2);
  ligaLed(Bola_Lin, Bola_Col, true); 
  
  delay(30); 
}
