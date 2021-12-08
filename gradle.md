# Building and Compiling kotlin based android app on Centos

1. Update the package repository to ensure you download the latest software:
```sudo yum update && sudo yum upgrade -y```
2. Now we have to install openjdk for gradle.
```sudo yum install java-11-openjdk-devel```
3. Verify java is installed by running 
```java -version```
![image.png](/.attachments/image-bd14ac57-14c8-43cd-bb5c-0ebe02d28283.png)
4. Download latest version of gradle from https://gradle.org/releases/
```wget https://downloads.gradle-dn.com/distributions/gradle-7.3.1-bin.zip -P /tmp```
5. Once downloaded you need **unzip** to extract .zip file. If not download it first 
```sudo yum install unzip -y```
6. Unzip the gradle archive
```sudo unzip -d /opt/gradle /tmp/gradle-7.3.1-bin.zip```
7. Verify gradle files are extracted by ```ls /opt/gradle/gradle-7.3.1```
![image.png](/.attachments/image-25c0ada0-ce58-4898-bd5d-ec001728a0d6.png)
8. Now we need to setup environment variable for gradle and add it to Path. So, create a new file inside ```/etc/profile.d/``` directory.
```sudo vi /etc/profile.d/gradle.sh```
9. Inside **gradle.sh** add the path to the binaries and append it to the default path.
```export GRADLE_HOME=/opt/gradle/gradle-7.3.1```
```export PATH=${PATH}:${GRADLE_HOME}/bin```
![image.png](/.attachments/image-4e8f18bf-a3fe-49a3-9a3c-9a9a0d9f45ab.png)
10. Add execution permission to the created .sh file.
```sudo chmod a+x /etc/profile.d/gradle.sh```
11. Load the environment variable using source.
```source /etc/profile.d/gradle.sh```
12. Now run ```gradle -v```
![image.png](/.attachments/image-dfb9bbd6-96ec-4cf0-b2d2-3abdbfa270a3.png)
13. Now we need to install Android managesdk and other build tools in order to build our apk. Go to https://developer.android.com/studio/#downloads and find the latest commandline utility for linux
```wget https://dl.google.com/android/repository/commandlinetools-linux-7583922_latest.zip -P /tmp```
14. Create a dir in ```/var/lib/android-sdk```
Run
```sudo unzip /tmp/commandlinetools-linux-7583922_latest.zip -d /var/lib/android-sdk```
15. After unzipping go to ```/var/lib/android-sdk```
![image.png](/.attachments/image-a430bff8-f4c6-4fd4-992d-c30441b0af93.png)
16. Inside the directory **cmdline-tools** create another directory called **tools**
```sudo mkdir tools```
```sudo mv * tools```
>Note sudo mv* tools would move all the files inside cmdline-tools to tools directory but will fail for the tool directory itself but we won't want that so we will ignore it.
17. Once everything is inside the tools directory your folder structure would be similar to 
``` /var/lib/android-sdk/cmdline-tools/tools/```
18. Now we have to add the binaries of android-sdk to our path variable. So we can invoke managesdk.
```sudo vi /etc/profile.d/android.sh```
19. Add these to android.sh
```export ANDROID_HOME=/var/lib/android-sdk```
```export PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/tools/bin```
20. Add permissions ```sudo chmod a+x /etc/profile.d/android.sh``` and load the env variables ```source /etc/profile.d/android.sh```
21. Once loaded now we have to install build dependencies for our android sdk. Since, /var/lib/ is write protected so in my case **centos** user can't download the dependencies into var directly, so you have to bind centos path variable to root path with sudo.
```sudo -E env "PATH=$PATH" sdkmanager --update```
```sudo -E env "PATH=$PATH" sdkmanager "build-tools;31.0.0"```
22. Now if you try to build with the installed build dependencies you might get **"Installed Build Tools revision 31.0.0 is corrupted"** In order to fix this 
```cd /var/lib/android-sdk/build-tools/31.0.0```
Rename ```sudo mv d8 dx ```
Rename ```sudo mv ./lib/d8.jar ./lib/dx.jar```
23. Now we are all set, go to your gradle kotlin app directory.
Run ```gradle wrapper```
Then ```sudo -E env "PATH=$PATH" ./gradlew tasks```
24. To create apk for debug (self signed)
``` sudo -E env "PATH=$PATH" ./gradlew assembleDebug```
25. If successful it will produce the apk in  outputs directory with debug.apk
![image.png](/.attachments/image-84b59de5-f1e8-4c9f-96bc-dd202c7e4cda.png)
![image.png](/.attachments/image-00d6917e-1747-40e3-8de5-576bcd900c0b.png)
26. We can run the debug apk in an emulator or android device.
