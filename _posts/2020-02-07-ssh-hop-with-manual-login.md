---
title: SSH Tunneling with a manual first hop
tags: [ ssh, tunnel, 2fa, keys, pci, duo, putty, windows]
---
Follow-up to:
1. [SSH key authentication with PuTTY]({% post_url 2011-02-03-ssh-key-authentication-with-putty %})
2. [SSH key authentication with PuTTY, Part 2, Tunneling with Plink]({% post_url 2011-02-03-tunneling-with-plink %})
3. [SSH Tips]({% post_url 2018-02-01-ssh-tips %})

What if that first hop doesn't allow you to background tunnel through it? Either because SSH key authentication isn't enabled, or because it uses a 2FA solution that requires manual interaction?

Thanks to colleague Dan Patterson, I've got steps that work out almost as quickly/simply as I had before.

* TOC
{:toc}

## First hop with manual authentication and primary tunnel

First, I'm opening a [PuTTY](https://www.putty.org/) session to the "jump" host where I have to both use a password rather than a key, and where I have to authorize 2FA through [Duo](https://duo.com/product/multi-factor-authentication-mfa/two-factor-authentication-2fa).

The PuTTY Session "Host Name" and "Port" are to the server I have to reach in order to then reach the other desired destinations. The server that requires password authentication and/or manual 2FA interaction. I'll call this the "_jump host_".

![putty-jump-session.png](/assets/putty-jump-session.png)

Then, in the **Connection** > **SSH** > **Tunnels** configuration, I'm going to tunnel local 2222 to the SSH port (normally 22) on a host in a network on the other side of the _jump host_. Let's call that the "_protected network_", and the "_tunnel host_".

![putty-tunnel-2222.png](/assets/putty-tunnel-2222.png)

Under **Connection**, I'm also setting a "keepalive" on this PuTTY session so that it will stay open even though I won't be interacting with it.

![putty-jump-keepalive.png](/assets/putty-jump-keepalive.png)

Login (to the _jump host_), fullfill the 2FA requirements, put the window somewhere you won't close it. (I also gave it a unique size and font color to remind myself that it's special.)

### Trusting the _tunnel host_ SSH host key

Note that if you've never connected directly from your local machine, through PuTTY/plink, to the _tunnel host_, you'll have to connect there first and (verify and) accept its host key. Start a command-line plink command to your `localhost:2222` equivalent:

```shell
plink -P 2222 localhost
```
You'll be prompted to trust the key presented. Once you do that, you don't need to continue the login, but now your PuTTY tunnels through this _tunnel host_ won't get stuck waiting for you to accept the host key.

## PuTTY Session for dynamic/on-the-fly to the _protected network_

Now we can use the above connection to tunnel, in one step, to another host in the _protected network_ (technically, anything reachable from that _tunnel host_, including perhaps other hosts that only it can reach, like via a VPN).

I've created another PuTTY Session for this, with no hostname specified, so that I can type in any host dynamically when I open it.

![putty-proxy-session.png](/assets/putty-proxy-session.png)

I've named the Session "localhost 2222 proxy" to let me know that I'm going to be connecting through the above tunnel.

Now, here I'm using the same local proxy "plink" approach [as described in my earlier post]({% post_url 2011-02-03-tunneling-with-plink %}), but with the proxy host being said localhost:2222.

![putty-proxy-proxy.png](/assets/putty-proxy-proxy.png)

The small addition needed to the prior plink command was the -P one for the custom port:

```
c:\Program Files (x86)\PuTTY\plink -l %user %proxyhost -P %proxyport -nc %host:%port
```
### Authentication to the Tunnel/Proxy host

You have to specify how to authenticate to your _tunnel host_ when proxying through it. Either:
1. the above `-l %user%` has to be specified, with the _Username_ field filled in, and you have to have SSH key authentication enabled on the _tunnel host_, with Pageant running locally, **or**
2. the _Username_ and _Password_ fields have to both be filled in **and** `-pw %pass` must be added to the plink command as well. 

#### PuTTY Default Settings session

I also discovered, BTW, that if you have just one user ID you use on every system, and if you specify that user ID in the special PuTTY "Default Settings" session's **Connection** > **Data** > **Auto-login username** field, this automatically supplies your user name in place of having to add the `plink ... -l` option at all. 

This fact has thrown me off a number of times when something works for me but not for a colleague who hasn't set up her Default Settings that way.

## Single Session with many tunneled ports

Finally, my "big" PuTTY session which automatically sets up many different tunneled ports for me, including database server ports and [the super-useful dynamic port that I can use as a browser SOCKS proxy](/2018/02/01/ssh-tips.html#dynamic-forwarding).

I used a hardcoded destination in the _protected network_, but that's not required.

![putty-proxy-tunnels-session.png](/assets/putty-proxy-tunnels-session.png)

Same proxy setup as above, just this time we're also going to store several tunnels to ports which are on different servers only accessible from the _protected network_. 

For instance, below I have a DB2 port tunneled on 60000, an HTTP port tunneled on 8081->8080, a Dynamic Socks tunnel on 8888, and I'm about to add a Windows Remote Desktop port on 3391->3389.

![putty-proxy-tunnels-tunnels.png](/assets/putty-proxy-tunnels-tunnels.png)

**NOTE** you _could_ run the dynamic tunnel - my 8888 - up in the initial "Jump" connection instead. If you only need SOCKS connectivity to things the _jump host_ can reach. As opposed to things the _tunnel host_ can also reach.

## Bonus Tip

Again, plug for the super useful [SwitchyOmega](https://addons.mozilla.org/en-US/firefox/addon/switchyomega/) Firefox plugin for routing different hosts & networks through different proxies. Like the 8888 "dynamic port" SOCKS proxy we created above.
