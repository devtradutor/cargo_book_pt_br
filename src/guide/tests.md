# Testes

O Cargo pode executar seus testes com o comando `cargo test`. O Cargo procura por testes
para executar em dois lugares: em cada um dos seus arquivos `src` e quaisquer testes em `tests/`.
Os testes em seus arquivos `src` devem ser testes unitários e [testes de documentação][documentation tests].
Os testes em `tests/` devem ser testes de  
estilo de integração. Como tal, você precisará
importar suas crates para os arquivos em `tests`.

Aqui está um exemplo de execução do `cargo test` em nosso [pacote][def-pacote], que
atualmente não tem testes:

```console
$ cargo test
   Compiling regex v1.5.0 (https://github.com/rust-lang/regex.git#9f9f693)
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
     Running target/test/hello_world-9c2b65bbb79eabce

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Se nosso pacote tivesse testes, nós veríamos mais saídas com o número correto de
testes.

Você também pode executar um teste específico passando por um filtro:

```console
$ cargo test foo
```

Isto irá executar qualquer teste com `foo` em seu nome.

O `cargo test` executa verificações adicionais também. Ele irá compilar quaisquer exemplos
que você incluiu para garantir que eles ainda compilam. Ele também executa testes de documentação
para garantir que seus exemplos de código dos comentários da documentação compilem.
Por favor, veja o [guia de teste][testing] na documentação do Rust para uma visão geral
 de como escrever e organizar testes. Veja [Alvos do Cargo: Testes][Cargo Targets: Tests] para saber mais
sobre os diferentes estilos de testes no Cargo.

[documentation tests]: ../../rustdoc/write-documentation/documentation-tests.html
[def-package]:  ../appendix/glossary.md#package  '"package" (glossary entry)'
[testing]: https://doc.rust-lang.org/book/ch11-00-testing.html
[Cargo Targets: Tests]: ../reference/cargo-targets.html#tests
