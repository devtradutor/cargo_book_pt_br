# Integração contínua

## Começando

Um CI básico irá construir e testar os seus projectos:

### GitHub Actions

Para testar seu pacote no GitHub Actions, aqui está um exemplo de arquivo `.github/workflows/ci.yml`:

```yaml
name: Cargo Build & Test

on:
  push:
  pull_request:

env: 
  CARGO_TERM_COLOR: always

jobs:
  build_and_test:
    name: Rust project - latest
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain:
          - stable
          - beta
          - nightly
    steps:
      - uses: actions/checkout@v3
      - run: rustup update ${{ matrix.toolchain }} && rustup default ${{ matrix.toolchain }}
      - run: cargo build --verbose
      - run: cargo test --verbose
  
```

Isso testará todos os três canais de lançamento (observe que uma falha em qualquer versão do toolchain resultará na falha do trabalho inteiro). Você também pode clicar em `"Actions" > "new workflow"` na interface do GitHub e selecionar Rust para adicionar a [configuração padrão](https://github.com/actions/starter-workflows/blob/main/ci/rust.yml) ao seu repositório. Consulte a [documentação do GitHub Actions](https://docs.github.com/en/actions) para obter mais informações.

### GitLab CI

Para testar seu pacote no GitLab Ci, aqui está um exemplo no arquivo `.gitlab-ci.yml`

```yaml
stages:
  - build

rust-latest:
  stage: build
  image: rust:latest
  script:
    - cargo build --verbose
    - cargo test --verbose

rust-nightly:
  stage: build
  image: rustlang/rust:nightly
  script:
    - cargo build --verbose
    - cargo test --verbose
  allow_failure: true
```

Isso será testado no canal estável e no canal noturno, mas qualquer
quebra no nightly não falhará sua compilação geral. Por favor, veja a
[Documentação do GitLab CI](https://docs.gitlab.com/ce/ci/yaml/index.html) para mais
informações.

### builds.sr.ht

Para testar seu pacote no sr.ht, aqui está um exemplo de arquivo `.build.yml`.
Certifique-se de mudar `<seu repo>` e `<seu projeto>` para o repo a ser clonado e
o diretório onde ele foi clonado.

```yaml
image: archlinux
packages:
  - rustup
sources:
  - <seu repo>
tasks:
  - setup: |
      rustup toolchain install nightly stable
      cd <seu projeto>/
      rustup run stable cargo fetch
  - stable: |
      rustup default stable
      cd <seu projeto>/
      cargo build --verbose
      cargo test --verbose
  - nightly: |
      rustup default nightly
      cd <seu projeto>/
      cargo build --verbose ||:
      cargo test --verbose  ||:
  - docs: |
      cd <seu projeto>/
      rustup run stable cargo doc --no-deps
      rustup run nightly cargo doc --no-deps ||:
```

Isto irá testar e construir a documentação no canal estável e no canal noturno
mas qualquer quebra no nightly não falhará sua compilação geral. Por favor
veja a [documentação builds.sr.ht](https://man.sr.ht/builds.sr.ht/) para mais
informações.

## Verificação das dependências mais recentes

Quando [especificando dependências](../reference/specifying-dependencies.md) no
`Cargo.toml`, elas geralmente correspondem a uma gama de versões.
Testar exaustivamente todas as combinações de versões seria muito complicado.
Verificar as últimas versões pelo menos testaria os usuários que executam [`cargo
add`] ou [`cargo install`].

Ao testar as versões mais recentes, algumas considerações são:
- Minimizar fatores externos que afetam o desenvolvimento local ou CI.
- Taxa de novas dependências sendo publicadas.
- Nível de risco que um projeto está disposto a aceitar.
- Custos de CI, incluindo custos indiretos, como se um serviço de CI tiver um limite 
  máximo para executores em paralelo, o que faz com que novos trabalhos sejam serializados quando atingem o máximo.

Algumas soluções possíveis incluem:
- [Não verificando o `Cargo.lock`](../faq.md#why-have-cargolock-in-version-control)
  - Dependendo da velocidade do PR, muitas versões podem não ser testadas
  - Isso vem com o custo de determinismo.
- Ter um trabalho de CI para verificar as dependências mais recentes, mas marcá-lo para "continuar em caso de falha".
  - Dependendo do serviço de CI, as falhas podem não ser óbvias.
  - Dependendo da velocidade dos PR, pode utilizar mais recursos do que o necessário.
- Ter uma tarefa de CI agendada para verificar as dependências mais recentes
  - Um serviço de CI hospedado pode desabilitar trabalhos agendados para repositórios que
    não tenham sido tocados há algum tempo, afetando pacotes mantidos passivamente
  - Dependendo do serviço de CI, as notificações podem não ser encaminhadas para as pessoas
    que possam atuar sobre a falha
  - Se não for equilibrado com a taxa de publicação de dependências, pode não testar versões suficientes
    ou pode efetuar testes redundantes
- Atualizar regularmente as dependências através de PRs, como com [Dependabot] ou [RenovateBot]
  - Pode isolar dependências para o seu próprio PR ou juntá-las num único PR
  - Só utiliza os recursos necessários
  - Pode configurar a frequência para equilibrar os recursos de CI e a cobertura das versões de dependência

Um exemplo de trabalho de CI para verificar as dependências mais recentes, usando GitHub Actions:
```yaml
jobs:
  latest_deps:
    name: Latest Dependencies
    runs-on: ubuntu-latest
    continue-on-error: true
    steps:
      - uses: actions/checkout@v3
      - run: rustup update stable && rustup default stable
      - run: cargo update --verbose
      - run: cargo build --verbose
      - run: cargo test --verbose
```
Para projetos com maior risco de falhas por plataforma ou por versão do Rust, 
pode ser necessário testar mais combinações.

[`cargo add`]: ../commands/cargo-add.md
[`cargo install`]: ../commands/cargo-install.md
[Dependabot]: https://docs.github.com/en/code-security/dependabot/working-with-dependabot
[RenovateBot]: https://renovatebot.com/
