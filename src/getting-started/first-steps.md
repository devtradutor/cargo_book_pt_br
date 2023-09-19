# Primeiros Passos com o Cargo

Esta seção fornece uma noção rápida da ferramenta de linha de comando do `cargo`.
 Nos demonstramos sua capacidade de gerar um novo [***pacote***][def-package] para nós,
 sua capacidade de compilar o [***crate***][def-crate] dentro do pacote, e sua
 capacidade de executar o resultado do programa.

Para iniciar um novo pacote com o Cargo, use  o `cargo new`:

```console
$ cargo new hello_world
```

O Cargo por padrão usa `--bin` para criar programa binário. Para fazer uma biblioteca,
 nós passaria o `--lib` em vez disso.

Vamos ver o que o Cargo gerou para nós:

```console
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

Isto é tudo que nos precisamos para começar. Primeiro, vamos verificar o `Cargo.toml`:

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
```

Isto é chamado de [***manifest***][def-manifest], e contém todos os metadados
 que o Cargo precisa para compilar seu pacote.

Aqui está o que está incluso no `src/main.rs`

```rust
fn main() {
    println!("Hello, world!");
}
```

O Cargo gerou um programa "hello world" para nós, também conhecido como [***crate binário***][def-crate].
 Vamos compilá-lo:

```console
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

E então execute-o:

```console
$ ./target/debug/hello_world
Hello, world!
```

Também podemos usar `cargo run` para compilar e depois executá-lo, tudo em uma única etapa:

```console
$ cargo run
     Fresh hello_world v0.1.0 (file:///path/to/package/hello_world)
   Running `target/hello_world`
Hello, world!
```

## Indo mais além

Para mais detalhes sobre como usar o Cargo, consulte o [guia do Cargo](../guide/index.md)

[def-crate]:     ../appendix/glossary.md#crate     '"crate" (glossary entry)'
[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
