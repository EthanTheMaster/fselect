# fselect
Find files with SQL-like queries

[![Crates.io](https://img.shields.io/crates/v/fselect.svg)](https://crates.io/crates/fselect)
[![Build Status](https://travis-ci.org/jhspetersson/fselect.svg?branch=master)](https://travis-ci.org/jhspetersson/fselect)

### Why use fselect?

While it doesn't tend to fully replace traditional `find` and `ls`, **fselect** has these nice features:

* complex queries
* SQL-like (not real SQL, but highly relaxed!) grammar easily understandable by humans
* search within archives
* `.gitignore` support (experimental)
* search by width and height of images
* search by MP3 info
* shortcuts to common file types
* various output formatting (CSV, JSON, and others)

More is under way!

### Installation

#### From source

* Install [Rust with Cargo](https://www.rust-lang.org/en-US/install.html) and its dependencies to build a binary
* Run `cargo install fselect`

#### Arch Linux

[AUR package](https://aur.archlinux.org/packages/fselect/), thanks to [@asm0dey](https://github.com/asm0dey)

#### Other Linux

[Static build with musl](https://github.com/jhspetersson/fselect/releases/download/0.4.4/fselect-x86_64-linux-musl.gz).

#### Windows 64bit

A statically precompiled [binary](https://github.com/jhspetersson/fselect/releases/download/0.4.4/fselect-x86_64-win.zip) is available at Github downloads.

#### Mac

[Precompiled on OS X Mavericks 10.9.5](https://github.com/jhspetersson/fselect/releases/download/0.4.4/fselect-x86_64-apple-darwin.zip)

#### Mac from brew

* Install [brew](https://docs.brew.sh/Installation)
* Run `brew install fselect`

### Usage

    fselect COLUMN[, COLUMN...] [from ROOT[, ROOT...]] [where EXPR] [order by COLUMNS] [limit N] [into FORMAT]

### Documentation

[More detailed description. Look examples first.](docs/usage.md)

### Examples

Find temporary or config files (full path and size):

    fselect size, path from /home/user where name = '*.cfg' or name = '*.tmp'
    
Windows users may omit the quotes:

    fselect size, path from C:\Users\user where name = *.cfg or name = *.tmp

Or put all the arguments into the quotes like this:

    fselect "name from /home/user/tmp where size > 0"

Find files (just names) with any content (size > 0):

    fselect name from /home/user/tmp where size gt 0

Specify file size and add it to the results:

    fselect size, path from /home/user/tmp where size gt 2g
    fselect fsize, path from /home/user/tmp where size = 5m
    fselect hsize, path from /home/user/tmp where size lt 8k
    
More complex query:

    fselect "name from /tmp where (name = *.tmp and size = 0) or (name = *.cfg and size > 1000000)"
    
Use single quotes if you need to address files with spaces:

    fselect path from '/home/user/Misc stuff' where name != 'Some file'
    
Regular expressions supported:

    fselect name from /home/user where path =~ '.*Rust.*'
    
And even simple glob will suffice:

    fselect name from /home/user where path = '*Rust*'
    
Classic LIKE:

    fselect path from /home/user where name like %report-2018-__-__???
    
Exact match operators to search with regexps disabled:

    fselect path from /home/user where name === 'some_*_weird_*_name'
    
Find files by date:

    fselect path from /home/user where created = 2017-05-01
    fselect path from /home/user where modified = today
    fselect path from /home/user where accessed = yesterday
    fselect path from /home/user where modified = 'apr 1'
    fselect path from /home/user where modified = 'last fri'
    
Be more specific to match all files created at interval between 3PM and 4PM:

    fselect path from /home/user where created = '2017-05-01 15'
    
And even more specific:

    fselect path from /home/user where created = '2017-05-01 15:10'
    fselect path from /home/user where created = '2017-05-01 15:10:30'
    
Date and time intervals possible (find everything updated since May 1st):

    fselect path from /home/user where modified gte 2017-05-01
    
Default is current directory:

    fselect path, size where name = '*.jpg'
    
Search within multiple locations:

    fselect path from /home/user/oldstuff, /home/user/newstuff where name = '*.jpg'
    
With maximum depth specified:

    fselect path from /home/user/oldstuff depth 5 where name = '*.jpg'
    fselect path from /home/user/oldstuff depth 5, /home/user/newstuff depth 10 where name = '*.jpg'

Optionally follow symlinks:

    fselect path, size from /home/user symlinks where name = '*.jpg'
    
Search within archives (currently only zip-archives are supported):

    fselect path, size from /home/user archives where name = '*.jpg'
    
Or in combination:

    fselect size, path from /home/user depth 5 archives symlinks where name = '*.jpg' limit 100

Enable `.gitignore` support:

    fselect size, path from /home/user/projects gitignore where name = '*.cpp'        
    
Search by image dimensions:

    fselect width, height, path from /home/user/photos where width gte 2000 or height gte 2000
    
Find old-school rap MP3 files:

    fselect path from /home/user/music where genre = Rap and bitrate = 320 and year lt 2000  
    
Shortcuts to common file extensions:

    fselect path from /home/user where is_archive = true
    fselect path from /home/user where is_audio = 1
    fselect path from /home/user where is_book != false
    fselect path from /home/user where is_doc != 1
    fselect path from /home/user where is_image = false
    fselect path from /home/user where is_video != true
    
Find files with dangerous permissions:
    
    fselect mode, path from /home/user where other_write = true or other_exec = true
    
Simple glob-like expressions or even regular expressions on file mode are possible:
    
    fselect mode, path from /home/user where mode = '*rwx'
    fselect mode, path from /home/user where mode =~ '.*rwx$'
    
Find files by owner's uid or gid:

    fselect uid, gid, path from /home/user where uid != 1000 or gid != 1000
    
Or by owner's or group's name:

    fselect user, group, path from /home/user where user = mike or group = mike

Find special files:

    fselect path from /tmp where is_pipe = true
    fselect path from /tmp where is_socket = 1
    
Include arbitrary text as columns:

    fselect name, ' has size of ', size, ' bytes'

Order results:

    fselect path from /tmp order by size desc, name
    fselect modified, fsize, path from ~ order by 1 desc, 3
    
Finally limit the results:

    fselect name from /home/user/samples limit 5 
    
Format output:

    fselect size, path from /home/user limit 5 into json
    fselect size, path from /home/user limit 5 into csv

### License

MIT/Apache-2.0
