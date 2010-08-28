Build CouchDB Fork for Red Hat Enterprise Linux
===============================================

I created this fork to document and share the steps on how to setup build-couchdb on RHEL.

I am running: 

 * Red Hat Enterprise Linux Server release 5.5 (Tikanga) 
 * Linux 2.6.18-194.3.1.el5 #1 SMP Sun May 2 04:22:18 EDT 2010 i686 athlon i386 GNU/Linux

I followed the Fedora instructions, with the following exceptions:

 * I did not use yum to install rubygem-rake or libcurl-devel.
 * I used [Ruby Version Manager][rvm] to install the latest release of Ruby v 1.8.7.  This installed a version of rake, too.
 * I downloaded and installed the latest release of [cURL][curl] from source.
 * I had to modify one file in the git repo: tasks/distro.rake

[rvm]: http://rvm.beginrescueend.com
[curl]: http://curl.haxx.se/docs/install.html

Build CouchDB
=============

Build CouchDB is a wrapper or master project which pulls in, from official
sources, CouchDB plus all of its dependencies. It is the most straightforward
and reliable procedure to build official CouchDB releases from source.

Build CouchDB builds an isolated, independent server. You do not need
administrator access to run it. You can run several couches (for example, 0.10,
0.11, 1.0 releases) side-by-side.

## Supported Platforms

Build CouchDB is developed and tested on the following operating systems:

 * Debian GNU/Linux 5.0 (Lenny)
 * Ubuntu 10.04 LTS (Lucid Lynx)
 * Ubuntu 9.10 (Karmic Koala)
 * Fedora 13
 * Apple OSX
 * OpenSUSE 11.3
 * Scientific Linux 5.3

The following systems are planned for support in the near future:

 * MS Windows Vista, Windows 7

## Requirements

You need only a few packages provided by the operating system. Copy and paste
the commands below.

On **Fedora**:

    sudo yum install gcc gcc-c++ libtool libcurl-devel \
                     zlib-devel openssl-devel rubygem-rake

On **Debian**, first install `sudo` and add yourself to `/etc/sudoers`.

    su -
    apt-get install sudo
    vi /etc/sudoers # Or your preferred editor

On **Ubuntu and Debian**:

    sudo apt-get install make gcc zlib1g-dev libssl-dev libreadline5-dev rake

On **OpenSUSE**:

    sudo zypper install flex lksctp-tools-devel zip \
				rubygem-rake gcc-c++ make m4 zlib-devel \
				libopenssl-devel libtool automake libcurl-devel

On **Scientific Linux**

    sudo yum install --enablerepo=dag gcc gcc-c++ libtool zlib-devel openssl-devel \
				autoconf213

You also must install a recent copy of Ruby and libcurl as the ones
available in the provided yum repositories are too old to use.

It is sufficient to perform a standard three-finger salute to install
ruby and libcurl

    ./configure
    make
    make install

On **OSX**, install XCode.


## Getting the Code

You will need the Git tool. Check out the code and pull in the third-party
submodules.

    git clone git://github.com/jhs/build-couchdb
    cd build-couchdb
    git submodule init
    git submodule update

## How to Build CouchDB

Just run Rake.

    rake

CouchDB and all its dependencies will install in the `build/`. To uninstall,
simply delete that directory.

## Usage

It's CouchDB! Just type `couchdb`. (But remember the path)

    $ build/bin/couchdb
    Apache CouchDB 0.12.0aa63efb6-git (LogLevel=info) is starting.
    Apache CouchDB has started. Time to relax.
    [info] [<0.33.0>] Apache CouchDB has started on http://127.0.0.1:5984/

You can of course call it by absolute path. If your username is `amit` and you
checked out the code in your home directory, you would run:

    /home/amit/build-couchdb/build/bin/couchdb

## Conveniently Add CouchDB to the PATH

The `env` task will output a script which will add this CouchDB build to your
path. Then you can simply type `couchdb`. To load these settings into your
current shell, run:

    eval `rake env --silent`

## Cheat Codes

Build CouchDB supports some fancy tricks by entering cheat codes to the Rake
command.

### Build any Git fork or tag of CouchDB

Add a `git` parameter with the repository URL, then a space, then the branch,
tag, or commit hash. (Remember to quote all of thos so Rake sees the space.)

Want to build [GeoCouch][geocouch]? No problem.

    rake git="git://github.com/vmx/couchdb geocouch"

### Install CouchDB somewhere besides `build/`.

Add a `couchdb_build` parameter to place the final couchdb binaries anywhere.
Note, you still need the main `build/` subdirectory because couchdb dependencies
such as Erlang and ICU reside there.

However, `couchdb_build` makes it trivial to install several couchdb versions
side-by-side.

    rake git="git://github.com/vmx/couchdb geocouch" couchdb_build=geocouch
    rake git="git://git.apache.org/couchdb.git trunk" couchdb_build=trunk
    for tag in 1.0.1 11.0 11.1; do
        rake git="git://git.apache.org/couchdb.git tags/$tag" couchdb_build=$tag
    done

### Get a manifest of all the components

To get a better idea of exactly what is going on, add a `manifest` parameter.

    rake manifest=1

That will produce additional files in `build/manifest` which indicate which
package (icu, erlang, spidermonkey, etc) owns which files within `build`. A
trick I do a lot is `cat build/manifest/couchdb | xargs rm` to "uninstall" only
couchdb so I can try a rebuild.

I have no idea how `manifest` interacts with `couchdb_build` as I have never
used them together.

 [geocouch]: http://vmx.cx/cgi-bin/blog/index.cgi/geocouch-the-future-is-now:2010-05-03:en,CouchDB,Python,Erlang,geo

### Do not strip down Erlang/OTP

Build CouchDB strips many modules out of the Erlang platform to reduce disk
usage. (You can see which ones at the top of `tasks/erlang.rake`.) To indicate
that a package should be kept, set the `otp_keep` variable to space-separated
library names.

    rake otp_keep="compiler eunit"

Or, you can keep them all this way:

    rake otp_keep="*"

vim: tw=80
