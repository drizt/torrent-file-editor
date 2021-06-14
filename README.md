[![Build Status](https://travis-ci.org/torrent-file-editor/torrent-file-editor.svg?branch=master)](https://travis-ci.org/torrent-file-editor/torrent-file-editor)
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/torrent-file-editor/localized.svg)](https://crowdin.com/project/torrent-file-editor)
[![Version](https://badge.fury.io/gh/torrent-file-editor%2Ftorrent-file-editor.svg)](https://badge.fury.io/gh/torrent-file-editor%2Ftorrent-file-editor)

Torrent File Editor
===================

Qt based GUI tool designed to create and edit .torrent files

Author: Ivan Romanov <[drizt72@zoho.eu](mailto:drizt72@zoho.eu)>  
License: GNU General Public License v3.0 or later  
Homepage: https://torrent-file-editor.github.io  
Sources: https://github.com/torrent-file-editor/torrent-file-editor  
Crowdin translations: https://crowdin.com/project/torrent-file-editor

Build Instructions
------------------

Need to have
 - CMake >= 2.8.11
 - Qt4 or Qt5
 - QJSON >= 0.8.0 if used Qt4
 - [Sparkle](http://sparkle-project.org/) only for Mac OS X

**Linux:**

Will build Qt4 version by default

    mkdir build && cd build
    cmake -DCMAKE_BUILD_TYPE=Release -DQT5_BUILD=OFF ..
    make

If building Qt5 version on Ubuntu 18.04+, install required Qt5LinguistTools from `qttools5-dev` package.

**Mac OS X:**

Only Qt5 version

    mkdir build && cd build
    cmake -DCMAKE_BUILD_TYPE=Release ..
    make
    make dmg # to build dmg package

**Windows important note**

Only Qt4 version for a while.
I use Fedora 26 MinGW to build Windows versions. Furthermore I build
portable static versions. Any other build way is not tested and may
not work. It is on my TODO list.

Fedora hasn't a MinGW QJSON package. You need to build your own version.
It is easy:

    wget https://github.com/flavio/qjson/archive/master.tar.gz -O qjson-master.tar.gz
    tar zxf qjson-master.tar.gz
    mkdir qjson-master/win32
    mkdir qjson-master/win64
    cd qjson-master/win32
    mingw32-cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DQT4_BUILD=ON  -DQT_INCLUDE_DIRS_NO_SYSTEM=ON -DQT_USE_IMPORTED_TARGETS=OFF ..
    make
    sudo make install # will be careful, it installs qjson to system folders
    cd ../win64
    mingw64-cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DQT4_BUILD=ON  -DQT_INCLUDE_DIRS_NO_SYSTEM=ON -DQT_USE_IMPORTED_TARGETS=OFF ..
    make
    sudo make install # be careful, it installs qjson to system folders

**Windows x32:**

    mkdir build && cd build
    mingw32-cmake -DCMAKE_BUILD_TYPE=Release ..
    make

**Windows x64:**

    mkdir build && cd build
    mingw64-cmake -DCMAKE_BUILD_TYPE=Release ..
    make

How Can I Help?
---------------

The Project is translated from English to several languages.
I would be glad if you add new translations. You can translate the
project to your native language with [Crowdin](https://crowdin.com/project/torrent-file-editor).
It is not difficult and no special knowledges are required.
Or you can correct my English. I know it is not good. Anyway you can
always email <[drizt72@zoho.eu](mailto:drizt72@zoho.eu)> me.

Also feel free to open an issue on GitHub or send me pull requests.

**Translations**

<img src="https://lipis.github.io/flag-icon-css/flags/4x3/za.svg" width="24" height="24">  Afrikaans - Afrikaans  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/sa.svg" width="24" height="24">  العربية - Arabic  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/bd.svg" width="24" height="24">  বাংলা - Bengali  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/cn.svg" width="24" height="24">  简体中文 - Chinese Simplified  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/tw.svg" width="24" height="24">  繁體中文 - Chinese Traditional  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/cz.svg" width="24" height="24">  Čeština - Czech  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/nl.svg" width="24" height="24">  Nederlands - Dutch  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/us.svg" width="24" height="24">  English - English  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/fi.svg" width="24" height="24">  Suomi - Finnish  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/fr.svg" width="24" height="24">  Français - French  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/de.svg" width="24" height="24">  Deutsch - German  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/il.svg" width="24" height="24">  עברית‎ - Hebrew  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/hu.svg" width="24" height="24">  Magyar - Hungarian  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/id.svg" width="24" height="24">  Indonesia - Indonesian  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/it.svg" width="24" height="24">  Italiano - Italian  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/jp.svg" width="24" height="24">  日本語 - Japanese  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/kr.svg" width="24" height="24">  한국어 - Korean  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/pl.svg" width="24" height="24">  Polski - Polish  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/br.svg" width="24" height="24">  Português  (Brasil) - Portuguese (Brazil)  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/ro.svg" width="24" height="24">  Română - Romanian  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/ru.svg" width="24" height="24">  Русский - Russian  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/es.svg" width="24" height="24">  Español - Spanish  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/tr.svg" width="24" height="24">  Türkçe - Turkish  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/vn.svg" width="24" height="24">  Tiếng Việt - Vietnamese  
<img src="https://lipis.github.io/flag-icon-css/flags/4x3/ua.svg" width="24" height="24">  Украї́нська - Ukrainian  
