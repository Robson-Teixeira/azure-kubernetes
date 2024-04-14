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

## Comandos

> Necessário Azure CLI instalado

- `az login` login no Azure
- `az aks get-credentials --name <nome-cluster> --resource-group <nome-resource-group>` obter credenciais (contexto de conexão) do recurso AKS para o cluster e resource group especificados
- `az acr login --name <nomeregistry>` login no ACR
- `docker tag <nome-usuário/nome-imagem>:versão <nomeregistry.azurecr.io>/<pasta>/<nome-imagem>:versão` registrar versão no ACR
- `docker push <nomeregistry.azurecr.io>/<pasta>/<nome-imagem>:versão` subir versão para o ACR

## Configurando ACR
- Acessar [Azure](https://portal.azure.com/)
    - Criar um recurso -> Contêineres -> Registro de Contêiner
        >Nome do registro `nomeregistry.azurecr.io`
        >Criar grupo de recursos (caso não exista), por exemplo, `nome-rg`
        >Habilitar `Usuário administrador`