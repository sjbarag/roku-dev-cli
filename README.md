# Overview

_roku-dev-cli_ is a command-line tool for loading builds onto a dev-enabled Roku
device.

Features:

- build zipping, installation, and logging
- project analysis
- network proxying

# Installation

_roku-dev-cli_ can be installed from the pypi repository using pip.

```shell
pip install roku-dev-cli
```

# Usage

`roku-cli --help`

# Usage: Loading a Build

To load a build on the Roku, first make sure the device is on the same network
and is in developer mode
([how to enter developer mode](https://sdkdocs.roku.com/display/sdkdoc/Loading+and+Running+Your+Application#LoadingandRunningYourApplication-EnablingDevelopmentModeonyourbox)).

In your terminal, go to the base directory of your Roku project where the
_manifest_ file exists. Then run the following command:

```shell
roku <IP_ADDRESS>
```

This will zip up the current directory and send it to the device. If the app is
successfully installed the tool will output the application's log to your
console as it executes. Hitting Ctrl-C will safely exit the tool without
stopping the application.

The script will do some basic checking of the project making sure all the
component files are valid XML.

# Usage: Loading a build from an existing zip file

If you already have a zip file you want to load, you can use the zip_file flag
instead of zipping the current directory:

```shell
roku --zip_file <ZIP_FILE_PATH> <IP_ADDRESS>
```

This will install the zip file and start logging the application output.

# Usage: Analysis

TODO: document analysis tools and graphing

# Usage: Network Proxying

The tool has some support for _mitmproxy_, which allows you to view and control
network traffic running through the box.

The Roku OS currently does not support HTTP proxy forwarding, so users must
first route traffic through a computer.

### Step 0: Install mitmproxy

On mac, run the following command:

```shell
brew install mitmproxy
```

### Step 1: Setup routing

#### On Linux:

```shell
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A PREROUTING -i wlan1 -p tcp --dport 443 -j REDIRECT --to-port 8080
sudo iptables -t nat -A PREROUTING -i wlan1 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

Replace _wlan1_ with the wifi interface the Roku device is connected to.

#### On Mac:

Enable port forwarding:

```shell
sudo sysctl -w net.inet.ip.forwarding=1
```

Update pf config. It should look something like this...

```shell
...
scrub-anchor "com.apple/*"
nat-anchor "com.apple/*"
rdr-anchor "com.apple/*"
# Port forwarding for mitmproxy
rdr pass on bridge100 inet proto tcp from any to any port 80 -> 127.0.0.1 port 8080
rdr pass on bridge100 inet proto tcp from any to any port 443 -> 127.0.0.1 port 8080
dummynet-anchor "com.apple/*"
anchor "com.apple/*"
...
```

Notice that the forwarding rules were inserted between some other existing
rules, if you append them to the end of the file you will get some warnings.

Finally run the following commands. The first will check file syntax while the
second one will restart the service loading the new rules.

```shell
sudo pfctl -v -n -f /etc/pf.conf
sudo pfctl -ef /etc/pf.conf
```

### Step 2: Start the proxy

Finally start the tool with the --proxy flag. This will add mitmproxy's
certificate to your build and replace certificate instances to use it instead.

```shell
roku --proxy <IP_ADDRESS>
```

At this point the application should be running through your proxy.

TODO: document use of mitmdump scripts

# Publishing to PyPi

1.  If necessary, create a personal account at pypi.org and ask to be added as a
    maintainer of https://pypi.org/project/roku-dev-cli/.

2.  If necessary, edit `~/.pypirc` to include your pypi.org credentials. For
    example:

    ```ini
    [distutils]
    index-servers =
      pypi

    [pypi]
    username: <your-pypi-username>
    ```

3.  Merge your pull request, including updating `version.py`.

4.  Publish to the default public pypi repo, you will be prompted to enter your
    pypi.org password:

    ```shell
    python setup.py sdist upload
    ```
