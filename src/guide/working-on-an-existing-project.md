# Tralhando em um Pacote Cargo Existente

Se você baixar um [pacote][def-package] existente que usa o Cargo, 
é realmente fácil começar.

Primeiro, obtenha o pacote de algum lugar. Neste exemplo, usaremos o `regex` clonado de seu repositório no GitHub:

```console
$ git clone https://github.com/rust-lang/regex.git
$ cd regex
```

Para construir, use `cargo build`:

```console
$ cargo build
   Compiling regex v1.5.0 (file:///path/to/package/regex)
```

Isso irá buscar todas as dependências e depois construí-las, 
juntamente com o pacote.

[def-package]:  ../appendix/glossary.md#package  '"package" (glossary entry)'
