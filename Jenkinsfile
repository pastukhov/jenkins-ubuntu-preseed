pipeline {
  agent any
  environment {
    ISO_FILENAME = 'ubuntu-16.04.6-server-amd64'  
  }
  parameters {
    string(name: 'LOCALE', defaultValue: 'ru_RU.UTF-8', description: 'Locale')
  	string(name: 'FULLNAME', defaultValue: 'Ubuntu', description: 'Full Name')
  	string(name: 'USERNAME', defaultValue: 'ubuntu', description: 'Username')
    string(name: 'PUBKEY', defaultValue: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCw73YVw5JKvNvATa7EAh/bQFDV41GKj2B2/bZ5QF+qJQXb08o1az0A8dsGbxlkkXPALxzmfWKcLxoDIOYD58kkPK/+eCbE+EDi/trpQ1cdltVvlC31cwfctlCOrdKboKjwqqUKsurfJY8zFlsYBras5IxdLSk/4VxOkC6O+/3+ptw+UAY5RGdAuwDppP60qi807t7ySPmtVx90I+I31rpzizzqfI/wkUutVonZKYn9A9nsF3Xkf2MQwtbA7OWfVv/0IkXdsTatQAFqkUPhO8ZOiMhCeb4E2juO/b0jCNxyieXfkxAkpONfLM0E+0HLvDsOjWE7mvcpuZ0hFDPFYk/d jenkins@jenkins', description: 'Public SSH key that will be added to authorized_keys for this user')
    string(name: 'NETIFACE', defaultValue: 'auto', description: 'Default network interface')
    string(name: 'HOSTNAME', defaultValue: 'unassigned-hostname', description: 'Hostname')
    string(name: 'DOMAIN', defaultValue: 'unassigned-domain', description: 'Domain')
    string(name: 'TIMEZONE', defaultValue: 'Europe/Moscow', description: 'Timezone')
    string(name: 'ROOT_DEV', defaultValue: '/dev/sda', description: 'System disk')
    string(name: 'ARTIFACT_NAME', defaultValue: 'ubuntu-16.04.6-server-amd64_unattend.iso')
  }
  options {
    buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '3', daysToKeepStr: '', numToKeepStr: ''))
  }
  stages {
    stage('Download ISO') {
      when {
        expression {
          return !(fileExists("${ISO_FILENAME}.iso"))
        }
        
      }
      steps {
        sh "curl -O http://releases.ubuntu.com/16.04/${ISO_FILENAME}.iso"
      }
    }
    stage('Mount ISO') {
      steps {                  
        sh '7z x ./${ISO_FILENAME}.iso -oiso'
      }
    }
    stage('Update initrd') {
	  steps {
	    dir('initrd') {
        sh 'echo "$PUBKEY" > ./userkey.pub'
	      sh 'cat ../iso/install/initrd.gz | gzip -d > "./initrd"'
	      sh 'echo "./userkey.pub" | fakeroot cpio -o -H newc -A -F "./initrd"'
	      sh 'cat "./initrd" | gzip -9c > "../iso/install/initrd.gz"'	      
	    }
	  }    
    }
    stage('Configure') {
      steps {
        sh 'echo en > ./iso/isolinux/lang'
        sh 'cp ./preseed/server.seed ./iso/preseed'        
        sh '''
        PASSWORD=$(openssl rand -base64 32)
        echo "Password for ${USERNAME} is: \'${PASSWORD}\', make sure to change it on first boot"
        PASSWORD_CRYPT=$(mkpasswd -m sha-512 -S $(pwgen -ns 16 1) ${PASSWORD})
        PASSWORD_CRYPT_ESCAPED=$(echo "$PASSWORD_CRYPT" | sed \'s/[&/\\]/\\\\&/g\')
        sed -i "s/PASSWORD_CRYPT/${PASSWORD_CRYPT_ESCAPED}/g" ./iso/preseed/server.seed
        '''                  
      	sh 'sed -i -r "s#timeout\\s+[0-9]+#timeout 10#g" ./iso/isolinux/isolinux.cfg'        
        sh 'sed -i "s#NETIFACE#${NETIFACE}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s#LOCALE#${LOCALE}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s/FULLNAME/${FULLNAME}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s/USERNAME/${USERNAME}/g" ./iso/preseed/server.seed'        
        sh 'sed -i "s/HOSTNAME/${HOSTNAME}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s/DOMAIN/${DOMAIN}/g" ./iso/preseed/server.seed'
        sh 'sed -i "s#ROOT_DEV#${ROOT_DEV}#g" ./iso/preseed/server.seed'
        sh 'sed -i "s#TIMEZONE#${TIMEZONE}#g" ./iso/preseed/server.seed'
        sh '''        	
			MD5_SUM=$(md5sum ./iso/preseed/server.seed)
      sed -i '/menuentry "Install Ubuntu Server"/imenuentry "Autoinstall" {\\n\\
        set gfxpayload=keep\\n\\
        linux /install/vmlinuz gfxpayload=800x600x16,800x600 hostname=${HOSTNAME} --- auto=true preseed/file=/cdrom/preseed/server.seed preseed/file/checksum=$MD5_SUM quiet\\n\\
        initrd /install/initrd.gz\\n\\}' ./iso/boot/grub/grub.cfg
			sed -i "/label install/ilabel autoinstall\\n\\
			  menu label ^Autoinstall Ubuntu 16.04 Server\\n\\
			  kernel /install/vmlinuz\\n\\
			  append file=/cdrom/preseed/ubuntu-server.seed initrd=/install/initrd.gz auto=true priority=high preseed/file=/cdrom/preseed/server.seed preseed/file/checksum=$MD5_SUM --" ./iso/isolinux/txt.cfg
        '''
      }
    }
    stage('Make ISO') {
      steps {
        sh 'dd if=${ISO_FILENAME}.iso bs=512 count=1 of=./iso/isolinux/isohdpfx.bin'              
        sh 'xorriso -as mkisofs -isohybrid-mbr ./iso/isolinux/isohdpfx.bin -c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -isohybrid-gpt-basdat -o ${ARTIFACT_NAME} ./iso'
      }
      post {
        success {          
          archiveArtifacts artifacts: "${ARTIFACT_NAME}", fingerprint: true
        }
      }
    }   
  }
  post {  
	  always {
	    dir('iso') {
        deleteDir()
      }
      dir('initrd') {
        deleteDir()
      }
	  }
	}   
}
