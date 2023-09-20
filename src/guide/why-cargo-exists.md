# Por que o Cargo Existe

## Preliminares

No Rust, como voce deve saber, uma biblioteca ou programa executável é chamado de
 [*crate*][def-crate]. As crates são compilada usando o compilador Rust, `rustc`.
 Ao começar com Rust, o primeiro código fonte que a maioria das pessoas encontra
 é o venerável programa “hello world”, que eles compilam invocando o `rustc` diretamente:

```console
$ rustc hello.rs
$ ./hello
Hello, world!
```

Observe que o comando acima exigiu que nos especificássemos o nome do arquivo
 explicitamente. Se fossemos usar `rustc` diretamente para compilar um programa diferente,
 uma invocação de linha de comando diferente seria necessária.  Se precisarmos especificar
 qualquer sinalizador específico do compilador ou incluir dependências externas, então o
 comando necessário seria ainda mais específico (e elaborado).

Além disso, a maiorias dos programas não triviais provavelmente terá dependências
 em bibliotecas externas e, portando, também dependerão transitivamente das *suas*
 dependências. Obter a versões corretas de todas dependências necessária e
 mantê-las atualizadas seria trabalhoso e protenso a erros se fosse feito manualmente.

Em vez de trabalhar com crates e `rustc`, nos podemos evitar o tédio manual
 envolvido na realização das tarefas acima, introduzindo uma abstração de nível
 superior ["*pacotes*"][def-package] e usando um [*gerenciador de pacotes*][def-package-manager].

## Entrar: Cargo

*Cargo* é o gerenciador de pacotes do Rust. É uma ferramenta que permite que
 [pacotes][def-package] Rust declarem suas várias dependências e garantem que
 você sempre obterá uma compilação repetível.

Para realizar esse objetivo, o Cargo faz quatro coisas:

* Introduz dois arquivos de metadados com vários pedaços de informação de pacote.
* Busca e constrói as dependências do seu pacote.
* Invoca `rustc` ou outras de ferramentas de compilação com os parâmetros corretos
 para construir seu pacote.
* Introduz convenções para facilitar o trabalho com pacotes Rust.

Em grande parte, o Cargo normaliza os comandos necessários para construir um determinado
programa ou biblioteca; este é um aspecto das convenções mencionadas acima.Como 
mostraremos mais tarde, o mesmo comando pode ser usado para construir diferentes 
[*artefatos*][def-artifact], independentemente de seus nomes. Ao invés de invocar
`rustc` diretamente, podemos invocar algo genérico como `cargo build` 
e deixar o Cargo se encarregar de construir a invocação correta do `rustc`.
Além disso, o Cargo buscará automaticamente um 
[*registro*][def-registry] de qualquer dependências que definimos para nosso artefato,
e providenciar para que eles sejam incorporados em nossa construção conforme necessário.

É apenas uma pequena exageração dizer que, uma vez que você saiba como construir um projeto baseado no Cargo, você sabe como construir *todos* eles.

[def-artifact]:         ../appendix/glossary.md#artifact         '"artifact" (glossary entry)'
[def-crate]:            ../appendix/glossary.md#crate            '"crate" (glossary entry)'
[def-package]:          ../appendix/glossary.md#package          '"package" (glossary entry)'
[def-package-manager]:  ../appendix/glossary.md#package-manager  '"package manager" (glossary entry)'
[def-registry]:         ../appendix/glossary.md#registry         '"registry" (glossary entry)'
