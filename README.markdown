aws-ddns
========

aws-ddns is a Bash script for rolling your own dynamic DNS solution on Amazon AWS Route 53.


Requirements
------------

- BSD/Linux/macOS
- awk, curl, grep, sed, tee
- [AWS Command Line Interface](https://docs.aws.amazon.com/cli/)


AWS configuration
-----------------

The following assumes you have already configured a Route 53 Hosted Zone for a domain you control.

Create a new IAM user and attach a permissions policy scoped to the Route 53 Hosted Zone ID and Record Name you wish to update. Example (replace `hosted-zone-id` and `record-name-no-trailing-dot` with yours):

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "List",
                "Effect": "Allow",
                "Action": [
                    "route53:ListResourceRecordSets"
                ],
                "Resource": [
                    "arn:aws:route53:::hostedzone/hosted-zone-id"
                ]
            },
            {
                "Sid": "Change",
                "Effect": "Allow",
                "Action": [
                    "route53:ChangeResourceRecordSets"
                ],
                "Resource": [
                    "arn:aws:route53:::hostedzone/hosted-zone-id"
                ],
                "Condition": {
                    "ForAllValues:StringEquals": {
                        "route53:ChangeResourceRecordSetsActions": "UPSERT",
                        "route53:ChangeResourceRecordSetsNormalizedRecordNames": [
                            "record-name-no-trailing-dot"
                        ],
                        "route53:ChangeResourceRecordSetsRecordTypes": [
                            "A",
                            "AAAA"
                        ]
                    }
                }
            },
            {
                "Sid": "Get",
                "Effect": "Allow",
                "Action": [
                    "route53:GetChange"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }

Install the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/).

    # Example for Fedora
    sudo dnf install awscli

    # Example for Debian/Ubuntu
    sudo apt install awscli

    # Example for other BSD/Linux
    # Replace x86_64 with aarch64 depending on your system architecture
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update

    # Example for macOS
    curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
    sudo installer -pkg AWSCLIV2.pkg -target /

Configure AWS CLI to use the credentials associated with the IAM user you created previously. Using the `--profile` option is recommended; see the Usage section for more information on using AWS CLI profiles with aws-ddns.

    aws configure --profile=ddns
    #AWS Access Key ID [None]:
    #AWS Secret Access Key [None]:


Installation
------------

Download the `aws-ddns` script, make it executable, and copy it to a directory in your `PATH` such as `/usr/local/bin`.

    curl -O https://raw.githubusercontent.com/bradleysepos/aws-ddns/refs/heads/master/aws-ddns
    chmod +x aws-ddns
    sudo mv aws-ddns /usr/local/bin


Usage
-----

The basic syntax is:

```
aws-ddns hosted-zone-id record-name
```

Replace `hosted-zone-id` with your Route 53 Hosted Zone ID and `record-name` with the Record Name (e.g., `mydomain.tld.` or `mysubdomain.mydomain.tld.`) you wish to point to your machine's external IP address.

To use an AWS CLI profile other than the default (recommended), specify the name using the `-p` argument.

```
aws-ddns -p ddns ...
```

Specify the record type `A` for IPv4 (default) or `AAAA` for IPv6.

```
aws-ddns -r A ...
aws-ddns -r AAAA ...
```

By default, aws-ddns queries one or more providers from a predefined list as needed to obtain the external IP address for your machine. Specify your own comma-separated list of any number of plain text external IP address providers using the `-i` argument.

```
aws-ddns -i https://ipv4.whatismyip.akamai.com,https://ifconfig.co ...
```

Or, manually specify the IP address, which avoids querying any external providers.

```
aws-ddns -e 127.0.0.1 ...
```

<abbr title="Time To Live">TTL</abbr> in seconds may be specified using the `-t` argument. The default is `300` seconds (5 minutes).

```
aws-ddns -t 300 ...
```

If you would like aws-ddns to use a cache location other than the default, specify the location using `-l`.

```
aws-ddns -l ~/.aws-ddns ...
```

Full usage:

```
aws-ddns -h
```


Automation
----------

### BSD/Linux

BSD/Linux automation can be implemented using a scheduler such as `cron`.

Example `cron` user schedule expression that runs every 5 minutes, using the AWS CLI profile `ddns` (replace `hosted-zone-id` and `record-name` with yours):

```
*/5 * * * * /usr/local/bin/aws-ddns -p ddns hosted-zone-id record-name
```

### macOS

Open `com.bradleysepos.aws-ddns.plist` in your preferred text editor.

- Replace `hosted-zone-id` and `record-name` with yours.
- Change the AWS CLI profile name from `default` to yours, if you have configured a different one (recommended).
- Optionally change `StartInterval` from the default `300` seconds (5 minutes) to how often you want the Launch Agent to run.

Copy the edited `com.bradleysepos.aws-ddns.plist` to `~/Library/LaunchAgents`. This can be done using the Finder or Terminal command line.

```
cp com.bradleysepos.aws-ddns.plist ~/Library/LaunchAgents
```

Load the Launch Agent using the Terminal command line.

```
launchctl load ~/Library/LaunchAgents/com.bradleysepos.aws-ddns.plist
```


License
-------

Copyright 2025 Bradley Sepos  
Released under the MIT License. See [LICENSE](LICENSE) for details.
