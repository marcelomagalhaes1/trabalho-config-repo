📘 Centralização de Configurações com Spring Cloud Config

Este repositório apresenta um projeto completo utilizando o Spring Cloud Config para gerenciamento centralizado de configurações em uma arquitetura de microserviços.
O ambiente é composto por um Config Server e três aplicações clientes que consomem essas configurações remotamente através de um repositório Git.

🗺️ Arquitetura do Sistema

A solução utiliza o padrão Configuração Centralizada, amplamente empregado em arquiteturas distribuídas.

🔧 Componentes:
Serviço	Porta	Função
config-server	8888	Lê configurações do Git e as entrega via HTTP
cliente-vendas	8081	Microserviço com configurações remotas
cliente-estoque	8082	Microserviço com configurações remotas
cliente-relatorios	8083	Microserviço com configurações remotas
🔗 Comunicação Cliente-Servidor

O Config Server atua como ponto central, lendo arquivos .properties de um repositório Git remoto (trabalho-config-repo).

Os Clientes usam a dependência spring-cloud-config-client para buscar suas configurações de forma automática antes mesmo de inicializar o Spring Boot.

A comunicação ocorre via HTTP com o endpoint base:

http://localhost:8888

⚙️ Fluxo de Inicialização e Obtenção de Configurações

O processo ocorre de forma determinística em etapas:

1️⃣ Inicialização do Config Server

Inicia na porta 8888

Utiliza @EnableConfigServer

Lê a propriedade:

spring.cloud.config.server.git.uri


Conecta-se ao repositório Git e carrega arquivos como:

cliente-vendas-dev.properties
cliente-estoque-dev.properties
cliente-relatorios-dev.properties

2️⃣ Inicialização de um Cliente

Um cliente, como cliente-vendas, inicia e imediatamente:

3️⃣ Busca Remota de Configuração

Com base no application.properties, o cliente monta automaticamente a seguinte URL:

http://localhost:8888/cliente-vendas/dev


Onde:

cliente-vendas = spring.application.name

dev = spring.profiles.active

4️⃣ Entrega da Configuração

O Config Server retorna um JSON contendo as propriedades, por exemplo:

{
  "server.port": 8081,
  "mensagem.boasvindas": "Bem-vindo ao sistema de Vendas - Ambiente DEV"
}

5️⃣ Carregamento e Inicialização Final

O cliente injeta essas propriedades no seu contexto Spring e só então completa sua inicialização.

🔑 Função de Cada Propriedade Importante
📌 No Config Server (config-server/application.properties)
Propriedade	Função
spring.cloud.config.server.git.uri	URL do repositório Git onde estão as configs
server.port	Porta do Config Server (8888)
📌 Nos Clientes (cliente-*/application.properties)
Propriedade	Função
spring.application.name	Nome do microserviço (define qual arquivo .properties buscar)
spring.profiles.active	Define ambiente (ex: dev)
spring.config.import	URL do Config Server (ex: optional:configserver:http://localhost:8888)
📌 No Repositório Git (*.properties)
Propriedade	Função
server.port	Porta onde o cliente irá rodar (vem do Git, não local)
mensagem.boasvindas	Mensagem exibida pelo endpoint /mensagem
🧪 Testes e Verificações
🔍 Testes no Config Server
URL	Função	Resultado
http://localhost:8888/cliente-vendas/dev	Verifica leitura do arquivo remoto	JSON com server.port:8081
http://localhost:8888/actuator/health	Verifica saúde do serviço	UP

✔️ Ambos funcionando corretamente.

🔍 Testes nos Clientes

O endpoint /mensagem foi testado em cada microserviço:

Serviço	Porta	URL	Resultado
Vendas	8081	http://localhost:8081/mensagem
	“Bem-vindo ao sistema de Vendas - Ambiente DEV”
Estoque	8082	http://localhost:8082/mensagem
	“Serviço de Estoque inicializado - Ambiente DEV”
Relatórios	8083	http://localhost:8083/mensagem
	“Serviço de Relatórios em execução - Ambiente DEV”

✔️ Todos os clientes receberam e injetaram suas configurações corretamente.

📝 Conclusão

O projeto comprova o funcionamento completo do Spring Cloud Config, garantindo:

Centralização de configurações

Versionamento via Git

Separação clara entre serviços

Flexibilidade para múltiplos ambientes

Redução de duplicação de configurações

A arquitetura mostrou-se estável e eficiente, com comportamento consistente entre todos os microserviços.
