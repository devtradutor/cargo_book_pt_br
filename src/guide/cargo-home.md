# Cargo Home (Casa do Cargo)

O "Cargo home" funciona como um cache de download e fonte. 
Ao construir uma [crate][def-crate], o Cargo armazena as dependências de compilação baixadas no Cargo home.
Você pode alterar o local do Cargo home definindo a [variável de ambiente][env] `CARGO_HOME`. 
O [crate](https://crates.io/crates/home) fornece uma API para obter essa localização, caso você precise dessa informação dentro do seu crate Rust. 
Por padrão, o Cargo home está localizado em `$HOME/.cargo/`.

Por favor, observe que a estrutura interna do Cargo home não está estabilizada e pode estar sujeita a alterações a qualquer momento.

O Cargo home consiste nos seguintes componentes:

## Arquivos:

* `config.toml`
    Arquivo de configuração global do Cargo, consulte a [entrada de configuração na referência][config].
* `credentials.toml`
    Credenciais de login privadas geradas pelo [`cargo login`] para fazer login em um [registro][def-registry].

* `.crates.toml`, `.crates2.json`
    Esses arquivos ocultos contêm informações de [pacote][def-package] de crates instalados via [`cargo install`]. NÃO edite manualmente!

## Diretórios:

* `bin`
O diretório bin contém executáveis das crates que foram instaladas via [`cargo install`] ou [`rustup`](https://rust-lang.github.io/rustup/). 
Para tornar esses binários acessíveis, adicione o caminho do diretório à variável de ambiente `$PATH`.

 *  `git`
	As fontes do Git são armazenadas aqui:

    * `git/db`
		Quando uma crate depende de um repositório Git, o Cargo clona o repositório como um repositório nu (bare) neste diretório e o atualiza, se necessário.

    * `git/checkouts`
		Se uma fonte Git for usada, o commit necessário do repositório é verificado a partir do repositório nu (bare) dentro de `git/db` para este diretório.
		Isso fornece ao compilador os arquivos reais contidos no repositório do commit especificado para essa dependência. 
		Múltiplos checkouts de commits diferentes do mesmo repositório são possíveis.

* `registry`
	Os pacotes e metadados de registros de crates (como [crates.io](https://crates.io/)) estão localizados aqui.

  * `registry/index`
		O índice é um repositório Git nu (bare) que contém os metadados (versões, dependências etc) de todas as crates disponíveis em um registro.

  *  `registry/cache`
		As dependências baixadas são armazenadas no cache. As crates são arquivos compactados no formato gzip com a extensão `.crate`.

  * `registry/src`
		Se um arquivo `.crate` baixado for necessário por um pacote, ele é descompactado na pasta `registry/src`, onde o rustc encontrará os arquivos `.rs`.


## Armazenamento em cache do Cargo home em CI

Para evitar o redownload de todas as dependências das crates durante a integração contínua, você pode fazer cache do diretório `$CARGO_HOME`.
No entanto, fazer cache de todo o diretório frequentemente é ineficiente, já que ele conterá as fontes baixadas duas vezes.
Se dependermos de uma crate como `serde 1.0.92` e fizermos cache de todo o `$CARGO_HOME`, na verdade estaríamos fazendo cache das fontes duas vezes, o arquivo `serde-1.0.92.crate` dentro de `registry/cache` e os arquivos `.rs` extraídos do serde dentro de `registry/src`.
Isto pode atrasar desnecessariamente a construção, uma vez que baixar, extrair, recomprimir e reenviar o cache para os servidores do CI pode levar algum tempo.

Se desejar fazer cache de binários instalados com [`cargo install`], você precisa fazer cache da pasta `bin/` e dos arquivos `.crates.toml` e `.crates2.json`.

Deve ser suficiente fazer cache dos seguintes arquivos e diretórios em todas as compilações:

* `.crates.toml`
* `.crates2.json`
* `bin/`
* `registry/index/`
* `registry/cache/`
* `git/db/`



## Fornecendo todas as dependências de um projeto

Veja o subcomando [`cargo vendor`].



## Limpando o cache

Em teoria, você sempre pode remover qualquer parte do cache, e o Cargo fará o melhor possível para restaurar as fontes se uma crate precisar delas, seja reextraindo um arquivo ou verificando um repositório nu ou simplesmente redownload das fontes da web.

Alternativamente, a crate [cargo-cache](https://crates.io/crates/cargo-cache) fornece uma ferramenta CLI simples para limpar apenas partes selecionadas do cache ou mostrar os tamanhos de seus componentes na linha de comando.

[`cargo install`]: ../commands/cargo-install.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo vendor`]: ../commands/cargo-vendor.md
[config]: ../reference/config.md
[def-crate]:     ../appendix/glossary.md#crate     '"crate" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
[def-registry]:  ../appendix/glossary.md#registry  '"registry" (glossary entry)'
[env]: ../reference/environment-variables.md
