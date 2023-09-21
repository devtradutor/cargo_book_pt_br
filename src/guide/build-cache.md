# Construção de Cache

O Cargo armazena a saída de uma compilação no diretório "target". 
Por padrão, este é o diretório chamado `target` na raiz do seu [*espaço de trabalho (workspace)*][def-workspace]. 
Para alterar o local, você pode definir a [variável de ambiente][environment variable] `CARGO_TARGET_DIR`, 
o valor de configuração [`build.target-dir`], ou a flag de linha de comando `--target-dir`.

O layout do diretório depende se você está ou não usando a flag `--target`
para compilar para uma plataforma específica. Se `--target` não for especificado, o Cargo
é executado em um modo onde ele constrói para a arquitetura do host. A saída vai para
raiz do diretório de destino, com cada [perfil (profile)][profile] armazenado em um
subdiretório separado:


Diretório | Descrição
----------|------------
<code style="white-space: nowrap">target/debug/</code> | Contém a saída para o perfil `dev` (desenvolvimento).
<code style="white-space: nowrap">target/release/</code> | Contém a saída para o perfil `release` (com a opção `--release`).
<code style="white-space: nowrap">target/foo/</code> | Contém a saída de compilação para o perfil `foo` (com a opção `--profile=foo`).

Por razões históricas, os perfis `dev` e `test` são armazenados no diretório `debug`, 
enquanto os perfis `release` e `bench` são armazenados no diretório `release`. 
Perfis definidos pelo usuário são armazenados em um diretório com o mesmo nome do perfil.

Ao compilar para outro alvo usando `--target`, a saída é colocada em um diretório com o nome do [target]:

Diretório | Exemplo
----------|--------
<code style="white-space: nowrap">target/&lt;triple&gt;/debug/</code> | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/debug/</code>
<code style="white-space: nowrap">target/&lt;triple&gt;/release/</code> | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/release/</code>

> **Nota**: Quando não se utiliza `--target`, isso tem a consequência de que o Cargo 
> compartilhará suas dependências com scripts de compilação e proc macros. [`RUSTFLAGS`] 
> será compartilhado com cada invocação do `rustc`. Com a flag `--target`, 
> scripts de compilação e proc macros são compilados separadamente (para a arquitetura do host) 
> e não compartilham `RUSTFLAGS`.

Dentro do diretório de perfil (como `debug` ou `release`), os artefatos são colocados nos seguintes diretórios:

Diretório | Descrição
----------|------------
<code style="white-space: nowrap">target/debug/</code> | Contém a saída do pacote sendo construído (os [executáveis binários][binary executables] e [targets(algos) de biblioteca][library targets]).
<code style="white-space: nowrap">target/debug/examples/</code> | Contém [targets(algos) de exemplos][example targets].

Alguns comandos colocam sua saída em diretórios dedicados no 
nível superior do diretório `target`:

Diretório | Descrição
----------|------------
<code style="white-space: nowrap">target/doc/</code> | Contém a documentação do rustdoc ([`cargo doc`]).
<code style="white-space: nowrap">target/package/</code> | Contém a saída dos comandos [`cargo package`] e [`cargo publish`].

O Cargo também cria vários outros diretórios e arquivos necessários para o processo de compilação. A sua disposição é considerada interna ao Cargo e está sujeita a alterações. Alguns desses diretórios são:

Diretório | Descrição
----------|------------
<code style="white-space: nowrap">target/debug/deps/</code> | Dependências e outros artefatos.
<code style="white-space: nowrap">target/debug/incremental/</code> | [Saída incremental][incremental output] do `rustc`, um cache usado para acelerar compilações subsequentes.
<code style="white-space: nowrap">target/debug/build/</code> | Saída de [scripts de compilação][build scripts].

## Arquivos de informações de dependência (dep-info files).

Ao lado de cada artefato compilado, há um arquivo chamado de "arquivo de informações de dependência" com um sufixo `.d`. 
Este arquivo possui uma sintaxe semelhante a um arquivo Makefile e indica todas as dependências de arquivo necessárias para reconstruir o artefato. 
Eles são destinados a serem usados com sistemas de compilação externos para que possam detectar se o Cargo precisa ser reexecutado. 
Por padrão, os caminhos no arquivo são absolutos. Consulte a opção de configuração [`build.dep-info-basedir`] 
para usar caminhos relativos.

```Makefile
# Exemplo de arquivo dep-info encontrado no target/debug/foo.d
/path/to/myproj/target/debug/foo: /path/to/myproj/src/lib.rs /path/to/myproj/src/main.rs
```

## Cache compartilhado

Uma ferramenta de terceiros, chamada [sccache], pode ser usada para compartilhar dependências compiladas entre diferentes espaços de trabalho.

Para configurar o `sccache`, instale-o com o comando `cargo install sccache` e 
defina a variável de ambiente `RUSTC_WRAPPER` como `sccache` antes de chamar o Cargo. 
Se você estiver usando o bash, faz sentido adicionar `export RUSTC_WRAPPER=sccache` ao seu arquivo `.bashrc`. Alternativamente, 
você pode definir [`build.rustc-wrapper`] na [configuração do Cargo][config]. Consulte a documentação do `sccache` para obter mais detalhes.


[`RUSTFLAGS`]: ../reference/config.md#buildrustflags
[`build.dep-info-basedir`]: ../reference/config.md#builddep-info-basedir
[`build.rustc-wrapper`]: ../reference/config.md#buildrustc-wrapper
[`build.target-dir`]: ../reference/config.md#buildtarget-dir
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo publish`]: ../commands/cargo-publish.md
[build scripts]: ../reference/build-scripts.md
[config]: ../reference/config.md
[def-workspace]:  ../appendix/glossary.md#workspace  '"workspace" (glossary entry)'
[target]: ../appendix/glossary.md#target '"target" (glossary entry)'
[environment variable]: ../reference/environment-variables.md
[incremental output]: ../reference/profiles.md#incremental
[sccache]: https://github.com/mozilla/sccache
[profile]: ../reference/profiles.md
[binary executables]: ../reference/cargo-targets.md#binaries
[library targets]: ../reference/cargo-targets.md#library
[example targets]: ../reference/cargo-targets.md#examples
