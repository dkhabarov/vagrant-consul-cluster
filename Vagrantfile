CONSUL_VERSION = "0.7.2"
MAX_INSTANCES = 3
addresses = Array.new

addresses[1] = "192.168.88.87"
addresses[2] = "192.168.88.95"
addresses[3] = "192.168.88.96"

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.ssh.forward_agent = true

  (1..MAX_INSTANCES).each do |instance|
    config.vm.define "node#{instance}" do |node|
      
      node.vm.hostname = "node#{instance}"
      node.vm.network "public_network", ip: addresses[instance]
      node.vm.network "forwarded_port", guest: 80, host: "850#{instance}".to_i

      node.vm.provision "shell", inline: <<-EOF
          echo node#{instance}.consul > /etc/hostname
          hostname -F /etc/hostname
          echo "#{addresses[instance]} node#{instance}.consul" >> /etc/hosts
          apt-get update
          apt-get -y upgrade
          apt-get -y install unzip nginx
          apt-get purge -y chef chef-zero puppet puppet-common
          wget https://releases.hashicorp.com/consul/#{CONSUL_VERSION}/consul_#{CONSUL_VERSION}_linux_amd64.zip -O /tmp/consul_#{CONSUL_VERSION}_linux_amd64.zip
          cd /tmp &&  unzip consul_#{CONSUL_VERSION}_linux_amd64.zip && \
          cp consul /usr/local/bin && rm -v consul_#{CONSUL_VERSION}_linux_amd64.zip
          adduser consul 
          mkdir -pv /etc/consul.d/{bootstrap,server,client}
          mkdir -pv /var/consul
          chown -v consul:consul /var/consul
          mkdir -p /opt/consul-ui && \
          cd /opt/consul-ui && \
          wget https://releases.hashicorp.com/consul/#{CONSUL_VERSION}/consul_#{CONSUL_VERSION}_web_ui.zip && \
          unzip -fv consul_#{CONSUL_VERSION}_web_ui.zip
      EOF

      if instance == 1
        node.vm.provision "shell", inline: <<-EOF
          cp -vf /vagrant/etc/consul-bootstrap.upstart.conf /etc/init/consul.conf
        EOF
      elsif instance > 1
        node.vm.provision "shell", inline: <<-EOF
          cp -vf /vagrant/etc/consul.upstart.conf /etc/init/consul.conf
        EOF
      end

      node.vm.provision "shell", inline: <<-EOF
        cp -vf /vagrant/etc/default.nginx.conf /etc/nginx/sites-available/default
        cp -vf /vagrant/etc/consul/bootstrap/config.json /etc/consul.d/bootstrap/
        cp -vf /vagrant/etc/consul/server/config.json /etc/consul.d/server/config.json
        service consul start
        service nginx restart
      EOF

      if instance > 1
        node.vm.provision "shell", inline: <<-EOF
          consul join #{addresses[1]}
        EOF
      end
    end
  end
end
