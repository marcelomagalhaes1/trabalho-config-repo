# 📘 Centralização de Configurações com Spring Cloud Config

Este projeto demonstra um ambiente completo utilizando **Spring Cloud
Config** para gerenciamento centralizado de configurações em uma
arquitetura de microserviços.\
A solução é composta por um **Config Server** e três aplicações clientes
que consomem configurações versionadas em um repositório Git remoto.

## 🗺️ Arquitetura do Sistema

O projeto utiliza o padrão **Configuração Centralizada**, muito comum em
sistemas distribuídos.

                   +------------------------+
                   |   Repositório Git     |
                   |  (trabalho-config-repo)|
                   +-----------+------------+
                               |
                               v
                   +------------------------+
                   |     CONFIG SERVER      |
                   |       Porta 8888       |
                   +-----------+------------+
            _________|__________|___________
           |          |          |          |
           v          v          v          v
    +-------------+ +-------------+ +-------------+
    | cliente     | | cliente     | | cliente     |
    | vendas 8081 | | estoque8082 | | relatorios8083 |
    +-------------+ +-------------+ +-------------+

## 🔧 Componentes do Sistema

  ---------------------------------------------------------------------------------
  Serviço                  Porta   Função
  ------------------------ ------- ------------------------------------------------
  **config-server**        8888    Lê configs do Git e entrega via HTTP

  **cliente-vendas**       8081    Microserviço que consome config remota

  **cliente-estoque**      8082    Microserviço que consome config remota

  **cliente-relatorios**   8083    Microserviço que consome config remota
  ---------------------------------------------------------------------------------

## 🔗 Comunicação Cliente ↔ Servidor

-   O Config Server lê arquivos `.properties` do repositório Git.
-   Os clientes usam `spring-cloud-config-client` para buscar
    configurações antes da inicialização.
-   A comunicação ocorre por HTTP através do endpoint:

```{=html}
<!-- -->
```
    http://localhost:8888

# ⚙️ Fluxo de Funcionamento

## 1️⃣ Inicialização do Config Server

-   Sobe na porta **8888**
-   Usa `@EnableConfigServer`
-   Lê o repositório Git configurado:

```{=html}
<!-- -->
```
    spring.cloud.config.server.git.uri

Arquivos lidos:

    cliente-vendas-dev.properties
    cliente-estoque-dev.properties
    cliente-relatorios-dev.properties

## 2️⃣ Inicialização dos Clientes

Cada microserviço inicia e automaticamente consulta o Config Server.

## 3️⃣ Busca Remota das Configurações

Exemplo para **cliente-vendas** com perfil `dev`:

    http://localhost:8888/cliente-vendas/dev

## 4️⃣ Resposta do Config Server

``` json
{
  "server.port": 8081,
  "mensagem.boasvindas": "Bem-vindo ao sistema de Vendas - Ambiente DEV"
}
```

## 5️⃣ Carregamento das Configurações

O cliente injeta as propriedades recebidas e só então finaliza sua
inicialização.

# 🔑 Propriedades Importantes

## 📌 No Config Server (`application.properties`)

  ------------------------------------------------------------------------------
  Propriedade                            Função
  -------------------------------------- ---------------------------------------
  `spring.cloud.config.server.git.uri`   Repositório Git com as configurações

  `server.port`                          Porta do servidor de configuração
                                         (8888)
  ------------------------------------------------------------------------------

## 📌 Nos Clientes (`application.properties`)

  -----------------------------------------------------------------------
  Propriedade                    Função
  ------------------------------ ----------------------------------------
  `spring.application.name`      Nome do microserviço / arquivo no Git

  `spring.profiles.active`       Ambiente desejado (dev, test, prod)

  `spring.config.import`         URL do Config Server
  -----------------------------------------------------------------------

## 📌 No Repositório Git (\*.properties)

  Propriedade             Função
  ----------------------- ------------------------------------------
  `server.port`           Porta de execução do cliente
  `mensagem.boasvindas`   Mensagem retornada no endpoint /mensagem

# 🧪 Testes e Validações

## 🔍 Testes no Config Server

  -----------------------------------------------------------------------------------
  URL                                          Função                  Resultado
                                                                       esperado
  -------------------------------------------- ----------------------- --------------
  `http://localhost:8888/cliente-vendas/dev`   Ler config do Git       JSON retornado

  `http://localhost:8888/actuator/health`      Checar saúde do serviço UP
  -----------------------------------------------------------------------------------

## 🔍 Testes nos Clientes

  ---------------------------------------------------------------------------
  Serviço         Porta    URL                               Resultado
  --------------- -------- --------------------------------- ----------------
  Vendas          8081     `/mensagem`                       "Bem-vindo ao
                                                             sistema de
                                                             Vendas -
                                                             Ambiente DEV"

  Estoque         8082     `/mensagem`                       "Serviço de
                                                             Estoque
                                                             inicializado -
                                                             Ambiente DEV"

  Relatórios      8083     `/mensagem`                       "Serviço de
                                                             Relatórios em
                                                             execução -
                                                             Ambiente DEV"
  ---------------------------------------------------------------------------

# 📝 Conclusão

O projeto demonstra:

-   ✔️ Centralização completa das configurações\
-   ✔️ Versionamento via Git\
-   ✔️ Isolamento por serviço\
-   ✔️ Separação clara por ambiente\
-   ✔️ Redução de duplicação\
-   ✔️ Arquitetura estável e consistente com microserviços
