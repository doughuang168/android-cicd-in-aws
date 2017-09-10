# android-cicd-in-aws 
Android CI/CD Automation in AWS 
========

CI/CD automation is achieved by using Jenkins:

 - AWS EC2 instance need to be ubuntu type, because only ubuntu have 32bit library
 - Needed software components are:
 - Java 8
 - Gradle
 - AWS CLI
 - Jenkins
 - Nodejs
 - Android SDK and lib32stdc++6 , lib32z1
 - Python


Installation Instruction
========
 


Java 8
-----------------

- sudo apt-get update
- sudo apt-get install oracle-java8-installer
- sudo add-apt-repository ppa:webupd8team/java
- sudo apt-get update
- sudo apt-get install oracle-java8-installer
- sudo update-alternatives --config java


Gradle
-----------------

- wget -c http://services.gradle.org/distributions/gradle-2.13-all.zip
- sudo apt-get install unzip
- sudo unzip gradle-2.13-all.zip -d /opt
- sudo ln -s /opt/gradle-2.13 /opt/gradle
- sudo printf "export GRADLE_HOME=/opt/gradle\nexport PATH=\$PATH:\$GRADLE_HOME/bin\n" > gradle.sh
- sudo cp gradle.sh /etc/profile.d/gradle.sh
- . /etc/profile.d/gradle.sh
- gradle –v


AWS CLI
-----------------

- sudo apt install awscli



Jenkins
-----------------

- wget -q -O - http://pkg.jenkins-ci.org/debian-stable/jenkins-ci.org.key | sudo apt-key add -
- sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
- sudo apt-get update
- sudo apt-get install jenkins
- Modify SecurityGroup of buildbox to allow inbound port 8080 
- At Your Android GitHub repo, Setting-> Integration&Service ->Add Service
- Select Jenkins (GitHub Plugin)
- enter url: http://<your-jenkins-build-box>:8080/github-webhook/



Node.js ionic cordova
-----------------

- curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
- sudo apt-get install -y nodejs
- nodejs –v
- npm –v
- sudo npm install -g ionic
- sudo npm install -g cordova



Android SDK R23
-----------------

- aws s3 cp s3://mysayapis3bucket/android-sdk_r23.0.2-linux.tgz /tmp
- cd /tmp
- tar -xvf android-sdk_r23.0.2-linux.tgz
- cd android-sdk-linux/tools
- ./android update sdk --no-ui
- mkdir –p /var/lib/jenkins/sdk
- sudo printf "export ANDROID_HOME=/var/lib/jenkins/sdk/android-sdk-linux\nexport PATH=\$PATH:\$ANDROID_HOME/tools:\$ANDROID_HOME/build-tools/23.0.1\n" > android.sh
- sudo cp android.sh /etc/profile.d/android.sh
- . /etc/profile.d/android.sh
- sudo apt-get install lib32stdc++6 
- sudo apt-get install lib32z1
- mkdir "$ANDROID_SDK/licenses" || true
- echo -e "\n8933bad161af4178b1185d1a37fbf41ea5269c55" > android-sdk-license
- echo -e "\n84831b9409646a918e30573bab4c9c91346d8abd" > android-sdk-preview-license



Python, Python3, pip, Google API Python Client
-----------------

- sudo apt-get install python3
- sudo apt-get install python-pip
- sudo apt-get install python3-pip
- pip install google-api-python-client





Jenkins Shell Scrips
-----------------
- export GRADLE_HOME=/opt/gradle
- export PATH=$PATH:$GRADLE_HOME/bin
- export ANDROID_HOME=/var/lib/jenkins/sdk/android-sdk-linux
- export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/build-tools/23.0.1
- cd device/<your device name>
- pwd
- #Pre-Build
- touch config.xml.new
- head -1 config.xml >> config.xml.new
- cp /var/lib/jenkins/key.pem ./key.p12
- sudo pip install pyOpenSSL
- sudo python /var/lib/jenkins/basic_list.py <your-android-id> >> config.xml.new
- tail -n +3 config.xml >> config.xml.new
- mv config.xml config.xml.orig
- mv config.xml.new config.xml
- #Build
- npm install
- sudo ionic platform rm android
- sudo ionic platform add https://github.com/apache/cordova-android.git#master
- sudo ionic resources --icon
- #sudo ionic resources --splash
- sudo ANDROID_HOME="/var/lib/jenkins/sdk/android-sdk-linux" ionic build android --release
- sudo cp platforms/android/build/outputs/apk/android-release-unsigned.apk platforms/android
- cp /var/lib/jenkins/my.keystore .
- sudo cp my.keystore platforms/android/
- cd platforms/android/
- sudo jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore mysay.keystore -storepass <your-pass> android-release-unsigned.apk mykey
- sudo ANDROID_HOME="/var/lib/jenkins/sdk/android-sdk-linux" $ANDROID_HOME/build-tools/23.0.1/zipalign -f -v 4 android-release-unsigned.apk <my-app>.apk

- #Post-Build / Deployment
- sudo cp /var/lib/jenkins/key.pem ./key.p12
- sudo python /var/lib/jenkins/basic_upload_apks_service_account.py <my-andorid-id> <my-app>.apk
