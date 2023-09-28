# Especificando Dependências

Seus crates podem depender de outras bibliotecas do [crates.io] ou de outros
registros, repositórios `git`, ou subdiretórios em seu sistema de arquivos local.
Você também pode temporariamente substituir a localização de uma dependência --- por exemplo,
para testar uma correção de bug na dependência em que você está trabalhando
localmente. Você pode ter diferentes dependências para plataformas diferentes e
dependências que são usadas apenas durante o desenvolvimento. Vamos dar uma olhada em como
fazer cada uma dessas coisas.

## Especificando dependências do crates.io

O Cargo é configurado para procurar dependências no [crates.io] por padrão. Nesse caso, apenas
o nome e uma sequência de versão são necessários. No [guia do cargo](../guide/index.md), especificamos
uma dependência com a crate `time`:

```toml
[dependencies]
time = "0.1.12"
```

A string `"0.1.12"` é um requisito de versão. 
Embora pareça ser uma *versão* específica da crate `time`, na verdade especifica um *intervalo* de versões e permite atualizações compatíveis com o [SemVer]. 
Uma atualização é permitida se o número da nova versão não modificar o número não-zero mais à esquerda no agrupamento de major, minor e patch. 
Neste caso, se executássemos `cargo update time`, o Cargo deveria nos atualizar para a versão `0.1.13` se esta for a versão mais recente da série `0.1.z`, mas não nos atualizaria para `0.2.0`. Se em vez disso tivéssemos especificado a sequência de versão como `1.0`, o Cargo deveria atualizar para `1.1` se esta fosse a versão mais recente da série `1.y`, mas não para `2.0`. A versão `0.0.x` não é considerada compatível com qualquer outra versão.

[SemVer]: https://semver.org

Aqui estão mais alguns exemplos de requisitos de versão e as versões que seriam
ser permitido com eles:

```notrust
1.2.3  :=  >=1.2.3, <2.0.0
1.2    :=  >=1.2.0, <2.0.0
1      :=  >=1.0.0, <2.0.0
0.2.3  :=  >=0.2.3, <0.3.0
0.2    :=  >=0.2.0, <0.3.0
0.0.3  :=  >=0.0.3, <0.0.4
0.0    :=  >=0.0.0, <0.1.0
0      :=  >=0.0.0, <1.0.0
```

Esta convenção de compatibilidade é diferente do SemVer 
na forma como trata as versões anteriores a 1.0.0. Enquanto o SemVer diz que não há compatibilidade antes de 1.0.0, 
o Cargo considera que `0.x.y` é compatível com `0.x.z`, onde `y ≥ z` e `x > 0`.

É possível ajustar ainda mais a lógica para selecionar versões compatíveis usando operadores especiais, 
embora na maioria das vezes isso não seja necessário.

## Sintaxe de requisito de versão

### Requisitos de acento (caret)

**Requisitos de acento (caret requirements)** são a estratégia padrão de requisitos de versão. 
Esta estratégia de versão permite atualizações compatíveis com o [SemVer]. 
Eles são especificados como requisitos de versão com um acento (^) na frente.

`^1.2.3` é um exemplo de um requisito de acento (caret requirement). 

Deixar de fora o acento (^) é uma sintaxe equivalente simplificada para usar requisitos de acento.
Embora os requisitos de acento sejam o padrão, é recomendável usar a sintaxe simplificada sempre que possível.

`log = "^1.2.3"` é exatamente equivalente a `log = "1.2.3"`.

### Requisitos com Tilde

Os **requisitos com tilde** especificam uma versão mínima com alguma capacidade de atualização. 
Se você especificar uma versão de grande, médio e pequeno porte, ou apenas uma versão de grande e médio porte, somente as alterações de nível de pequeno porte são permitidas. 
Se você especificar apenas uma versão de grande porte, então as alterações de médio e pequeno porte também são permitidas.

`~1.2.3` é um exemplo de um requisito com tilde.
```notrust
~1.2.3  := >=1.2.3, <1.3.0
~1.2    := >=1.2.0, <1.3.0
~1      := >=1.0.0, <2.0.0
```

### Requisitos com Asterisco

Os **requisitos com asterisco** permitem qualquer versão em que o asterisco esteja posicionado.

`*`, `1.*` e `1.2.*` são exemplos de requisitos com asterisco.
```notrust
*     := >=0.0.0
1.*   := >=1.0.0, <2.0.0
1.2.* := >=1.2.0, <1.3.0
```

> **Observação**: [crates.io] não permite versões com apenas o asterisco (`*`).

### Requisitos de Comparação

Os **requisitos de comparação** permitem especificar manualmente um intervalo de versões ou uma versão exata para depender.

Aqui estão alguns exemplos de requisitos de comparação:
```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```

### Requisitos de múltiplas versões

Conforme mostrado nos exemplos acima, é possível separar múltiplos requisitos de versão com uma vírgula, por exemplo, `>= 1.2, < 1.5`.

> **Recomendação:** Quando em dúvida, use o operador de requisito de versão padrão.

>
> Em circunstâncias raras, um pacote com uma "dependência pública" (que reexporta a dependência ou interage com ela em sua API pública)
> que seja compatível com várias versões incompatíveis de acordo com o SemVer (por exemplo, usa apenas um tipo simples que não mudou entre as versões, como um `Id`) 
> pode permitir que os usuários escolham qual versão da "dependência pública" usar. Nesse caso, um requisito de versão como `">=0.4, <2"` pode ser de interesse. 
> No entanto, os usuários do pacote provavelmente encontrarão erros e precisarão selecionar manualmente uma versão da "dependência pública" via `cargo update` se também dependerem dela, 
> já que o Cargo pode escolher diferentes versões da "dependência pública" ao [resolver as versões de dependências](resolver.md) (consulte [#10599]).
>
> Evite restringir o limite superior de uma versão a qualquer coisa menor do que a próxima versão incompatível com o SemVer (por exemplo, evite `">=2.0, <2.4"`) pois outros pacotes na árvore de dependências podem exigir uma versão mais recente, o que levaria a um erro insolucionável (consulte #6584). Considere se controlar a versão em seu [`Cargo.lock`] seria mais apropriado.
> 
> Em algumas instâncias, isso pode não importar ou os benefícios podem superar os custos, incluindo:
> - Quando ninguém mais depende do seu pacote, por exemplo, se ele tiver apenas um `[[bin]]`.
> - Ao depender de um pacote de pré-lançamento e desejar evitar mudanças que quebrem a compatibilidade, uma especificação completa como `"=1.2.3-alpha.3"` pode ser justificada (consulte [#2222]).
> - Quando uma biblioteca reexporta um proc-macro, mas o proc-macro gera código que chama a biblioteca que o reexporta, então uma especificação completa `=1.2.3` 
> pode ser justificada para garantir que o proc-macro não seja mais recente do que a biblioteca que o reexporta e gerando código que usa partes da API que não existem na versão atual.

[`Cargo.lock`]: ../guide/cargo-toml-vs-cargo-lock.md
[#2222]: https://github.com/rust-lang/cargo/issues/2222
[#6584]: https://github.com/rust-lang/cargo/issues/6584
[#10599]: https://github.com/rust-lang/cargo/issues/10599

## Especificando dependências de outros registros

Para especificar uma dependência de um registro que não seja o [crates.io], 
primeiro o registro deve ser configurado em um arquivo `.cargo/config.toml`. Consulte a [documentação de registros][registries documentation] para obter mais informações. 
Na dependência, defina a chave `registry` com o nome do registro a ser usado.

```toml
[dependencies]
some-crate = { version = "1.0", registry = "my-registry" }
```

> **Nota**: [crates.io] não permite que pacotes sejam publicados com dependências em código publicado fora do [crates.io].

[registries documentation]: registries.md

## Especificando Dependências a partir de repositórios `git`

Para depender de uma biblioteca localizada em um repositório `git`, a informação mínima que você precisa especificar é a localização do repositório com a chave `git`:

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```
O Cargo buscará o repositório `git` nessa localização e, em seguida, procurará um arquivo `Cargo.toml` para a crate solicitada em qualquer lugar dentro do repositório `git` 
(não necessariamente na raiz - por exemplo, especificar o nome de uma crate membro em um espaço de trabalho e definir `git` como o repositório que contém o espaço de trabalho).

Como não especificamos nenhuma outra informação, o Cargo assume que pretendemos usar o commit mais recente no branch padrão para construir nosso pacote, que pode não ser necessariamente o branch principal. 
Você pode combinar a chave `git` com as chaves `rev`, `tag` ou `branch` para especificar algo diferente. 
Aqui está um exemplo de especificação para usar o commit mais recente em um branch chamado `next`:

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", branch = "next" }
```

Qualquer coisa que não seja um branch ou tag cai na categoria `rev`. Isso pode ser um hash de commit, como `rev = "4c59b707"`, ou uma referência nomeada exposta pelo repositório remoto, como `rev = "refs/pull/493/head"`. 
As referências disponíveis variam dependendo de onde o repositório está hospedado; o GitHub, em particular, expõe uma referência para o commit mais recente de cada pull request, 
como mostrado, mas outros hosts de git frequentemente fornecem algo equivalente, possivelmente sob um esquema de nomenclatura diferente.

Depois que uma dependência `git` for adicionada, o Cargo irá travar essa dependência no commit mais recente naquele momento. 
Novos commits não serão baixados automaticamente depois que o travamento estiver em vigor. No entanto, eles podem ser baixados manualmente com o comando `cargo update`.

Consulte [Autenticação Git](../appendix/git-authentication.md) para obter ajuda com autenticação git para repositórios privados.

> **Nota**: Nem a chave `git` nem a chave `path` alteram o significado da chave `version`: a chave `version` sempre implica que o pacote está disponível em um registro. As chaves `version`, `git` e `path` são consideradas [locais separados](#multiple-locations) para resolver a dependência.

> Quando a dependência é obtida a partir do `git`, a chave `version` _não_ afetará qual commit é usado, mas as informações de versão no arquivo `Cargo.toml` da dependência ainda serão validadas em relação ao requisito de `version`.

> **Nota**: O [crates.io] não permite que pacotes sejam publicados com dependências em código publicado fora de [crates.io] em si ([dev-dependencies] são ignoradas). Consulte a seção [Múltiplos locais](#multiple-locations) para uma alternativa de fallback para dependências `git` e `path`.


## Especificando dependências de caminho

Com o tempo, nosso pacote `hello_world` do [guia](../guide/index.md) cresceu significativamente em tamanho! Chegou ao ponto em que provavelmente queremos separar uma crate separada para que outros a utilizem. Para fazer isso, o Cargo oferece suporte às **dependências de caminho**, que são normalmente sub-crates que residem dentro de um único repositório. Vamos começar criando uma nova crate dentro do nosso pacote `hello_world`:

```console
# dentro do hello_world/
$ cargo new hello_utils
```

Isso criará uma nova pasta `hello_utils`, dentro da qual um `Cargo.toml` e uma pasta `src` estão prontos para serem configurados. 
Para informar ao Cargo sobre isso, abra o `Cargo.toml` de `hello_world` e adicione `hello_utils` às suas dependências:

```toml
[dependencies]
hello_utils = { path = "hello_utils" }
```

Isso informa ao Cargo que dependemos de uma crate chamada `hello_utils` que é encontrada na pasta `hello_utils` (relativa ao `Cargo.toml` no qual está escrito).

E é isso! O próximo `cargo build` automaticamente construirá o `hello_utils` e todas as suas próprias dependências, e outros também podem começar a usar a crate. 
No entanto, as crates que usam dependências especificadas apenas com um caminho não são permitidas no [crates.io]. 
Se quisermos publicar nossa crate `hello_world`, precisaríamos publicar uma versão de `hello_utils` no [crates.io] e especificar sua versão na linha de dependências também:
```toml
[dependencies]
hello_utils = { path = "hello_utils", version = "0.1.0" }
```

> **Nota**: Nem a chave `git` nem a chave `path` alteram o significado da chave `version`: 
> a chave `version` sempre implica que o pacote está disponível em um registro. 
> As chaves `version`, `git` e `path` são consideradas [locais separados](#multiple-locations) para resolver a dependência.
> **Nota**: O [crates.io] não permite que pacotes sejam publicados com dependências em código publicado fora do [crates.io] em si (as [dev-dependencies] são ignoradas). 
> Consulte a seção [Locais múltiplos](#multiple-locations) para uma alternativa de fallback para dependências `git` e `path`.

## Locais Múltiplos {#multiple-locations}

É possível especificar tanto uma versão de registro quanto uma localização `git` ou `path`. A dependência `git` ou `path`(caminho) será usada localmente (nesse caso, a `version` é verificada em relação à cópia local), e quando publicada em um registro como [crates.io], ela usará a versão do registro. Outras combinações não são permitidas. Exemplos:

```toml
[dependencies]
# Uses `my-bitflags` when used locally, and uses
# version 1.0 from crates.io when published.
bitflags = { path = "my-bitflags", version = "1.0" }

# Uses the given git repo when used locally, and uses
# version 1.0 from crates.io when published.
smallvec = { git = "https://github.com/servo/rust-smallvec.git", version = "1.0" }

# N.B. that if a version doesn't match, Cargo will fail to compile!
```

Um exemplo em que isso pode ser útil é quando você dividiu uma biblioteca em
vários pacotes dentro do mesmo espaço de trabalho. Você pode então usar as dependências `path`
para apontar para os pacotes locais dentro do espaço de trabalho e usar a
versão local durante o desenvolvimento e, em seguida, usar a versão [crates.io] uma vez que
esteja publicado. Isso é semelhante a especificar uma
[substituição(override)](overriding-dependencies.md), mas se aplica apenas a esta
declaração de dependência específica.

## Dependências específicas da plataforma

As dependências específicas da plataforma seguem o mesmo formato, mas são listadas dentro de uma seção `target`. Normalmente, será usada uma sintaxe semelhante a [`#[cfg]`](../../reference/conditional-compilation.html) em Rust para definir essas seções:

```toml
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"

[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"

[target.'cfg(target_arch = "x86")'.dependencies]
native-i686 = { path = "native/i686" }

[target.'cfg(target_arch = "x86_64")'.dependencies]
native-x86_64 = { path = "native/x86_64" }
```

Assim como no Rust, a sintaxe aqui suporta os operadores `not`, `any` e `all` para combinar várias pares de nome/valor de cfg.

Se você deseja saber quais alvos de cfg estão disponíveis em sua plataforma, execute `rustc --print=cfg` a partir da linha de comando. Se você quiser saber quais alvos de cfg estão disponíveis para outra plataforma, como o Windows de 64 bits, execute `rustc --print=cfg --target=x86_64-pc-windows-msvc`.

Ao contrário do seu código-fonte Rust, você não pode usar `[target.'cfg(feature = "fancy-feature")'.dependencies]` para adicionar dependências com base em recursos opcionais. Em vez disso, utilize [a seção `[features]`](features.md):

```toml
[dependencies]
foo = { version = "1.0", optional = true }
bar = { version = "1.0", optional = true }

[features]
fancy-feature = ["foo", "bar"]
```

O mesmo se aplica a `cfg(debug_assertions)`, `cfg(test)` e `cfg(proc_macro)`. Esses valores não funcionarão como o esperado e sempre terão o valor padrão retornado por `rustc --print=cfg`. 
Atualmente, não há como adicionar dependências com base nesses valores de configuração.

Além da sintaxe `#[cfg]`, o Cargo também suporta a listagem completa do alvo ao qual as dependências se aplicariam:

```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```

### Especificações de destino personalizadas

Se você estiver usando uma especificação de destino personalizada (como `--target foo/bar.json`), use o nome de arquivo base sem a extensão `.json`:

```toml
[target.bar.dependencies]
winhttp = "0.4.0"

[target.my-special-i686-platform.dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
```

> **Nota**: As especificações de destino personalizadas não são utilizáveis no canal estável.

## Dependências de desenvolvimento

Você pode adicionar uma seção `[dev-dependencies]` ao seu `Cargo.toml` cujo formato é equivalente ao `[dependencies]`:

```toml
[dev-dependencies]
tempdir = "0.3"
```

As dependências de desenvolvimento não são usadas ao compilar um pacote para construção, mas são usadas para compilar testes, exemplos e benchmarks.

Essas dependências *não* são propagadas para outros pacotes que dependem deste pacote.

Você também pode ter dependências de desenvolvimento específicas do alvo usando `dev-dependencies` no cabeçalho da seção de alvo em vez de `dependencies`. Por exemplo:

```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```

> **Nota**: Quando um pacote é publicado, somente as dev-dependências que especificam uma `version` serão incluídas na crate publicada. 
> Para a maioria dos casos de uso, as dev-dependências não são necessárias quando publicadas, embora alguns usuários 
> (como empacotadores de sistemas operacionais) possam querer executar testes dentro de uma crate, então fornecer uma `version`, se possível, ainda pode ser benéfico.

## Dependências de compilação

Você pode depender de outras crates baseadas no Cargo para uso em seus scripts de compilação. As dependências são declaradas por meio da seção `build-dependencies` do manifesto:

```toml
[build-dependencies]
cc = "1.0.3"
```


Você também pode ter dependências de compilação específicas do alvo usando `build-dependencies` no cabeçalho da seção de alvo em vez de `dependencies`. Por exemplo:

```toml
[target.'cfg(unix)'.build-dependencies]
cc = "1.0.3"
```

Nesse caso, a dependência será construída apenas quando a plataforma de hospedagem corresponder ao alvo especificado.

O script de compilação **não** tem acesso às dependências listadas na seção `dependencies` ou `dev-dependencies`. 
As dependências de compilação também não estarão disponíveis para o pacote em si, a menos que também estejam listadas na seção `dependencies`. 
Um pacote em si e seu script de compilação são construídos separadamente, então suas dependências não precisam coincidir. O Cargo é mantido mais simples e limpo ao usar dependências independentes para fins independentes.

## Escolhendo recursos

Se um pacote no qual você depende oferece recursos condicionais, você pode especificar quais usar:

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # não incluir os recursos padrão, e opcionalmente
                         # selecionar recursos individuais específicos.
features = ["secure-password", "civet"]
```

Mais informações sobre recursos podem ser encontradas no [capítulo de recursos](features.md#dependency-features).

## Renomeando dependências no `Cargo.toml`

Ao escrever uma seção `[dependencies]` no `Cargo.toml`, a chave que você escreve para uma dependência geralmente corresponde ao nome da crate que você importa no código. Para alguns projetos, você pode desejar fazer referência à crate com um nome diferente no código, independentemente de como ela é publicada no crates.io. Por exemplo, você pode desejar:

* Evitar a necessidade de `use foo as bar` no código Rust.
* Dependência de múltiplas versões de uma crate.
* Dependência de crates com o mesmo nome de diferentes registros.

Para dar suporte a isso, o Cargo oferece uma chave `package` na seção `[dependencies]` na qual você especifica em qual pacote deve ser dependido:

```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project.git", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```

In this example, three crates are now available in your Rust code:

```rust,ignore
extern crate foo; // crates.io
extern crate bar; // git repositório
extern crate baz; // registro `custom`
```

Todas essas três crates têm o nome do pacote `foo` em seus próprios `Cargo.toml`, então estamos usando explicitamente a chave `package` para informar ao Cargo que queremos o pacote `foo`, mesmo que o chamemos de algo diferente localmente. A chave `package`, se não especificada, terá como padrão o nome da dependência solicitada.

Observe que se você tiver uma dependência opcional como:

```toml
[dependencies]
bar = { version = "0.1", package = 'foo', optional = true }
```

Você está dependendo da crate `foo` do crates.io, mas a sua crate tem um recurso `bar` 
em vez de um recurso `foo`. Ou seja, os nomes dos recursos seguem o nome da dependência, não o nome do pacote, quando renomeados.

Habilitar dependências transitivas funciona de maneira semelhante, por exemplo, poderíamos adicionar o seguinte ao manifesto acima:

```toml
[features]
log-debug = ['bar/log-debug'] # Usar 'foo/log-debug' seria um erro!
```

## Herdando uma dependência de uma Área de trabalho

As dependências podem ser herdadas de uma área de trabalho especificando a dependência na tabela [`[workspace.dependencies]`][workspace.dependencies] da área de trabalho. Em seguida, adicione-a à tabela `[dependencies]` com `workspace = true`.

Junto com a chave `workspace`, as dependências também podem incluir estas chaves:
- [`optional`][optional]: Note que a tabela `[workspace.dependencies]` não pode especificar `optional`.
- [`features`][features]: Estes são adicionais com os recursos declarados em `[workspace.dependencies]`.

Além de `optional` e `features`, as dependências herdadas não podem usar nenhuma outra chave de dependência (como `version` ou `default-features`).

As dependências nas seções `[dependencies]`, `[dev-dependencies]`, `[build-dependencies]` e `[target."...".dependencies]` 
suportam a capacidade de fazer referência à definição de dependências em `[workspace.dependencies]`.

```toml
[package]
name = "bar"
version = "0.2.0"

[dependencies]
regex = { workspace = true, features = ["unicode"] }

[build-dependencies]
cc.workspace = true

[dev-dependencies]
rand = { workspace = true, optional = true }
```


[crates.io]: https://crates.io/
[dev-dependencies]: #development-dependencies
[workspace.dependencies]: workspaces.md#the-dependencies-table
[optional]: features.md#optional-dependencies
[features]: features.md

<script>
(function() {
    var fragments = {
        "#overriding-dependencies": "overriding-dependencies.html",
        "#testing-a-bugfix": "overriding-dependencies.html#testing-a-bugfix",
        "#working-with-an-unpublished-minor-version": "overriding-dependencies.html#working-with-an-unpublished-minor-version",
        "#overriding-repository-url": "overriding-dependencies.html#overriding-repository-url",
        "#prepublishing-a-breaking-change": "overriding-dependencies.html#prepublishing-a-breaking-change",
        "#overriding-with-local-dependencies": "overriding-dependencies.html#paths-overrides",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>
