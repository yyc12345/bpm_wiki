# User Guide

This manual will guide a beginner of bpm how to use bpm correctly. But, bpm is a console application. Abviously, it is not prepared for newbee user of computer. Please comfirm that you have rich computer experience. Then, you can consider to read this guide.

I will use plain words to describe how to use bpm. If you still couldn't comperhend it. You can contact the people, who know how to use bpm, and ask them for help.

## Configuration and installation

bpm, just like common software, you need to install it. You can download compressed package on Release panel of bpm Github Repository. You should decompress the install package totally into a blank folder. Then, we get the main body of bpm. But, please wait. bpm still couldn't be run currently. You should read the following corresponding chapter which is related to your computer OS type. For common user, please read chapter Windows.

### Windows

**If you only can run app via double-clicking app icon and you don't want to know any other way to launch app. Please close this document and delete bpm totally. bpm is not suit for you.**

#### Configure install environment

First, let we install environment. [Click here](https://dotnet.microsoft.com/download) to visit .Net Core download page and download corresponding Windows installation package(Download Runtime package is enough for you).

#### Check installation

Use Windows's search function(It is at the bottom of Start Menu on Windows 7. Use Cortana on Windows 10) and search `Cmd`. When you find it, right click it and select Run As Admin.

Then, go to the folder where you store bpm. Copy this folder's path. It just like this: `D:\MySoftware\bpm`. Let we name it `xxx`.

Turn on the Cmd window which you open previously. Input the command `cd /d xxx`. `xxx` is your copied folder's path just now. Then, press Enter.

And, you can input `bpm help` and press Enter. If Cmd outputs a string of bpm help document, congratulation, you install bpm successfully. Otherwise, you should check your settings. If you are still confused about it, you can try to ask someone for help.

#### Global configuration

The method which I intorduce above is still complex to operate. Because the step is too much. But, we can let bpm to be global command. So you can use bpm command in everywhere.

Right click This PC(This PC on Windows 10, Computer on Windows 7). Select Property. Select Senior System Settings at left panel. Go to Senior tab and click Environment Value.

Select `PATH` in User Value group(If `PATH` is not existed, create it pls). Do you remember `xxx`? Add it into `PATH`.

Now, we can use bpm everywhere.

### Linux

We think you, who use Linux, know all of essential computer knowledge.

#### Configure install environment

Please go to [.Net Core download website](https://dotnet.microsoft.com/download) to ckeck out how to configure .Net Core runtime environment correctly.

#### Check installation

Open Terminal Emulator in the folder where you store bpm.

You can rub `bpm.sh help` to make sure that your bpm has been configured correctly.

#### Global configuration

I think you can do it on your own.

#### Tips

`bpm.sh` is not be tested by me. It might contain some issues(such as runtime folder issue). Limited by my ability, I couldn't fix and improve it in a short time. I will appreciate it if you can share your startup script.

### macOS

Do it by yourself. ~~I don't have enough money to buy a Apple device and test bpm.~~

## Command input

bpm's work method use command parameters. It mean that only one command will be executed during once running. 

Command formation is `bpm command-name parameter-1 parameter-2 ...`. Input it in Terminal or Cmd directly. UTF-8 support is unknow. But it support UTF-8 on my computer(Windows 10 + PowerShell).

## Command description

* `list`
  - `list`：List infomation of installed packages.
* `update`
  - `update`：Sync package database with bpm server.
* `show`
  - `show package-name`：Output specific package's info. Package-name need version.
* `search`
  - `search key-word-1 key-word-2 ...`：Search related package according to provided key words in database. It will seach package name and aka. You can provide more than one key words. The count of key words is unlimited.
* `install`
  - `install package-name`：Install specific package. Package-name can don't contain version(Install the lastest version automatically).
* `remove`
  - `remove package-name`：Remove specific package. Package-name can don't contain version(Remove all versions automatically).
* `guide`
  - `guide package-name`：Display specific package's help. Almost help will help you to comfirm the parameter of deploy. Package-name need version.
* `deploy`
  - `deploy package-name parameter`：Deploy a package. Such as map, bgm and so on. Package-name need version.
* `clean`
  - `clean`：Remove all cache files(Downloaded ZIP files and JSON files)
  - `clean package-name`：Remove specific package's cache file. Package-name can don't contain version(Remove all versions'cache files automatically).
* `config`
  - `config`：List all settings item and its value
  - `config item-name`：List specific item and its value
  - `config item-name new-value`：Change specific item's value

## Attention

* bpm will automatically establish some folders when it run firstly. Don't delete these folders in any case.
* You will get `package.db` after you execute `bpm update`. Don't delete it in any case.
* You will get `InstallRecord.db` after you install a new package successfully. Don't delete it in any case.
* bpm couldn't detect your installed Ballance. Oppositely, we suggest you should install bpm on the computer, which don't have Ballance or uninstall installed Ballance previously. And then, let bpm install Ballance for your computer. This will promise that bpm can control everything. You need to use `bpm config GamePath xxx` to point out the folder where you want to store Ballance in advance. bpm will store every of game files in that folder. If you insist that you need to use the Ballance installed on your own, you only need to use `bpm config GamePath xxx` and configure game path to be your Ballance root folder's path. But, you should make sure that your Ballance is original Ballance version, without any edition, otherwise we don't provide any support. The command of install Ballance in bpm is `bpm install ballance@v1.13`
* For above statement, we don't provide the services of downloading Ballance currently because we are in BETA mode now.
* Default language is English. Use `bpm config Language xxx` to change display language. We support `en-us` and `zh-cn` now.
* In default, bpm will connect bpm global resources server. Use `bpm config Sources IP:Port` to change default settings if you wnat to connect other servers.
* You should execute `bpm update` to update package database after run bpm firstly. And then, you can execute almost command.
* It is good on Windows environment that run bpm with admin permission. Especially your game is installed in system folder. Because bpm need change game files. bpm also need to change Registry to register Ballance when installing Ballance. Limited permission will result in some unexpected bugs and errors. We couldn't provide any help and undertake any result if you don't use admin permission.
* bpm *might* not need `root` permission on Linux. But we are not clear about whether bpm need permission when use wine to install Ballance.
