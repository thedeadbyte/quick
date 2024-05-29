---
layout: post
pinned: true
title: 'Article Example'
description: 'Vivamus gravida efficitur orci facilisis interdum. Donec consectetur convallis dapibus. Pellentesque vestibulum magna sit amet porta porta. Duis et augue sit amet eros ullamcorper porta non sit amet mauris. Etiam rutrum, nisl laoreet mattis condimentum, sem neque congue sapien, in feugiat massa tellus eget turpis. Morbi ligula lectus, iaculis id tristique luctus, pharetra finibus ante.'
icon: fishbone.gif
tags: [example, article, fancy]
banner: /assets/posts/quick-theme-markdown/banner.png
author: 'Ricardo C.'
author_url: 'https://twitter.com/astreuw'
---

# Introdução

O mundo da cibersegurança evolui a cada dia, diversas vulnerabilidades e mecanismos de defesa surgem constantemente. Como costumam dizer, é uma “briga de gato e rato”, onde por um lado pesquisadores buscam proteger sistemas, do outro "*malfeitores*" buscam por meios de quebrá-los.

Uma área mais específica da cibersegurança na qual essa descrição se encaixa perfeitamente é a análise e desenvolvimento de malwares, isso pois para que hoje se tenha meios de defesa contra um determinado malware, foi um dia necessário efetuar a análise sobre o comportamento e funcionamento daquele.

Entretanto, assim como existem mecanismos de defesa para sistemas, malwares mundo afora criaram técnicas para se "defender" dos antivírus, ou melhor, dos analistas que buscarão entender seu funcionamento. Quais são esses meios, como e por quais motivos malwares modernos buscam se defender dos analistas?

# Sobre os Mecanismos de Defesa

Como dissertado nos artigos [Técnicas de Ofuscação, pt. 1 e 2](https://inferi.zip/paper/tecnicas-de-ofuscacao-em-c-pt-1), uma das formas de um malware esconder suas intenções é através da ofuscação, seja de suas estruturas internas como funções e instruções, quanto dos dados que esse busca em um sistema. Entretanto, apesar dessa técnica ser relativamente eficaz contra mecanismos de defesa baseados em assinatura (i.e antivírus), para soluções avançadas como EDRs e análises dinâmicas, está longe de ser o suficiente.

## Análise Dinâmica

Hoje, uma das formas mais avançadas de detecção se dá através da análise dinâmica, geralmente feita de forma automatizada, mas nem sempre limitada a isso. Nessa técnica a aplicação é executada primeiramente em um ambiente controlado (geralmente uma VM ou sandbox) e monitorado, sendo suas ações observadas e classificadas como maliciosas ou não, dependendo da interação com os recursos do sistema. Hoje, soluções avançadas como EDRs efetuam a análise de forma automatizada, porém em casos mais severos, um analista de malware é o encarregado por essa tarefa. Nesse cenário, o analista busca compreender não só estaticamente a estrutura da aplicação como bibliotecas usadas, como também seu funcionamento em tempo execução, onde faz-se o uso geralmente de debuggers.

# Debuggers

```js
var undefined,
    xui,
    window     = this,
    string     = new String('string'),
    document   = window.document,
    simpleExpr = /^#?([\w-]+)$/,
```

```rb
desc "Edit a post (defaults to most recent)"
task :edit_post, :title do |t, args|
  args.with_defaults(:title => false)
  posts = Dir.glob("#{source_dir}/#{posts_dir}/*.*")
  post = (args.title) ? post = posts.keep_if {|post| post =~ /#{args.title}/}.last : posts.last
  if post
    puts "Opening #{post} with #{editor}..."
    system "#{ENV['EDITOR']} #{post} &"
  else
    puts "No posts were found with \"#{args.title}\" in the title."
  end
end
```


Os debuggers, também chamados de depuradores, são ferramentas extremamente poderosas que em princípio, servem para entender o funcionamento de um determinado programa, buscar por erros como memory leak, etc. Através dessas ferramentas, um analista possui quase que o controle total da aplicação, podendo entender seu funcionamento em nível de código (assembly), modificar o fluxo de execução, entender os recursos do sistema em uso, além é claro de um dos recursos mais importantes, definir breakpoints.

## Breakpoints

Os breakpoints são um dos recursos mais úteis para um analista, sendo basicamente indicações de que a execução do programa deve ser pausada em uma determinada instrução ou endereço de memória. Com essa pausa, o analista pode observar não só as instruções seguintes, como também modificar o fluxo de execução, inspecionar registradores e entender a memória da aplicação naquele exato momento. Majoritariamente existem dois tipos de breakpoints, os hardware breakpoints e software breakpoints, foquemos no último por agora.

Um debugger é capaz de definir um software breakpoint em um programa modificando uma de suas instruções para `int 0x3`, que possui o opcode `0xCC`. Essa instrução quando executada, emite um sinal (exceção) do tipo `BREAKPOINT_EXCEPTION`, a execução da aplicação é pausada e o controle passado ao debugger até que esse sinal seja tratado pelo mesmo. Quando tratado, a execução é retomada e a aplicação continua normalmente, podendo seguir um caminho diferente dependendo das modificações efetuadas pelo analista.

# Anti-Debugging

Manter o funcionamento do código obscuro é imprescíndível para um malware bem escrito, isso pois ao esconder suas intenções, classificá-lo como malicioso ou não se torna uma tarefa complicada. A técnica que abordarei, chamada de anti-debugging, consiste basicamente no uso de recursos do sistema ou interações com certas aplicações como debuggers para evitar o entendimento da aplicação. Técnicas de anti-debugging são comum em outros cenários também, como na indústria de jogos eletrônicos entre outros.

## IsDebuggerPresent()

Uma abordagem bastante simples e intuitiva para evitar o debugging de um programa é utilizar de chamadas de API. A API do Windows por exemplo, disponibiliza uma função chamada `IsDebuggerPresent()` para checar a presença de um debugger atrelado ao processo. Essa função possui o seguinte protótipo: `BOOL IsDebuggerPresent();`

Abaixo um exemplo de uso:

```c
#include <stdio.h>
#include <Windows.h>

int main(void) {
	if(IsDebuggerPresent()) {
		puts("Debugger detected!");
		return 1;
	}
	puts("You won!");
	return 0;
}

```

No código acima, caso a função retorne 1, indica a presença de um debugger, logo o bloco do if é executado, finalizando a execução do programa. Caso contrário, o bloco é ignorado e a mensagem “You won!” é exibida.

Infelizmente, essa é uma das técnicas mais fáceis de contornadas, já que debuggers modernos são capazes de modificar a flag da PEB consultada por essa função.

## INT 3

Caso tenha feito uma leitura focada até aqui, deve lembrar que INT 3 `0xCC` é a instrução utilizada por debuggers para indicar um software breakpoint. Assim como debuggers podem injetar essa instrução para isso, malwares também podem fazê-lo visando o anti-debugging.

<img src="/assets/img/tdb0-1.png">

*Na imagem acima uma demonstração da instrução sendo interpretada pelo debugger, mesmo não tendo sido inserida por ele.*

Acima, apesar do breakpoint ter sido inserido pelo programador com `__asm int 3;`, foi interpretado pelo debugger como um breakpoint válido. Dessa forma, a exceção `BREAKPOINT_EXCEPTION` é emitida ao debugger para que seja tratada.

Para tornar essa técnica útil, é preciso validar se a exceção foi tratada externamente, o que indicaria nesse caso, a presença de um debugger, mas como?

## MSVC Structured Exception Handling

Através de uma extensão do MSVC chamada Structured Exception Handling, podemos tratar certas exceções como acessos de memória inválidos, divisões por zero e, a pérola do dia, exceções de breakpoints.

A sintaxe SEH é bastante simples:

```c
__try {
	// Bloco a ser executado e testado
} __except(/* Exceção a ser tratada */) {
	// Bloco que trata a exceção
}

```

No nosso caso, no bloco `__try` inserimos o código com a instrução do breakpoint `int 3`, já em `__except`, indicamos para que o controle seja mantido no exception handler através da macro `EXCEPTION_EXECUTE_HANDLER`, o que nesse caso indica a não presença de um debugger.

Pode parecer um pouco confuso, mas colocando em ordem, temos basicamente o seguinte:

1. O programa é executado
2. Caso haja um debugger e esse encontre a instrução 0xCC (breakpoint), a exceção é emitida ao debugger, que trata e retoma o controle para aplicação. Dessa forma o bloco `__try` é executado normalmente, indicando a presença de um debugger.
3. Caso não haja um debugger, a instrução 0xCC ainda sim é emitida, porém o bloco `__except` é o encarregado de tratá-la, indicando então a não presença de um debugger.

```c
#include <stdio.h>
#include <Windows.h>

int main(void) {
    __try {
        __asm int 3;
        puts("Debugger detected!");
    }
    __except (EXCEPTION_EXECUTE_HANDLER) {
        puts("No debugger detected!");
    }
    return 0;
}

```

Execução sem debugger:

<img src="/assets/img/tdb0-2.png">

*Na imagem acima, a exceção é tratada internamente pela aplicação, logo em teoria não há debugger.*

Execução com debugger:

<img src="/assets/img/tdb0-3.png">

*Na imagem acima a exceção é emitida ao debugger, que a trata e retoma o controle para a aplicação.*

## INT 0x2D

Outra instrução que funciona de uma forma semelhante é `int 0x2d`, emitindo também a exceção `EXCEPTION_BREAKPOINT` porém incrementando EIP e apontando para a próxima instrução. Essa instrução é interessante devido ao seu comportamento com EIP, já que ao apontar para a próxima instrução pode ser perigoso para o debugger, além disso, com ela podemos em teoria escolher qual exceção tratar no exception handler ao especificar uma instrução para tal.

Abaixo um breve exemplo onde é tratada a exceção `EXCEPTION_INT_DIVIDE_BY_ZERO` (emitida por `i = i / 0`) ao invés de `BREAKPOINT_EXCEPTION`:

```c
#include <stdio.h>
#include <Windows.h>

int main(void) {
    int i = 1;
    __try {
        __asm int 2dh;
        i = i / 0; // EXCEPTION_INT_DIVIDE_BY_ZERO
        puts("Debugger detected! BREAKPOINT_EXCEPTION");
    }
    __except (EXCEPTION_EXECUTE_HANDLER) {
        if (GetExceptionCode() == EXCEPTION_INT_DIVIDE_BY_ZERO) {
            puts("Debugger detected! EXCEPTION_INT_DIVIDE_BY_ZERO");
            exit(1);
        }
        puts("No debugger detected!");
    }
    return 0;
}

```

<img src="/assets/img/tdb0-4.png">

# Conclusão

Anti-debugging é uma área vasta e extremamente complexa, proporcionando muito o que aprender e explorar. Conhecer não só o sistema operacional mas como também o funcionamento dessas aplicações como debuggers é essencial para técnicas desse tipo, nos próximos artigos comentarei sobre outras baseadas em recursos do sistema operacional e timing. Por hora é isso, até a próxima!
