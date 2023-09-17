# sprint.edge
# README - Medidor de Energia IoT com Docker Compose

Este é um projeto que demonstra a implementação de um medidor de energia IoT utilizando um medidor de corrente ACS712 de 30A, um microcontrolador ESP32, e um conversor de energia para 5V. O projeto é parte de uma aplicação IoT mais ampla que usa Docker Compose para gerenciar componentes de backend e frontend.

## Recursos Necessários
- Núcleos de Processamento - 1vCPU
- Memória RAM - 1GB
- Armazenamento Secundário Mínimo - 20GB HD e/ou SSD (Depende dos requisitos da aplicação).
### Hardware:
- Dispositivos IoT (medidores de energia) - um para cada tomada a ser monitorada.
- ESP32.
- Medidor de corrente ACS712 de 30A.
- Conversor de energia para 5V.
- Cabos e conexões adequadas.

### Software:
- Docker Compose.
- Imagem Docker do backend da aplicação IoT.
- Imagem Docker do frontend da aplicação IoT.

## Funcionamento do Medidor de Energia

O medidor de energia (MeterWatt) é colocado em cada tomada da casa para medir o consumo de energia em tempo real. Ele funciona da seguinte maneira:

1. **Conexão de Hardware**:
   - A porta VCC do ACS712 está conectada à porta VIN do ESP32.
   - O GND do ACS712 está conectado ao GND do ESP32.
   - A porta OUT do ACS712 está conectada à porta VP do ESP32.

2. **Código do Processo**:

int pino_sensor = 36;
int menor_valor;
int valor_lido;
int menor_valor_acumulado = 0;
int ZERO_SENSOR = 0;
float corrente_pico;
float corrente_eficaz;
double maior_valor=0;
double corrente_valor=0;

void setup() {
  Serial.begin(9600);
  pinMode(pino_sensor,INPUT);
delay(3000);
 //Fazer o AUTO-ZERO do sensor
Serial.println("Fazendo o Auto ZERO do Sensor...");
menor_valor = 4095;
 
  for(int i = 0; i < 10000 ; i++){
  valor_lido = analogRead(pino_sensor);
  if(valor_lido < menor_valor){
  menor_valor = valor_lido;    
  }
  delayMicroseconds(1);  
  }
  ZERO_SENSOR = menor_valor;
  Serial.print("Zero do Sensor:");
  Serial.println(ZERO_SENSOR);
  delay(3000)
 }

void loop() {

  //Zerar valores
  menor_valor = 4095;
 
  for(int i = 0; i < 1600 ; i++){
  valor_lido = analogRead(pino_sensor);
  if(valor_lido < menor_valor){
  menor_valor = valor_lido;    
  }
  delayMicroseconds(10);  
  }

  
  Serial.print("Menor Valor:");
  Serial.println(menor_valor);
  
  //Transformar o maior valor em corrente de pico
  corrente_pico = ZERO_SENSOR - menor_valor; 
  corrente_pico = corrente_pico*0.805; // 
  corrente_pico = corrente_pico/66;   // COnverter o valor de tensão para corrente de acordo com o modelo do sensor. No meu caso, esta sensibilidade vale 66mV/                       
  Serial.print("Corrente de Pico:");
  Serial.print(corrente_pico);
  Serial.print(" A");
  Serial.print("     ");
  Serial.print(corrente_pico*1000);
  Serial.println(" mA");
  
 
  //Converter para corrente eficaz  
  corrente_eficaz = corrente_pico/1.4;
  Serial.print("Corrente Eficaz:");
  Serial.print(corrente_eficaz);
  Serial.print(" A");
  Serial.print("     ");
  Serial.print(corrente_eficaz*1000);
  Serial.println(" mA");
 
 delay(5000);
}

## Instruções de Uso

1. **Preparação do Hardware**:
   - Conecte o ACS712 ao ESP32 conforme descrito na seção "Conexão de Hardware".
   - Certifique-se de que os dispositivos IoT estejam instalados em cada tomada.

2. **Configuração do Software**:
   - Certifique-se de ter o Docker Compose instalado na sua máquina.

3. **Execução do Projeto IoT**:
   - Clone este repositório git https://github.com/fabiocabrini/fiware
   - Use o Docker Compose para iniciar os serviços do backend e frontend da aplicação IoT.

   ```bash
   docker-compose up -d
   ```

4. **Acessando o Frontend**:
   - Abra um navegador da web e acesse o frontend da aplicação em `http://localhost:8080`.

5. **Visualizando os Dados**:
   - Na interface do frontend, você poderá visualizar os dados de consumo de energia em tempo real.

## Requisitos e Dependências

Certifique-se de que você atenda aos seguintes requisitos e dependências:

- Hardware conforme descrito na seção "Recursos Necessários".
- Docker Compose instalado na sua máquina.
- Imagens Docker do backend e frontend da aplicação IoT disponíveis.
