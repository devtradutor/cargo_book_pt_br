# Dependências

[crates.io] é o [registro de pacotes][def-package-registry] central da comunidade Rust 
que serve como um local para descobrir e baixar
[pacotes][def-package]. O `cargo` está configurado para usá-lo por padrão para encontrar
pacotes solicitados.

Para depender de uma biblioteca hospedada no [crates.io], adicione-a ao seu `Cargo.toml`.

[crates.io]: https://crates.io/

## Adicionando uma dependência

Se o seu `Cargo.toml` ainda não tem uma seção `[dependencies]`,  adicione
isso, então liste o nome e a versão do [crate][def-crate] que você gostaria de
usar. Este exemplo adiciona uma dependência da crate `time`:

```toml
[dependencies]
time = "0.1.12"
```

A string de versão é um requisito de versão [SemVer]. A documentação sobre 
[especificar dependências](../reference/specifying-dependencies.md) fornece mais informações sobre as opções disponíveis aqui.

[SemVer]: https://semver.org

Se também quiséssemos adicionar uma dependência no crate `regex`, não seria necessário adicionar 
`[dependencies]` para cada crate listado. Aqui está como ficaria todo o seu arquivo 
`Cargo.toml` com dependências das crates `time` e `regex`:


```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

Execute novamente `cargo build`, e o Cargo buscará as novas dependências e todas as 
suas dependências, compilando todas elas e atualizando o `Cargo.lock`:

```console
$ cargo build
      Updating crates.io index
   Downloading memchr v0.1.5
   Downloading libc v0.1.10
   Downloading regex-syntax v0.2.1
   Downloading memchr v0.1.5
   Downloading aho-corasick v0.3.0
   Downloading regex v0.1.41
     Compiling memchr v0.1.5
     Compiling libc v0.1.10
     Compiling regex-syntax v0.2.1
     Compiling memchr v0.1.5
     Compiling aho-corasick v0.3.0
     Compiling regex v0.1.41
     Compiling hello_world v0.1.0 (file:///path/to/package/hello_world)
```

Nosso `Cargo.lock` contém informações precisas sobre qual revisão 
de todas essas dependências nós utilizamos.

Agora, se o `regex` for atualizado, ainda construiremos com a 
mesma revisão até que optemos por executar `cargo update`.

Agora você pode usar a biblioteca `regex` no `main.rs`

```rust,ignore
use regex::Regex;

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2014-01-01"));
}
```

A execução mostrará:

```console
$ cargo run
   Running `target/hello_world`
Did our date match? true
```

[def-crate]:             ../appendix/glossary.md#crate             '"crate" (glossary entry)'
[def-package]:           ../appendix/glossary.md#package           '"package" (glossary entry)'
[def-package-registry]:  ../appendix/glossary.md#package-registry  '"package-registry" (glossary entry)'
