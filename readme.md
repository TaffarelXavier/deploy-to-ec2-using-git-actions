# Passo 1)

Aqui estão os passos para configurar a instância EC2 e permitir a conexão SSH:

1. **Crie uma instância EC2**:
   - Crie uma instância EC2 com Ubuntu 20.04 na sua conta AWS.
   - Certifique-se de que a instância tenha acesso à internet e uma chave SSH para se conectar a ela.

2. **Adicione as regras de segurança**:
   - Na página de detalhes da instância EC2, vá para a guia "Security" (Segurança) na seção "Networking".
   - Verifique se há uma regra de entrada (Inbound) que permite o tráfego SSH na porta 22. Caso não haja, adicione uma regra para permitir o tráfego SSH vindo do seu IP ou de qualquer IP (para testes) na porta 22.

3. **Conecte-se à instância via SSH**:
   - Abra um terminal local no seu computador.
   - Use o comando SSH para se conectar à instância EC2. Substitua `<ENDEREÇO_IP>` pelo endereço IP público da sua instância EC2.

   ```bash
   ssh -i /caminho/para/sua/chave.pem ubuntu@<ENDEREÇO_IP>
   ```

   Certifique-se de substituir "/caminho/para/sua/chave.pem" pelo caminho correto para o arquivo de chave SSH (.pem) que você usou ao criar a instância EC2.

4. **Atualize o sistema**:
   - Uma vez conectado à instância via SSH, execute os seguintes comandos para atualizar o sistema:

   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

5. **Configure as chaves SSH**:
   - No seu terminal local, execute o seguinte comando para gerar um novo par de chaves SSH:

   ```bash
   ssh-keygen -m PEM -t rsa -b 4096
   ```
   
   - Siga as instruções para salvar a chave no diretório `~/.ssh/id_rsa` (chave privada) e `~/.ssh/id_rsa.pub` (chave pública).

   - No servidor Ubuntu, adicione sua chave pública ao arquivo `~/.ssh/authorized_keys` com o seguinte comando:

   ```bash
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   ```

   - Configure as permissões corretas no arquivo `~/.ssh` e `~/.ssh/authorized_keys`:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

6. **Reinicie o serviço SSH**:
   - Reinicie o serviço SSH no servidor Ubuntu com o seguinte comando:

   ```bash
   sudo service ssh restart
   ```

Agora, você deve poder se conectar à sua instância EC2 usando o comando SSH mencionado anteriormente, substituindo `<ENDEREÇO_IP>` pelo endereço IP público da sua instância EC2. Após configurar corretamente a instância EC2, você pode prosseguir com a configuração do Git


## Passo 2)
Criar uma secret key no Git hub

cat /home/ubuntu/.ssh/id_rsa


Passo 3)

Como adicionar o usuário ubuntu para ter permissões?
Para adicionar permissões para o usuário "ubuntu" no diretório "/var/www/app", você pode seguir estas etapas:

1. Conecte-se à sua instância EC2 usando SSH como usuário "ubuntu". Por exemplo:
   ```
   ssh ubuntu@<endereço_IP_da_instância>
   ```

2. Verifique as permissões atuais do diretório "/var/www/app" usando o comando `ls -l`:
   ```
   ls -l /var/www/app
   ```

3. Se o diretório "/var/www/app" pertencer a outro usuário, como "root", você precisará alterar o proprietário para o usuário "ubuntu". Use o comando `sudo chown` para fazer isso:
   ```
   sudo chown -R ubuntu:ubuntu /var/www/app
   ```

   Esse comando alterará o proprietário e o grupo do diretório "/var/www/app" para "ubuntu".

4. Verifique novamente as permissões do diretório usando `ls -l` para garantir que o usuário "ubuntu" agora tenha permissões adequadas:
   ```
   ls -l /var/www/app
   ```

5. Agora, o usuário "ubuntu" deve ter as permissões necessárias para acessar e gravar no diretório "/var/www/app". Tente executar o comando rsync novamente e verifique se o erro de permissão negada foi resolvido.

Certifique-se de usar os comandos com cuidado e esteja ciente das implicações de segurança ao conceder permissões em diretórios específicos. Certifique-se também de ajustar o caminho do diretório conforme necessário, se estiver diferente no seu ambiente.

Passo 4)

name: Push-to-EC2
on:
  push:
    branches:
      - main
jobs:
  deploy:
    name: Deploy to EC2
    runs-on: ubuntu-latest
    steps:
    - name: Checkout the files
      uses: actions/checkout@v3
    - name: Copy files with SSH
      uses: easingthemes/ssh-deploy@main
      env:
        SSH_PRIVATE_KEY: ${{secrets.EC2_SSH_KEY}}
        ARGS: "-rltgoDzvO --delete"
        SOURCE: "./"
        REMOTE_HOST: "ec2-184-73-57-154.compute-1.amazonaws.com"
        REMOTE_USER: "ubuntu"
        TARGET: "/var/www/demo-silig"
