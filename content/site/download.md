+++
title = "downloads"
layout = "sidebar"
+++

# <a id="influxdb"></a>InfluxDB Downloads

## Version 0.9.4.2 (Stable)

#### OS X

- Via [Homebrew](http://brew.sh/)

		brew update
		brew install influxdb

#### Ubuntu & Debian

- 64-bit system install instructions

		wget https://s3.amazonaws.com/influxdb/influxdb_0.9.4.2_amd64.deb
		sudo dpkg -i influxdb_0.9.4.2_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

		wget https://s3.amazonaws.com/influxdb/influxdb-0.9.4.2-1.x86_64.rpm
		sudo yum localinstall influxdb-0.9.4.2-1.x86_64.rpm

#### Standalone Binary

- 64-bit system download & decompress instructions

		wget https://s3.amazonaws.com/influxdb/influxdb_0.9.4.2_x86_64.tar.gz
		tar xvfz influxdb_0.9.4.2_x86_64.tar.gz

#### Windows

- <p>64-bit system install via the [InfluxDB Windows Installer](https://s3.amazonaws.com/influxdb/influxdb_0.9.4.2_amd64.msi)</p>

## Version 0.9.5 (Nightly)
Nightly builds are created once-a-day, at midnight San Francisco, CA time, using the top-of-tree of [master](https://github.com/influxdb/influxdb/tree/master) source code. These builds will include all the latest fixes, but also undergo only basic testing.

#### Ubuntu & Debian

- 64-bit system install instructions

        wget https://s3.amazonaws.com/influxdb/influxdb_nightly_amd64.deb
        sudo dpkg -i influxdb_nightly_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

        wget https://s3.amazonaws.com/influxdb/influxdb-nightly-1.x86_64.rpm
        sudo yum localinstall influxdb-nightly-1.x86_64.rpm

#### Standalone Binary

- 64-bit system download & decompress instructions

		wget https://influxdb.s3.amazonaws.com/influxdb_nightly_x86_64.tar.gz
		tar xvfz influxdb_nightly_x86_64.tar.gz

### 32-Bit Packages
The industry is gradually [moving away from support for 32-bit x86 architectures](https://golang.org/doc/go1.5) so we do not provide packaged 32-bit binaries. However, we do endeavour to ensure the source can be compiled for a 32-bit x86 architecture at all times. To that end our [CI system](https://circleci.com/gh/influxdb/influxdb/tree/master) currently compiles 32-bit binaries and runs the unit test suite against the 32-bit build, in addition to the main 64-bit build. If compilation or unit testing for 32-bit architecture fails, we fix it.

However, we do reserve the right to break 32-bit compatibilty in the future, should design and implementation require it, but there are no plans to do so at this time.

### Deprecated Releases

Deprecated versions are no longer actively developed.

- [version 0.8](/docs/v0.8/introduction/installation.html)


# <a id="telegraf"></a>Telegraf Downloads

## Version 0.2.0

#### OS X

- Via [Homebrew](http://brew.sh/)

		brew update
		brew install telegraf

#### Ubuntu & Debian

- 64-bit system install instructions

		wget https://s3.amazonaws.com/get.influxdb.org/telegraf/telegraf_0.2.0_amd64.deb
		sudo dpkg -i telegraf_0.2.0_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

		wget https://s3.amazonaws.com/get.influxdb.org/telegraf/telegraf-0.2.0-1.x86_64.rpm
		sudo yum localinstall telegraf-0.2.0-1.x86_64.rpm

- 64-bit system download & decompress instructions

		wget https://s3.amazonaws.com/get.influxdb.org/telegraf/telegraf_linux_amd64_0.2.0.tar.gz
		tar xvfz telegraf_linux_amd64_0.2.0.tar.gz

# <a id="chronograf"></a>Chronograf Downloads

## Version 0.2.0

#### OS X

- Via [Homebrew](http://brew.sh/)

		brew update
		brew install homebrew/binary/chronograf

#### Ubuntu & Debian

- 64-bit system install instructions

		wget https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf_0.2.0_amd64.deb
		sudo dpkg -i chronograf_0.2.0_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

		wget https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf-0.2.0-1.x86_64.rpm
		sudo yum localinstall chronograf-0.2.0-1.x86_64.rpm

#### Standalone OS X Binary

- 64-bit system download & decompress instructions

		wget https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf-0.2.0-darwin_amd64.tar.gz
		tar xvfz chronograf-0.2.0-darwin_amd64.tar.gz

#### Windows

- <p>64-bit system install via the [Chronograf Windows Installer](https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf_0.2.0_amd64.msi)</p>

## Usage

If you installed Chronograf via a Debian or RPM package, you should be able to simply run `sudo service chronograf start`.
The Chronograf startup script needs root permission to ensure that it can write to /var/log, but the actual executable runs as a normal user.

If you did not install Chronograf via a package, you can just directly run the executable, e.g.

```
/opt/chronograf/chronograf -config=/opt/etc/chronograf.toml
```

## Registration

Every install of Chronograf requires a free registration on InfluxData Enterprise. Registration allows us to improve release management and functionality for Chronograf.

<script>
    if (typeof Cookies.get("submitted") === 'undefined') {
        var inst = $('[data-remodal-id=download]').remodal();
        inst.open();

        $(document).on('confirmation', '.remodal', function () {
            var form = $("form#download");
            var url = form.attr("action") ;
            var data = form.serialize();
            var email = $("input#email");

            if (email.val() != "") {
                $.post(url, data);
                Cookies.set("submitted", true);
            }
        })
    }
</script>


