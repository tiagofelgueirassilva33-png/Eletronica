#include <LedControl.h>

// Pinos: DIN, CLK, CS, Número de módulos
LedControl lc = LedControl(12, 11, 10, 4);

#define xPin1 A0 
#define xPin2 A1 
#define buttonPin 2

float Bola_Lin = 4, Bola_Col = 15; 
float Vel_Lin = 0.5; // Velocidade ajustada para ser mais suave
float Vel_Col = -0.5;

// Variáveis para evitar o efeito de piscar (flicker)
int posAntiga[2] = {-1, -1};


void setup() {
  for (int m = 0; m < 4; m++) { 
    lc.shutdown(m, false);   
    lc.setIntensity(m, 8);
    lc.clearDisplay(m);
  }

  pinMode(buttonPin, INPUT_PULLUP);
  randomSeed(analogRead(A2)); 
  Serial.begin(9600);
}

void ligaLed(float linha, float coluna, bool liga) { 
  // Garante que não tentamos desenhar fora dos limites 0-31 e 0-7
  int c = round(coluna);
  int l = round(linha);
  if (c >= 0 && c <= 31 && l >= 0 && l <= 7) {
    lc.setLed(3 - (c / 8), l, c % 8, liga); 
  }
}

void raquetes(int jogador, int posAtual) {  
  // Só redesenha se a posição mudou, evitando o "flicker"
  if (posAtual != posAntiga[jogador]) {
    int coluna = (jogador == 0) ? 0 : 31;
    // Limpa a coluna do jogador antes de desenhar a nova posição
    lc.setColumn(3 - (coluna / 8), coluna % 8, B00000000); 
    
    ligaLed(posAtual - 1, coluna, true); 
    ligaLed(posAtual,     coluna, true); 
    ligaLed(posAtual + 1, coluna, true);

    // Guarda a posição atual para a próxima comparação
    posAntiga[jogador] = posAtual;
    
  }
}

void loop() {
  // Ler joysticks e mapear posição (limite 1 a 6 para a raquete caber no ecrã)
  int xVal1 = map(analogRead(xPin1), 10, 1000, 1, 6); 
  int xVal2 = map(analogRead(xPin2), 10, 1000, 1, 6); 

  raquetes(0, xVal1); 
  raquetes(1, xVal2);

  // Lógica da Bola
  

  // Colisão com as paredes de cima e baixo
  if (Bola_Lin + Vel_Lin > 7 || Bola_Lin + Vel_Lin < 0) { 
    Vel_Lin *= -1; 
  } 
  
  // Colisão com as laterais (ponto ou rebote)
  if (Bola_Col + Vel_Col > 30 || Bola_Col + Vel_Col < 1) { 
    if (Bola_Col < 1) 
      if (BolaCol >= xVal1-1 && BolaCol <= xVal1+1){
        Vel_Col *= -1;
      }
      else{
        delay(5000); // perdeu o 1
      }
    else{  // teste jogador 2
      if (BolaCol >= xVal2-1 && BolaCol <= xVal2+1){
        Vel_Col *= -1;
      }
      else{
        delay(5000); // perdeu o 2
      }
    }
     

    // Pequena variação aleatória no rebote para o jogo não ser infinito
    Vel_Lin = (random(5, 15) / 10.0) * (Vel_Lin > 0 ? 1 : -1); 
  }

  ligaLed(Bola_Lin, Bola_Col, false); // Apaga posição anterior
  Bola_Lin += Vel_Lin;
  Bola_Col += Vel_Col;
  if (1){
    ligaLed(Bola_Lin, Bola_Col, true); // Desenha nova posição
  }
  delay(30); 
}
