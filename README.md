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
   - O código no ESP32 realiza a leitura da corrente elétrica da tomada por meio do sensor ACS712.
   - O sensor é calibrado para garantir leituras precisas.
   - O código calcula a corrente de pico e a corrente eficaz.
   - Os valores da corrente são impressos no monitor serial para exibição.

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
