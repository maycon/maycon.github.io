---
layout: post
title:  "Embedded Hacking: Acessando a interface UART/Serial"
date:   2018-03-10
categories: embedded-hacking
tags: embedded-devices hacking-embedded IoT-hacking uart serial
lang: br
ref: embedded-hacking-interface-uart
has-asciinema: true
---

Na última publicação foi falado sobre uma forma de se iniciar o processo de análise dinâmica de alguns binários utilizando o qemu. A principal vantagem dessa técnica é a possibilidade de conseguir dinamicamente analisar alguns binários sem a necessidade de possuir o hardware em mãos. Porém, tendo o hardware em mãos, muitas vezes é possível identificar uma interface serial no dispositivo, que resulta praticamente no acesso a um console do dispositivo. Nessa publicação será explicado como podemos identificar e acessar essa interface serial presente em praticamente todos os dispositivos embarcados.

### A interface UART

A interface UART (Universal Asynchronous Receiver-Transmitter), comumente encontrada em dispositivos embarcados, consiste em um hardware que fornece uma interface serial de comunicação assíncrona, e seu principal objetivo é fornecer um mecanismo de análise e depuração do dispositivo. Atualmente essa interface já é implementada em microcontroladores e microprocessadores, e é ferramenta fundamental para a engenharia reversa do dispositivo, por fornecer um acesso dinâmico ao dispositivo.

Geralmente a interface UART é identificada na placa do dispositivo com 4 conectores. Esses conectores podem estar com pinos, o que facilita bastante o trabalho, porém algumas vezes será necessário soltar ou prender pinos à placa para se ter uma forma de usar a conexão. Em alguns casos esses conectores sequer estão próximos um do outros, tornando necessário uma análise mais minuciosa da placa para identificá-los. 

### O Console Serial Linux

Quem já foi usuário de distribuições GNU/Linux ditas mais hards (Slackware, Gentoo, ArchLinux etc) provavelmente já é familiarizando com editar a configuração do bootloaders (grub ou Lilo) e alterar parâmetros da linha de comando da inicialização do Kernel. Um desses parâmetros do kernel diz respeito ao [Serial Console](https://www.kernel.org/doc/html/v4.12/admin-guide/serial-console.html) e, por não termos muitas alternativas de baixo custo, praticamente todo dispositivo embarcado possui essa configuração habilitada por padrão.

O console serial é responsável por redirecionar as mensagens do Kernel (que ficariam sendo exibidas em um monitor, por exemplo) para a saída serial. A partir daí é possível plugar um dispositivo serial na outra ponta e, sem muita dificuldade, visualizar e interagir com o console do Linux (shell) em tempo real, em execução no dispositivo ligado. Quando passamos esse parâmetro para o Kernel, é necessário fornecer o dispositivo do console que será utilizado (por exemplo o /dev/console) e o baudrate de transmissão. Caso esses parâmetros não sejam especificados, o Kernel irá utilizar o primeiro dispositivo disponível capaz de fornecer uma interface serial, e o baudrate de 9600n8, que significa uma transmissão de 9600bps (sim, bits por segundo), com 8 bits de dado e sem bit de paridade.

### Identificação da Pinagem

Para se identificar a pinagem do dispositivo temos que primeiramente abri-lo. Dependendo da criticidade do dispositivo que você está analisando, talvez exista técnicas anti-tampering, que detecta a abertura e destrói qualquer informação sensível ou relevante no dispositivo, impossibilitando a análise. Porém, devido ao custo de se empregar isso em larga escala, essas técnicas só são aplicadas em dispositivos críticos que armazenam dados sensíveis, como máquinas de cartão de crédito. Quem sabe em um futuro não muito próximo eu entre mais a fundo nesse assunto. Entretanto, caso o dispositivo analisado seja um roteador doméstico (semelhante ao que tenho utilizado nessa série de tutoriais), é praticamente impossível que existam tais técnicas de proteção implementadas. Então podemos abri-lo sem medo. 

Como eu havia falado, a interface UART geralmente é composta por 4 pinos, então abrir o dispositivo procure por 4 conectores próximos um ao outro. Caso você tenha sorte, esses conectores serão facilmente achados e já com os pinos disponíveis, sem necessitar soldar. Esses pinos são (não respectivamente):

- **VCC**: Uma fonte de alimentação (geralmente 3.3V ou 5V).
- **TX**: Pino de transmissão de dados.
- **RX**: Pino de recepção de dados
- **GND**: Aterramento (ground).

No equipamento que tenho utilizado para escrever essa série de publicações (o _MitraStar DSL-100HN-T1-NV_) esses pinos foram facilmente identificados e, para minha sorte, os mesmos já estava disponível com toda a pinagem disponível, sem existir a necessidade de se soldar os pinos à placa.

Abaixo é possível ter uma visão da placa interna do modem:
![DSL-100HN-T1-NV - Placa Interna]({{ "/assets/images/posts/2018/03/vivo-board.jpg" | absolute_url }})

Reparem que, apesar de existirem 5 conectores próximos um do outro, somente 4 deles possuem pinos para conexão. O quinto conector poderia ser parte da interface UART, porém obviamente testei primeiro os 4 com pinos disponíveis e, como de imaginar, foi suficiente para acessar a interface serial do dispositivo.


#### Detectando a Pinagem

Agora que já temos o candidato a possível interface UART identificada, precisamos detectar quem é quem (**RX**, **TX**, **VCC** e **GND**). Para isso vamos utilizar um multímetro.

Os principais a identificarmos são o **VCC** e o **GND**, pois a ordem do **TX** e do **RX** não corre o risco de danificar o aparelho. Já se invertermos VCC ou GND ou ligarmos alguma coisa errada nesses pinos podemos não só danificar nossa placa, como podemos danificar o dispositivo conversor para USB que utilizaremos ou, até mesmo, a controladora USB da máquina que utilizamos para os testes.


Ai ligar o equipamento, o processo de boot carrega todas as mensagens do Kernel, enviando uma quantidade grande dados para o bit de transmissão **TX**. Caso o multímetro não consiga identificar essa oscilação, talvez o resultado confunda o pino de **TX** com o pino de **VCC**, e isso não seria agradável. Portanto uma alternativa que eu utilizo é esperar o equipamento terminar o processo de boot e então medir a tensão em cada um dos pinos. Dessa forma os pinos que não são o VCC ficarão com uma tensão baixa (ou zero) e o VCC fornecerá uma tensão contínua de 3.3V ou 5V.

Tendo identificado o pino de VCC, não existe mais muito risco em confundir os demais pinos. Com isso podemos simplesmente utilizar o multímetro para identificar o pino de aterramento(0V), e podemos utilizar tentativa e erro para identificar quem é RX e quem é TX, apesar de que durante o processo de boot recebemos várias informações em **TX** e praticamente nada em **RX**.

### Conectamos os Pinos

O próximo passo após identificar a pinagem UART na placa do dispositivo embarcado é ligá-la corretamente a um computador para que possamos acessar a interface serial. Para isso podemos utilizar um simples conversor USB/Serial, que é super barato mesmo comprando no [Mercado Livre](https://eletronicos.mercadolivre.com.br/pecas-componentes-eletricos/conversor-usb-serial-ttl). Esses conversores possuem preços a partir de R$ 8,00, o que o torna acessível para a maioria das pessoas.

![Conversor USB-TTL]({{ "/assets/images/posts/2018/03/conversor-usb-ttl.png" | absolute_url }})

Para fazer a ligação, iremos utilizar TRÊS dos QUATRO pinos detectados anteriormente. É importante relembrar que NÃO UTILIZAREMOS a pinagem VCC em momento algum para acessar a interface UART do dispositivo. Se por acaso você ligar o VCC da sua placa com o VCC do conversor, lembre-se que ambas estão FORNECENDO corrente e, por isso, o resultado não será agradável. Dito isso, a ligação deve ser feita da seguinte forma:

- Pino de transmissão (TX) do dispositivo ligado no pino de recepção (RX) do conversor
- Pino de recepção (RX) do dispositivo ligado no pino de transmissão (TX) do conversor
- Pino de aterramento (GND) do dispositivo ligado no pino de aterramento (GND) do conversor

Com isso teremos uma ligação semelhante ao demostrado na figura abaixo:

![Conversor USB-TTL]({{ "/assets/images/posts/2018/03/conexao-uart.jpg" | absolute_url }})

### Detecção do Baudrate

Como eu havia comentado, existe um baudrate padrão definido pelo Kernel, porém caso não seja esse o especificado, será necessário descobrir qual é o utilizado.

Não existe uma gama imensa possíveis baudrates utilizados por padrão nos dispositivos. Na verdade, os baudrates geralmente é um dos cinco seguintes: 9600, 38400, 19200, 57600 e 115200. Portanto você pode tentar um por vez ou simplesmente utilizar o script [Baudrate](https://github.com/devttys0/baudrate/), desenvolvida pelo pessoal do [devttys0](http://www.devttys0.com/).

Como a interface UART não possui um clock para sincronizar a transmissão/recepção de dados, os dados (bytes ou caracteres) são identificados a partir do baudrate. Portanto se você fornecer um baudrate incorreto os bits serão agrupados de maneira incorreta e o resultado será um monte de lixo ilegível na tela. Caso seja fornecido um baudrate correto, os bits serão agrupados corretamente e será exibido as mensagens do Kernel para o console.

### Acessando o Console

Tendo tudo verificado e conectado, agora basta acessar a interface fornecida pelo conversor serial e acessar o console do dispositivo embarcado. Esse acesso pode ser feito pela aplicação [minicom](https://en.wikipedia.org/wiki/Minicom) ou, se preferir, pode utilizar a própria ferramenta [GNU Screen](https://www.gnu.org/software/screen/) para acessar tal interface. Ambos são disponíveis em praticamente todas os repositórios oficiais das distribuições Linux. Para amantes do Windows, o acesso pode ser feito pelo [g]old HyperTerminal ou, se preferir, pela ferramenta Putty, mas como fazer isso fica ao encargo e cada um.

Após conectar o screen na interface serial e ligar o dispositivo, caso tudo esteja corretamente configurado, será possível visualizar o processo de boot desde o kernel até a shell Linux. Abaixo é possível visualizar o resultado do boot do _MitraStar DSL-100HN-T1-NV_ através da interface serial:


<asciinema-player src="/assets/cast/DSL-100HN-T1-NV-boot.json" preload="true" rows="25" cols="100" poster="npt:0:11"></asciinema-player>

Só a mensagem de boot acima fornece diversas informações úteis para análise futuras (inclusive um _Segmentation Fault_). Alguns dispositivos já fornecem diretamente uma shell root, outros permitem login utilizando as mesmas credenciais da interface web, alguns possuem senhas hardcoded que podem ser obtidas através da [extração do firmware]({{ site.baseurl }}{% link _posts/2018/01/2018-01-26-iot-hacking-extracao-de-firmware-usando-spi.markdown %}) etc; cada caso é um caso. Mas, como de praxe, isso ficará para postagens futuras.

### Conclusão

Com esta publicação acredito eu que se encerra as etapas fundamentais para análise tanto estática quanto dinâmica de dispositivos embarcados. A partir das próximas publicações teremos algo bem específico do dispositivo em questão e, é claro, iremos cada vez mais descer o nível de profundidade em nossas análises.

Espero que quem está acompanhando a série esteja gostando. E caso tenham alguma dúvida ou sugestão basta utilizar os campos de comentários abaixo ou entrar em contato diretamente comigo pelos meios de contato na barra lateral.

Cheers,
