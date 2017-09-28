SSH
=====

## Allowing Root Access for Specific Commands

There are some cases where you might want to disable root access generally, but enable it in order to allow certain applications to run correctly. An example of this might be a backup routine.

This can be accomplished through the root user's `authorized_keys` file, which contains SSH keys that are authorized to use the account.

1. Add the key from your local computer that you wish to use for this process (we recommend creating a new key for each automatic process) to the root user's authorized_keys file on the server. We will demonstrate with the ssh-copy-id command here, but you can use any of the methods of copying keys we discuss in other sections:
`ssh-copy-id root@remote_host`
2. Now, log into the remote server. We will need to adjust the entry in the authorized_keys file, so open it with root or sudo access:
`vim /root/.ssh/authorized_keys`
3. At the beginning of the line with the key you uploaded, add a command= listing that defines the command that this key is valid for. This should include the full path to the executable, plus any arguments:
`command="/path/to/command arg1 arg2" ssh-rsa ...`
4. Save and close the file when you are finished.
5. Now, open the sshd_config file with root or sudo privileges:
`vim /etc/ssh/sshd_config`
6. Find the directive `PermitRootLogin`, and change the value to forced-commands-only. This will only allow SSH key logins to use root when a command has been specified for the key:
`PermitRootLogin forced-commands-only`
7. Save and close the file.
8. Restart the SSH daemon to implement your changes.

## Fingerprints and ssh-copy-id
---
Each SSH key pair share a single cryptographic "fingerprint" which can be used to uniquely identify the keys. This can be useful in a variety of situations.
To find out the fingerprint of an SSH key, type:

`ssh-keygen -l`

Enter file in which the key is (`~/.ssh/id_rsa`):

You can press ENTER if that is the correct location of the key, else enter the revised location. You will be given a string which contains the bit-length of the key, the fingerprint, and account and host it was created for, and the algorithm used:

`4096 8e:c4:82:47:87:c2:26:4b:68:ff:96:1a:39:62:9e:4e  demo@test (RSA)`

To copy your public key to a server, allowing you to authenticate without a password, a number of approaches can be taken.

### ssh-copy-id

If you currently have password-based SSH access configured to your server, and you have the ssh-copy-id utility installed, this is a simple process. The `ssh-copy-id` tool is included in many Linux distributions' OpenSSH packages, so it very likely may be installed by default.

If you have this option, you can easily transfer your public key by typing:

`ssh-copy-id username@remote_host`

This will prompt you for the user account's password on the remote system:

```
The authenticity of host '111.111.11.111 (111.111.11.111)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
demo@111.111.11.111's password:
```
After typing in the password, the contents of your `~/.ssh/id_rsa.pub` key will be appended to the end of the user account's `~/.ssh/authorized_keys` file:

`Number of key(s) added: 1`

Now try logging into the machine, with:

`ssh 'demo@111.111.11.111'`

and check to make sure that only the key(s) you wanted were added. You can now log into that account without a password:

`ssh username@remote_host Copying your Public SSH Key to a Server Without SSH-Copy-ID`

### Access with no ssh-copy-id

If you do not have the `ssh-copy-id` utility available, but still have password-based SSH access to the remote server, you can copy the contents of your public key in a different way.
You can output the contents of the key and pipe it into the ssh command. On the remote side, you can ensure that the `~/.ssh` directory exists, and then append the piped contents into the `~/.ssh/authorized_keys` file:

`cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"`

You will be asked to supply the password for the remote account:

```
The authenticity of host '111.111.11.111 (111.111.11.111)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
demo@111.111.11.111's password:
```

After entering the password, your key will be copied, allowing you to log in without a password:

`ssh username@remote_IP_host`

### Copying your Public SSH Key to a Server Manually

If you do not have password-based SSH access available, you will have to add your public key to the remote server manually. The reason I mention this is a lot of embedded devices will come this way, so it's nice to set them up for SSH at least for testing images.

On your local machine, you can find the contents of your public key file by typing:

```
cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCqql6MzstZYh1TmWWv11q5O3pISj2ZFl9HgH1JLknLLx44+tXfJ7mIrKNxOOwxIxvcBF8PXSYvobFYEZjGIVCEAjrUzLiIxbyCoxVyle7Q+bqgZ8SeeM8wzytsY+dVGcBxF6N4JS+zVk5eMcV385gG3Y6ON3EG112n6d+SMXY0OEBIcO6x+PnUSGHrSgpBgX7Ks1r7xqFa7heJLLt2wWwkARptX7udSq05paBhcpB0pHtA1Rfz3K2B+ZVIpSDfki9UVKzT8JUmwW6NNzSgxUfQHGwnW7kj4jp4AT0VZk3ADw497M2G/12N0PPB5CnhHf7ovgy6nL1ikrygTKRFmNZISvAcywB9GVqNAVE+ZHDSCuURNsAInVzgYo9xgJDW8wUw2o8U77+xiFxgI5QSZX3Iq7YLMgeksaO4rBJEa54k8m5wEiEE1nUhLuJ0X/vh2xPff6SQ1BL/zkOhvJCACK6Vb15mDOeCSq54Cr7kvS46itMosi/uS66+PujOO+xt/2FWYepz6ZlN70bRly57Q06J+ZJoc9FfBCbCyYH7U/ASsmY095ywPsBo1XQ9PqhnN1/YOorJ068foQDNVpm146mUpILVxmq41Cj55YKHEazXGsdBIbXWhcrRf4G2fJLRcGUr9q8/lERo9oxRm5JFX6TCmj6kmiFqv+Ow9gI0x8GvaQ== demo@test
```

On the remote server, create the `~/.ssh` directory if it does not already exist:

`mkdir -p ~/.ssh`

Afterwards, you can create or append the `~/.ssh/authorized_keys` file by typing:

`echo public_key_string >> ~/.ssh/authorized_keys`

You should now be able to log into the remote server without a password.

## Adding your SSH Keys to an SSH Agent to Avoid Typing the Passphrase

If you have an passphrase on your private SSH key, you will be prompted to enter the passphrase every time you use it to connect to a remote host.

To avoid having to repeatedly do this, you can run an SSH agent. This small utility stores your private key after you have entered the passphrase for the first time. It will be available for the duration of your terminal session, allowing you to connect in the future without re-entering the passphrase.

Note: This is also important if you need to forward your SSH credentials (shown below).

1. To start the SSH Agent, type the following into your local terminal session:
```
eval $(ssh-agent)
Agent pid 10891
```
2. This will start the agent program and place it into the background. Now, you need to add your private key to the agent, so that it can manage your key:
```
ssh-add
Enter passphrase for /home/demo/.ssh/id_rsa:
Identity added: /home/demo/.ssh/id_rsa (/home/demo/.ssh/id_rsa)
```
3. You will have to enter your passphrase (if one is set). Afterwards, your identity file is added to the agent, allowing you to use your key to sign in without having re-enter the passphrase again.

## Forwarding your SSH Credentials to Use on a Server
---
If you wish to be able to connect without a password to one server from within another server, you will need to forward your SSH key information. This will allow you to authenticate to another server through the server you are connected to, using the credentials on your local computer.

To start, you must have your SSH agent started and your SSH key added to the agent (see above). After this is done, you need to connect to your first server using the -A option. This forwards your credentials to the server for this session:

`ssh -A username@remote_host`

From here, you can SSH into any other host that your SSH key is authorized to access. You will connect as if your private SSH key were located on this server.

## Disabling Host Checking
---
By default, whenever you connect to a new server, you will be shown the remote SSH daemon's host key fingerprint.
```
The authenticity of host '111.111.11.111 (111.111.11.111)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
```
This is configured so that you can verify the authenticity of the host you are attempting to connect to and spot instances where a malicious user may be trying to masquerade as the remote host.

In certain circumstances, you may wish to disable this feature.

Note: This can be a big security risk, so make sure you know what you are doing if you set your system up like this.

To make the change, the open the `~/.ssh/config` file on your local computer:

`vim ~/.ssh/config`

If one does not already exist, at the top of the file, define a section that will match all hosts. Set the `StrictHostKeyChecking` directive to "no" to add new hosts automatically to the `known_hosts` file. Set the `UserKnownHostsFile` to `/dev/null` to not warn on new or changed hosts:

```
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
```

You can enable the checking on a case-by-case basis by reversing those options for other hosts. The default for `StrictHostKeyChecking` is "ask":

```
Host *
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null

Host testhost
    HostName example.com
    StrictHostKeyChecking ask
    UserKnownHostsFile /home/demo/.ssh/known_hosts
```

## Tunneling
---
1. Establish the SOCKS5 Proxy
`ssh -D 8080 -f -C -q -N 54.190.50.202 `
-D is the local port your SOCKS5 proxy will run on.
2. Setup Firefox for SOCKS5 with localhost:8080 and remove ignore DNS resolution

## SSH Proxying Config Example
---

A note about the below; `Host` does not need to be a resolveable host fqdn.

```
Host <bastion machine fqdn>
  Hostname <bastion machine fqdn>
  User ec2-user
  IdentityFile ~/.ssh/keys/identity_file.pem

Host <CIDR Block>
  ProxyCommand ssh -W %h:%p <bastion machine fqdn>
  IdentityFile ~/.ssh/keys/identity_file.pem
  User ec2-user
  StrictHostKeyChecking no

Host <CIDR Block>
  ProxyCommand ssh -W %h:%p <bastion machine fqdn>
  IdentityFile ~/.ssh/keys/identity_file.pem
  User ec2-user
  StrictHostKeyChecking no
```

## Keeping Connections Alive to Avoid Timeout
---
If you find yourself being disconnected from SSH sessions before you are ready, it is possible that your connection is timing out.
You can configure your client to send a packet to the server every so often in order to avoid this situation:

1. On your local computer, you can configure this for every connection by editing your `~/.ssh/config` file. Open it now:
`vim ~/.ssh/config`
2. If one does not already exist, at the top of the file, define a section that will match all hosts. Set the ServerAliveInterval to "120" to send a packet to the server every two minutes. This should be enough to notify the server not to close the connection:
```
Host *
    ServerAliveInterval 120
```
3. Save and close the file when you are finished.

## Private Key Forwarding
---
Set the private key through ssh forwarding to another server.

For example A -> B -> C then A and B need to be set.

1. Make sure ssh forwarding works: `eval ssh-agent -s`
2. `ssh-add`
3. Ensure ssh-agent is running: `ps aux | grep ssh`
4. `vim ~ /.ssh/config` add the following two lines: `ServerAliveInterval 90` & `ForwardAgent Yes`

## HOWTO: Use Wireshark over SSH
---
What you need:

Source system (the server you want to capture packets on) that you have SSH access to, with tcpdump installed, and available to your user (either directly, or via sudo without password). Destination system (where you run graphical Wireshark) with wireshark installed and working, and `mkfifo` available.

### Procedure:

On the destination system, if you haven’t already done so,

`mkfifo /tmp/packet_capture`

This creates a named pipe where the source packet data (via ssh) will be written and Wireshark will read it from. You can use any name or location you want, but `/tmp/packet_capture` is pretty logical.

On your destination system, open up Wireshark (we do this now, since on many systems it required the root password to start). In the “Capture” menu, select “Options”. In the “Interface” box, type in the path to the FIFO you created (/tmp/packet_capture). You should press the Start button before running the next command - I recommend typing the command in a terminal window, pressing start, then hitting enter in the terminal to run the command.

On the destination system, run

`ssh user@source-hostname "sudo /usr/sbin/tcpdump -s 0 -U -n -w - -i eth0 not port 22" > /tmp/packet_capture`

This will SSH to the source system (source-hostname, either by hostname or IP) as the specified user (user) and execute `/usr/sbin/tcpdump`. Omit the `sudo` if you don’t need it, though if you do, you’ll need passwordless access. Options passed to tcpdump are: `-s 0` snarf entire packets, no length limit; `-U` packet-buffered output - write each complete packet to output once it’s captured, rather than waiting for a buffer to fill up; `-n` don’t convert addresses to hostnames; `-w -` write raw packets to `STDOUT` (which will be passed through the SSH tunnel and become STDOUT of the `ssh` command on the destination machine); `-i eth0` capture on interface eth0; “not port 22” a tcpdump filter expression to prevent capturing our own SSH packets (more on this below). The final `> /tmp/packet_capture` redirects the STDOUT of the ssh program (the raw packets from tcpdump on the source machine) to the `/tmp/packet_capture` FIFO.

When you’re ready to stop the capture, just Ctrl+C the SSH command in the terminal window. Wireshark will automatically stop capturing, and you can save the capture file or play around with it. To capture again, you’ll need to restart the capture in Wireshark and then run the ssh command again.

### A note on network usage and tcpdump filters

This is a relatively bandwidth intensive procedure. If you use the “not port 22” tcpdump filter (shown above) on the source machine, all traffic over eth0 (other than SSH) on that machine will be duplicated within an SSH tunnel. So you have double the traffic, plus the overhead of tunneling all that within SSH to the destination machine. If you’re capturing data from a busy machine this way, you could easily saturate the uplink and wreak all sorts of havoc. As a result, I’d recommend making the tcpdump filter as specific as you can while still retaining the data you need. If you can replace it with a filter for specific ports (i.e. '(port 67 or port 68)' for DHCP) or specific hosts, that should cut down on the amount of data you actually have to pass through the tunnel.

`ssh remote-host "tcpdump -s0 -w - 'port 8080'" | wireshark -k -i -`

This will run tcpdump on host “remote-host” and capture full packages (`-s0`) on port 8080. The output is sent over SSH to the local host’s “stdout” where Wireshark is waiting on “stdin” for input. (-k means start immediately).

There are a few things that may make the line above not work in your case. Make sure tcpdump is on the path on your remote host or change the line to include the path a la:

`ssh remote-host "/usr/sbin/tcpdump -s0 -w - 'port 8080'" | wireshark -k -i -`

You may also need to run tcpdump with sudo which means you need to change the command to:

`ssh remote-host "sudo /usr/sbin/tcpdump -s0 -w - 'port 8080'" | wireshark -k -i -`

Warning: Such a remote capture session can be pretty heavy on the network depending on the application. Make sure you filter as much as possible on the remote side using tcpdump’s filters.
