# Automação de Deploy com GitHub Actions

Este projeto configura a automação de deploy para dois servidores: um **servidor de teste** e um **servidor de produção**. Através do uso de **GitHub Actions**, o código será automaticamente enviado para o **servidor de teste** sempre que houver um `push` para a branch `staging`, e para o **servidor de produção** após aprovação, quando houver um `push` para a branch `main`.

## Fluxo de Trabalho

1. **Servidor de Teste**:
   - O código é **enviado automaticamente para o servidor de teste** toda vez que o código é **enviado para a branch `staging`**.
   - Os gestores podem verificar a versão do site no servidor de teste para validar as alterações.
   
2. **Servidor de Produção**:
   - Após a validação no servidor de teste, as alterações aprovadas são enviadas para o **servidor de produção** ao fazer um **merge** para a branch **`main`**.

## Pré-requisitos

- **GitHub** com acesso ao repositório.
- **SSH** configurado nos servidores de teste e produção para autenticação sem senha.
- Acesso de **escrita** para configurar **secrets** no GitHub.

## Estrutura das pastas:

```
.introduction-github-actions/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── src/ (onde os arquivos do site ficam)
├── docker-compose.yml
```

## Passos para Configuração

### 1. **Gerar Chaves SSH**
Você precisa gerar um par de chaves SSH para permitir o acesso remoto aos servidores.

1. **No terminal, gere as chaves SSH:**

   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

    Isso gerará duas chaves:

    ```bash
    Chave pública (~/.ssh/id_rsa.pub).
    Chave privada (~/.ssh/id_rsa).
    ```

2. **Adicionar a Chave Pública ao Servidor**

    - Adicione a `chave pública (id_rsa.pub)` nos servidores de teste e produção.

    - Copie o conteúdo de id_rsa.pub para o arquivo ~/.ssh/authorized_keys de cada servidor.

3. **Adicionar a Chave Privada como um Secret no GitHub**

    - No GitHub, abra o repositório onde o código do site está hospedado.

    - Vá até Settings > Secrets > New repository secret.

    - Crie um secret chamado SSH_PRIVATE_KEY e cole o conteúdo da chave privada (id_rsa).

### 2. **Configuração do GitHub Actions**
No seu repositório, crie o arquivo de workflow para automação do deploy.
   
1. Crie a estrutura de diretórios /.github/workflows/ se ainda não existir.

2. Crie o arquivo deploy.yml dentro de .github/workflows/ e adicione o seguinte conteúdo:

        name: Deploy Site para Teste e Produção

        on:
        push:
            branches:
            - staging  # Quando o código for enviado para 'staging', o deploy vai para o servidor de teste
            - main     # Quando o código for enviado para 'main', o deploy vai para o servidor de produção

        jobs:
        deploy_to_test:
            runs-on: ubuntu-latest
            if: github.ref == 'refs/heads/staging'  # Só roda se o push for na branch 'staging'
            steps:
            - name: Check out repository
                uses: actions/checkout@v2

            - name: Set up SSH
                uses: webfactory/ssh-agent@v0.5.3
                with:
                ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

            - name: Deploy para o Servidor de Teste
                run: |
                ssh -o StrictHostKeyChecking=no user@<IP_SERVER_TESTE> 'cd /path/to/test/server && git pull origin staging && docker-compose up --build'

        deploy_to_production:
            runs-on: ubuntu-latest
            if: github.ref == 'refs/heads/main'  # Só roda se o push for na branch 'main'
            steps:
            - name: Check out repository
                uses: actions/checkout@v2

            - name: Set up SSH
                uses: webfactory/ssh-agent@v0.5.3
                with:
                ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

            - name: Deploy para o Servidor de Produção
                run: |
                ssh -o StrictHostKeyChecking=no user@<IP_SERVER_PRODUCAO> 'cd /path/to/production/server && git pull origin main && docker-compose up --build'


### 3. **Configuração dos Servidores**
Configurações a serem feitas nos servidores:

1. Servidor de Teste:

    - Crie o diretório no servidor onde o site será hospedado.

    - Garanta que o Git e o Docker estejam instalados no servidor.

    - Configure o Nginx ou outro servidor web conforme necessário.

2. Servidor de Produção:

    - O mesmo processo do servidor de teste, mas este será o ambiente de produção.

3. **Verificando o Processo**

    - Após configurar o workflow, quando você fizer um push para a branch staging, o código será enviado automaticamente para o servidor de teste.

    Servidor de Teste:
    - Verifique o log do GitHub Actions para ver o progresso do deploy no servidor de teste.

    Servidor de Produção:
    - Após aprovação dos gestores, faça o merge para a branch main e o código será automaticamente enviado para o servidor de produção.

4. **Exemplo de Como o Deploy Funciona**
    - Para enviar para o servidor de teste, faça um push para a branch staging:
    
    ```bash
    git checkout staging
    git push origin staging
    ```
    - O GitHub Actions rodará automaticamente e enviará os arquivos para o servidor de teste.

    - Após aprovação, enviar para o servidor de produção: Faça o merge da branch staging para a branch main (normalmente no GitHub, após revisão).

5. **Verificando Logs e Erros**

    - Se ocorrer algum erro durante o deploy, verifique os logs do GitHub Actions para depurar o que deu errado. Isso pode incluir problemas de configuração de SSH, diretórios incorretos ou falha no comando de deploy.

## Para verificar os logs:

    Acesse a aba Actions no GitHub e selecione o workflow específico.
    Verifique as mensagens de erro ou sucesso para identificar o que precisa ser corrigido.

## Considerações Finais
    Usar GitHub Actions para automatizar o deploy para ambientes de teste e produção é uma ótima maneira de garantir que os gestores possam validar alterações antes de publicá-las oficialmente. Ao automatizar o processo, você ganha em eficiência, segurança e confiabilidade, além de garantir que o fluxo de trabalho seja transparente e controlado.
