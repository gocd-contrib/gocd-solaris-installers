Vagrant.configure('2') do |config|

  full_version = ENV['GO_VERSION']

  version, epoch = full_version.split('-') if full_version

  config.vm.define 'package' do |package_vm|
    package_vm.vm.box = 'tnarik/solaris10-minimal'

    package_vm.vm.provider 'virtualbox' do |vb|
      vb.gui = false
      vb.memory = '1024'
      vb.cpus = 1
    end

    package_vm.vm.provision :shell, inline: 'sudo pkgutil --catalog'
    package_vm.vm.provision :shell, inline: 'sudo pkgutil --upgrade -y CSWcacertificates CSWcas-sslcert CSWwget CSWopenssl-utils coreutils'

    if full_version
      version, epoch = full_version.split('-')
      package_vm.vm.provision 'shell', inline: <<-SHELL
          set -ex
          cd /vagrant

          echo "Downloading go-server binary"
          wget -cq https://download.go.cd/binaries/#{full_version}/generic/go-server-#{full_version}.zip
          wget -cq https://download.go.cd/binaries/#{full_version}/generic/go-server-#{full_version}.zip.sha1sum

          echo "Downloading go-agent binary"
          wget -cq https://download.go.cd/binaries/#{full_version}/generic/go-agent-#{full_version}.zip
          wget -cq https://download.go.cd/binaries/#{full_version}/generic/go-agent-#{full_version}.zip.sha1sum

          echo "Verifying checksums"
          gsha1sum -c *.sha1sum
        SHELL

      package_vm.vm.provision :shell, inline: <<-SHELL
          set -ex
          echo foo
          # clean and prepare the working directory
          rm    -rf target/sol
          mkdir -p  target/sol

          mkdir -p  target/sol/go-{agent,server}-#{version}
          mkdir -p  target/sol/go-{agent,server}-#{version}/install

          echo "Extracting go-agent and go-server zip files"

          rm -rf go-agent-#{version} go-server-#{version}
          unzip -qo /vagrant/go-server-#{full_version}.zip
          unzip -qo /vagrant/go-agent-#{full_version}.zip

          mv go-agent-#{version}/* target/sol/go-agent-#{version}/
          mv go-server-#{version}/* target/sol/go-server-#{version}/

          # Copy over files that must be packaged
          cp /vagrant/installers/go-agent/{svc.xml,go-agent,Prototype}  target/sol/go-agent-#{version}/
          mv target/sol/go-agent-#{version}/config target/sol/go-agent-#{version}/install
          cp /vagrant/installers/go-server/{svc.xml,go-agent,Prototype} target/sol/go-server-#{version}/

          chmod 755 target/sol/go-agent-#{version}/go-agent
          chmod 755 target/sol/go-server-#{version}/go-server

          # Create the Prototype (manifest) file
          (
            set -e
            cd target/sol/go-agent-#{version}
            /usr/bin/pkgproto .=go-agent >> Prototype
            gsed -i -e 's/^f none go-agent\\/Prototype.*$\\\\n//g' Prototype
          )

          (
            set -e
            cd target/sol/go-server-#{version}
            /usr/bin/pkgproto .=go-server >> Prototype
            gsed -i -e 's/^f none go-server\\/Prototype.*$\\\\n//g' Prototype
          )

          # Copy over install scripts
          cp /vagrant/installers/go-agent/*  target/sol/go-agent-#{version}/
          cp /vagrant/installers/go-server/*  target/sol/go-server-#{version}/

          gsed -i -e 's/ root root$/ root other/g' target/sol/go-{agent,server}-#{version}/Prototype
          gsed -i -e 's/@VERSION@/#{version}.#{epoch}/g' target/sol/go-{agent,server}-#{version}/pkginfo

          cp target/sol/go-agent-#{version}/LICENSE target/sol/go-agent-#{version}/copyright
          cp target/sol/go-server-#{version}/LICENSE target/sol/go-server-#{version}/copyright

          (
            set -e
            cd target/sol/go-agent-#{version}
            /usr/bin/pkgmk -o -r . -d .. -f Prototype
          )
          (
            set -e
            cd target/sol/go-server-#{version}
            /usr/bin/pkgmk -o -r . -d .. -f Prototype
          )

          /usr/bin/pkgtrans -s target/sol go-agent-#{version}.#{epoch}-solaris TWSgo-agent
          /usr/bin/pkgtrans -s target/sol go-server-#{version}.#{epoch}-solaris TWSgo-server

          gzip target/sol/go-agent-#{version}.#{epoch}-solaris
          gzip target/sol/go-server-#{version}.#{epoch}-solaris
          mv target/sol/go-{agent,server}-#{version}.#{epoch}-solaris.gz /vagrant
        SHELL
    else
      $stderr.puts 'Please specify GO_VERSION environment variable to provision. For e.g.'
      $stderr.puts '$ GO_VERSION=16.9.0-4001 vagrant [up|provision]'
    end
  end

  config.vm.define 'test' do |test_vm|
    test_vm.vm.box = 'tnarik/solaris10-minimal'

    test_vm.vm.provider 'virtualbox' do |vb|
      vb.gui = false
      vb.memory = '4096'
      vb.cpus = 2
    end

    test_vm.vm.provision :shell, inline: 'sudo pkgutil --catalog'
    test_vm.vm.provision :shell, inline: "sudo pkgutil --upgrade -y jre8 coreutils"
    test_vm.vm.provision :shell, inline: "sudo ln -sf /opt/csw/java/jre/jre8/bin/java /usr/bin/java"
    test_vm.vm.provision :shell, inline: "sudo ln -sf /opt/csw/bin/greadlink /usr/bin/readlink"
    test_vm.vm.provision :shell, inline: "cat /vagrant/go-agent-#{version}.#{epoch}-solaris.gz | gunzip > go-agent-#{version}.#{epoch}-solaris"
    test_vm.vm.provision :shell, inline: "cat /vagrant/go-server-#{version}.#{epoch}-solaris.gz | gunzip > go-server-#{version}.#{epoch}-solaris"

    test_vm.vm.provision :shell, inline: "sudo pkgadd -y -d go-agent-#{version}.#{epoch}-solaris"
    test_vm.vm.provision :shell, inline: "sudo pkgadd -y -d go-server-#{version}.#{epoch}-solaris"
  end
end
