# Instalação

## Instalando o Rust e Cargo

A maneira mais fácil de obter o Cargo é instalando a versão estável atual do [Rust]
usando [rustup]. Instalando o Rust usando o `rustup` também instalará o `cargo`

Nos sistemas Linux e macOS, isto é feito da seguinte forma:

```console
curl https://sh.rustup.rs -sSf | sh
```

Isto vai baixar um script, e iniciará a instalação. se tudo correr bem,
 você verá isto aparecer:

```console
Rust is installed now. Great!
```

No Windows, baixe e execute [rustup-init.exe]. Isto iniciará a instalação em
 um console e apresentará a mensagem acima em caso de sucesso.

Depois disso você pode usar o comando `rustup` para instalar os
 canais `beta` ou `nightly` para o Rust e Cargo.

Para outras opções de instalação e informações, visite a
página de [instalação][install-rust] do site Rust.

## Construa e instale o Cargo a partir do código fonte

Alternativa, você pode [construir o Cargo a partir do código fonte][compiling-from-source].

[rust]: https://www.rust-lang.org/pt-BR/
[rustup]: https://rustup.rs/
[rustup-init.exe]: https://win.rustup.rs/
[install-rust]: https://www.rust-lang.org/tools/install
[compiling-from-source]: https://github.com/rust-lang/cargo#compiling-from-source
