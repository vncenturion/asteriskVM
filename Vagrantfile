# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  
  # Configurações e definição da MV
  config.vm.box = "ubuntu/focal64"
  config.vm.network "public_network"
  config.vm.define "vm_asterisk"
  
  # Provisionamento da máquina virtual com um script shell
  config.vm.provision "shell", inline: <<-SHELL
    sudo -i

    # Atualiza os pacotes do sistema
    apt-get update
    apt-get upgrade -y

    # Instalação de pacotes necessários para o Asterisk
    apt-get install -y aptitude
    aptitude install libsrtp2-dev libssl-dev libopus-dev gcc build-essential wget git gcc++ make uuid-dev -y
    
    # Download e instalação do Asterisk
    cd /usr/src
    wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
    sudo tar -zxvf asterisk-20-current.tar.gz
    cd /usr/src/asterisk-20.*/contrib/scripts
    ./install_prereq install
    ./get_mp3_source.sh
    cd /usr/src/asterisk-20.*
    ./configure
    make
    make install
    make samples
    make config
    make install-logrotate
    
    # Faz backup do arquivo http.conf e cria uma nova configuração
    mv /etc/asterisk/http.conf /etc/asterisk/http.conf.bkp
    mkdir /etc/asterisk/keys
    echo -e "[general]\nenabled=yes\nbindaddr=0.0.0.0\nbindport=8088\n;tlsenable=yes\n;tlsbindaddr=0.0.0.0:8089\n;tlscertfile=/etc/asterisk/keys/asterisk.crt\n;tlsprivatekey=/etc/asterisk/keys/asterisk.key" | sudo tee -a /etc/asterisk/http.conf

    # Faz backup do arquivo pjsip.conf e cria uma nova configuração
    mv /etc/asterisk/pjsip.conf /etc/asterisk/pjsip.conf.bkp
    echo -e "[transport-wss]\ntype=transport\nprotocol=wss\nbind=0.0.0.0\n; All other transport parameters are ignored for wss transports.\n\n" | sudo tee /etc/asterisk/pjsip.conf
    echo -e "[webrtc_client]\ntype=aor\nmax_contacts=5\nremove_existing=yes\n\n[webrtc_client]\ntype=auth\nauth_type=userpass\nusername=webrtc_client\npassword=webrtc_client ; This is a completely insecure password!  Do NOT expose this\n                       ; system to the Internet without utilizing a better password.\n\n[webrtc_client]\ntype=endpoint\naors=webrtc_client\nauth=webrtc_client\ndtls_auto_generate_cert=yes\nwebrtc=yes\n; Setting webrtc=yes is a shortcut for setting the following options:\n; use_avpf=yes\n; media_encryption=dtls\n; dtls_verify=fingerprint\n; dtls_setup=actpass\n; ice_support=yes\n; media_use_received_transport=yes\n; rtcp_mux=yes\ncontext=default\ndisallow=all\nallow=opus,ulaw" | sudo tee -a /etc/asterisk/pjsip.conf
    
    # Inicia e habilita o serviço do Asterisk
    systemctl start asterisk
    systemctl enable asterisk

  SHELL

end