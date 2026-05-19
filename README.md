# PROJETO DE ENCERRAMENTO DE RESIDÊNCIA
# DISPOSITIVO IoT PARA MONITORAMENTO HOSPITALAR E ASSISTÊNCIA AO LEITO 

## 1. O Problema
Sou Diretor de Inovação e Enfermagem de um hospital de alta complexidade. Um dos nossos maiores desafios é otimizar o tempo de resposta das equipes médicas em casos de intercorrências nas enfermarias e melhorar o conforto dos pacientes internados. O método atual de "botão de chamada de enfermeira" padrão é limitado, pois não diz à central qual é a gravidade da situação antes de o profissional se deslocar até o quarto. Além disso, pacientes acamados dependem constantemente de técnicos de enfermagem para ajustar manualmente o ângulo e a inclinação da cama (via manivela física) para evitar lesões por pressão (escaras) ou melhorar o conforto.
Precisamos de um sistema embarcado de assistência à beira do leito. Esse dispositivo deve monitorar continuamente a temperatura do quarto e os batimentos cardíacos do paciente (via sensor externo acoplado ao dedo). No leito, o paciente terá acesso a uma interface com dois botões físicos bem sinalizados: um de Chamada Simples e um de Emergência. Para o conforto, o sistema deve ler a inclinação do próprio dispositivo (usando um acelerômetro) e acionar um servo motor acoplado mecanicamente à estrutura da cama para ajustar a inclinação automaticamente para a esquerda ou direita com base no ângulo desejado. Todas as informações de sinais vitais, status de chamados e alertas devem ser exibidas em tempo real em um display local e enviadas para uma Dashboard central do posto de enfermagem.

Temos duas regras críticas de acionamento:
Chamada Simples: 
1. Se o botão de chamada for pressionado, um alerta visual deve ser enviado imediatamente para a Dashboard do posto de enfermagem.

2. Emergência com Áudio: Se o botão de emergência for acionado, o sistema deve entrar em estado de prioridade máxima. Enquanto o botão estiver sendo pressionado, o microfone interno deve gravar o áudio do quarto (limitado a um teto máximo de 10 segundos) e enviar o arquivo de áudio para o servidor de forma assíncrona, onde um sistema externo fará a transcrição de voz para a equipe saber exatamente o que está acontecendo (ex: "paciente caiu da cama") antes mesmo de chegar ao quarto.

## 2. Recomendações de Firmware e Conectividade
* **Conectividade e Telemetria de Sinais Vitais:** O sistema deve se conectar à rede local e transmitir em tempo real (ou em janelas curtíssimas de poucos segundos) a frequência cardíaca do paciente, a temperatura do ambiente e o status dos alertas para o servidor.

* **Tratamento de Interrupções (Botões):** Os botões de Chamada e Emergência devem ser configurados utilizando Interrupções de Hardware (ISR) no firmware. O sistema não pode ignorar ou atrasar o clique desses botões devido a outras tarefas paralelas (como atualização do display ou leitura de sensores).
Gravação Dinâmica por Pressionamento: A lógica do microfone deve monitorar o estado do botão de emergência. A gravação começa quando o botão é pressionado e se encerra assim que o botão for solto OU quando atingir o limite estrito de 10 segundos. O buffer de áudio deve ser fechado e transmitido rapidamente para o servidor.

* **Controle de Posicionamento (Acelerômetro + Servo):** O firmware deve converter os dados brutos de aceleração dos eixos (X, Y, Z) para calcular o ângulo de inclinação atual. Com base nesse ângulo, o sinal PWM enviado ao servo motor deve ser ajustado para rotacionar o motor, simulando a automação da manivela física da cama para o lado esquerdo ou direito.

* **Interface Local (Display):** Atualizar constantemente a tela com o nome/ID do paciente fictício, batimentos por minuto (BPM), temperatura e uma indicação visual piscante bem clara se houver um chamado ativo ou emergência em andamento.

## 3. Requisitos de Modelagem 3D
Este projeto possui um desafio duplo de modelagem 3D, dividindo-se em duas peças fundamentais:
* **Gabinete Principal (Beira do Leito):** Um case para abrigar a placa educacional (ESP32-S3, display e microfone). Deve possuir aberturas funcionais para a tela do display, furações para a captação limpa do microfone e acesso para os cabos dos botões externos e da alimentação. Deve conter abas para fixação na parede ou no suporte de soro ao lado da cama.

* **Acessório do Sensor Cardíaco (Oxímetro de Dedo):** Desenvolver um case específico e ergonômico para o sensor externo de batimentos cardíacos. Esse modelo 3D deve mimetizar o formato de um oxímetro de pulso hospitalar, permitindo que o paciente encaixe confortavelmente o dedo de forma firme. O design deve garantir que o sensor fique em contato direto com a pele, bloqueando o máximo de luz externa possível (o que melhora a precisão da leitura do sensor óptico).

* **Acoplamento do Sensor Mecânico:** O projeto também deve prever geometricamente como o case principal se posicionará para que o acelerômetro interno registre a inclinação correta do leito, ou como o servo motor se acoplará mecanicamente à simulação da manivela da cama.

## 4. Entregável Inicial: Documento de Arquitetura e Viabilidade Técnica
Antes de iniciar o desenvolvimento prático do firmware ou a modelagem dos cases 3D, a equipe deverá entregar um relatório técnico de arquitetura contendo obrigatoriamente:
### 4.1. Mapeamento de Hardware e Componentes
A equipe deve identificar e documentar os periféricos internos e externos utilizados:
* **Identificação:** Modelos exatos do sensor de temperatura, do sensor de batimentos cardíacos externo, do acelerômetro, do display, do microfone interno, do servo motor e o tipo de botões que serão implementados.
* **Pinout (Tabela de GPIOs):** Tabela detalhada mapeando as conexões de todos os pinos com o ESP32-S3, destacando quais pinos suportam interrupções externas (para os botões) e qual pino possui saída PWM (para o servo motor).
* **Barramentos Internos:** Indicação das interfaces de comunicação (ex: I2C para display/acelerômetro, I2S ou ADC para o microfone, sinal digital ou analógico para o sensor de pulso).
### 4.2. Diagrama de Blocos do Sistema
Apresentar um esquema gráfico modular do hardware. No centro deve figurar o microcontrolador ESP32-S3 e, ao redor, blocos conectados por setas indicando o fluxo de sinal:
Setas de entrada (In): Sensor de temperatura, Sensor de batimentos externo, Acelerômetro, Microfone e os dois Botões (Chamada e Emergência) apontando para o ESP32-S3.
* **Setas de saída (Out):** ESP32-S3 apontando para o Display e para o Servo Motor.
* **Fluxo de rede:** Indicação de comunicação de rede sem fio enviando dados de telemetria, sinalizações de alertas e o buffer de áudio gravado para o servidor.
### 4.3. Especificação do Protocolo de Comunicação e IoT
* **Protocolo de Aplicação e Rede:** Definição se o fluxo de telemetria e botões utilizará MQTT ou HTTP, e qual será a estratégia de rede para transmitir o arquivo de áudio gerado pelo botão de emergência sem bloquear o envio dos batimentos cardíacos em tempo real.
* **Estrutura de Dados (Payloads):** Exemplos estruturados em JSON de como as mensagens de batimentos e temperatura serão enviadas, bem como a formatação do pacote que sinaliza ao posto de enfermagem qual botão foi ativado.
* **Arquitetura do Servidor:** Descrição de onde a Dashboard de monitoramento será criada e como ela exibirá visualmente o alerta crítico de emergência para atrair a atenção imediata dos enfermeiros.

