---
layout: post
title:  "IoT Hacking: Extração de Firmware usando SPI"
date:   2018-01-26 00:00:00 +0000
categories: embedded-hacking
tags: embedded-devices hacking-embedded IoT-hacking
lang: br
ref: firmware-extraction-spi
---

O objetivo desse post é dar inicio a uma série de publicações sobre hacking de embedded devices. Apesar de eu não ser nenhum mestre nisso, acredito que já tenha feito experimentos o suficiente para ajudar aos iniciantes que tiveram pouco ou nenhum contato com hacking de dispositivos embarcados. E, como de praxe, uma das primeiras partes geralmente envolve a obtenção do firmware do dispositivo. E é sobre isso que iremos falar nesse post. Caso identifiquem algum erro basta entrar em contato comigo pelos comentários no final do post ou por qualquer um dos contatos disponíveis na barra lateral do site.

## O que é um firmware?

Eu espero não ter que delongar muito sobre isso. Mas só para deixar claro, vamos ver o que é, e o que podemos ter em um firmware de um dispositivos embarcado.

O firmware (também conhecido como software embarcado -- _embedded software_) é parte do sistema embarcado responsável por armazenar o software que irá executar e controlar todo o sistema. Presente em diversos dispositivos, os firmwares podem ser encontrados em processadores, em microcontroladores, na BIOS do seu computador, em impressoras e nos nossos tão preciosos modems domésticos. Basicamente qualquer dispositivo que não foi feito 100% em hardware precisa de um firmware para funcionar. E esse pequeno "programa" será nosso alvo de estudo de engenharia reversa para descoberta de vulnerabilidades.

Quando fazemos o update do firmware de um dispositivo, podemos ter alguma funcionalidade completamente transparente para esse processo, onde simplesmente clicamos em algum botão e o firmware se atualiza sozinho. Porém, algumas vezes é necessários baixar um arquivo no site do fabricante/fornecedor e fazer o update manualmente através de uma ferramenta de auxílio. Essa ferramenta pode variar desde uma aplicação de celular até uma funcionalidade de atualização da própria versão atual do firmware.

Mas o que temos nesse arquivo? Pois bem, abaixo seguem algumas coisas que podemos ter dentro desses arquivos. Mas, lembre-se que isso varia de firmware para firmware, ou seja, um firmware em questão pode ter nenhum ou todas as opções abaixo:

#### Bootloader

Esse cara é o startup do dispositivo embarcado. Geralmente ele que é responsável por carregar na inicialização do aparelho e contém aquelas funcionalidades de "segurar algumas teclas" e entrar em algum menu de administração (ex: recovery). Basicamente, se você corromper seu dispositivo, mas o bootloader ainda estiver intacto, as chances de você recuperar seu aparelho é quase que certa. Para se ter uma ideia mais abrangente, o bootloader em um PC é chamado após o processo de teste de inicialização (power-on self-test - POST) e da inicialização da BIOS. Quem já utilizou o *GR*and *U*nifield *B*ootloader (grub) ou o *LI*nux *LO*ader (lilo) sabe exatamente do que eu estou falando.

Existem alguns tipos diferentes de Bootloader para embarcados, e eles possuem funcionalidades que podem danificar seu dispositivo, como a leitura e escrita diretamente na memória flash do aparelho (EEPROM). Porém, os dispositivos embarcados possuem, em sua grande maioria, o [Das U-Boot](https://www.denx.de/wiki/U-Boot), um bootloader open source compatível com mais de uma dezena de arquiteturas. Entretanto, como o U-Boot é open source, é comum encontrar versões limitadas (sem funcionalidades) ou modificada dele sendo executada em alguns dispositivos.

Só para fazer uma referência, em minha [publicação sobre Bug Bounty]({% post_url 2018-01-22-talk-is-cheap-show-me-the-money %}), eu descrevo superficialmente como eu consegui danifica um equipamento após executar alguns comandos no bootloader do aparelho de maneira "despreocupada".

É importante ressaltar que muitas vezes a atualização do firmware não atualiza o bootloader do dispositivo, mas somente seu sistemas de arquivos (descrito logo abaixo).

#### Kernel

O kernel é o núcleo do sistema em execução e sim, muitos deles rodam uma versão mínima do kernel do linux para embarcados. Geralmente o kernel é armazenado em uma área específica da memória, que é chamada diretamente pelo bootloader através de algum comando, passando como argumento o endereço de onde está o kernel na memória.

Como o kernel é um Linux para embarcados e ele é chamado pelo bootloader, é possível alterar parâmetros da inicialização e fazer algumas coisas simples, como desativar alguma feature ou enviar o console para uma porta serial (se não for default) de forma que consigamos acessar o console do kernel através de uma interface UART, por exemplo. Se for de interesse, essa comunicação com a interface serial pode ser detalhado em alguma publicação futura. Basta comentar sobre o interesse no campos de comentários no final da página.

#### Sistema de Arquivos

O sistema de arquivos (File System) é responsável por ter tudo do sistema operacional em execução. Ele contém os arquivos de configuração, os binários das aplicações em execução, a aplicação web (se for o caso) etc.

Ao acessar o sistema de arquivos durante uma pequisa por vulnerabilidades, é comum como primeiros passos procurar por usuários não documentados ([backdoor?](http://embeddedsw.net/doc/Embeddedsw_news_Undocumented_backdoor_account_in_dbltek_goip.html)), chaves privadas de criptografia hardcoded ([backdoorr??](https://www.wired.com/2015/12/juniper-networks-hidden-backdoors-show-the-risk-of-government-backdoors/)) ou algum outro mecanismo de autenticação ([backdooorrrr???](http://securityaffairs.co/wordpress/18645/hacking/d-link-backdoor.html)).

#### Memória Protegida

Aqui fica um pouco difícil de explicar. Mas vamos tentar.

O que estou chamando de memória protegida geralmente é mapeada a partir da imagem do firmware como uma partição somente leitura. Geralmente ela contém informações de configuração padrão que são utilizadas no caso do _reset_ do firmware para os padrões de fábrica, por exemplo.

## Obtenção do Firmware

Já sabemos quais as principais parte de um firmware e já podemos ter uma ideia do quão importante é obter o firmware no processo de hacking de algum dispositivo embarcado. Agora vamos ver de maneira geral alguns métodos que podemos utilizar para a obtenção do firmware:

#### Site do fabricante

Esse sem dúvidas é a forma mais simples de se obter o firmware. Basta identificar o fabricante do dispositivo e verificar no site do mesmo se existe alguma versão do firmware disponível para download.

#### Processo de Atualização

Essa forma de obtenção do firmware não a mais fácil mas também não é a das mais difíceis. Ela consiste basicamente em utilizar a _feature_ presente de atualização do firmware e tentar interceptar o tráfego diretamente no _gateway_ de saída, ou fazendo um ataque de [man-in-the-middle](https://pt.wikipedia.org/wiki/Ataque_man-in-the-middle) no aparelho. Algumas vezes será necessário instalar um certificado digital no aparelho (para interceptar HTTPS) ou até mesmo tentar fazer algum _downgrade attack_, como o SSLStrip.

#### Extração através do Bootloader

Lembra-se que eu falei que o bootloader possui funcionalidades para leitura e escrita na memória flash? Então! Geralmente a memória flash é praticamente uma imagem idêntica ao firmware propriamente dito. Com isso, utilizando os comandos de leitura da memória (ex: [md.b - memory display](http://www.denx.de/wiki/view/DULG/UBootCmdGroupMemory#Section_5.9.2.5.)) é possível fazer um dump em hexadecimal da memória flash e, com um simples script em python, converter o hexa para o arquivo original.

#### Leitura diretamente da memória Flash

Apesar de ser uma dos formas mais complexas de se obter o firmware, é a forma que possui assertividade em praticamente 100% dos casos. Ela pode ser considerada difícil por precisar de interação diretamente com o hardware porém, tendo um certo cuidado e o equipamento necessário, ela passa a ser a forma preferida de se obter o firmware de um dispositivo.

Muitos dispositivos possuem para seu armazenamento uma memória flash (EEPROM) convencional. Essa memória geralmente tem 8 ou 16 pinos, e armazenado nela temos tudo o que precisamos. Então só precisamos extrair o conteúdo. Para isso é necessário abrir o aparelho, identificar a memória flash (fabricante e modelo), descobrir a pinagem dela (pela especificação/datasheet) e ler todo conteúdo dela (com o aparelho desligado, é claro).

Vamos ver com mais detalhes como podemos fazer isso utilizando SPI.

## O que é SPI?

O _Serial Peripheral Interface_ (SPI) é uma interface de comunicação serial que permite a comunicação com diversos componentes. Nas memórias flash, essa interface de comunicação possibilita ao microcontrolador ler e gravar informações e, no nosso caso, nos possibilitará a extração completa da memória.

A comunicação serial por SPI se dá basicamente através dos seguintes 4 canais:

- **Clock**: O clock é o pulso que será enviado para a memória com o objetivo de sincronizar e controlar a comunicação.

- **Serial In (SI)**: Esse canal é responsável por **receber** dados de maneira serial (seguindo a frequência do clock)

- **Serial Out (SO)**: Semelhante ao anterior, porém responsável por **enviar** dados de maneira serial.

- **Chip Select (CS)**: Esse sinal é responsável por habilitar o chip propriamente dito. Ele geralmente é ativo em nível lógico baixo, e sua nomenclatura vem como CS# ou como CS com um traço em cima.


## A Extração do Firmware

Nessa parte irei demonstrar na prática como foi possível extrair o firmware do meu modem doméstico da Vivo. No caso irei demonstrar o processo no modem **MITRASTAR DSL-100HN-T1-NV**. Para isso foi utilizado o seguinte software e hardware:

- O modem da Mitrastar propriamente dito
- Um [BusPirate](http://dangerousprototypes.com/docs/Bus_Pirate)
- Pinças de teste (pode utiliza uma [pinça SOIC8](https://lista.mercadolivre.com.br/pin%C3%A7a-soic-8))
- O software [flashrom](https://www.flashrom.org/Flashrom)

O BusPirate é uma ferramenta hacker open-source desenvolvida pela [Dangerous Prototypes](http://dangerousprototypes.com/) e ele possui a capacidade de conversar com dispositivos eletrônicos, permitindo a programação de FPGAs, programação de microcontroladores, acesso à interface JTAG e, como no nosso caso, comunicação com memória flash.

Como o BusPirate é um hardware não muito acessível, gostaria de deixar registrado que é possível fazer os mesmos procedimentos aqui [utilizando um RaspberryPi](https://github.com/bibanon/Coreboot-ThinkPads/wiki/Hardware-Flashing-with-Raspberry-Pi).

Então! Temos nosso modem em mãos:

![Modem MITRASTAR DSL-100HN-T1-NV (VIVO)]({{ "/assets/images/posts/2018/01/modem.png" | absolute_url }})

A primeira coisa a fazer é abrir o modem e acessar sua placa de circuito. Nesse modelo o procedimento é bem simples, bastando desparafusar 4 parafusos na parte de baixo do modem e desencaixar com cuidado as duas partes de plástico, a de cima e a de baixo. Com isso temos acesso à placa do modem:

![Placa interna]({{ "/assets/images/posts/2018/01/MITRASTAR-board.png" | absolute_url }})

Para identificar a memória flash que ele utiliza temos que começar procurando por chips que possuem 4 ou 8 pinos. Analisando esses chips com uma lupa (pra quem é ruim das vistas) e uma boa iluminação, é possível identificar o que tem escrito em cada um deles e, então, perguntar ao google mais detalhes sobre o que ele é.

Procurando sobre cada possível candidato à memória flash visível na placa, foi possível identificar a MX25L12835F da Macronix, com capacidade de 16MB.

![Memória Flash MX25L12835F]({{ "/assets/images/posts/2018/01/MX25L12835F-chip.png" | absolute_url }})

Tendo o modelo da memória flash, sem muita dificuldade foi possível achar a especificação([datasheet](http://datasheet.octopart.com/MX25L12835FZ2I-10G-Macronix-datasheet-14372549.pdf)) da mesma. Essa documentação contém todas as informações referentes ao chip, inclusive sua pinagem, que é o que mais nos interessa:

![Pinagem da Flash MX25L12835F]({{ "/assets/images/posts/2018/01/MX25L12835F-pinout.png" | absolute_url }})

Vamos ver o que temos no nosso chip e como iremos conecta-lo:
- **CS#:** Esse é o _chip select_, e ele é utilizado para ativar ou não o chip. Lembre-se que quando temos esse # no final da identificação significa que nosso pino será ATIVO em nível lógico zero. Então para ativar ele precisa estar em zero. Mas no caso só precisamos de ligá-lo ao pino CS do BusPirate.

- **SI:** Esse pino é o Serial In, e deve ser ligado ao MOSI (_Master Out Slave In_) do BusPirate, pois o nosso BusPirate será o mestre e a memória flash será o escravo.

- **SO:** Semelhante ao pino SI, porém esse é o _Serial Out_ e deverá ser ligado no MISO (_Master In Slave Out_) do BusPirate.

- **SCLK:** O clock (sem mais), ligado no clock do BusPirate.

- **WP#:** Esse pino significa Write Protection e, como o nome diz, ele é responsável por impedir a escrita no chip, habilitado somente a leitura. Porém, como já falado, o símbolo # no final da nomenclatura significa que esse pino é ativado em nível lógico zero.

> É importante ressaltar que não ligar um pino não significa que ele está em nível lógico zero. Na verdade ele fica em um estado chamado _floating_, que não vamos entrar em detalhes aqui. Eu, particularmente, não precisei ligar esse pino. Mas para ativar o **WP#** é recomendado que ele seja ligado no terra (GND) para garantir nível lógico zero e evitar que você escreva acidentalmente na sua memória flash.

- **RESET#:** Não utilizamos esse pino, mas ele serve para dar um hardware reset na memória.

- **VCC:** Nossa alimentação, que de acordo com o datasheet, é de +3V. Ligada ao 3V3 (3.3V) do BusPirate.

- **GND:** Nosso aterramento, ligado ao ground (GND) do BusPirate.

Como já identificamos as pinagens responsáveis pelo SPI, então agora, com o aparelho desligado, temos tudo para conectar o BusPirate diretamente na memória flash. Eu utilizei umas pinças de testes que possuo, mas o mesmo pode ser feito utilizando uma pinça SOIC8:

![BusPirate conectado na memória Flash]({{ "/assets/images/posts/2018/01/buspirate-connected.png" | absolute_url }})

Tendo tudo conectado, ligamos o BusPirate no computador e utilizamos o aplicativo _flashrom_ para testar a comunicação com a memória flash:

```bash
[maycon@DayOfDevil ~]$ flashrom -p buspirate_spi:dev=/dev/buspirate,spispeed=1M
flashrom v0.9.9-r1955 on Linux 4.14.7-1-ARCH (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Calibrating delay loop... OK.
Found Macronix flash chip "MX25L12805D" (16384 kB, SPI) on buspirate_spi.
Found Macronix flash chip "MX25L12835F/MX25L12845E/MX25L12865E" (16384 kB, SPI) on buspirate_spi.
Multiple flash chip definitions match the detected chip(s): "MX25L12805D", "MX25L12835F/MX25L12845E/MX25L12865E"
Please specify which chip definition to use with the -c <chipname> option.
```

Como é possível notar, o flashrom identificou nossa memória flash MX25L12835F com o identificador "MX25L12835F/MX25L12845E/MX25L12865E".

Apesar de ter utilizado o device `/dev/buspirate`, esse device só existe por uma regra que adicionei no udev do meu sistema operacional. Caso esse device não existe no seu sistema, tente os padrões `/dev/ttyUSB0` ou similar. Em caso de dúvidas conecte o BUsPirate na USB da máquina e utilize o comando `dmesg` para identificar se ele foi detectado corretamente e qual dispositivo está sendo associado à ele.

Tendo identificado o dispositivo, agora basta utilizar o `flashrom` para extrair o conteúdo da memória flash para um arquivo:


```bash
[maycon@DayOfDevil ~]$ time flashrom -p buspirate_spi:dev=/dev/buspirate,spispeed=1M -c "MX25L12835F/MX25L12845E/MX25L12865E" -r flash.dump
flashrom v0.9.9-r1955 on Linux 4.14.7-1-ARCH (x86_64)
flashrom is free software, get the source code at https://flashrom.org

Calibrating delay loop... OK.
Found Macronix flash chip "MX25L12835F/MX25L12845E/MX25L12865E" (16384 kB, SPI) on buspirate_spi.
Reading flash... done.

real    44m58.310s
user    0m2.127s
sys     0m3.399s
```

Cronometrei o tempo para deixar claro que, apesar da memória ter somente 16MB de armazenamento, a leitura bit-a-bit dela levou cerca de 45 minutos. Então para memórias maiores (ex: 128MB) é esperado que o tempo de extração seja de horas e horas.

Tendo nosso dump da memória flash, já é possível acessar as partes do firmware e partir para a engenharia reversa e análise de vulnerabilidade:

```bash
[maycon@DayOfDevil ~]$ binwalk flash.dump

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
26112         0x6600          LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 107920 bytes
69600         0x10FE0         LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 42848 bytes
197120        0x30200         LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 3482360 bytes
1372293       0x14F085        Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 6471498 bytes, 1385 inodes, blocksize: 131072 bytes, created: 2016-07-15 11:37:07
8389120       0x800200        LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 3482360 bytes
9564293       0x91F085        Squashfs filesystem, little endian, version 4.0, compression:lzma, size: 6471498 bytes, 1385 inodes, blocksize: 131072 bytes, created: 2016-07-15 11:37:07

```

Espero que tenha gostado do post. Caso tenha gostado deixei seu like e eu poderei avaliar quais tipos de publicação são de maior interesse de vocês. Se tiverem alguma dúvida basta colocar nos comentários ou utilizar qualquer forma de contato disponível no menu lateral.

Hack N' Roll<br />
Maycon Vitali

## Bônus: Oh Really?!?!

![BusPirate conectado na memória Flash]({{ "/assets/images/posts/2018/01/bonus-really.png" | absolute_url }})