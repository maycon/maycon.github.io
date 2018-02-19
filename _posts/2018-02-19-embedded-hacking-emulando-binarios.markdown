---
layout: post
title:  "Embedded Hacking: Emulando binários dos firmwares"
date:   2018-02-19
categories: embedded-hacking
tags: embedded-devices hacking-embedded IoT-hacking qemu mips
lang: br
ref: embedded-hacking-emulando-binarios
has-asciinema: true
---

Seguindo a série sobre hacking de dispositivo embarcados, nessa postagem será demonstrado uma forma simples de iniciar a análise do firmware extraído na postagem anterior. Nessa etapa iremos somente fazer uma análise do firmware, sem acesso ao dispositivo propriamente dito. Isso pode ser útil para identificar vulnerabilidades em firmwares que, por algum motivo, não possuímos o hardware para testes. Para isso continuaremos utilizando como teste o formware dispositivo da MITRASTAR, o *DSL-100HN-T1-NV*, [extraído diretamente da memória flash]({% post_url 2018-01-26-iot-hacking-extracao-de-firmware-usando-spi %}) do aparelho.

## O dump do Firmware

Na [postagem anterior]({% post_url 2018-01-26-iot-hacking-extracao-de-firmware-usando-spi %}) dessa série foi explicado quais elementos podem ser encontrados no firmware de um dispositivo embarcado. No firmware que extraímos, é possível identificar e extrair todas esses elementos separadamente e, então, analisá-los. Para isso vamos utilizar uma ferramente já bem conhecido pelo pessoal que faz engenharia reversa de firmware e perícia forense, o [binwalk](https://github.com/ReFirmLabs/binwalk).

## Introdução ao Binwalk

O [binwalk](https://github.com/ReFirmLabs/binwalk) é uma ferramente utilizada para análise, extração e engenharia reversa de firmwares. Ela possui dezenas de funcionalidades, desde a extração de firmware, disassembly de código (utilizando o [capstone](http://www.capstone-engine.org/)) e análise de entropia, muito útil para identificar se todo ou parte de um arquivo está criptografado ou compactado. 

Basicamente quando queremos saber o tipo de um arquivo desconhecido nós utilizamos o comando `file` passando o arquivo como argumento. Esse comando possui uma base de assinatura (aka `magic numbers`) de binários e utiliza essa base de assinatura para verificar o tipo do arquivo. Porém o comando file faz o match da assinatura no começo do arquivo, diferente da ferramenta binwalk.

Além de ter uma base de assinatura mais específica e complexa, o binwalk percorre todo o binário procurando por formatos conhecidos, como bootloaders, sistemas de arquivos, imagens compactadas etc. Além disso, é possível extrair esse conteúdo (ao invés de fazer na mão com o comando `dd`) e, caso necessário, fazer esse processo de extração de maneira recursiva.

Por exemplo, supomos que uma imagem de um firmware possui um sistema de arquivos compactado, e nesse sistema de arquivos possui um arquivo compactado com tar.gz. O binwalk com os argumentos certos iria identificar e extrair o sistema de arquivos principal e, então, identificar e extrair o arquivo tar.gz compactado.

### Instalando

Na minha distribuição (ArchLinux) o binwalk já está disponível por padrão no repositório _community_. Portanto é possível instalá-lo em um simples comando:
<asciinema-player src="/assets/cast/binwalk-install.json" rows="20" preload="true" poster="npt:0:11"></asciinema-player>

Caso esteja utilizando alguma outra distribuição, verifique nos repositórios oficiais se a ferramenta está disponível. Caso contrário, a instalação a partir do github seria a melhor alternativa.

Abaixo temos a lista completa de opções disponíveis na ferramenta:
```
Binwalk v2.1.1
Craig Heffner, http://www.binwalk.org

Usage: binwalk [OPTIONS] [FILE1] [FILE2] [FILE3] ...

Disassembly Scan Options:
    -Y, --disasm                 Identify the CPU architecture of a file using the capstone disassembler
    -T, --minsn=<int>            Minimum number of consecutive instructions to be considered valid (default: 500)
    -k, --continue               Don't stop at the first match

Signature Scan Options:
    -B, --signature              Scan target file(s) for common file signatures
    -R, --raw=<str>              Scan target file(s) for the specified sequence of bytes
    -A, --opcodes                Scan target file(s) for common executable opcode signatures
    -m, --magic=<file>           Specify a custom magic file to use
    -b, --dumb                   Disable smart signature keywords
    -I, --invalid                Show results marked as invalid
    -x, --exclude=<str>          Exclude results that match <str>
    -y, --include=<str>          Only show results that match <str>

Extraction Options:
    -e, --extract                Automatically extract known file types
    -D, --dd=<type:ext:cmd>      Extract <type> signatures, give the files an extension of <ext>, and execute <cmd>
    -M, --matryoshka             Recursively scan extracted files
    -d, --depth=<int>            Limit matryoshka recursion depth (default: 8 levels deep)
    -C, --directory=<str>        Extract files/folders to a custom directory (default: current working directory)
    -j, --size=<int>             Limit the size of each extracted file
    -n, --count=<int>            Limit the number of extracted files
    -r, --rm                     Delete carved files after extraction
    -z, --carve                  Carve data from files, but don't execute extraction utilities

Entropy Analysis Options:
    -E, --entropy                Calculate file entropy
    -F, --fast                   Use faster, but less detailed, entropy analysis
    -J, --save                   Save plot as a PNG
    -Q, --nlegend                Omit the legend from the entropy plot graph
    -N, --nplot                  Do not generate an entropy plot graph
    -H, --high=<float>           Set the rising edge entropy trigger threshold (default: 0.95)
    -L, --low=<float>            Set the falling edge entropy trigger threshold (default: 0.85)

Binary Diffing Options:
    -W, --hexdump                Perform a hexdump / diff of a file or files
    -G, --green                  Only show lines containing bytes that are the same among all files
    -i, --red                    Only show lines containing bytes that are different among all files
    -U, --blue                   Only show lines containing bytes that are different among some files
    -w, --terse                  Diff all files, but only display a hex dump of the first file

Raw Compression Options:
    -X, --deflate                Scan for raw deflate compression streams
    -Z, --lzma                   Scan for raw LZMA compression streams
    -P, --partial                Perform a superficial, but faster, scan
    -S, --stop                   Stop after the first result

General Options:
    -l, --length=<int>           Number of bytes to scan
    -o, --offset=<int>           Start scan at this file offset
    -O, --base=<int>             Add a base address to all printed offsets
    -K, --block=<int>            Set file block size
    -g, --swap=<int>             Reverse every n bytes before scanning
    -f, --log=<file>             Log results to file
    -c, --csv                    Log results to file in CSV format
    -t, --term                   Format output to fit the terminal window
    -q, --quiet                  Suppress output to stdout
    -v, --verbose                Enable verbose output
    -h, --help                   Show help output
    -a, --finclude=<str>         Only scan files whose names match this regex
    -p, --fexclude=<str>         Do not scan files whose names match this regex
    -s, --status=<int>           Enable the status server on the specified port

```


## Extração do firmware

Para extrair o sistema de arquivos do firmware do nosso device, iremos executar o binwalk com o argumento -e (de `extract`) no dump da memória flash, extraída por SPI na publicação anterior. Caso você esteja analisando algum outro firmware obtido por outro meio (ex: página de download do fabricante), o procedimento seria o mesmo.

Abaixo é possível ver o processo de extração e o resultado obtido:
<asciinema-player src="/assets/cast/binwalk-extract.json" rows="20" preload="true" poster="npt:0:22"></asciinema-player>

A imagem do firmware contém alguns elementos. Porém o que nos interessa a princípio é o sistema de arquivos Squashfs. O binwalk com o arquivo `-e` já extrai e monta o sistema de arquivos, e ao acessar a pasta destino foi possível ver os arquivos do dispositivo. Nessa etapa já é possível comectar a procurar por alguns ítens interessantes, como por exemplo:
- Serviços inicializando
- Usuários não documentados
- Chaves privadas ou certificados digitais
- Alguma senha hardcoded de autenticação diferente das fornecidas na documentação
- Etc.

## Execução de Binários

Uma coisa muito interessante para se fazer é executar os binários disponíveis no firmware para uma análise dinâmica prévia. Isso pode ser útil para identificar vulnerabilidades e/ou simplesmente analisar o funcionamento de alguma aplicação legacy disponível no firmware.

Para a execução de binários, temos que identificar a arquitetura do dispositivo. Geralmente a arquitetura é MIPS ou ARM, mas para tira a dúvida basta executar o comando `file` em algum dos binários disponíveis no sistema de arquivos do firware.

Abaixo o comando `file` comprova que estamos trabalhando com um dispositivo com arquitetura MIPS:

<asciinema-player src="/assets/cast/file-binary.json" rows="10" preload="true" poster="npt:0:8"></asciinema-player>

Para executar os binários, vamos utilizar uma combinação do `chroot` + `qemu`. O chroot irá alterar o root do sistema para o root do sistema de arquivos do firmware. Dessa forma todas as bibliotecas (também MIPS) e arquivos de configuração estarão disponíveis. Já o qemu irá executar o binário propriamente dito. Porém para isto o qemu deve ser compilado de maneira estática, para não depender de qualquer biblioteca que esteja fora do ambiente chroot. No archlinux existe um pacote AUR chamado [qemu-user-static](https://aur.archlinux.org/packages/qemu-user-static/), que contém o QEMU compilado de maneira estática para diversas arquiteturas diferentes.

Para executar os binários MIPS do firmware primeiramente copiamos o binário estático do qemu para arquitetura MIPS (no caso `qemu-mips-static`) para dentro do squashfs-root e, então, executamos o binário copiado dentro de um ambiente de chroot.

O vídeo abaixo demostra a execução do binário MIPS de listagem de arquivos (ls):
<asciinema-player src="/assets/cast/qemu-embedded-binary.json" rows="20" preload="true" poster="npt:0:2"></asciinema-player>

## Conclusão

Apesar dessa publicação ter sido razoavelmente simples, acredito que a utilização do `chroot` + `qemu` para análise dinâmica de binários seja extremamente útil. Porém, infelizmente, nem todos os binário executam com sucesso através desse método. Isso ocorre pelo motivo óbvio que o qemu não emula o hardware completo do nosso dispositivo embarcado. Entretanto a maioria das dificuldades conseguem ser contornadas utilizando alguns artifícios como uma melhoria no setup do ambiente (ex: montar o `/proc` e o `/dev`), seja mudando algum arquivo de configuração, seja rodando o binário com LD_PRELOAD para burlar algumas chamadas específicas de biblioteca (ex: acesso a `nvram`) ou, no pior caso, fazendo um patch no binário em execução.

Recomendo aos que estão acompanhando a série a praticarem com algum dispositivo que possuem ou a escolherem algum firmware na internet para seguir o processo. Caso tenham qualquer dúvida ou comentário basta entrar em contato ou utilizar os campos de comentário no final da página.

Cheers,

Maycon Vitali<br />
Hack N' Roll