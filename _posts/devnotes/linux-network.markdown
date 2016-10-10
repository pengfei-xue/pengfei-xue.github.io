## second nameserver in /etc/resolv.conf not picked up by wget

The default behavior for resolv.conf and the resolver is to try the servers in the
order listed.

The resolver will only try the next nameserver if the first nameserver times out.
The resolv.conf manpage says:

    nameserver Name server IP address

    Internet address (in dot notation) of a name server that the resolver should query.
    Up to MAXNS (currently 3, see ) name servers may be listed, one per keyword.
    If there are multiple servers, the resolver library queries them in the order listed.

And:

    (The algorithm used is to try a name server, and if the query times out, try the next, until out of name servers,
    then repeat trying all the name servers until a maximum number of retries are made.)
    Also see the resolver(5) manual page for more information.

You can alter the resolver's behavior using rotate, which will query the Nameservers in a round-robin order:

    rotate sets RES_ROTATE in res.options, which causes round robin selection of nameservers from among those listed.
    This has the effect of spreading the query load among all listed servers, rather than having all clients try the first listed server first every time.

However, nslookup will use the second nameserver if it receives a SERVFAIL from the first nameserver. From the nslookup manpage:

    [no]fail Try the next nameserver if a nameserver responds with SERVFAIL or a referral (nofail) or terminate query (fail) on such a response.
    (Default = nofail)

[resolv.conf man page](https://linux.die.net/man/5/resolv.conf)


## SSH access problem: debug1: expecting SSH2_MSG_KEX_DH_GEX_REPLY

从ci ssh 到 backend.test 不成功 hang在 expecting SSH2_MSG_KEX_DH_GEX_REPLY

    ip li set mtu 1400 dev wlan0

https://serverfault.com/questions/210408/ssh-access-problem-debug1-expecting-ssh2-msg-kex-dh-gex-reply/676026#676026
