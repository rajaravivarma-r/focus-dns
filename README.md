First things first: I copied code from https://github.com/hubdotcom/marlon-tools
and https://github.com/amoffat/focus to build a custom DNS server facilitating my needs.

(focus-dns.py) Focus.py helps you keep focused by applying schedulable firewall rules
to distracting websites.  An example firewall rule looks like this:

``` python
def domain_reddit_com(dt):
    return dt.hour == 21 # allow from 9-10pm
```

Starting
========

## Now start Focus:

    sudo FOCUS_ROOT="$HOME/focus/" python focus-dns.py

where `FOCUS_ROOT` is where the **focus_blacklist.py** will be created.

Personally I like to keep my **hosts** and **focus_blacklist.py** file in the same directory as the **focus-dns.py** file.

Filtering Domains
=================

Firewall rules involving schedules and timeframes can get complicated fast.
For this reason, the scheduling specification is pure Python, so you can make
your filtering rules as simple or as complex as you want.

The default filter rules is created on first startup in `$FOCUS_ROOT/focus_blacklist.py`:

```python
def domain_ycombinator_com(dt):
    # return dt.hour % 2 # every other hour
    return False

def domain_reddit_com(dt):
    # return dt.hour in (12, 21) # at noon-1pm, or from 9-10pm
    return False

def domain_facebook_com(dt):
    return False

def default(domain, dt):
    # do something with regular expressions here?
    return True
```

The format is simple; Just define a function named like the domain you
want to block, preceeded by "domain_".  Have it take a single datetime object
and have it return True or False.  In the body, you can write whatever logic
makes the most sense for
you.  Maybe you want to write your own Pomodoro routine, or maybe you want to
scrape your google calendar for exam dates, and block certain websites on those dates.

For sites without their own scheduler function, the default() function is called.

There's no need to restart Focus if you redefine your schedules.

# It is also
A simple DNS proxy server, support wilcard hosts, IPv6, cache.

Instructions:
```
1. Edit /etc/hosts, add:
127.0.0.1 *.local
-2404:6800:8005::62 *.blogspot.com

2. startup dnsproxy (here using Google DNS server as delegating server):
$ sudo python FOCUS_ROOT="$HOME/focus/" focus-dns.py -s 8.8.8.8

3. Then set system dns server as 127.0.0.1, you can verify it by dig:
$ dig test.local
The result should contain 127.0.0.1.

```

Usage:
```
focus-dns.py [options]

Options:
  -h, --help            show this help message and exit
  -f <file>, --hosts-file=<file>
                        specify hosts file, default /etc/hosts
  -H HOST, --host=HOST  specify the address to listen on
  -p PORT, --port=PORT  specify the port to listen on
  -s SERVER, --server=SERVER
                        specify the delegating dns server
  -C, --no-cache        disable dns cache
```

How it works
============

Focus.py is, at its core, a DNS server.  By making it your primary nameserver,
it receives all DNS lookup requests.  Based on the domain name being requested,
it either responds with a "fail ip" address (blocked), or passes the request
on to your other nameservers (not blocked).  In both cases, Focus adjusts the TTL of each
DNS response so that the service requesting the DNS lookup will do minimal
caching on the IP, allowing Focus's filtering rules to be more immediate.


FAQ
===

- Q: I started Focus, but it's not blacklisting the site I picked.
- A: Your browser may be caching that site's ip.  Give it a few minutes.

- Q: Why do I need to start Focus with sudo?
- A: Focus needs to listen on a privileged port as a DNS server.

- Q: How do I stop Focus?
- A: Press `Ctrl-c`, just like you would terminate any running bash process.
