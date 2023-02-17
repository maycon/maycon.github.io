---
layout: post
title:  "Serial Fishing"
date:   2009-03-03
categories: reverse-engineering
tags: reverse-engineering assembly serial-phishing
lang: br
ref: serial-phishing
has-asciinema: false
---
Estava procurando rever algumas coisas de engenharia reversa e fui ‘brincar’ com serial Fishing. Serial fishing é possível quando a aplicação, ao tentar verificar se o serial digitado foi fornecido corretamente, gera o serial original para fazer uma comparação. Vamos ver como podemos tirar proveito disto.

Como o serial gerado pela aplicação esta residente na memória, basta abrir o processo e pegar ele bonitinho na memória. A parte mais emocionante é encontrar o serial na memória (isto se ele estiver lá). Para fins de uso pessoal estudo, achando tendo o serial na memória já basta, porém como bons nerds atoas que somos, nós escrevemos um código que pegue e exiba o serial na tela.

O programa que estava testar é o jogo Xadrez Master na versão 5.8.6.0, ao abri-lo no OllyDbg e digitar qualquer nome e serial, é exibida uma mensagem informando que o serial está inválida. Sem fechar a janela, voltei para o Olly, dei pause no processo e então pude analisar o seguinte na memória:

```
CPU Dump:
0012EA48  |00F2F1A8  ASCII "Maycon Maia Vitali (0ut0fBound)"
0012EA4C  |00F36868  ASCII "XAD-BXLT4-TLKXJ"
0012EA50  |00F1C8A4  ASCII "XAD"
0012EA54  |00F2F18C  ASCII "AABBCCDDEEFF"
0012EA58  |00EDCB64  ASCII "AAB"
0012EA5C  |00F1CEAC  ASCII "Maycon Maia Vitali (0ut0fBound)"
0012EA60  |00F2F170  ASCII "XAD-BXLT4-TLKXJ"
0012EA64  |00F1CE90  ASCII "AABBCCDDEEFF"
0012EA68  |00ED1D64  ASCII "AABBCCDDEEFF"
0012EA6C  |00ED1D80  ASCII "AABBCCDDEEFF"
0012EA70  |00F1CE74  ASCII "AABBCCDDEEFF"
0012EA74  |00ED1D0C  ASCII "Maycon Maia Vitali (0ut0fBound)"
0012EA78  |00ED1D38  ASCII "Maycon Maia Vitali (0ut0fBound)"
```

Estes dados foram obtidos da pilha, e podemos notar o serial `XAD-BXLT4-TLKXJ` para o nome `Maycon Maia Vitali (0ut0fBound)` em diversos endereços diferentes. Vamos pegar só um exemplo:

```
0012EA48  |00F2F1A8  ASCII "Maycon Maia Vitali (0ut0fBound)"
0012EA4C  |00F36868  ASCII "XAD-BXLT4-TLKXJ"
```

Neste caso, nos endereços `0012EA48` e `0012EA4C` temos respectivamente os endereços de memória onde estão o nome e serial válidos. Portanto nosso “Serial Fisher” deve obter antes os endereços das strings na memória para depois le-las em sí.

Sem muitas firulas, segue um código-rápido que fiz para fazer _Serial Fishing_ desse software. Lembrando que é necessário digitar um serial qualquer e pressionar o botão de registrar e, sem fechar a caixa da mensagem informando serial inválido, rodar o "fisher":

```cpp
/**
 * Serial fisher para RKSoft Xadrez Master 5.8.6.0
 * Por Maycon M. Vitali (0ut0fBound)
 *
 * ATENÇÃO: Este código foi criado para fins de estudo, caso deseje
 * obter o software é necessário que se page a licensa estipulada
 * pelo fabricante.
 */
 
#include <stdio.h>
#include <string.h>
#include <windows.h>
 
#define NAME_PTR_ADDR   0x0012EA48
#define SERIAL_PTR_ADDR 0x0012EA4C
 
#define TITULO_JANELA "Xadrez Master"
 
#define MAX_SIZE 100
 
int main()
{
    HWND cmeHandle;                // Handle da janela do processo
    DWORD cmePid;                  // PID do processo crackme
    HANDLE cmdProcHandle;          // Handle do processo ( depois de aberto )
 
    long lngAddrNome, lngAddrSerial;
    char strNome[MAX_SIZE], strSerial[MAX_SIZE];
 
    char strMensagem[MAX_SIZE*2 + 20];
 
    // Procuramos pela janela do crackme
    if (!(cmeHandle = FindWindow(0, TITULO_JANELA)))
    {
        MessageBoxA(
            0,
            "Execute o xadrez.exe e tente registrar com qualquer serial antes.",
            "Serial Fishing",
            0
        );
        exit(-1);
    }
 
    // Pega e abre processo
    GetWindowThreadProcessId(cmeHandle, &cmePid);
    cmdProcHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, cmePid);
 
    // Obtém os endereços das informações na pilha
    ReadProcessMemory(
      cmdProcHandle, (void *)NAME_PTR_ADDR, &lngAddrNome, sizeof(lngAddrNome), NULL
    );
    ReadProcessMemory(
      cmdProcHandle, (void *)SERIAL_PTR_ADDR, &lngAddrSerial, sizeof(lngAddrSerial), NULL
    );
 
    // Pega os valores nos endereços obtidos
    ReadProcessMemory(
      cmdProcHandle, (const void *)lngAddrNome, strNome, sizeof(strNome) - 1, NULL
    );
    ReadProcessMemory(
      cmdProcHandle, (const void *)lngAddrSerial, strSerial, sizeof(strSerial) - 1, NULL
    );
 
    snprintf (
      strMensagem, sizeof(strMensagem) - 1, "Nome: %s\nSerial: %s", strNome, strSerial
    );
 
    MessageBoxA(0, strMensagem, "Enjoy!", 0);
 
    printf ("Nome ...: %s\n", strNome);
    printf ("Serial .: %s\n", strSerial);
 
    return 0;
}
```

Como dito no comentário, não forneci esse código com fins de prejudicar a equipe que desenvolve a ferramenta. Tentarei futuramente disponibilizar análise de malwares e afins para não ter problemas futuros.