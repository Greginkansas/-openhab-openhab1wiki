OpenHAB has a great development environment with the "openHAB Designer".
But in some cases you want to use another editor to make changes to the configuration files of openHAB.
To make this more effectiv there are some files to enable syntax highlighting for openHAB-files in these editors.

* [mcedit](Syntax-Highlighting-for-external-editors#mcedit)
* [Notepad++](Syntax-Highlighting-for-external-editors#notepad)
* [vi](Syntax-Highlighting-for-external-editors#vi)
* [nano](Syntax-Highlighting-for-external-editors#nano)
* [Textwrangler](Syntax-Highlighting-for-external-editors#textwrangler)

## mcedit

mcedit is an editor which comes with mc (Midnight Commander).


### Installing the syntax-files

- download copy the syntax-files to */usr/share/mc/syntax/*
- https://code.google.com/p/openhab-samples/source/browse/syntaxhl/mc/?repo=wiki

- insert the following lines to the file *Syntax* in */usr/share/mc/syntax/*
```
    file ..\*\\.(items)$ openHAB\sItems 
    include openhab-items.syntax  
     
    file ..\*\\.(sitemap)$ openHAB\sSitemap 
    include openhab-sitemap.syntax
     
    file ..\*\\.(persist)$ openHAB\sPersistence
    include openhab-persist.syntax
     
    file ..\*\\.(rules)$ openHAB\sRules
    include openhab-rules.syntax 
```
- edit the Debian-line from
`file (rules|rocks)$ Debian\srules`
to 
`file (rocks)$ Debian\srules`
because it interferes with openHABs rules-files.

### Screenshots
![Items](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_items.png "Items")
![Rules](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_rules.png "Rules")
![Sitemap](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_mc_sitemap.png "Sitemap")

## Notepad++

Notepad++ Version 6.2 or above is required to support UDL2 (User Defined Language v2).
# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"# Comments
color brightgreen "//.*"
color brightgreen start="/\*" end="\*/"
color brightgreen start="/\*\*" end="\*/"
http://notepad-plus-plus.org/news/notepad-6.2-release-udl2.html

### How to import UDL2-files?

- Download the UDL2-Files (openHAB-`*`.xml)
- https://code.google.com/p/openhab-samples/source/browse/syntaxhl/npp/?repo=wiki
- Install or update Notepad++ if necessary
- http://notepad-plus-plus.org/download/
- Open Notepad++
- Click "Language" (1)
- Click "Define your language.." (2)
- Click "Import..." (3)
- Select one of the downloaded XML-files
- Done.

![Import](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_import_udl2.png "Import")

### Screenshots

![Items](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_items.png "Items")
![Rules](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_rules.png "Rules")
![Sitemap](http://wiki.openhab-samples.googlecode.com/hg/screenshots/syntaxhl_npp_sitemap.png "Sitemap")

## vi

### Installing the syntax-files
#### Automatic installation
Paste the following code into a commandline
```
mkdir -p ~/.vim/{ftdetect,syntax} && \
curl -L -o ~/.vim/syntax/openhab.vim https://github.com/cyberkov/openhab-vim/raw/master/syntax/openhab.vim && \
curl -L -o ~/.vim/ftdetect/openhab.vim https://github.com/cyberkov/openhab-vim/raw/master/ftdetect/openhab.vim
```

#### Manual installation
- Download the syntax files from https://github.com/cyberkov/openhab-vim
- place them in your home directory under ~/.vim/
- start vim with an openHAB configuration file and it should work.

## nano

Nano is a common editor in linux systems

### Installing the syntax-files
- Download the syntax file openhab.nanorc from https://github.com/airix1/openhabnano
- place them in your nanorc directory ie: ~/.nano or /usr/share/nano. Then simply include the openhab.nanorc file in your ~/.nanorc or /etc/nanorc file ie:
````
## Openhab
include /usr/share/nano/openhab.nanorc
````

## Textwrangler
Textwrangler is a text and code editor for MAC OS X.

### Manual installation

- Download the syntax file from https://github.com/GrisoMG/openhab-syntax-textwrangler/blob/master/openhab.plist
- Copy the file openhab.plist into ~/Library/Application Support/TextWrangler/Language Modules
