- 👋 Hi, I’m @MartinDojcinoski23
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
MartinDojcinoski23/MartinDojcinoski23 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
# coding=utf-8
import os
import sys
import webbrowser
from platform import system
from traceback import print_exc
from typing import Any
from typing import Callable
from typing import List
from typing import Tuple


def clear_screen():
    if system() == "Linux":
        os.system("clear")
    if system() == "Windows":
        os.system("cls")


def validate_input(ip, val_range):
    try:
        ip = int(ip)
        if ip in val_range:
            return ip
        else:
            return None
    except:
        return None


class HackingTool(object):
    # About the HackingTool
    TITLE: str = ""  # used to show info in the menu
    DESCRIPTION: str = ""

    INSTALL_COMMANDS: List[str] = []
    INSTALLATION_DIR: str = ""

    UNINSTALL_COMMANDS: List[str] = []

    RUN_COMMANDS: List[str] = []

    OPTIONS: List[Tuple[str, Callable]] = []

    PROJECT_URL: str = ""

    def __init__(self, options = None, installable: bool = True,
                 runnable: bool = True):
        if options is None:
            options = []
        if isinstance(options, list):
            self.OPTIONS = []
            if installable:
                self.OPTIONS.append(('Install', self.install))
            if runnable:
                self.OPTIONS.append(('Run', self.run))
            self.OPTIONS.extend(options)
        else:
            raise Exception(
                "options must be a list of (option_name, option_fn) tuples")

    def show_info(self):
        desc = self.DESCRIPTION
        if self.PROJECT_URL:
            desc += '\n\t[*] '
            desc += self.PROJECT_URL
        os.system(f'echo "{desc}"|boxes -d boy | lolcat')

    def show_options(self, parent = None):
        clear_screen()
        self.show_info()
        for index, option in enumerate(self.OPTIONS):
            print(f"[{index + 1}] {option[0]}")
        if self.PROJECT_URL:
            print(f"[{98}] Open project page")
        print(f"[{99}] Back to {parent.TITLE if parent is not None else 'Exit'}")
        option_index = input("Select an option : ")
        try:
            option_index = int(option_index)
            if option_index - 1 in range(len(self.OPTIONS)):
                ret_code = self.OPTIONS[option_index - 1][1]()
                if ret_code != 99:
                    input("\n\nPress ENTER to continue:")
            elif option_index == 98:
                self.show_project_page()
            elif option_index == 99:
                if parent is None:
                    sys.exit()
                return 99
        except (TypeError, ValueError):
            print("Please enter a valid option")
            input("\n\nPress ENTER to continue:")
        except Exception:
            print_exc()
            input("\n\nPress ENTER to continue:")
        return self.show_options(parent = parent)

    def before_install(self):
        pass

    def install(self):
        self.before_install()
        if isinstance(self.INSTALL_COMMANDS, (list, tuple)):
            for INSTALL_COMMAND in self.INSTALL_COMMANDS:
                os.system(INSTALL_COMMAND)
            self.after_install()

    def after_install(self):
        print("Successfully installed!")

    def before_uninstall(self) -> bool:
        """ Ask for confirmation from the user and return """
        return True

    def uninstall(self):
        if self.before_uninstall():
            if isinstance(self.UNINSTALL_COMMANDS, (list, tuple)):
                for UNINSTALL_COMMAND in self.UNINSTALL_COMMANDS:
                    os.system(UNINSTALL_COMMAND)
            self.after_uninstall()

    def after_uninstall(self):
        pass

    def before_run(self):
        pass

    def run(self):
        self.before_run()
        if isinstance(self.RUN_COMMANDS, (list, tuple)):
            for RUN_COMMAND in self.RUN_COMMANDS:
                os.system(RUN_COMMAND)
            self.after_run()

    def after_run(self):
        pass

    def is_installed(self, dir_to_check = None):
        print("Unimplemented: DO NOT USE")
        return "?"

    def show_project_page(self):
        webbrowser.open_new_tab(self.PROJECT_URL)


class HackingToolsCollection(object):
    TITLE: str = ""  # used to show info in the menu
    DESCRIPTION: str = ""
    TOOLS = []  # type: List[Any[HackingTool, HackingToolsCollection]]

    def __init__(self):
        pass

    def show_info(self):
        os.system("figlet -f standard -c {} | lolcat".format(self.TITLE))
        # os.system(f'echo "{self.DESCRIPTION}"|boxes -d boy | lolcat')
        # print(self.DESCRIPTION)

    def show_options(self, parent = None):
        clear_screen()
        self.show_info()
        for index, tool in enumerate(self.TOOLS):
            print(f"[{index} {tool.TITLE}")
        print(f"[{99}] Back to {parent.TITLE if parent is not None else 'Exit'}")
        tool_index = input("Choose a tool to proceed: ")
        try:
            tool_index = int(tool_index)
            if tool_index in range(len(self.TOOLS)):
                ret_code = self.TOOLS[tool_index].show_options(parent = self)
                if ret_code != 99:
                    input("\n\nPress ENTER to continue:")
            elif tool_index == 99:
                if parent is None:
                    sys.exit()
                return 99
        except (TypeError, ValueError):
            print("Please enter a valid option")
            input("\n\nPress ENTER to continue:")
        except Exception as e:
            print_exc()
            input("\n\nPress ENTER to continue:")
        return self.show_options(parent = parent)

# coding=utf-8
import re

from core import HackingTool
from core import HackingToolsCollection
from main import all_tools


def sanitize_anchor(s):
    return re.sub(r"\W", "-", s.lower())


def get_toc(tools, indentation = ""):
    md = ""
    for tool in tools:
        if isinstance(tool, HackingToolsCollection):
            md += (indentation + "- [{}](#{})\n".format(
                tool.TITLE, sanitize_anchor(tool.TITLE)))
            md += get_toc(tool.TOOLS, indentation = indentation + '    ')
    return md


def get_tools_toc(tools, indentation = "##"):
    md = ""
    for tool in tools:
        if isinstance(tool, HackingToolsCollection):
            md += (indentation + "# {}\n".format(tool.TITLE))
            md += get_tools_toc(tool.TOOLS, indentation = indentation + '#')
        elif isinstance(tool, HackingTool):
            if tool.PROJECT_URL:
                md += ("- [{}]({})\n".format(tool.TITLE, tool.PROJECT_URL))
            else:
                md += ("- {}\n".format(tool.TITLE))
    return md


def generate_readme():
    toc = get_toc(all_tools[:-1])
    tools_desc = get_tools_toc(all_tools[:-1])

    with open("README_template.md") as fh:
        readme_template = fh.read()

    readme_template = readme_template.replace("{{toc}}", toc)
    readme_template = readme_template.replace("{{tools}}", tools_desc)

    with open("README.md", "w") as fh:
        fh.write(readme_template)


if __name__ == '__main__':
    generate_readme()

##!/usr/bin/env python3
# -*- coding: UTF-8 -*-
# Version 1.1.0
import os
import webbrowser
from platform import system
from time import sleep

from core import HackingToolsCollection
from tools.anonsurf import AnonSurfTools
from tools.ddos import DDOSTools
from tools.exploit_frameworks import ExploitFrameworkTools
from tools.forensic_tools import ForensicTools
from tools.information_gathering_tools import InformationGatheringTools
from tools.other_tools import OtherTools
from tools.payload_creator import PayloadCreatorTools
from tools.phising_attack import PhishingAttackTools
from tools.post_exploitation import PostExploitationTools
from tools.remote_administration import RemoteAdministrationTools
from tools.reverse_engineering import ReverseEngineeringTools
from tools.sql_tools import SqlInjectionTools
from tools.steganography import SteganographyTools
from tools.tool_manager import ToolManager
from tools.webattack import WebAttackTools
from tools.wireless_attack_tools import WirelessAttackTools
from tools.wordlist_generator import WordlistGeneratorTools
from tools.xss_attack import XSSAttackTools

logo = """\033[33m
   ▄█    █▄       ▄████████  ▄████████    ▄█   ▄█▄  ▄█  ███▄▄▄▄      ▄██████▄           ███      ▄██████▄   ▄██████▄   ▄█       
  ███    ███     ███    ███ ███    ███   ███ ▄███▀ ███  ███▀▀▀██▄   ███    ███      ▀█████████▄ ███    ███ ███    ███ ███       
  ███    ███     ███    ███ ███    █▀    ███▐██▀   ███▌ ███   ███   ███    █▀          ▀███▀▀██ ███    ███ ███    ███ ███       
 ▄███▄▄▄▄███▄▄   ███    ███ ███         ▄█████▀    ███▌ ███   ███  ▄███                 ███   ▀ ███    ███ ███    ███ ███       
▀▀███▀▀▀▀███▀  ▀███████████ ███        ▀▀█████▄    ███▌ ███   ███ ▀▀███ ████▄           ███     ███    ███ ███    ███ ███       
  ███    ███     ███    ███ ███    █▄    ███▐██▄   ███  ███   ███   ███    ███          ███     ███    ███ ███    ███ ███       
  ███    ███     ███    ███ ███    ███   ███ ▀███▄ ███  ███   ███   ███    ███          ███     ███    ███ ███    ███ ███▌    ▄ 
  ███    █▀      ███    █▀  ████████▀    ███   ▀█▀ █▀    ▀█   █▀    ████████▀          ▄████▀    ▀██████▀   ▀██████▀  █████▄▄██ 
                                         ▀                                                                            ▀                             
                                    \033[34m[✔] https://github.com/Z4nzu/hackingtool   [✔]
                                    \033[34m[✔]            Version 1.1.0               [✔]
                                    \033[91m[X] Please Don't Use For illegal Activity  [X]
\033[97m """

all_tools = [
    AnonSurfTools(),
    InformationGatheringTools(),
    WordlistGeneratorTools(),
    WirelessAttackTools(),
    SqlInjectionTools(),
    PhishingAttackTools(),
    WebAttackTools(),
    PostExploitationTools(),
    ForensicTools(),
    PayloadCreatorTools(),
    ExploitFrameworkTools(),
    ReverseEngineeringTools(),
    DDOSTools(),
    RemoteAdministrationTools(),
    XSSAttackTools(),
    SteganographyTools(),
    OtherTools(),
    ToolManager()
]


class AllTools(HackingToolsCollection):
    TITLE = "All tools"
    TOOLS = all_tools

    def show_info(self):
        print(logo + '\033[0m \033[97m')


if __name__ == "__main__":
    try:
        if system() == 'Linux':
            fpath = "/home/hackingtoolpath.txt"
            if not os.path.exists(fpath):
                os.system('clear')
                # run.menu()
                print("""
                        [@] Set Path (All your tools will be installed in that directory)
                        [1] Manual 
                        [2] Default
                """)
                choice = input("Z4nzu =>> ")

                if choice == "1":
                    inpath = input("Enter Path (with Directory Name) >> ")
                    with open(fpath, "w") as f:
                        f.write(inpath)
                    print(f"Successfully Set Path to: {inpath}")
                elif choice == "2":
                    autopath = "/home/hackingtool/"
                    with open(fpath, "w") as f:
                        f.write(autopath)
                    print(f"Your Default Path Is: {autopath}")
                    sleep(3)
                else:
                    print("Try Again..!!")
                    exit(0)

            with open(fpath) as f:
                archive = f.readline()
                if not os.path.exists(archive):
                    os.mkdir(archive)
                os.chdir(archive)
                all_tools = AllTools()
                all_tools.show_options()

        # If not Linux and probably Windows
        elif system() == "Windows":
            print(
                "\033[91m Please Run This Tool On A Debian System For Best Results " "\e[00m")
            sleep(2)
            webbrowser.open_new_tab("https://tinyurl.com/y522modc")

        else:
            print("Please Check Your System or Open New Issue ...")

    except KeyboardInterrupt:
        print("\nExiting ..!!!")
        sleep(2)

#!/bin/bash
clear

BLACK='\e[30m'
RED='\e[31m'
GREEN='\e[92m'
YELLOW='\e[33m'
ORANGE='\e[93m'
BLUE='\e[34m'
PURPLE='\e[35m'
CYAN='\e[36m'
WHITE='\e[37m'
NC='\e[0m'
purpal='\033[35m'

echo -e "${ORANGE} "
echo ""
echo "   ▄█    █▄       ▄████████  ▄████████    ▄█   ▄█▄  ▄█  ███▄▄▄▄      ▄██████▄           ███      ▄██████▄   ▄██████▄   ▄█       ";
echo "  ███    ███     ███    ███ ███    ███   ███ ▄███▀ ███  ███▀▀▀██▄   ███    ███      ▀█████████▄ ███    ███ ███    ███ ███       ";
echo "  ███    ███     ███    ███ ███    █▀    ███▐██▀   ███▌ ███   ███   ███    █▀          ▀███▀▀██ ███    ███ ███    ███ ███       ";
echo " ▄███▄▄▄▄███▄▄   ███    ███ ███         ▄█████▀    ███▌ ███   ███  ▄███                 ███   ▀ ███    ███ ███    ███ ███       ";
echo "▀▀███▀▀▀▀███▀  ▀███████████ ███        ▀▀█████▄    ███▌ ███   ███ ▀▀███ ████▄           ███     ███    ███ ███    ███ ███       ";
echo "  ███    ███     ███    ███ ███    █▄    ███▐██▄   ███  ███   ███   ███    ███          ███     ███    ███ ███    ███ ███       ";
echo "  ███    ███     ███    ███ ███    ███   ███ ▀███▄ ███  ███   ███   ███    ███          ███     ███    ███ ███    ███ ███▌    ▄ ";
echo "  ███    █▀      ███    █▀  ████████▀    ███   ▀█▀ █▀    ▀█   █▀    ████████▀          ▄████▀    ▀██████▀   ▀██████▀  █████▄▄██ ";
echo "                                         ▀                                                                            ▀         ";                         

echo -e "${BLUE}                                    https://github.com/Z4nzu/hackingtool ${NC}"

echo -e "${RED}                                   [!] This Tool Must Run As ROOT [!]${NC}"
echo ""
echo -e ${CYAN}              "Select Best Option : "
echo ""
echo -e "${WHITE}              [1] Kali Linux / Parrot-Os "
echo -e "${WHITE}              [0] Exit "
echo -n -e "Z4nzu >> "
read choice
INSTALL_DIR="/usr/share/doc/hackingtool"
BIN_DIR="/usr/bin/"
if [ $choice == 1 ]; then 
	echo "[*] Checking Internet Connection .."
	wget -q --tries=10 --timeout=20 --spider https://google.com
	if [[ $? -eq 0 ]]; then
	    echo -e ${BLUE}"[✔] Loading ... "
	    sudo apt-get update && apt-get upgrade 
	    sudo apt-get install python-pip
	    echo "[✔] Checking directories..."
	    if [ -d "$INSTALL_DIR" ]; then
	        echo "[!] A Directory hackingtool Was Found.. Do You Want To Replace It ? [y/n]:" ;
	        read input
	        if [ "$input" = "y" ]; then
	            rm -R "$INSTALL_DIR"
	        else
	            exit
	        fi
	    fi
    		echo "[✔] Installing ...";
		echo "";
		git clone https://github.com/Z4nzu/hackingtool.git "$INSTALL_DIR";
		echo "#!/bin/bash
		python3 $INSTALL_DIR/hackingtool.py" '${1+"$@"}' > hackingtool;
		sudo chmod +x hackingtool;
		sudo cp hackingtool /usr/bin/;
		rm hackingtool;
		echo ""; 
		echo "[✔] Trying to installing Requirements ..."
		sudo pip3 install lolcat
		sudo apt-get install -y figlet
		sudo pip3 install boxes
		sudo apt-get install boxes
		sudo pip3 install flask
		sudo pip3 install requests
	else 
		echo -e $RED "Please Check Your Internet Connection ..!!"
	fi

    if [ -d "$INSTALL_DIR" ]; then
        echo "";
        echo "[✔] Successfuly Installed !!! ";
        echo "";
        echo "";
        echo -e $ORANGE "		[+]+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++[+]"
        echo 		"		[+]						      		[+]"
        echo -e $ORANGE  "		[+]     ✔✔✔ Now Just Type In Terminal (hackingtool) ✔✔✔ 	[+]"
        echo 		"		[+]						      		[+]"
        echo -e $ORANGE "		[+]+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++[+]"
    else
        echo "[✘] Installation Failed !!! [✘]";
        exit
    fi
elif [ $choice -eq 0 ];
then
    echo -e $RED "[✘] THank Y0u !! [✘] "
    exit
else 
    echo -e $RED "[!] Select Valid Option [!]"
fi

MIT License

Copyright (c) 2020 Mr.Z4nzu

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

### All in One Hacking tool For Hackers🥇
![](https://img.shields.io/github/license/Z4nzu/hackingtool)
![](https://img.shields.io/github/issues/Z4nzu/hackingtool)
![](https://img.shields.io/github/issues-closed/Z4nzu/hackingtool)
![](https://img.shields.io/badge/Python-3-blue)
![](https://img.shields.io/github/forks/Z4nzu/hackingtool)
![](https://img.shields.io/github/stars/Z4nzu/hackingtool)
![](https://img.shields.io/github/last-commit/Z4nzu/hackingtool)
[![HitCount](http://hits.dwyl.com/Z4nzu/hackingtool.svg)](http://hits.dwyl.com/Z4nzu/hackingtool)
![](https://img.shields.io/badge/platform-Linux%20%7C%20KaliLinux%20%7C%20ParrotOs-blue)

#### Install Kali Linux in WIndows10 Without VirtualBox [YOUTUBE](https://youtu.be/BsFhpIDcd9I)

## Update Available V1.1.0 🚀 
- [x] Added New Tools 
    - [x] Reverse Engineering
    - [x] RAT Tools
    - [x] Web Crawling 
    - [x] Payload Injector
- [x] Multitor Tools update
- [X] Added Tool in wifijamming


# Hackingtool Menu 🧰
- [Anonymously Hiding Tools](#anonymously-hiding-tools)
- [Information gathering tools](#information-gathering-tools)
- [Wordlist Generator](#wordlist-generator)
- [Wireless attack tools](#wireless-attack-tools)
- [SQL Injection Tools](#sql-injection-tools)
- [Phishing attack tools](#phishing-attack-tools)
- [Web Attack tools](#web-attack-tools)
- [Post exploitation tools](#post-exploitation-tools)
- [Forensic tools](#forensic-tools)
- [Payload creation tools](#payload-creation-tools)
- [Exploit framework](#exploit-framework)
- [Reverse engineering tools](#reverse-engineering-tools)
- [DDOS Attack Tools](#ddos-attack-tools)
- [Remote Administrator Tools (RAT)](#remote-administrator-tools--rat-)
- [XSS Attack Tools](#xss-attack-tools)
- [Steganograhy tools](#steganograhy-tools)
- [Other tools](#other-tools)
    - [SocialMedia Bruteforce](#socialmedia-bruteforce)
    - [Android Hacking tools](#android-hacking-tools)
    - [IDN Homograph Attack](#idn-homograph-attack)
    - [Email Verify tools](#email-verify-tools)
    - [Hash cracking tools](#hash-cracking-tools)
    - [Wifi Deauthenticate](#wifi-deauthenticate)
    - [SocialMedia Finder](#socialmedia-finder)
    - [Payload Injector](#payload-injector)
    - [Web crawling](#web-crawling)
    - [Mix tools](#mix-tools)


### Anonymously Hiding Tools
- [Anonmously Surf](https://github.com/Und3rf10w/kali-anonsurf)
- [Multitor](https://github.com/trimstray/multitor)
### Information gathering tools
- [Network Map (nmap)](https://github.com/nmap/nmap)
- [Dracnmap](https://github.com/Screetsec/Dracnmap)
- Port scanning
- Host to IP 
- [Xerosploit](https://github.com/LionSec/xerosploit)
- [RED HAWK (All In One Scanning)](https://github.com/Tuhinshubhra/RED_HAWK)
- [ReconSpider(For All Scaning)](https://github.com/bhavsec/reconspider)
- IsItDown (Check Website Down/Up)
- [Infoga - Email OSINT](https://github.com/m4ll0k/Infoga)
- [ReconDog](https://github.com/s0md3v/ReconDog)
- [Striker](https://github.com/s0md3v/Striker)
- [SecretFinder (like API & etc)](https://github.com/m4ll0k/SecretFinder)
- [Find Info Using Shodan](https://github.com/m4ll0k/Shodanfy.py)
- [Port Scanner - rang3r](https://github.com/floriankunushevci/rang3r)
- [Breacher](https://github.com/s0md3v/Breacher)
### Wordlist Generator
- [Cupp](https://github.com/Mebus/cupp.git)
- [WordlistCreator](https://github.com/Z4nzu/wlcreator)
- [Goblin WordGenerator](https://github.com/UndeadSec/GoblinWordGenerator.git)
- [Password list (1.4 Billion Clear Text Password)](https://github.com/Viralmaniar/SMWYG-Show-Me-What-You-Got)
### Wireless attack tools
- [WiFi-Pumpkin](https://github.com/P0cL4bs/wifipumpkin3)
- [pixiewps](https://github.com/wiire/pixiewps)
- [Bluetooth Honeypot GUI Framework](https://github.com/andrewmichaelsmith/bluepot)
- [Fluxion](https://github.com/thehackingsage/Fluxion)
- [Wifiphisher](https://github.com/wifiphisher/wifiphisher)
- [Wifite](https://github.com/derv82/wifite2)
- [EvilTwin](https://github.com/Z4nzu/fakeap)
- [Fastssh](https://github.com/Z4nzu/fastssh)
- Howmanypeople
### SQL Injection Tools
- [Sqlmap tool](https://github.com/sqlmapproject/sqlmap)
- [NoSqlMap](https://github.com/codingo/NoSQLMap)
- [Damn Small SQLi Scanner](https://github.com/stamparm/DSSS)
- [Explo](https://github.com/dtag-dev-sec/explo)
- [Blisqy - Exploit Time-based blind-SQL injection](https://github.com/JohnTroony/Blisqy)
- [Leviathan - Wide Range Mass Audit Toolkit](https://github.com/leviathan-framework/leviathan)
- [SQLScan](https://github.com/Cvar1984/sqlscan)
### Phishing attack tools
- [Setoolkit](https://github.com/trustedsec/social-engineer-toolkit)
- [SocialFish](https://github.com/UndeadSec/SocialFish)
- [HiddenEye](https://github.com/DarkSecDevelopers/HiddenEye)
- [Evilginx2](https://github.com/kgretzky/evilginx2)
- [I-See_You(Get Location using phishing attack)](https://github.com/Viralmaniar/I-See-You)
- [SayCheese (Grab target's Webcam Shots)](https://github.com/hangetzzu/saycheese)
- [QR Code Jacking](https://github.com/cryptedwolf/ohmyqr)
- [ShellPhish](https://github.com/An0nUD4Y/shellphish)
- [BlackPhish](https://github.com/iinc0gnit0/BlackPhish)
### Web Attack tools
- [Web2Attack](https://github.com/santatic/web2attack)
- Skipfish
- [SubDomain Finder](https://github.com/aboul3la/Sublist3r)
- [CheckURL](https://github.com/UndeadSec/checkURL)
- [Blazy(Also Find ClickJacking)](https://github.com/UltimateHackers/Blazy)
- [Sub-Domain TakeOver](https://github.com/m4ll0k/takeover)
- [Dirb](https://gitlab.com/kalilinux/packages/dirb)
### Post exploitation tools
- [Vegile - Ghost In The Shell](https://github.com/Screetsec/Vegile)
- [Chrome Keylogger](https://github.com/UndeadSec/HeraKeylogger)
### Forensic tools
- Autopsy
- Wireshark
- [Bulk extractor](https://github.com/simsong/bulk_extractor)
- [Disk Clone and ISO Image Aquire](https://guymager.sourceforge.io/)
- [Toolsley](https://www.toolsley.com/)
### Payload creation tools
- [The FatRat](https://github.com/Screetsec/TheFatRat)
- [Brutal](https://github.com/Screetsec/Brutal)
- [Stitch](https://nathanlopez.github.io/Stitch)
- [MSFvenom Payload Creator](https://github.com/g0tmi1k/msfpc)
- [Venom Shellcode Generator](https://github.com/r00t-3xp10it/venom)
- [Spycam](https://github.com/thelinuxchoice/spycam)
- [Mob-Droid](https://github.com/kinghacker0/Mob-Droid)
- [Enigma](https://github.com/UndeadSec/Enigma)
### Exploit framework
- [RouterSploit](https://github.com/threat9/routersploit)
- [WebSploit](https://github.com/The404Hacking/websploit )
- [Commix](https://github.com/commixproject/commix)
- [Web2Attack](https://github.com/santatic/web2attack)
### Reverse engineering tools
- [Androguard](https://github.com/androguard/androguard )
- [Apk2Gold](https://github.com/lxdvs/apk2gold )
- [JadX](https://github.com/skylot/jadx)
### DDOS Attack Tools
- SlowLoris
- [Asyncrone | Multifunction SYN Flood DDoS Weapon](https://github.com/fatihsnsy/aSYNcrone)
- [UFOnet](https://github.com/epsylon/ufonet)
- [GoldenEye](https://github.com/jseidl/GoldenEye)
### Remote Administrator Tools (RAT)
- [Stitch](https://github.com/nathanlopez/Stitch)
- [Pyshell](https://github.com/knassar702/pyshell)
### XSS Attack Tools
- [DalFox(Finder of XSS)](https://github.com/hahwul/dalfox)
- [XSS Payload Generator](https://github.com/capture0x/XSS-LOADER.git)
- [Extended XSS Searcher and Finder](https://github.com/Damian89/extended-xss-search)
- [XSS-Freak](https://github.com/PR0PH3CY33/XSS-Freak)
- [XSpear](https://github.com/hahwul/XSpear)
- [XSSCon](https://github.com/menkrep1337/XSSCon)
- [XanXSS](https://github.com/Ekultek/XanXSS)
- [Advanced XSS Detection Suite](https://github.com/UltimateHackers/XSStrike)
- [RVuln](https://github.com/iinc0gnit0/RVuln)
### Steganograhy tools
- SteganoHide
- StegnoCracker
- [Whitespace](https://github.com/beardog108/snow10)
### Other tools
#### SocialMedia Bruteforce
- [Instagram Attack](https://github.com/chinoogawa/instaBrute)
- [AllinOne SocialMedia Attack](https://github.com/Matrix07ksa/Brute_Force)
- [Facebook Attack](https://github.com/Matrix07ksa/Brute_Force)
- [Application Checker](https://github.com/jakuta-tech/underhanded)
#### Android Hacking tools
- [Keydroid](https://github.com/F4dl0/keydroid)
- [MySMS](https://github.com/papusingh2sms/mysms)
- [Lockphish (Grab target LOCK PIN)](https://github.com/JasonJerry/lockphish)
- [DroidCam (Capture Image)](https://github.com/kinghacker0/WishFish)
- [EvilApp (Hijack Session)](https://github.com/crypticterminal/EvilApp)
- [HatCloud(Bypass CloudFlare for IP)](https://github.com/HatBashBR/HatCloud)
#### IDN Homograph Attack
- [EvilURL](https://github.com/UndeadSec/EvilURL)
#### Email Verify tools
- [Knockmail](https://github.com/4w4k3/KnockMail)
#### Hash cracking tools
- [Hash Buster](https://github.com/s0md3v/Hash-Buster)
#### Wifi Deauthenticate
- [WifiJammer-NG](https://github.com/MisterBianco/wifijammer-ng)
- [KawaiiDeauther](https://github.com/aryanrtm/KawaiiDeauther)
#### SocialMedia Finder
- [Find SocialMedia By Facial Recognation System](https://github.com/Greenwolf/social_mapper)
- [Find SocialMedia By UserName](https://github.com/xHak9x/finduser)
- [Sherlock](https://github.com/sherlock-project/sherlock)
- [SocialScan | Username or Email](https://github.com/iojw/socialscan)
#### Payload Injector
- [Debinject](https://github.com/UndeadSec/Debinject)
- [Pixload](https://github.com/chinarulezzz/pixload)
#### Web crawling
- [Gospider](https://github.com/jaeles-project/gospider)
#### Mix tools
- Terminal Multiplexer


![](https://github.com/Z4nzu/hackingtool/blob/master/images/A00.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A0.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A1.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A2.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A4.png)

## Installation For Linux <img src="https://konpa.github.io/devicon/devicon.git/icons/linux/linux-original.svg" alt="linux" width="25" height="25"/></p><p align="center">

#### This Tool Must Run As ROOT !!!

    git clone https://github.com/Z4nzu/hackingtool.git
    
    chmod -R 755 hackingtool  
    
    cd hackingtool
    
    sudo pip3 install -r requirement.txt
    
    bash install.sh
    
    sudo hackingtool

 After Following All Steps Just Type In Terminal **root@kaliLinux:~** **hackingtool**

#### Thanks to original Author of the tools used in hackingtool

<img src ="https://img.shields.io/badge/Important-notice-red" />
<h4>Please Don't Use for illegal Activity</h4>

### To do 
- [ ] Release Tool 
- [ ] Add Tools for CTF
- [ ] Want to do automatic 

## Social Media :mailbox_with_no_mail:
[![Twitter](https://img.shields.io/twitter/url?color=%231DA1F2&label=follow&logo=twitter&logoColor=%231DA1F2&style=flat-square&url=https%3A%2F%2Fwww.reddit.com%2Fuser%2FFatChicken277)](https://twitter.com/_Zinzu07)
[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat-square&logo=github&link=https://github.com/Z4nzu/)](https://github.com/Z4nzu/)
##### Your Favourite Tool is not in hackingtool or Suggestions Please [CLICK HERE](https://forms.gle/b235JoCKyUq5iM3t8)
![Z4nzu's github stats](https://github-readme-stats.vercel.app/api?username=Z4nzu&show_icons=true&title_color=fff&icon_color=79ff97&text_color=9f9f9f&bg_color=151515)

<a href="https://www.buymeacoffee.com/Zinzu" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/arial-yellow.png" alt="Buy Me A Coffee" style="height: 50px !important;width: 50px !important;"></a>

#### Don't Forgot to share with Your Friends 
### The new Update get will soon stay updated
#### Thank you..!!

### All in One Hacking tool For Hackers🥇
![](https://img.shields.io/github/license/Z4nzu/hackingtool)
![](https://img.shields.io/github/issues/Z4nzu/hackingtool)
![](https://img.shields.io/github/issues-closed/Z4nzu/hackingtool)
![](https://img.shields.io/badge/Python-3-blue)
![](https://img.shields.io/github/forks/Z4nzu/hackingtool)
![](https://img.shields.io/github/stars/Z4nzu/hackingtool)
![](https://img.shields.io/github/last-commit/Z4nzu/hackingtool)
[![HitCount](http://hits.dwyl.com/Z4nzu/hackingtool.svg)](http://hits.dwyl.com/Z4nzu/hackingtool)
![](https://img.shields.io/badge/platform-Linux%20%7C%20KaliLinux%20%7C%20ParrotOs-blue)

#### Install Kali Linux in WIndows10 Without VirtualBox [YOUTUBE](https://youtu.be/BsFhpIDcd9I)

## Update Available V1.1.0 🚀 
- [x] Added New Tools 
    - [x] Reverse Engineering
    - [x] RAT Tools
    - [x] Web Crawling 
    - [x] Payload Injector
- [x] Multitor Tools update
- [X] Added Tool in wifijamming


# Hackingtool Menu 🧰
{{toc}}

{{tools}}

![](https://github.com/Z4nzu/hackingtool/blob/master/images/A00.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A0.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A1.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A2.png)
![](https://github.com/Z4nzu/hackingtool/blob/master/images/A4.png)

## Installation For Linux <img src="https://konpa.github.io/devicon/devicon.git/icons/linux/linux-original.svg" alt="linux" width="25" height="25"/></p><p align="center">

#### This Tool Must Run As ROOT !!!

    git clone https://github.com/Z4nzu/hackingtool.git
    
    chmod -R 755 hackingtool  
    
    cd hackingtool
    
    sudo pip3 install -r requirement.txt
    
    bash install.sh
    
    sudo hackingtool

 After Following All Steps Just Type In Terminal **root@kaliLinux:~** **hackingtool**

#### Thanks to original Author of the tools used in hackingtool

<img src ="https://img.shields.io/badge/Important-notice-red" />
<h4>Please Don't Use for illegal Activity</h4>

### To do 
- [ ] Release Tool 
- [ ] Add Tools for CTF
- [ ] Want to do automatic 

## Social Media :mailbox_with_no_mail:
[![Twitter](https://img.shields.io/twitter/url?color=%231DA1F2&label=follow&logo=twitter&logoColor=%231DA1F2&style=flat-square&url=https%3A%2F%2Fwww.reddit.com%2Fuser%2FFatChicken277)](https://twitter.com/_Zinzu07)
[![GitHub](https://img.shields.io/badge/-GitHub-181717?style=flat-square&logo=github&link=https://github.com/Z4nzu/)](https://github.com/Z4nzu/)
##### Your Favourite Tool is not in hackingtool or Suggestions Please [CLICK HERE](https://forms.gle/b235JoCKyUq5iM3t8)
![Z4nzu's github stats](https://github-readme-stats.vercel.app/api?username=Z4nzu&show_icons=true&title_color=fff&icon_color=79ff97&text_color=9f9f9f&bg_color=151515)

<a href="https://www.buymeacoffee.com/Zinzu" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/arial-yellow.png" alt="Buy Me A Coffee" style="height: 50px !important;width: 50px !important;"></a>

#### Don't Forgot to share with Your Friends 
#### Thank you..!!

lolcat
boxes
flask
requests

echo "███████╗██╗  ██╗███╗   ██╗███████╗██╗   ██╗    ";
echo "╚══███╔╝██║  ██║████╗  ██║╚══███╔╝██║   ██║    ";
echo "  ███╔╝ ███████║██╔██╗ ██║  ███╔╝ ██║   ██║    ";
echo " ███╔╝  ╚════██║██║╚██╗██║ ███╔╝  ██║   ██║    ";
echo "███████╗     ██║██║ ╚████║███████╗╚██████╔╝    ";
echo "╚══════╝     ╚═╝╚═╝  ╚═══╝╚══════╝ ╚═════╝     ";
echo "                                               ";

clear

sudo chmod +x /etc/

clear

sudo chmod +x /usr/share/doc

clear

sudo rm -rf /usr/share/doc/hackingtool/

clear

cd /etc/

clear

sudo rm -rf /etc/hackingtool

clear

mkdir hackingtool

clear

cd hackingtool

clear

git clone https://github.com/Z4nzu/hackingtool.git

clear

cd hackingtool

clear

sudo chmod +x install.sh

clear

./install.sh

clear
