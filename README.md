# Out Of Office
An autoresponder bash script for postfix

## About Out Of Office

This is a replica of a script called autoresponse that used to be hosted on nefaria.com, but is now offline. It was originally uploaded to the repository CodingConnected/autoresponse by Menno van der Woude for safekeeping and future reference, since it provides an awesomely easy way to enable automatic email out of office response on postfix mail servers.

The README is an adapted version of [this page](https://www.howtoforge.com/how-to-set-up-a-postfix-autoresponder-with-autoresponse).

## Installing Out Of Office

To install, download and store the outofoffice script to the server where postfix is running. Then do the following:

    sudo useradd -d /var/spool/outofoffice -s `which nologin` outofoffice
    sudo mkdir -p /var/spool/outofoffice/log /var/spool/outofoffice/responses
    sudo cp ./outofoffice /usr/local/sbin/
    sudo chown -R outofoffice:outofoffice /var/spool/outofoffice
    sudo chmod -R 0770 /var/spool/outofoffice

Now we edit /etc/postfix/master.cf:

    nano /etc/postfix/master.cf

At the beginning of the file, you should see the line

    [...]
    smtp      inet  n       -       -       -       -       smtpd
    [...]

Modify it so that it looks as follows (the second line must begin with at least one whitespace!):

    [...]
    smtp      inet  n       -       -       -       -       smtpd
      -o content_filter=autoresponder:dummy
    [...]

At the end of the file, append the following two lines (again, the second line must begin with at least one whitespace!):

    [...]
    autoresponder unix - n n - - pipe
      flags=Fq user=outofoffice argv=/usr/local/sbin/outofoffice -s ${sender} -r ${recipient} -S ${sasl_username} -C ${client_address}

Then run...

    postconf -e 'autoresponder_destination_recipient_limit = 1'

... and restart Postfix:

    sudo systemctl restart restart

If you have users with shell access, and you want these users to be able to create out of office messages themselves on the shell, you must add each user account to the outofoffice group, e.g. as follows for the system user john:

    sudo usermod -G outofoffice john 

However, this is not necessary if you want to create all out of office messages as root (or use the email feature to create autoresponder messages - I'll come to that in a moment).
 
## Using OutOfOffice

Run

    outofoffice -h

to learn how to use OutOfOffice:

    server1:~# outofoffice -h

A help text will be displayed detailing all the options the script provides. To create an out of office message for the account john@example.com, we run...

    outofoffice -e john@example.com

... and type in the out of office text:

    I will be out the week of March 2 with very limited access to email.
    I will respond as soon as possible.
    Thanks!
    John

(You cannot set the subject using this method; by default, the subject of the out of office messages will be Out of Office.)

Now send an email to john@example.com from a different account, and you should get the out of office message back.

To disable an existing out of office, run

    outofoffice -d john@example.com

To enable a deactivated out of office, run

    outofoffice -E john@example.com

To delete an out of office, run

    outofoffice -D john@example.com

You can modify the RESPONSE_RATE variable in /usr/local/sbin/outofoffice. It defines the time limit (in seconds) that determines how often an out of office message will be sent, per email address. The default value is 86400 (seconds) which means if you send an email to john@example.com and receive an out of office message and send a second email to john@example.com within 86400 seconds (one day), you will not receive another out of office message.

    sudo nano /usr/local/sbin/outofoffice

    [...]
    declare RESPONSE_RATE="86400"
    [...]

## Creating/Deleting Out Of Office Messages By Email

Instead of creating out of office messages on the command line, this can also be done by email. If you want to create an out of office message for the email address john@example.com, send an email from john@example.com to john+outofoffice@example.com (this works only if you've set up SMTP-AUTH on your server). The subject of that email will become the subject of the out of office message (that way you can define subjects different from Out of Office), and the email body will become the out of office text.

If you create an out of office this way, it will send you an email back like this one (so that you know if the operation was successful):

    Out of office enabled for john@example.com  by SASL authenticated user: john@example.com  from: 192.168.0.200   

If there's already an active out of office for that email address, it will be disabled (i.e., there's no active out of office at all for that address anymore, and you will receive an email telling you so:

    Out of office disabled for john@example.com by SASL authenticated user: john@example.com from: 192.168.0.200

).

This means the email feature is a toggle switch - if there's no out of office, it will be created, and if there's an out of office, it will be disabled. 
