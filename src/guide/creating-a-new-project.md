# Criando um Novo Pacote

Para iniciar um novo [pacote][def-package] com o Cargo, use `cargo new`

```console
$ cargo new hello_world --bin
```

Estamos passando `--bin` porque estamos fazendo um programa binário: se nós
 estivéssemos fazendo uma biblioteca, nós passaríamos `--lib`. Isso também inicializa um novo
 repositório `git` por padrão. se você não quer que ele faça isso, passe `--vcs none`.

Vamos conferir o que o Cargo gerou para nós:

```console
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

Vamos dar uma olhada no `Cargo.toml`:

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]

```

Isso é chamado de [***manifest***][def-manifest], e contém todos os
 metadados que o Cargo precisa para compilar o seu pacote. Este arquivo é escrito no formato
 [TOML] (pronunciado /tɑməl/).

Aqui está o que está em `src/main.rs`:

```rust
fn main() {
    println!("Hello, world!");
}
```

O Cargo gerou um programa "hello world" para nós, também conhecido como uma
[*crate binária*][def-crate]. Vamos compilá-lo:
```console
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

E então executá-lo:

```console
$ ./target/debug/hello_world
Hello, world!
```

Também podemos usar `cargo run` para compilar e depois executar em um único passo (Você
 não verá a linha `Compiling` se não tiver feito nenhuma alteração desde a última compilação):

```console
$ cargo run
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

Agora você notará um novo arquivo, `Cargo.lock`. Ele contém informações sobre nossas dependências. 
Como ainda não temos nenhuma, não é muito interessante neste momento.

Quando estiver pronto para o lançamento, você pode usar `cargo build --release` para compilar
seus arquivos com as otimizações ativadas:

```console
$ cargo build --release
   Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

O `cargo build --release` coloca o binário resultante na pasta `target/release` ao invés de
`target/debug`.

A compilação no modo de depuração(debug) é o padrão para o desenvolvimento. O tempo de compilação é mais curto, já que o compilador não realiza otimizações, mas o código será executado mais lentamente. O modo de lançamento(release) leva mais tempo para compilar, mas o código será executado mais rápido.

[TOML]: https://toml.io/
[def-crate]:     ../appendix/glossary.md#crate     '"crate" (glossary entry)'
[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
