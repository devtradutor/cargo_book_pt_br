# Layout do Pacote

O Cargo utiliza convenções para a organização de arquivos a fim de facilitar o entendimento de um 
novo [pacote][def-package] do Cargo:

```text
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

* `Cargo.toml` e `Cargo.lock` são armazenados na raiz do seu pacote (*base do pacote*).
* O código-fonte é colocado no diretório `src`.
* O arquivo de biblioteca padrão é `src/lib.rs`.
* O arquivo executável padrão é `src/main.rs`.
    * Outros executáveis podem ser colocados em `src/bin/`.
* Os benchmarks são colocados no diretório `benches`.
* Exemplos são colocados no diretório `examples`.
* Testes de integração são colocados no diretório `tests`.

Se um binário, exemplo, benchmark ou de integração de teste  consistir em vários arquivos de origem, 
coloque um arquivo `main.rs` junto com os [módulos][def-module] 
adicionais dentro de um subdiretório do diretório `src/bin`, `examples`, `benches` ou `tests`. 
O nome do executável será o nome do diretório.

Você pode aprender mais sobre o sistema de módulos do Rust no [livro][book-modules].

Consulte [Configurando um alvo][Configuring a target] para obter mais detalhes sobre a configuração manual de alvos.
Consulte [Descoberta automática de alvos][Target auto-discovery] para obter mais informações sobre como controlar como 
o Cargo automaticamente infere nomes de alvos.

[book-modules]: ../../book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[Configuring a target]: ../reference/cargo-targets.md#configuring-a-target
[def-package]:           ../appendix/glossary.md#package          '"package" (glossary entry)'
[def-module]:            ../appendix/glossary.md#module           '"module" (glossary entry)'
[Target auto-discovery]: ../reference/cargo-targets.md#target-auto-discovery
