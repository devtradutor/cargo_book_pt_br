# Cargo.toml vs Cargo.lock

`Cargo.toml` e `Cargo.lock` servem a dois propósitos diferentes. 
Antes de falarmos sobre eles, aqui está um resumo:

* `Cargo.toml` descreve suas dependências em um sentido amplo e é
  escrito por você.
* `Cargo.lock` contém informações exatas sobre suas dependências. Isso é
  mantido pelo Cargo e não deve ser editado manualmente.

Em caso de dúvida, verifique `Cargo.lock` no sistema de controle de versão (por exemplo, Git).
Para uma melhor compreensão do por que e quais podem ser as alternativas, consulte
["Porquê ter o Cargo.lock no controle de versão?" no FAQ](../faq.md#why-have-cargolock-in-version-control).
Recomendamos combinar isso com 
[Verificação das últimas dependências](continuous-integration.md#verifying-latest-dependencies)

Vamos aprofundar um pouco mais.

O arquivo `Cargo.toml` é um arquivo [**manifesto**][def-manifest] no qual podemos especificar um
um monte de metadados diferentes sobre nosso pacote. Por exemplo, nós podemos dizer que nós
dependemos de outro pacote:

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

Este pacote tem uma única dependência, na biblioteca `regex`. Nós declarámos
este caso que estamos confiando em um repositório Git em particular que vive no
GitHub. Como não especificamos nenhuma outra informação, o Cargo assume que
pretendemos usar o último commit no branch padrão para construir nosso pacote.

Parece-lhe bem? Bem, há um problema: se você construir este pacote hoje, e
e depois enviar uma cópia para mim, e eu construir este pacote amanhã, algo mau
pode acontecer. Poderia haver mais commits para `regex` nesse meio tempo, e minha
compilação incluiria novos commits enquanto a sua não. Portanto, nós
teríamos compilações diferentes. Isso seria ruim porque nós queremos compilações reproduzíveis.

Nós poderíamos resolver esse problema definindo um valor `rev` específico no nosso `Cargo.toml`,
para que o Cargo pudesse saber exatamente qual revisão usar ao construir o pacote:

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git", rev = "9f9f693" }
```

Agora as nossas construções serão iguais. Mas há uma grande desvantagem: agora temos que
pensar manualmente nos SHA-1s sempre que quisermos atualizar a nossa biblioteca. Isso é
tedioso e propenso a erros.

Agora, entramos no `Cargo.lock`. Devido à sua existência, não precisamos acompanhar manualmente 
as revisões exatas: o Cargo fará isso por nós. Quando temos um manifesto como este:

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
regex = { git = "https://github.com/rust-lang/regex.git" }
```

O Cargo vai pegar o último commit e escrever essa informação no nosso 
`Cargo.lock` quando construirmos pela primeira vez. Esse arquivo será parecido com este:

```toml
[[package]]
name = "hello_world"
version = "0.1.0"
dependencies = [
 "regex 1.5.0 (git+https://github.com/rust-lang/regex.git#9f9f693768c584971a4d53bc3c586c33ed3a6831)",
]

[[package]]
name = "regex"
version = "1.5.0"
source = "git+https://github.com/rust-lang/regex.git#9f9f693768c584971a4d53bc3c586c33ed3a6831"
```

Pode ver que há muito mais informação aqui, incluindo a revisão exata 
que usamos para construir. Agora, quando você der seu pacote para outra pessoa,
ela usará exatamente o mesmo SHA, mesmo que não o tenhamos especificado em nosso
`Cargo.toml`.

Quando estivermos prontos para optar por uma nova versão da biblioteca, o Cargo pode recalcular as dependências e atualizar as coisas para nós:

```console
$ cargo update         # atualiza todas as dependências
$ cargo update regex   # atualização apenas do “regex”
```

Isto irá escrever um novo `Cargo.lock` com a informação da nova versão. Note
que o argumento para `cargo update` é na verdade uma
[Especificação de ID do pacote](../reference/pkgid-spec.md) e `regex` é apenas uma
especificação curta.

[def-manifest]:  ../appendix/glossary.md#manifest  '"manifest" (glossary entry)'
[def-package]:   ../appendix/glossary.md#package   '"package" (glossary entry)'
