# Autoresponse
An autoresponder bash script for postfix

## About Autoresponse

This is a repolica of a script that used to hosted on nefaria.com, but is now offline. I put it here for safekeeping and future reference, since it provides an awesomely easy way to enable automatic email response for postfix on my private mail servers.

The README is an adapted version of [this page](https://www.howtoforge.com/how-to-set-up-a-postfix-autoresponder-with-autoresponse).

## Installing Autoresponse

To install, download and store the autoresponse script to the server where postfix is running. Then do the following:

    sudo useradd -d /var/spool/autoresponse -s `which nologin` autoresponse
    sudo mkdir -p /var/spool/autoresponse/log /var/spool/autoresponse/responses
    sudo cp ./autoresponse /usr/local/sbin/
    sudo chown -R autoresponse:autoresponse /var/spool/autoresponse
    sudo chmod -R 0770 /var/spool/autoresponse

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
      flags=Fq user=autoresponse argv=/usr/local/sbin/autoresponse -s ${sender} -r ${recipient} -S ${sasl_username} -C ${client_address}

Then run...

    postconf -e 'autoresponder_destination_recipient_limit = 1'

... and restart Postfix:

    sudo systemctl restart postfix

If you have users with shell access, and you want these users to be able to create autoresponder messages themselves on the shell, you must add each user account to the autoresponse group, e.g. as follows for the system user john:

    sudo usermod -G autoresponse john 

However, this is not necessary if you want to create all autoresponder messages as root (or use the email feature to create autoresponder messages - I'll come to that in a moment).
 
## Using Autoresponse

Run

    autoresponse -h

to learn how to use Autoresponse:

    server1:~# autoresponse -h

A help text will be displayed detailing all the options the script provides. To create an autoresponder message for the account john@example.com, we run...

    autoresponse -e john@example.com

... and type in the autoresponder text:

    I will be out the week of March 2 with very limited access to email.
    I will respond as soon as possible.
    Thanks!
    John

(You cannot set the subject using this method; by default, the subject of the autoresponder messages will be Out of Office.)

Now send an email to john@example.com from a different account, and you should get the autoresponder message back.

To disable an existing autoresponder, run

    autoresponse -d john@example.com

To enable a deactivated autoresponder, run

    autoresponse -E john@example.com

To delete an autoresponder, run

    autoresponse -D john@example.com

You can modify the RESPONSE_RATE variable in /usr/local/sbin/autoresponse. It defines the time limit (in seconds) that determines how often an autoresponder message will be sent, per email address. The default value is 86400 (seconds) which means if you send an email to john@example.com and receive an autoresponder message and send a second email to john@example.com within 86400 seconds (one day), you will not receive another autoresponder message.

    sudo nano /usr/local/sbin/autoresponse

    [...]
    declare RESPONSE_RATE="86400"
    [...]

## Creating/Deleting Autoresponder Messages By Email

Instead of creating autoresponder messages on the command line, this can also be done by email. If you want to create an autoresponder message for the email address john@example.com, send an email from john@example.com to john+autoresponse@example.com (this works only if you've set up SMTP-AUTH on your server). The subject of that email will become the subject of the autoresponder message (that way you can define subjects different from Out of Office), and the email body will become the autoresponder text.

If you create an autoresponder this way, Autoresponse will send you an email back like this one (so that you know if the operation was successful):

    Autoresponse enabled for john@example.com  by SASL authenticated user: john@example.com  from: 192.168.0.200   

If there's already an active autoresponder for that email address, it will be disabled (i.e., there's no active autoresponder at all for that address anymore, and you will receive an email telling you so:

    Autoresponse disabled for john@example.com by SASL authenticated user: john@example.com from: 192.168.0.200

).

This means the email feature is a toggle switch - if there's no sautoresponder, it will be created, and if there's an autoresponder, it will be disabled. 
