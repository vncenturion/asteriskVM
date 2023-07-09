# Asterisk-VM
Criação e configuração de servidor asterisk com webrtc em vm linux ubuntu

## Pré-requisitos

* Hipervisor [ORACLE VirtualBox](https://www.virtualbox.org/wiki/Linux_Downloads).
* Vagrant
* Vagrantfile (contido no repositório)
* Vagrantbox ubuntu/focal64 (será baixada automaticamente ao ser executado o vagrantfile)

  O Asterisk atualmente contém duas pilhas SIP: o driver de canal SIP chan_sip original , que é uma implementação autônoma completa, esteve presente em todas as versões anteriores do Asterisk e não recebe mais suporte principal , e a pilha SIP chan_pjsip mais recente, baseada no " pjproject " de Teluu pilha SIP.
  
  A partir do Asterisk 13.8.0, uma versão estável do pjproject foi incluída no diretório ./third-party do Asterisk e habilitada com a --with-pjproject-bundledopção ./configure. A partir do Asterisk 15.0.0, ele é ativado por padrão. Na atividade é instalada a versão atual do asterisk (20.3.x), de modo que não necessidade de instalar e configurar o pjproject.

## Executando o Vagrantfile

1. Criar um diretório para o vagrantfile:

   ```shell
      > mkdir ~/asterisk-vm
   ```

2. Baixar o vagrantfile para o direitório

   ```shell
      > cd ~/asterisk-vm
      > wget https://raw.githubusercontent.com/vncenturion/asteriskVM/main/Vagrantfile
   ```
   
3. Criar a máquina a partir do vagrantfile

   ```shell
      > vagrant up
   ```

4. Acessar a máquina

   ```shell
      > vagrant up
   ```
   
5. Criando certificados

   ```shell
      > cd /usr/src/asterisk*
      > sudo contrib/scripts/ast_tls_cert -C pbx.<dominio.com> -O "<Nome da Organização>" -b 2048 -d /etc/asterisk/keys
   ```

   O asterisk conta com um scripct de geração de certificado autoassinado, contudo, só é aconselhado sua utilização em ambientes de teste.

6. Alterando o arquivo /etc/asterisk/http.conf

   Descomente as linhas correspondente a habilitação do uso de TLS e suas configurações.

   ```shell
      > sudo vim /etc/asterisk/http.conf
   ```

8. Reiniciando o asterisk

   ```shell
      > sudo systemctl stop asterisk
      > sudo systemctl start asterisk
   ```

   Observação: em alguns testes, quando utilizado o comando de reinicialização, o serviço apresentou problemas, exibindo um alerta quando visualizado o status. Razão pela qual se preferiu interromper o serviço e depois inicializá-lo.

## Analisando as configurações adotadas:

1. Servidor HTTP integrado:

   `/etc/asterisk/http.conf`
   ```
      [general]
      enabled=yes
      bindaddr=0.0.0.0
      bindport=8088
      tlsenable=yes
      tlsbindaddr=0.0.0.0:8089
      tlscertfile=/etc/asterisk/keys/asterisk.crt
      tlsprivatekey=/etc/asterisk/keys/asterisk.key
   ```

2. Definição de transporte PJSIP básico:

   `/etc/asterisk/pjsip.conf`
   ```
      [transport-wss]
      type=transport
      protocol=wss
      bind=0.0.0.0
   ```

3. Definição de transporte PJSIP básico:

   `/etc/asterisk/pjsip.conf`
   ```
      [webrtc_client]
      type=aor
      max_contacts=5
      remove_existing=yes
  
      [webrtc_client]
      type=auth
      auth_type=userpass
      username=webrtc_client
      password=webrtc_client ; This is a completely insecure password!  Do NOT expose this
                             ; system to the Internet without utilizing a better password.
 
      [webrtc_client]
      type=endpoint
      aors=webrtc_client
      auth=webrtc_client
      dtls_auto_generate_cert=yes
      webrtc=yes
      ; Setting webrtc=yes is a shortcut for setting the following options:
      ; use_avpf=yes
      ; media_encryption=dtls
      ; dtls_verify=fingerprint
      ; dtls_setup=actpass
      ; ice_support=yes
      ; media_use_received_transport=yes
      ; rtcp_mux=yes
      context=default
      disallow=all
      allow=opus,ulaw
   ```

## Considerações

   Seu cliente WebRTC deve ser capaz de registrar e fazer chamadas. No entanto, se você usou certificados autoassinados, seu navegador pode não permitir a conexão e, como a tentativa não é de um URI normal fornecido pelo usuário, o usuário pode nem ser notificado de que há um problema. Você pode conseguir que o navegador aceite o certificado visitando " https://pbx.<dominio.com>:8089/ws" diretamente. 
   
   Isso geralmente resultará em um aviso do navegador e pode dar a você a oportunidade de aceitar o certificado autoassinado e/ou criar uma exceção. Se você gerou seu certificado de uma autoridade de certificação local pré-existente, também pode importar o certificado dessa autoridade de certificação para seu armazenamento confiável, mas esse procedimento está além do escopo deste documento.
