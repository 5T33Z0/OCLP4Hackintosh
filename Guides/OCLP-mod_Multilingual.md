# How to compile the and use the multilingual version of OCLP-Mod 

1. Install Python if you haven't already
2. Clone the OCLP-Mod repo: 
	```
	git clone https://github.com/laobamac/OCLP-Mod.git
	```
3. Open Finder. Press CMF+G and enter 
	```
	~/OCLP-Mod
	```
4. Open 
	```text
	requirements.txt
	```
5. Replace the long string `wxPython @…` by 
	```text
	wxPython==4.2.2
	```
6. Save the file.
7. Back in Terminal, navigate to the OCLP-Mod folder:
	```
	cd OCLP-Mod
	```
8. Switch to the Multilingual branch:
	```
	git checkout multilingual
	```
9. Install the requirements:
	```
	pip3 install -r requirements.txt
	```
10. Next, run the following command. Otherwise the pathcer won't work:
	```
	python3 Build-Project.command --git-branch multilingual --reset-dmg-cache --reset-pyinstaller-cache
	```
11. In Finder double-click on `OCLP-Mod-GUI.command` to run the the Patcher.

## Screenshot

![](/Volumes/macOS Sonoma - Data/Users/5t33z0/Desktop/GUI_ml.png)

## Notes
The translation is incomplete.

## Credits

Based on [guide by to schrup21](https://www.hackintosh-forum.de/forum/thread/60350-wwdc-2025-macos-26-hackintosh/?postID=803772#post803772) from Hackintosh-Forum.de 
