aws-ddns
======

aws-ddns is a Bash script for rolling your own dynamic DNS solution on Amazon AWS Route 53.


Requirements
------------

- OS X 10.9 Mavericks or later suggested (not tested on other operating systems)
- [AWS Command Line Tools 1.8.2 or later](http://docs.aws.amazon.com/cli/latest/userguide/installing.html) (not tested on earlier versions)


Installation
------------

1. Copy `aws-ddns` to a directory in your `PATH` such as `/usr/local/bin` and make it executable.
2. Add your Hosted Zone ID and Record Set name (domain or subdomain) to `com.bradleysepos.aws-ddns.plist` 
3. Copy the updated `com.bradleysepos.aws-ddns.plist` to `~/Library/LaunchAgents`.
4. Open Terminal and load the Launch Agent:

```
launchctl load ~/Library/LaunchAgents/com.bradleysepos.aws-ddns.plist
```


Usage
-----

The basic syntax is:

```
aws-ddns hosted-zone-id record-set-name
```

Replace `hosted-zone-id` with your Route 53 Hosted Zone ID and `record-set-name` with the Record Set name you wish to map to your machine's external IP address (e.g. `mydomain.com.` or `something-clever.mydomain.com.`).

By default, aws-ddns queries up to three predefined providers for your machine's external IP address. Specify your own comma-separated list of any number of external IP address providers using the `-i` argument.

```
aws-ddns -i https://api.ipify.org,https://wtfismyip.com/text,https://icanhazip.com hosted-zone-id record-set-name
```

<abbr title="Time To Live">TTL</abbr> in seconds may be set using the `-t` argument. The default is `300` (5 minutes).

```
aws-ddns -t 300 hosted-zone-id record-set-name
```

If you would like aws-ddns to use a cache location other than the default, specify the location using `-l`.

```
aws-ddns -l "~/.aws-ddns" hosted-zone-id record-set-name
```

Full usage:

```
aws-ddns -h
```


License
-------

Copyright 2017 Bradley Sepos  
Released under the MIT License. See [LICENSE](LICENSE) for details.
