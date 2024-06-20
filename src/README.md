## Definições
- AKS: Azure Kubernetes Service
- ACR: Azure Container Registry

## Ferramentas

- [Azure CLI](https://learn.microsoft.com/pt-br/cli/azure/)

## Sequência deploy

- db
    - permissoes.yaml
    - statefulset.yaml
    - servico-banco.yaml

- app
    - deployment.yaml
    - servico-aplicacao.yaml

## Preparação Banco de Dados
- `kubectl get pods` listar POD's
- `kubectl exec -it <nome-pod> sh` acessar shell do POD
- Acessar MySQL:
    ```
    mysql - u root
    ```

- Criar estrutura do banco:
    ```
    use loja
    create table produtos (id integer auto_increment primary key, nome varchar(255), preco decimal(10,2));
    alter table produtos add column usado boolean default false;
    alter table produtos add column descricao varchar(255);
    create table categorias (id integer auto_increment primary key, nome varchar(255));
    insert into categorias (nome) values ("Futebol"), ("Volei"), ("Tenis");
    alter table produtos add column categoria_id integer;
    update produtos set categoria_id = 1;
    ```

## Configurando AKS
- Acessar [Azure](https://portal.azure.com/)
    - Criar um recurso -> Contêineres -> Kubernetes Service
        >Criar grupo de recursos (caso não exista), por exemplo, `nome-rg`

        >**Nome do cluster do Kubernetes** e **Prefixo do nome DNS** podem ser iguais, por exemplo, `nome-k8s`
        
        >Na configuração de escala o valor do tamanho será aplicado para todos os nós.
        Já a contagem de nós **NÃO** deve incluir o master (abstraído pelo AKS), mas sim as máquinas que de fato farão o trabalho.

        >O **Service Principal** será utilizado para criação e dimensionamento das VM's, inibindo assim a necessidade de uma conta administradora do Azure. Além disso, as aplicações o usarão por meio de **ClientID** e **ClientSecret**

        >Para atualizar o cluster, ir até configurações e evoluir de 1 em 1 o 2º número da versão

## Comandos

> Necessário Azure CLI instalado

- `az login` login no Azure
- `az aks get-credentials --name <nome-cluster> --resource-group <nome-resource-group>` obter credenciais (contexto de conexão) do recurso AKS para o cluster e resource group especificados
- `az group create --name <nome-grupo-rg> --location eastus` criar grupo de recursos na região Leste dos EUA
- `az group list` listar grupo de recursos
    - `az group list --output table` formata o output no modo tabela
- `az aks get-versions --location eastus --output table` listar versões Kubernetes por localidade
- `az aks create -h` ajuda para o comando `aks create`
    - `az aks create --name <nome-k8s> --kubernetes-version "<versão>" --node-count <valor> --resource-group <nome-grupo-rg> --location eastus --generate-ssh-keys` cria cluster AKS. Especificado a versão, quantidade de máquinas, grupo de recursos, localização e a chave para a comunicação entre as máquinas e master
- `az aks list --output table` listar clusters
- `az aks scale --node-count <quantidade> --name <nome-registry> --resource-group <nome-grupo-rg>` alterar número de nós do cluster
- `az aks get-upgrades -n <nome-registry> -g <nome-grupo-rg>` listar versão atual e versões disponíveis para atualização do cluster
- `az aks upgrade -k "<versão>" -n <nome-registry> -g <nome-grupo-rg>` atualiza o cluster para a versão indicada
- `az acr -h` ajuda para o comando
    - `az acr create -h` ajuda para o comando `acr create`
        - `az acr create --name <nome-registry> --resource-group <nome-grupo-rg> --sku Basic --location eastus` cria registro. Especificado grupo de recursos, tipo de armazenamento (_Basic_) e localização
    - `az acr login --name <nomeregistry>` login no ACR
    - `az acr update -n <nome-registry> --admin-enabled true` ativar usuário administrador do registro
    - `az acr credential -h` ajuda para o comando `acr credential`
        - `az acr credential show --name <nome-registry>` listar credenciais do ACR (necessário usuário administrador ativo)
- `docker tag <nome-usuário/nome-imagem>:versão <nomeregistry.azurecr.io>/<pasta>/<nome-imagem>:versão` registrar versão no ACR
- `docker push <nomeregistry.azurecr.io>/<pasta>/<nome-imagem>:versão` subir versão para o ACR
- `kubectl create secret docker-registry <nomeregistry.secret> --docker-server <nomeregistry.azurecr.io> --docker-username <usuário-administrador-registry> --docker-password <senha-administrador-registry> --docker-email <e-mail>` cria secret no ACR

## Configurando ACR
- Acessar [Azure](https://portal.azure.com/)
    - Criar um recurso -> Contêineres -> Registro de Contêiner
        >Nome do registro `nomeregistry.azurecr.io`
        >Criar grupo de recursos (caso não exista), por exemplo, `nome-rg`
        >Habilitar `Usuário administrador`

## Custos
- [AKS](https://azure.microsoft.com/en-us/pricing/details/kubernetes-service/)