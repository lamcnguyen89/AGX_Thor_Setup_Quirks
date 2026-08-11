To install Visual Studio Code (VS Code) for ARM on Ubuntu, you can use either the official Microsoft APT repository or download the direct  package. [1, 2]  
Method 1: Using the APT Repository (Recommended) 
This method installs the official software and ensures VS Code updates automatically when you run regular system updates. 

1. Install required dependencies: 
```bash
sudo apt update
sudo apt install software-properties-common apt-transport-https wget curl gpg -y
```
2. Import the Microsoft GPG encryption key: 
```bash
curl -fSsL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/microsoft.gpg > /dev/null
```
3. Add the official ARM64 VS Code repository: 
```bash
echo "deb [arch=arm64] https://packages.microsoft.com/repos/vscode stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```
4. Update your package list and install VS Code: [4, 5, 6, 7, 8]  
```bash
sudo apt update
sudo apt install code -y
```

Method 2: Direct  Package Installation 
Use this quick, alternative route to pull the package straight from Microsoft's servers via the command line. 

1. Download the ARM64 package: 

```bash
curl -L "https://aka.ms/linux-arm64-deb" -o vscode_arm64.deb
```
2. Install the downloaded file: [9]  
```bash
sudo apt install ./vscode_arm64.deb -y
```

Launching the Application 
Once the installation completes, start the editor by searching for "Visual Studio Code" in your system applications or by typing the following into your terminal: [1, 3, 10, 11, 12]  

```bash
code
```


[1] https://www.youtube.com/watch?v=McD2r-YPdnk&vl=en
[2] https://creatronix.de/how-to-install-vs-code-on-ubuntu-for-arm64/
[3] https://www.youtube.com/watch?v=n4cSQ-21xcM
[4] https://graphite.com/guides/install-vs-code-windows-mac-ubuntu
[5] https://www.youtube.com/watch?v=Iqp3qMeUes0
[6] https://utho.com/docs/linux/debian/how-to-install-visual-studio-code-on-debian-10
[7] https://github.com/microsoft/vscode/issues/50300
[8] https://devblogs.microsoft.com/cppblog/visual-studio-code-c-extension-arm-and-arm64-support/
[9] https://superuser.com/questions/1488578/how-do-i-install-the-linux-version-of-vscode-on-an-arm64-chromebook
[10] https://askubuntu.com/questions/1410992/vscode-not-opening-on-arm64-ubuntu-20-04
[11] https://www.youtube.com/watch?v=pCufz6iMySo
[12] https://www.youtube.com/watch?v=QslFQKGJAwQ

