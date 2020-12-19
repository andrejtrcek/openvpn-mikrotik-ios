# OpenVPN server on RouterOS with iOS client

Create your own VPN server on Mikrotik (RouterOS) with OpenVPN and connect with iOS clients (iPhone, iPad).


# Generate and sign certificates

### Generate and sign a Self-Signed certificate
```sh
/certificate add name=cert.ca common-name=cert.ca key-usage=key-cert-sign,crl-sign trusted=yes
/certificate sign cert.ca
```

### Generate and sign a certificate for OpenVPN server
```sh
/certificate add name=cert.server common-name=cert.server
/certificate sign cert.server ca=cert.ca
/certificate set trusted=yes cert.server
```

### Generate and sign a certificate for iOS client
```sh
/certificate add name=ios.client common-name=ios.client
/certificate sign ios.client ca=cert.ca
/certificate set trusted=yes ios.client
```

### Export certificate and pcks12
```sh
/certificate export-certificate cert.ca
/certificate export-certificate ios.client export-passphrase=password type=pkcs12
```


# Get the exported files

Download the exported "cert.ca" and "cert_export_ios.client.p12" which are located in Files (Webfig > Files).


# OpenVPN / RouterOS configuration

Create a new IP pool
```sh
/ip pool add name=OpenVPN ranges=10.10.23.1-10.10.23.200
```

Create a profile
```sh
/ppp profile add local-address=OpenVPN name=OpenVPN use-encryption=yes
```

Add a new secret
```sh
/ppp secret add name=ios-client password=password profile=OpenVPN service=ovpn
```

Configure OpenVPN server
```sh
/interface ovpn-server server set auth=sha1 certificate=cert.server cipher=aes256 default-profile=OpenVPN enabled=yes port=1194 require-client-certificate=yes
```

Enable your clients to connect to the OpenVPN server by opening a port (1194/TCP) in your firewall
```sh
/ip firewall filter add action=accept chain=input comment=OpenVPN disabled=yes dst-port=1194 protocol=tcp
```

# Create .ovpn for iOS

Create a new file with .ovpn extension (for example: myiphone.ovpn). Use and modify the following config:

```sh
client
proto tcp
dev tun                             
remote 0.0.0.0
port 1194
nobind
tun-mtu 1492
mssfix 1400
resolv-retry infinite
persist-key
persist-tun
auth-user-pass
auth SHA1
cipher AES-256-CBC
remote-cert-tls server
redirect-gateway def1
verb 5

<ca>
-----BEGIN CERTIFICATE-----
<copy/paste cert.ca contents>
-----END CERTIFICATE-----
</ca>

```


# Install and copy configuration

### Install the OpenVPN app

Connecting to OpenVPN server with iOS device is possible by using the OpenVPN Connect application, which can be downloaded from the AppStore:

https://apps.apple.com/us/app/openvpn-connect/id590379981


### Copy the files to iOS device

Connect your iOS device to the computer with an USB cable.

Open iTunes (Windows) or Finder (OSX), select "Files" tab and upload both files (cert_export_ios.client.p12, myiphone.ovpn) to OpenVPN app.


# Author

Andrej Trcek

Web: http://www.andrejtrcek.com

E-mail: me@andrejtrcek.com
