---
layout: post
title:  "Análise do Virus TR/ATRAPS.Gen – Parte 1"
date:   2009-04-12
categories: malwares
tags: urban hacking-tricks gold
lang: br
ref: analise-do-virus-tr-atraps-parte-i
has-asciinema: false
---
Hoje cheguei em casa umas 12h e notei que tinha uma mensagem no MSN com o seguinte:

```
Esta foto te miras padre!

http://img456.myspace-imagen.info/img456/my.php?id=MVC-IMAGEN41.jpeg
```

Então pensei: Ou a minha tia está aprendendo espanhol e esta querendo praticar comigo, ou ela esta com um vírus de MSN. Para tirar minhas dúvidas cliquei no link e, como de se esperar, o navegador abriu uma janela para eu efetuar download do arquivo `MVC-IMAGEN41.jpeg.src`. Então o salvei e comecei a fazer a análise dele.

Ao submeter para o [VirusTotal](http://www.virustotal.com/pt/analisis/8d997936bda52f125e4f961dfd6c12b2), notei que dos 37 anti-virus com base de assinaturas registradas, somente 4 o detectaria como vírus. Para maiores detalhes da submissão do vírus acesse VirusTotal

Abri no PEiD para ver o que ele me falava do arquivo, e obtive a seguinte resposta:

![x]({{ "/assets/images/posts/2009/04/figura01.jpg" | absolute_url }})

Pelo PEiD foi possível notar que não existe nenhum tipo de compactação nem criptografia das sessões do binário. Então bastamos abrir o binário para uma análise direta.

Abri o binário no OllyDbg e logo no inicio vemos a seguinte chamada:

![x]({{ "/assets/images/posts/2009/04/figura02.jpg" | absolute_url }})

Inicialmente ele aloca memória na pilha e em seguida move uma constante para o registrador `EAX` e efetua uma chamada. Ao analisarmos a chamada temos o seguinte:

![x]({{ "/assets/images/posts/2009/04/figura03.jpg" | absolute_url }})

Nesta chamada é salvo o parâmetro que está em EAX no registrador EBX em seguida move-se o valor 0 (zero) para a posição `[44064]` que representa o `TLS (Thread-Local Storage)` do processo, utilizada para criar um mecanismo de IPC. Por padrão, às variáveis locais de uma função são únicas para cada thread que execute a função e as variáveis globais e estáticas (static) são compartilhadas por cada processo. Com TLS, utilizando um índice global, é possível prover um dado único para cada thread que o processo pode acessar.

Após zerar o TLS, o processo efetua uma chamada a API `GetModuleHandleA`[1] passando 0 (zero) como parâmetro. A API `GetModuleHandleA()` retorna o handle (ponteiro) para o módulo passado como parâmetro, o módulo no caso seria algum outro processo ou DLL carregado na mesma região de memória do processo atual. Como foi passado como parâmetro o valor 0 (zero) a API retornará o Handle do arquivo que criou o processo atual, no caso o nosso m4ware.

Em seguida o processo salva o resultado nos endereço `DS:[4566C]` e `DS:[4406C]`, então para facilitar o trabalho colocamos um label (rótulo) nesses endereços.

![Figura 04]({{ "/assets/images/posts/2009/04/figura04.jpg" | absolute_url }})

Nesta sub-rotina, vemos um exemplo clássico de anti-debugger. Esse trick consiste é pegar o uptime da máquina em milissegundos (`GetTrickCount`[2]), em seguida efetuar uma pausa (`Sleep`) e pegar novamente o numero de milissegundos. Com isto verifica-se se as diferenças foram compatíveis. No exemplo, o malw4re pega o ms da máquina, da uma pause de 501ms e em seguida pega o ms novamente e então verifica se a diferença foi de 501ms. Então a sub-rotina retorna 1 (um) em `EAX` caso ocorra a diferença (debug detectado) ou 0 (zero) caso não tenha detectado. Após a verificação, o malw4are altera seu fluxo caso tenha sido detectado o debug, executando o salto condicional (endereço `00043616`).

Não tenho detectado o debuger, o malw4are prossegue normalmente e em seguida executa a seguinte rotina passando o valor `6F` (`111` dec.) no registrador `EAX`:

![Figura 05]({{ "/assets/images/posts/2009/04/figura05.jpg" | absolute_url }})

No inicio temos o prelúdio normal de uma rotina (salva pilha e separa nova pilha). Em seguida temos a alocação de uma região da pilha (variáveis locais). Salva-se o valor de `EAX` (parâmetro) em `EBX` e executa um `Sleep` de 50ms. Após a pausa, o m4lware soma Trick do sistema e com o parâmetro passado e executa um salto incondicional para o meio de loop (endereço `000434D4`). O loop executa diversas leituras de mensagens do sistemas. A estrutura de resposta da mensagem fica armazenada na pilha no endereço `SS:[EBP-1C], então o processo verifica pela mensagem de ID 12h para então sair[3].

Ao sair desta rotina, o m4lware passa para a seguinte rotina:

![x]({{ "/assets/images/posts/2009/04/figura06.jpg" | absolute_url }})

Inicialmente temos o prelúdio com a alocação e inicialização (com zeros) das variáveis locais da sub-rotina. É possível notar diversas chamadas a API `LoadLibraryA`[4], que é responsável por carregar uma determinada biblioteca (DLL) dinamicamente, ou seja, sem estar no `IMPORT_TABLE` do binário. Porém, antes de cada chamada, temos a chamada a três sub-rotinas do binário. Analisando superficialmente, é possível notar que a primeira rotina carrega uma mensagem codificada, a segunda decodifica a mensagem e a terceira verifica se teve sucesso ou não. Com isto, é possível verificar que as chamadas as APIs `LoadLibraryA`() importam as bibliotecas `kernel32.dll`, `advapi32.dll` e `ntdll.dll` respectivamente. Em seguira é feito uma chamada a API `GetProcAddress`, que é responsável por, dado um handle de uma biblioteca (retorno da função `LoadLibrary`()) e um nome de API, retorna um ponteiro para a rotina da API. A chamada à `GetProcAddress`() busca e retorna o endereço da própria API `GetProcessAddress` em `Kernel32.dll`, armazenando seu retorno em `DS:[ESI]`.

Após a obtenção do endereço de `GetProceAddres` em `DS:[ESI]`, a rotina efetua diversas rotinas como as seguintes:

![x]({{ "/assets/images/posts/2009/04/figura07.jpg" | absolute_url }})

Com isto, o m4lware carrega, não necessariamente nesta ordem, as seguintes APIs:


| - RegCloseKey | - RegQueryValueExA | - RegOpenKeyA | - EnumResourceNamesA
| - GetModuleHandleA | - GetComputerNameA | - GetUserNameA | - GetFileAttributesA
| - FreeLibrary | - FreeResource | - ExitProcess | - SizeofResource
| - LoadResource | - LockResource | - FindResourceA | - SetThreadContext
| - TerminateProcess | - ZwUnmapViewOfSection | - VirtualAllocEx | - WriteProcessMemory
| - CreateProcessA | - GetThreadContext | - ReadProcessMemory | - SetThreadContext
| - ResumeThread | - VirtualProtectEx


No final, esta rotina libera o handle das biblitecas `kernel32.dll`, `advapi.dll` e `ntdll.dll` carregados no inicio.

Bem, esta foi a primeira parte da análise do m4lware TR/ATRAPS.Gen, até então não encontramos nada de muito complicado nem ameaçador. Nos post futuros continuarei com a análise e talvez tenha algo de mais interessante.

### Referências
1. [GetModuleHandle Function](http://msdn.microsoft.com/en-us/library/ms683199(VS.85).aspx)
2. [Thread-Local Storage](http://msdn.microsoft.com/en-us/library/ms686749.aspx)
3. [MSG Structure](http://msdn.microsoft.com/en-us/library/ms644958(VS.85).aspx)
4. [LoadLibrary Function](http://msdn.microsoft.com/en-us/library/ms684175(VS.85).aspx)